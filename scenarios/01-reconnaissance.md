# Escenario 01 — Reconocimiento de Red
**Fecha:** 2026-06-09 | **Analista:** Katalina Sepúlveda López

## Ejecución desde Kali Linux

```bash
# Descubrimiento de hosts
sudo nmap -sn 192.168.56.0/24

# Escaneo completo de servicios
nmap -sV -sC 192.168.56.20
```

## Resultado

| Puerto | Servicio | Riesgo |
|---|---|---|
| 53/tcp | DNS — Simple DNS Plus | Bajo |
| 80/tcp | HTTP — IIS 10.0 | HTTP TRACE habilitado ⚠️ |
| 135/tcp | MSRPC | Bajo |
| 445/tcp | SMB | Signing NO obligatorio 🚨 |
| 5985/tcp | WinRM | Acceso remoto expuesto ⚠️ |

**Wazuh generó 401 alertas** — pico visible exactamente a las 17:53.
Hostname revelado: `WIN-J36GU3SVEOC` | SO: Windows Server 2022

# Alertas Wazuh — Reconocimiento

## Resumen
- **Total alertas:** 401 eventos
- **Hora del pico:** 17:53 UTC
- **Grupos:** windows, sca, windows_system, WEF
- **Alerta más relevante:** EventID 4672

## Evento más sospechoso detectado

```
EventID:     4672
Descripción: Special privileges assigned to new logon
Rule ID:     67028 | Level: 3
MITRE:       T1484 — Domain Policy Modification
Táctica:     Defense Evasion + Privilege Escalation

Usuario:     DefaultAppPool (IIS APPPOOL)
Privilegios: SeAssignPrimaryTokenPrivilege
             SeAuditPrivilege
             SeImpersonatePrivilege
Timestamp:   2026-05-31T21:51:15Z
```

## Análisis
Comportamiento normal de IIS al recibir petición HTTP. En entorno real debe correlacionarse
con otros eventos — SeImpersonatePrivilege puede ser explotado (PrintSpoofer, JuicyPotato).

# MITRE ATT&CK — Reconocimiento

| Táctica | Técnica | ID | Herramienta |
|---|---|---|---|
| Reconnaissance | Network Service Scanning | T1046 | Nmap 7.95 |

## IOCs
- IP atacante: `192.168.56.30`
- Hostname víctima: `WIN-J36GU3SVEOC`
- SMB Signing no obligatorio → vulnerable a relay attacks
- HTTP TRACE habilitado