# Configuración del Lab — VirtualBox + Wazuh SIEM

> Guía completa para reproducir este home lab desde cero.

---

## 📋 Requisitos del host

| Recurso    | Mínimo                | Recomendado|
|------------|-----------------------|------------|
| RAM        | 12 GB                 | 16 GB      |
| CPU        | 4 cores               | 6+ cores   |
| Disco      | 80 GB libres          | 120 GB SSD |
| OS Host    | Windows 10/11 o Linux | cualquiera |
| VirtualBox | 7.x                   | 7.x        |

---

## 🖥️ Máquinas virtuales

| VM       | OS                  | Rol                       | RAM  | CPU | Disco |       IP      |
|----------|---------------------|---------------------------|------|-----|-------|---------------|
| SIEM     | Ubuntu 22.04 LTS    | Wazuh Manager + ELK       | 4 GB | 2   | 40 GB | 192.168.56.30 |
| Victim   | Windows Server 2022 | Target + Active Directory | 4 GB | 2   | 50 GB | 192.168.56.20 |
| Attacker | Kali Linux 2024     | Red Team                  | 2 GB | 1   | 30 GB | 192.168.56.10 |

---

## 🌐 Configuración de red (Host-Only)

Todas las VMs se comunican en una red interna aislada. Sin acceso a internet desde Kali durante ataques.

**Crear la red en VirtualBox:**

1. Abrir VirtualBox → `Archivo` → `Administrador de red de anfitrión`
2. Crear red `vboxnet0`
3. Configurar:
   - **IPv4:** `192.168.56.1`
   - **Máscara:** `255.255.255.0`
   - **DHCP:** ❌ Desactivado

**En cada VM → Configuración → Red:**
- Adaptador 1: `NAT`

- Adaptador 2: `Red interna` → `labsoc`

---

## ⚙️ Paso 1 — Instalar Wazuh en Ubuntu 22.04

```bash
# Conectarse a la VM de Ubuntu
# Ejecutar como root o con sudo

# 1. Descargar el instalador de Wazuh
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh

# 2. Instalar todo en un solo nodo (manager + indexer + dashboard)
sudo bash wazuh-install.sh -a

# 3. Al terminar, guardar las credenciales que muestra en pantalla
# Usuario: admin
# Contraseña: (generada automáticamente)

# 4. Acceder al dashboard
# https://192.168.56.30 (desde el navegador del host)
```

> ⚠️ La instalación tarda entre 10 y 20 minutos. No cerrar la terminal.

**Verificar que Wazuh está corriendo:**
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

---

## ⚙️ Paso 2 — Instalar Wazuh Agent en Windows Server 2022

1. Descargar el agente desde el dashboard de Wazuh:
   - `https://192.168.56.30` → `Agents` → `Deploy new agent`
   - Seleccionar: **Windows**
   - Manager address: `192.168.56.30`

2. Ejecutar el instalador `.msi` en Windows Server como Administrador

3. O instalar vía PowerShell (como Administrador):

```powershell
# Descargar agente
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.0-1.msi `
  -OutFile wazuh-agent.msi

# Instalar apuntando al manager
msiexec.exe /i wazuh-agent.msi /q `
  WAZUH_MANAGER="192.168.56.30" `
  WAZUH_AGENT_NAME="WinServer2022"

# Iniciar el servicio
NET START WazuhSvc
```

4. Verificar en el dashboard que el agente aparece como **Active** ✅

---

## ⚙️ Paso 3 — Configurar IPs estáticas

**En Ubuntu (Wazuh):**
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```
```yaml
network:
  ethernets:
    enp0s3:
      addresses: [192.168.56.30/24]
      nameservers:
        addresses: [8.8.8.8]
  version: 2
```
```bash
sudo netplan apply
```

**En Windows Server:**
- Panel de control → Centro de redes → Cambiar configuración del adaptador
- Ethernet → Propiedades → IPv4
- IP: `192.168.56.20` | Máscara: `255.255.255.0`

**En Kali Linux:**
```bash
sudo nano /etc/network/interfaces
# Agregar:
# iface eth0 inet static
#   address 192.168.56.10
#   netmask 255.255.255.0
```

---

## ✅ Verificar conectividad

Desde Kali, confirmar que ve las otras VMs:
```bash
ping 192.168.56.20   # debe responder (Windows Server)
ping 192.168.56.30   # debe responder (Wazuh)
nmap -sn 192.168.56.0/24  # ver todos los hosts activos
```

---

## 📸 Snapshots recomendados

Antes de cada escenario de ataque, tomar snapshot de cada VM:

| Snapshot | Cuándo tomar |
|---|---|
| `baseline-clean` | Después de instalar todo, antes de cualquier ataque |
| `pre-scenario-XX` | Justo antes de ejecutar cada escenario |
| `post-analysis` | Después del análisis, antes de limpiar |

**Cómo tomar snapshot en VirtualBox:**
- VM seleccionada → `Máquina` → `Tomar instantánea`
- O: `Ctrl + Shift + S`

---

## 🔁 Flujo de trabajo para cada escenario

```
1. Restaurar snapshot "baseline-clean" en todas las VMs
2. Verificar que Wazuh dashboard está activo
3. Ejecutar el ataque desde Kali
4. Capturar alertas en Wazuh (screenshots)
5. Analizar logs y documentar en /scenarios/XX-nombre/
6. Escribir reporte en /incident-reports/IR-00X.md
7. Restaurar snapshot para el siguiente escenario
```

---

## 🛠️ Herramientas preinstaladas en Kali (usadas en este lab)

| Herramienta  | Escenario        | Comando base                                            |
|--------------|------------------|---------------------------------------------------------|
| Nmap         | Reconocimiento   | `nmap -sV -sC 192.168.56.20`                            |
| Netdiscover  | Reconocimiento   | `netdiscover -r 192.168.56.0/24`                        |
| Hydra        | Brute Force      | `hydra -L users.txt -P rockyou.txt rdp://192.168.56.20` |
| CrackMapExec | Lateral Movement | `crackmapexec smb 192.168.56.20`                        |
| Impacket     | Lateral Movement | `python3 psexec.py admin@192.168.56.20`                 |

---

*Home Lab documentado por Katalina Sepúlveda — Blue Team / SOC Analyst (en formación)*