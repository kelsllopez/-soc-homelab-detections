# IR-001 — Reconocimiento de Red
| Campo | Valor |
|---|---|
| **ID** | IR-001 |
| **Fecha** | 2026-06-09 |
| **Severidad** | 🟡 Media |
| **Estado** | ✅ Cerrado |
| **MITRE** | T1046 — Network Service Scanning |
| **Analista** | Katalina Sepúlveda López |

## Descripción
Escaneo de red desde `192.168.56.30` (Kali) hacia `192.168.56.0/24`. Identificó 3 activos, puertos abiertos y versiones de servicios.

## Hallazgos técnicos
- Puertos expuestos en `.20`: 53, 80, 135, 139, 445, 5985
- SMB Signing NO obligatorio
- HTTP TRACE habilitado en IIS 10.0
- WinRM expuesto sin restricción

## Contención
- No requerida (no hubo acceso)

## Recomendaciones
- 🔴 Deshabilitar HTTP TRACE en IIS
- 🔴 Hacer obligatorio SMB Signing
- 🟡 Restringir WinRM a IPs autorizadas

## Lecciones aprendidas
El reconocimiento exitoso proveyó el mapa de ataque completo para los días siguientes. La superficie de ataque era demasiado amplia.