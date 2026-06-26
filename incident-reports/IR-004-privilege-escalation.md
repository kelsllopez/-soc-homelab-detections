# IR-004 — Escalada de Privilegios
| Campo | Valor |
|---|---|
| **ID** | IR-004 |
| **Fecha** | 2026-06-17 |
| **Severidad** | 🔴 Crítica |
| **Estado** | ✅ Cerrado |
| **MITRE** | T1136, T1078, T1098, T1484, T1531 |
| **Ticket** | INC-004 |
| **Escalamiento** | Reportado a SOC Tier 2 |
| **Analista** | Katalina Sepúlveda López |

## Descripción
Creación de cuenta no autorizada `hackersoc` y adición a Domain Admins y Administradores locales. Acceso administrativo total al dominio `soc.local`.

## Cronología del ataque
| Hora (UTC) | EventID | Descripción |
|---|---|---|
| 21:16:06 | 4672 | Privilegios especiales asignados |
| 21:22:37 | 4720 | Cuenta `hackersoc` CREADA |
| 21:23:48 | 4732 | `hackersoc` → Administradores (Level 12 🔴) |
| 21:35:55 | 4728 | `hackersoc` → Domain Admins |
| 22:06:37 | 4726 | `hackersoc` eliminada (contención) |

## Impacto
- 🔴 Acceso administrativo total al dominio
- 🔴 Capacidad de crear/modificar/eliminar usuarios
- 🔴 Posible acceso a todos los equipos del dominio
- 🔴 Capacidad de modificar políticas de seguridad

## Acciones de contención
```powershell
# 1. Eliminar cuenta maliciosa
net user hackersoc /delete

# 2. Remover de Domain Admins
net group "Domain Admins" hackersoc /delete /domain

# 3. Remover de Administradores locales
net localgroup Administradores hackersoc /delete

# 4. Verificar contención en Wazuh (EventID 4726)
```

## Recomendaciones
- 🔴 Auditar TODOS los grupos administrativos del dominio
- 🔴 Alertas automáticas nivel CRÍTICO para EventID 4728
- 🟡 Principio de mínimo privilegio en todas las cuentas
- 🟡 MFA obligatorio para cuentas administrativas
- 🟡 Monitoreo de cambios en grupos privilegiados

## Lecciones aprendidas
- EventID 4728 debe generar alerta nivel ALTO automáticamente
- La cadena de ataque completa debe ser visible en el SIEM
- Monitoreo de grupos privilegiados es crítico en entornos AD