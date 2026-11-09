# windows_ltsc

Mis notas de post-instalación de Windows 11 LTSC en mi Lenovo Legion. El otro SSD lleva Fedora, y cambio de disco a mano en lugar de hacer dual boot.

Elegí LTSC por su estado minimalista: sin el bloat de IA y de apps preinstaladas que traen Home y Pro.

> Son notas personales para mi equipo, no una guía general. Revisa cada comando antes de ejecutarlo en el tuyo.

## Antes de empezar

Abre PowerShell **como administrador** para evitar avisos de permisos durante todo el proceso.

Para poder ejecutar scripts descargados, habilito la ejecución solo en la sesión actual:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

`-Scope Process` hace que el cambio dure únicamente mientras esa ventana de PowerShell esté abierta; al cerrarla, la política vuelve a la de siempre.

## Limpieza y ajustes del sistema

Uso el script de Chris Titus Tech (WinUtil), que reúne ajustes de privacidad, servicios y apps:

```powershell
irm https://christitus.com/win | iex
```

`irm` descarga el script desde internet e `iex` lo ejecuta de inmediato. Por eso conviene leerlo antes en [su repositorio](https://github.com/ChrisTitusTech/winutil) y no correrlo a ciegas.

## Instalar winget

LTSC no trae winget, así que lo instalo a mano. Primero, el paquete `.msixbundle` del instalador de aplicaciones (App Installer) descargado previamente:

```powershell
Add-AppxPackage -Path "C:\Ruta\Al\Archivo.msixbundle"
```

Después, el módulo de PowerShell de winget y la reparación del administrador de paquetes para todos los usuarios:

```powershell
Install-Module -Name Microsoft.WinGet.Client -Force
Repair-WinGetPackageManager -AllUsers
winget source update
```

## Aplicaciones

Primero los runtimes de Visual C++:

```powershell
winget install abbodi1406.vcredist
```

Después, todo lo demás en un solo comando. Las banderas aceptan los acuerdos de las fuentes y hacen la instalación silenciosa:

```powershell
winget install blizzard.battlenet Discord.Discord OBSProject.OBSStudio Valve.Steam BraveSoftware.Brave VideoLAN.VLC 7zip.7zip HeroicGamesLauncher.HeroicGamesLauncher yt-dlp.yt-dlp ffmpeg Microsoft.VCRedist.2015+.x64 Microsoft.DirectX StirlingTools.StirlingPDF flameshot --source winget --accept-source-agreements --accept-package-agreements --silent
```

| Categoría | Programas |
|---|---|
| Juegos | Battle.net, Steam, Heroic Games Launcher |
| Streaming y grabación | OBS Studio |
| Comunicación | Discord |
| Navegador y multimedia | Brave, VLC, yt-dlp, ffmpeg |
| Utilidades | 7-Zip, Stirling-PDF, Flameshot |
| Runtimes | Visual C++, DirectX |

Nota: Battle.net se instala en `C:\Program Files (x86)\Battle.net`.

### Descargar video con yt-dlp

```powershell
yt-dlp -o "$env:USERPROFILE\Videos\YT_Downloads\%(title)s.%(ext)s" -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best" "https://youtube.com/watch?v=..."
```

Guarda el video en la carpeta Videos del usuario, con el título como nombre de archivo, y prefiere mp4.

## Acceso a mi servidor Samba (red de casa)

Estos ajustes son para conectarme al servidor Samba de mi homelab (Debian) desde este equipo:

```powershell
Set-SmbClientConfiguration -RequireSecuritySignature $false -Force
Set-SmbClientConfiguration -EnableInsecureGuestLogons $true -Force
Restart-Service LanmanWorkstation -Force
```

- `RequireSecuritySignature $false`: deja de exigir firma obligatoria en las conexiones SMB.
- `EnableInsecureGuestLogons $true`: permite entrar a recursos compartidos como invitado, sin credenciales.
- `Restart-Service LanmanWorkstation`: aplica los cambios.

> **Advertencia:** ambos ajustes bajan la seguridad del cliente SMB. Los uso solo porque mi servidor está en mi red doméstica y es de confianza. No los apliques en una red que no controles.
