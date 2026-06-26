# Escenario 04 — Escalada de Privilegios
**Fecha:** 2026-06-17 | **Hora inicio:** 21:16:06 | **Analista:** Katalina Sepúlveda López

## Contexto
El atacante tiene acceso con la cuenta `usuarioprueba` (usuario normal). Objetivo: convertirse en Domain Admin.

## Comandos ejecutados (PowerShell como Admin en Windows Server)

### Paso 1 — Crear cuenta maliciosa
```powershell
net user hackersoc Password123! /add /domain
```

### Paso 2 — Agregar a Administradores locales
```powershell
net localgroup Administradores hackersoc /add
```

### Paso 3 — Agregar a Domain Admins
```powershell
net group "Domain Admins" hackersoc /add /domain
# (también realizado manualmente vía GUI Active Directory)
```

### Paso 4 — Verificar privilegios
```powershell
whoami /priv
net user hackersoc
query user
```

## Cronología de ataque

| Hora (UTC) | EventID | Acción |
|---|---|---|
| 21:16:06 | 4672 | Privilegios especiales asignados (Administrador) |
| 21:22:37 | 4720 | Cuenta `hackersoc` CREADA |
| 21:23:48 | 4732 | `hackersoc` agregado a Administradores locales |
| 21:35:55 | 4728 | `hackersoc` agregado a Domain Admins |
| 22:06:37 | 4726 | Cuenta `hackersoc` eliminada (contención SOC) |


# Alertas Wazuh — Escalada de Privilegios

## Cadena de ataque en Wazuh

### EventID 4720 — Cuenta creada
```
Búsqueda: data.win.system.eventID: 4720
Quién:    Administrador
Qué:      Creó usuario hackersoc
Cuándo:   21:22:37 UTC
MITRE:    T1136 — Create Account
```

### EventID 4732 — Agregado a grupo local
```
Búsqueda: data.win.system.eventID: 4732
Quién:    Administrador
Qué:      hackersoc → Administradores (Builtin)
Cuándo:   21:23:48 UTC
Level:    12 (CRÍTICO) ← Wazuh detecta esto como crítico
```

### EventID 4728 — Agregado a Domain Admins
```
Búsqueda: data.win.system.eventID: 4728
Quién:    Administrador
Qué:      hackersoc → Domain Admins
Cuándo:   21:35:55 UTC
MITRE:    T1098 — Account Manipulation
```

### EventID 4726 — Cuenta eliminada (contención)
```
Búsqueda: data.win.system.eventID: 4726
Quién:    Administrador (SOC)
Qué:      Eliminó usuario hackersoc
Cuándo:   22:06:37 UTC
Estado:   AUDIT_SUCCESS
MITRE:    T1531 — Account Access Removal
```

## Tabla de EventIDs clave para Privilege Escalation

| EventID | Qué significa | Severidad SOC |
|---|---|---|
| 4720 | Cuenta nueva creada | Alta |
| 4722 | Cuenta habilitada | Media |
| 4728 | Agregado a grupo global (Domain Admins) | Alta |
| 4732 | Agregado a grupo local | Alta |
| 4672 | Privilegios especiales | Alta |
| 4726 | Cuenta eliminada | Media |
| 4740 | Cuenta bloqueada | Alta |


# MITRE ATT&CK — Escalada de Privilegios

| Técnica | ID | Descripción |
|---|---|---|
| Create Account | T1136 | Creación de cuenta hackersoc |
| Valid Accounts | T1078 | Uso de cuenta para acceso |
| Account Manipulation | T1098 | Agregar usuario a grupo admin |
| Domain Policy Modification | T1484 | Modificación de políticas |
| Account Access Removal | T1531 | Eliminación de cuenta (contención) |

## Privilegios obtenidos por hackersoc
```
SeSecurityPrivilege
SeTakeOwnershipPrivilege
SeLoadDriverPrivilege
SeBackupPrivilege
SeRestorePrivilege
SeDebugPrivilege
SeImpersonatePrivilege
SeEnableDelegationPrivilege
```