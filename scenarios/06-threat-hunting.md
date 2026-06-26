# Escenario 06 — Threat Hunting Proactivo
**Fecha:** 2026-06-20 | **Analista:** Katalina Sepúlveda López

## Concepto

> El analista reactivo espera alertas. El Threat Hunter **asume que ya hay algo malo** y sale a buscarlo.

---

## 7 hipótesis ejecutadas

### H1 — Logins en horario inusual (00:00–06:00)
```
Filtro: data.win.system.eventID: 4624
Rango:  00:00 - 06:00
Resultado: 1 login a las 02:23 AM — usuario kels, IP 192.168.1.105
Acción: Investigar si fue tarea automatizada o acceso no autorizado
```

### H2 — Cuentas que nunca deberían loguearse
```
Filtro 1: data.win.eventdata.targetUserName: Invitado  → 0 eventos ✅
Filtro 2: data.win.eventdata.targetUserName: krbtgt     → 0 eventos ✅
Conclusión: Sin evidencia de ataque Golden Ticket
```

### H3 — Servicios con ejecutables sospechosos
```
Filtro: data.win.system.eventID: 7045
Hallazgos:
  ❌ LogService → C:\Users\kels\backdoor.ps1 (SOSPECHOSO)
  ✅ WindowsUpdate → svchost.exe (legítimo)
  ⚠️ BackupService → C:\Scripts\backup.ps1 (INVESTIGAR)
```

### H4 — Scripts PowerShell en rutas inusuales
```
Filtro: data.win.system.eventID: 4104
Hallazgos:
  ❌ C:\Users\kels\backdoor.ps1 (conocido del Día 5)
  ❌ C:\Windows\Temp\temp_install.ps1 (SOSPECHOSO)
  ⚠️ C:\Scripts\backup.ps1 (verificar legitimidad)
```

### H5 — Modificaciones en Registro Run
```
Filtro: data.win.system.eventID: 4657
Hallazgos:
  ❌ HKLM\...\Run → SecurityHelper = backdoor.ps1 a las 02:23 AM
  ⚠️ HKLM\...\RunOnce → Cleanup = cleanup.ps1 a las 08:15 AM
Acción: Eliminar SecurityHelper inmediatamente
```

### H6 — FIM — Integridad de archivos
```
Filtro: rule.groups: syscheck
Hallazgos:
  ✅ 77 eventos syscheck detectados
  ✅ svcrestarttask modificado en tiempo real
  Hash MD5 cambió: ffab8f → a6a582
  MITRE T1565.001 confirmado
```

### H7 — Tácticas MITRE de la semana
| Táctica | Técnicas | Cantidad |
|---|---|---|
| Initial Access | T1078, T1190 | 2 |
| Persistence | T1543.003, T1547.001, T1053.005 | 3 |
| Privilege Escalation | T1078.002, T1548 | 1 |
| Defense Evasion | T1036, T1070.001 | 2 |
| Discovery | T1087, T1018 | 1 |
| Lateral Movement | T1021, T1550 | 1 |

---

## Reglas personalizadas creadas (Día 6)

Ver [`../../detections/wazuh-rules/custom-rules.xml`](../../detections/wazuh-rules/custom-rules.xml)

| Regla | Detecta | Nivel |
|---|---|---|
| 100001 | Servicio instalado con .ps1 | 12 |
| 100002 | Usuario a Domain Admins | 15 |
| 100003 | 5+ logins fallidos en 2 min | 14 |
| 100004 | Archivo .ps1 en System32 | 12 |
| 100005 | Nueva cuenta creada | 10 |