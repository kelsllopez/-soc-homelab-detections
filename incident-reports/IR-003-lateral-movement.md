# IR-003 — Movimiento Lateral
| Campo | Valor |
|---|---|
| **ID** | IR-003 |
| **Fecha** | 2026-06-11 |
| **Hora** | 21:44:00 UTC |
| **Severidad** | 🔴 Alta |
| **Estado** | ✅ Cerrado |
| **MITRE** | T1021, T1135, T1087, T1046, T1078 |
| **Analista** | Katalina Sepúlveda López |

## Descripción
El atacante utilizó credenciales válidas (`usuarioprueba:Password123!`) para enumerar recursos SMB, listar 14 usuarios del dominio y conectarse vía WinRM.

## Cronología
```
21:44:00 → Enumeración SMB iniciada
21:44:30 → Recursos ADMIN$, C$, SYSVOL, NETLOGON expuestos
21:44:45 → 14 usuarios del dominio enumerados
21:44:50 → nmap confirma SMB solo en .20
21:45:00 → Conexión WinRM exitosa [~]
21:50-54 → Wazuh registra 125+ EventID 4624
```

## Recursos comprometidos
| Recurso | Impacto |
|---|---|
| ADMIN$ | Acceso total al sistema |
| C$ | Acceso completo al disco |
| SYSVOL | Políticas de dominio expuestas |
| NETLOGON | Scripts de sesión expuestos |

## Evidencia Wazuh
- EventID 4624: 125+ eventos (logins exitosos)
- EventID 5140: ~15 eventos (acceso a shares)

## Recomendaciones
- 🔴 Bloquear IP atacante en firewall
- 🔴 Deshabilitar WinRM si no es necesario operacionalmente
- 🔴 Cambiar contraseña de `usuarioprueba`
- 🟡 Restringir enumeración anónima de usuarios AD
- 🟡 Implementar segmentación de red