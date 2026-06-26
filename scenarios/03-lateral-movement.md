# Escenario 03 — Movimiento Lateral
**Fecha:** 2026-06-11 | **Hora:** 21:44:00 | **Analista:** Katalina Sepúlveda López

## Cronología del ataque

```
[21:44:00] Kali inició enumeración SMB
[21:44:30] Enumeración de recursos compartidos
[21:44:45] Enumeración de 14 usuarios del dominio
[21:44:50] Escaneo de red con nmap (puerto 445)
[21:45:00] Verificación de WinRM
[21:50-21:54] Wazuh registra 125+ eventos 4624
```

## Comandos ejecutados (Kali Linux)

### Paso 1 — Verificar herramientas
```bash
which crackmapexec
which smbclient
```

### Paso 2 — Enumerar recursos compartidos SMB
```bash
crackmapexec smb 192.168.56.20 -u usuarioprueba -p Password123! --shares
```

**Resultado:**
```
SMB  192.168.56.20  445  WIN-J36GU3SVEOC  [+] soc.local\usuarioprueba:Password123!
SMB  ...  ADMIN$   Admin remota         ← acceso total al sistema
SMB  ...  C$       Recurso predeterminado
SMB  ...  IPC$     IPC remota
SMB  ...  NETLOGON Inicio de sesión
SMB  ...  SYSVOL   Políticas de dominio  ← scripts de dominio expuestos
```

### Paso 3 — Listar usuarios del dominio
```bash
crackmapexec smb 192.168.56.20 -u usuarioprueba -p Password123! --users
```

**14 usuarios expuestos:**
`ksepulveda, klopez, analista1, analista2, soc1, soc2, chefsito, anime1, anime2, Administrador, krbtgt, Invitado, usuarioprueba (badpwdcount: 21)`

### Paso 4 — Escanear SMB en toda la red
```bash
nmap -p 445 192.168.56.0/24 --open
```

### Paso 5 — Conexión WinRM (CRÍTICO)
```bash
crackmapexec winrm 192.168.56.20 -u usuarioprueba -p Password123!
```
**Resultado:** `[~]` = autenticación exitosa → **el atacante puede ejecutar comandos remotos en PowerShell**


# Alertas Wazuh — Movimiento Lateral

## Búsqueda 1: EventID 4624 (Logins exitosos)
```
Filtro: data.win.system.eventID: 4624
Total:  125+ eventos
Significado: Login exitoso desde Kali hacia Windows Server
Rule:   60106 (Windows Logon Success) — Level 3
```

## Búsqueda 2: Por IP de Kali
```
Filtro: 192.168.56.30
Muestra TODOS los eventos generados por el atacante
Incluye: logins, accesos a shares, enumeraciones
```

## Evidencia completa

| EventID | Cantidad | Significado |
|---|---|---|
| 4624 | 125+ | Logins exitosos desde Kali |
| 5140 | ~15 | Accesos a ADMIN$, C$, SYSVOL |
| 4625 | 0 | Sin fallos (credenciales correctas) |


# MITRE ATT&CK — Movimiento Lateral

| Técnica | ID | Código que la generó |
|---|---|---|
| Valid Accounts | T1078 | `-u usuarioprueba -p Password123!` |
| Network Share Discovery | T1135 | `--shares` |
| Account Discovery | T1087 | `--users` |
| Network Service Scanning | T1046 | `nmap -p 445` |
| Remote Services (WinRM) | T1021 | `crackmapexec winrm` |

## Recursos comprometidos

| Recurso | Riesgo |
|---|---|
| ADMIN$ | Acceso total al sistema |
| C$ | Acceso completo al disco |
| SYSVOL | Políticas de dominio expuestas |
| NETLOGON | Scripts de inicio de sesión |