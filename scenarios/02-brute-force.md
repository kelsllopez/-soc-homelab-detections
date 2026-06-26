# Escenario 02 — Brute Force SMB
**Fecha:** 2026-06-11 | **Hora:** 16:32:44 | **Analista:** Katalina Sepúlveda López

## Contexto
Con los puertos descubiertos el Día 1, el atacante apunta al servicio SMB (puerto 445) con fuerza bruta de credenciales.

## Ejecución desde Kali Linux

```bash
# Intentos múltiples fallidos contra SMB
smbclient -L //192.168.56.20 -U usuarioprueba --password=contraseñafalsa

# Loop de 20 intentos automatizados
for i in {1..20}; do
  smbclient -L //192.168.56.20 -U usuarioprueba --password=incorrecta$i
done
```

## Resultado
- **20+ intentos fallidos** en menos de 2 minutos
- Wazuh disparó Rule 60122 (Logon Failure)
- EventID 4625 generado múltiples veces
- Ataque **fallido** — sin acceso logrado en esta etapa

## Credenciales objetivo
- Usuario: `usuarioprueba`
- Contraseña usada luego exitosamente (Día 3): `Password123!`

