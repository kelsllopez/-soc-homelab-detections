# 🛡️ SOC Home Lab — CyberCorp S.A. Detection Engineering

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh_4.7-blue?style=flat-square&logo=linux)
![MITRE](https://img.shields.io/badge/Framework-MITRE_ATT%26CK-red?style=flat-square)
![Status](https://img.shields.io/badge/Lab-Active-brightgreen?style=flat-square)
![ISC2](https://img.shields.io/badge/Cert-ISC2_CC_(en_progreso)-orange?style=flat-square)
![Scenarios](https://img.shields.io/badge/Escenarios-6-purple?style=flat-square)
![Reports](https://img.shields.io/badge/Reportes_IR-5-teal?style=flat-square)

Simulación de una **semana completa como SOC Analyst Junior** en la empresa ficticia CyberCorp S.A. Cada día representa una etapa real de la kill chain de un atacante, detectada y contenida con Wazuh SIEM y documentada con formato NIST IR + MITRE ATT&CK.

> **Todos los ataques se ejecutaron en entorno VirtualBox aislado, sin conexión a internet.**

---

## 🖥️ Arquitectura del laboratorio

```
┌──────────────────────────────────────────────────────────┐
│                     VirtualBox Host                      │
│                                                          │
│  ┌─────────────────┐        ┌──────────────────────┐     │
│  │   Kali Linux    │──────▶ │   Windows Server     │     │
│  │   (Attacker)    │        │   2022 + AD          │     │
│  │  192.168.56.30  │        │   192.168.56.20      │     │
│  └─────────────────┘        └──────────┬───────────┘     │
│                                        │ Wazuh Agent     │
│                             ┌──────────▼───────────┐     │
│                             │   Ubuntu 22.04       │     │
│                             │   Wazuh Manager SIEM │     │
│                             │   192.168.56.10      │     │
│                             └──────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

| VM | OS | Rol | IP |
|---|---|---|---|
| Attacker | Kali Linux 2025.4 | Red Team — generador de ataques | 192.168.56.30 |
| Victim | Windows Server 2022 | Target — Active Directory (soc.local) | 192.168.56.20 |
| SIEM | Ubuntu 22.04 LTS | Wazuh 4.7 Manager + Dashboard | 192.168.56.10 |

**Dominio AD:** `soc.local` | **Hostname víctima:** `WIN-J36GU3SVEOC`

Configuración detallada → [`lab-architecture/vm-setup.md`](./lab-architecture/vm-setup.md)

---

## 📁 Estructura del repositorio

```
soc-homelab-detections/
│
├── lab-architecture/
│   ├── vm-setup.md                        # Guía completa para reproducir el lab
│   └── network-diagram.png                # Diagrama visual de la red
│
├── scenarios/
│   ├── 01-reconnaissance/                 # Día 1 — Nmap + Netdiscover
│   ├── 02-brute-force/                    # Día 2 — Hydra + smbclient
│   ├── 03-lateral-movement/               # Día 3 — CrackMapExec + WinRM
│   ├── 04-privilege-escalation/           # Día 4 — Cuenta hackersoc → Domain Admins
│   ├── 05-persistence/                    # Día 5 — Backdoor + Servicio + Registro
│   └── 06-threat-hunting/                 # Día 6 — Hunting proactivo en Wazuh
│
├── incident-reports/
│   ├── IR-001-reconnaissance.md           # Reporte formal NIST IR
│   ├── IR-002-brute-force.md
│   ├── IR-003-lateral-movement.md
│   ├── IR-004-privilege-escalation.md
│   ├── IR-005-persistence.md
│   └── WEEKLY-REPORT-SOC-2026-001.md     # Informe semanal ejecutivo
│
├── detections/
│   └── wazuh-rules/
│       └── custom-rules.xml               # 5 reglas personalizadas CYBERCORP
│
└── scripts/
    └── lab-setup.sh                       # Script para levantar el lab
```

---

## 🎯 Escenarios documentados — Kill Chain completa

| Día | Escenario | Herramientas | MITRE Táctica | MITRE ID | Severidad |
|---|---|---|---|---|---|
| 1 | Reconocimiento de red | Nmap, Netdiscover | Reconnaissance | T1046 | 🟡 Media |
| 2 | Brute Force SMB | smbclient, Hydra | Credential Access | T1110.001 | 🔴 Alta |
| 3 | Movimiento Lateral | CrackMapExec, WinRM | Lateral Movement | T1021, T1135, T1087 | 🔴 Alta |
| 4 | Escalada de Privilegios | PowerShell, net.exe | Privilege Escalation | T1098, T1136 | 🔴 Crítica |
| 5 | Persistencia / Backdoor | PowerShell, sc.exe | Persistence | T1543.003, T1053.005, T1547.001 | 🔴 Crítica |
| 6 | Threat Hunting | Wazuh + reglas custom | — | — | Proactivo |

### Cadena de ataque completa

```
ETAPA 1          ETAPA 2          ETAPA 3          ETAPA 4          ETAPA 5
Reconocimiento → Fuerza Bruta  → Mov. Lateral  → Escalada Privs → Persistencia
   Nmap              smbclient      CrackMapExec    hackersoc→DA    backdoor.ps1
   T1046             T1110.001      T1021/T1135     T1098/T1136     T1543.003
```

---

## 🔍 Detecciones destacadas en Wazuh

### Event IDs críticos trabajados

| EventID | Descripción | Escenario |
|---|---|---|
| 4625 | Login fallido | Brute Force |
| 4624 | Login exitoso | Lateral Movement |
| 4720 | Cuenta nueva creada | Privilege Escalation |
| 4728 | Usuario agregado a grupo global (Domain Admins) | Privilege Escalation |
| 4732 | Usuario agregado a grupo local (Administradores) | Privilege Escalation |
| 4672 | Privilegios especiales asignados | Privilege Escalation |
| 4726 | Cuenta eliminada (contención) | IR Response |
| 7045 | Nuevo servicio instalado | Persistence |
| 4698 | Tarea programada creada | Persistence |
| 4657 | Modificación del registro | Persistence |
| 550 | Integridad de archivo modificada (FIM) | Persistence |

### Ejemplo real — Alerta de Persistencia detectada

```json
{
  "timestamp": "Jun 17, 2026 @ 20:07:07",
  "agent.name": "WindowsServer",
  "data.win.system.eventID": "7045",
  "data.win.eventdata.serviceName": "Windows Update Helper",
  "data.win.eventdata.imagePath": "C:\\Windows\\System32\\backdoor.ps1",
  "data.win.eventdata.startType": "inicio automático",
  "data.win.eventdata.accountName": "LocalSystem",
  "rule.id": "61138",
  "rule.description": "New Windows Service Created",
  "rule.level": 5,
  "rule.mitre.id": "T1543.003",
  "rule.mitre.tactic": "Persistence, Privilege Escalation"
}
```

---

## 🔧 Reglas personalizadas creadas (CYBERCORP)

5 reglas propias escritas en `/var/ossec/etc/rules/local_rules.xml`:

| Regla | EventID | Detecta | Nivel |
|---|---|---|---|
| 100001 | 7045 | Servicio instalado con ejecutable `.ps1` | 12 — Alto |
| 100002 | 4728 | Usuario agregado a Domain Admins | 15 — Crítico |
| 100003 | 60122 | 5+ logins fallidos en 2 minutos (brute force) | 14 — Alto |
| 100004 | syscheck | Archivo `.ps1` creado en `C:\Windows\System32` | 12 — Alto |
| 100005 | 4720 | Nueva cuenta de usuario creada | 10 — Medio |

Ver reglas completas → [`detections/wazuh-rules/custom-rules.xml`](./detections/wazuh-rules/custom-rules.xml)

---

## 📄 Reportes de incidente

Cada escenario genera un reporte siguiendo el lifecycle **NIST SP 800-61**:

1. Preparación — contexto y entorno
2. Detección y análisis — alertas, IOCs, timeline
3. Contención — acciones inmediatas
4. Erradicación y recuperación — limpieza
5. Lecciones aprendidas — mejoras propuestas

**Métricas de la semana simulada:**

| Métrica | Valor |
|---|---|
| Incidentes detectados | 5 |
| Incidentes contenidos | 5 (100%) |
| Tiempo promedio detección | < 5 minutos |
| Compromisos de datos | 0 |
| Reglas personalizadas creadas | 5 |
| Técnicas MITRE documentadas | 15+ |
| EventIDs estudiados | 12 |

---

## 🛠️ Stack tecnológico

| Categoría | Tecnología |
|---|---|
| SIEM | Wazuh 4.7 + OpenSearch Dashboard |
| Virtualización | VirtualBox 7.x |
| Atacante | Kali Linux 2025.4 |
| Víctima | Windows Server 2022 Standard (Build 20348) |
| Monitor | Ubuntu 22.04 LTS |
| Herramientas ataque | Nmap, smbclient, CrackMapExec, Hydra, PowerShell |
| Scripting | Python 3, PowerShell, Bash |

---

## 📚 Cómo reproducir el lab

```bash
# Clonar el repositorio
git clone https://github.com/kelsllopez/soc-homelab-detections.git
cd soc-homelab-detections

# Ver guía de configuración
cat lab-architecture/vm-setup.md
```

Requisitos mínimos del host: **12 GB RAM, 4 cores CPU, 80 GB disco libre**

---

## 👩‍💻 Sobre este proyecto

Este repositorio documenta mi transición hacia ciberseguridad defensiva (Blue Team). Vengo de experiencia real en soporte TI empresarial — Active Directory, incidentes N1/N2, SLA — en Portuaria Corral S.A. con +80 usuarios.

Actualmente preparando **ISC2 CC** (Certified in Cybersecurity).

Mi objetivo es especializarme en **análisis forense digital**, con SOC como punto de entrada.

📧 kels.sepulvedaa@gmail.com
📍 Valdivia / Temuco, Chile — disponible para trabajo remoto
🔗 [LinkedIn](https://linkedin.com/in/katalina-sepulveda)

---

*Todos los ataques se ejecutaron en entorno controlado y aislado. Fines educativos y de práctica profesional.*
