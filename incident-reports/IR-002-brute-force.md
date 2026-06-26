# IR-002 — Ataque de Fuerza Bruta
| Campo | Valor |
|---|---|
| **ID** | IR-002 |
| **Fecha** | 2026-06-11 |
| **Hora** | 16:32:44 UTC |
| **Severidad** | 🔴 Alta |
| **Estado** | ✅ Cerrado |
| **MITRE** | T1110.001 — Brute Force: Password Guessing |
| **Analista** | Katalina Sepúlveda López |

## Descripción
20+ intentos de autenticación fallidos contra SMB (445/tcp) desde `192.168.56.30` en menos de 2 minutos.

## Evidencia Wazuh
```
Rule:    60122 — Logon Failure
EventID: 4625
Level:   5
Usuario: usuarioprueba
IP:      192.168.56.30
Tiempo:  < 2 minutos
```

## Resultado del ataque
**FALLIDO** — Sin acceso logrado. Las credenciales eran incorrectas en esta etapa.

## Contención aplicada
- Recomendación: bloqueo de IP en firewall
- Recomendación: Account Lockout Policy (5 intentos → bloqueo 15 min)

## Recomendaciones
- 🔴 Implementar Account Lockout Policy
- 🔴 Alerta automática para 5+ EventID 4625 desde misma IP en 60s
- 🟡 Revisar contraseñas del dominio (política de complejidad)