# IR-005 — Persistencia / Backdoor
| Campo | Valor |
|---|---|
| **ID** | IR-005 |
| **Fecha** | 2026-06-18 |
| **Severidad** | 🔴 Crítica |
| **Estado** | ✅ Cerrado |
| **MITRE** | T1543.003, T1053.005, T1547.001, T1036 |
| **Ticket** | INC-005 |
| **Escalamiento** | Reportado a SOC Tier 2 |
| **Analista** | Katalina Sepúlveda López |

## Descripción
El atacante instaló 4 capas de persistencia para garantizar acceso aunque sea descubierto y cambien contraseñas.

## Mecanismos detectados
| Capa | Mecanismo | Ruta/Nombre |
|---|---|---|
| 1 | Archivo malicioso | `C:\Windows\System32\backdoor.ps1` |
| 2 | Servicio falso (Auto) | `WindowsUpdateHelper` |
| 3 | Tarea programada | `WindowsSecurityUpdate` |
| 4 | Registro Run | `HKLM\...\Run\SecurityHelper` |

## Cronología
| Hora | Acción | Detección |
|---|---|---|
| 20:00:00 | `backdoor.ps1` creado | FIM Syscheck |
| 20:00:30 | Servicio instalado | EventID 7045 |
| 20:01:00 | Tarea creada | EventID 4698 |
| 20:01:30 | Registro modificado | FIM Registry |
| 21:00:00 | Analista detecta y contiene | Investigación SOC |

## Acciones de contención
```powershell
Remove-Item "C:\Windows\System32\backdoor.ps1" -Force
Stop-Service "WindowsUpdateHelper" -Force
sc.exe delete "WindowsUpdateHelper"
Unregister-ScheduledTask -TaskName "WindowsSecurityUpdate" -Confirm:$false
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -Name "SecurityHelper" -Force
```

## Estado final: 🟢 SISTEMA LIMPIO

## Recomendaciones
- 🔴 Auditar servicios instalados — detectar nombres que imitan Windows
- 🔴 Revisar tareas programadas activas
- 🔴 FIM habilitado en System32, Tasks, y registro Run
- 🟡 Alertas automáticas EventID 7045 (nivel > 10)
- 🟡 Application Whitelisting

## Lecciones aprendidas
- FIM es crítico para detectar persistencia en tiempo real
- La persistencia puede sobrevivir a cambios de contraseña
- Múltiples capas hacen más difícil la eliminación completa