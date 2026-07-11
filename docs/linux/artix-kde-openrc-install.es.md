---
description: >-
  Instalación manual paso a paso de Artix Linux con OpenRC, subvolúmenes Btrfs, zram y KDE Plasma en hardware AMD.
---

<!--
Source: linux/artix-kde-openrc-install.md
Last translated: 2026-07
-->

# Guía de instalación manual de Artix Linux

**Equipo objetivo:** PC / portátil (procesador AMD Ryzen y GPU AMD)

**Sistema de archivos:** Btrfs con subvolúmenes

**Swap:** zram (sin partición de intercambio)

**Escritorio:** KDE Plasma (lo mínimo indispensable)

**Init:** OpenRC

**Unidad:** SSD NVMe (`/dev/nvme0n1`)

La wiki de Artix también es un gran recurso de referencia adicional y solución de problemas: [wiki.artixlinux.org/](https://wiki.artixlinux.org/)

Los pasos generales para instalar Artix (similares a los de la mayoría de distribuciones Linux) incluyen: conectarse a internet, particionar el disco, formatear las particiones, montar las particiones, instalar el kernel de Linux y los paquetes del sistema base, configurar la localización, instalar el gestor de arranque, añadir usuarios y establecer contraseñas, configurar la red, instalar el entorno de escritorio y el gestor de inicio de sesión, activar los servicios esenciales y realizar la configuración posterior a la instalación.

* * *

## 1\. Arrancar la ISO live

Descarga la ISO base con OpenRC desde [artixlinux.org/download.php](https://artixlinux.org/download.php). Grábala en una unidad USB

- PulsarTECH tiene una excelente guía de USB multiarranque con Ventoy + Linux en: [youtube.com/watch?v=BfjLJ0CqWsY](https://www.youtube.com/watch?v=BfjLJ0CqWsY)

Comando de Linux para el USB:

```bash
sudo dd bs=4M if=artix-base-openrc-YYYYMMDD-x86_64.iso of=/dev/sdX status=progress oflag=sync
```

Arranca el PC desde el USB. Aquí tienes una lista de referencia con las teclas del menú de arranque de varios fabricantes:

| Fabricante | Tecla del menú de arranque | Tecla de BIOS/UEFI |
| :--- | :--- | :--- |
| **Acer** | F12, Esc, F9 | F2, Supr |
| **Asus** | F8, Esc | F2, Supr |
| **Dell** | F12 | F2  |
| **HP** | F9, Esc | F10 |
| **Lenovo** | F12, F10, F8 (o botón Novo) | F2, F1 |
| **MSI** | F11 | Supr |
| **Toshiba** | F12 | F2, F1, Esc |
| **Samsung** | F12, F2, Esc | F2  |
| **Sony VAIO** | F11, F10, Esc (o botón Assist) | F2, F1, F3 |
| **Gigabyte** | F12 | Supr, F2 |
| **Intel NUC** | F10 | F2  |

Una vez en la terminal, el inicio de sesión por defecto es `root` y la contraseña es `artix`.

* * *

## 2\. Conectarse a internet

**Ethernet**:

Si usas ethernet (recomendado), ve directo al paso "Verificar la conectividad" más abajo.

**Wi-Fi**:

La interfaz inalámbrica puede estar bloqueada por software (soft-blocked) por defecto. Desbloquéala primero:

```bash
rfkill unblock wifi
ip link set wlan0 up
```

Después conéctate con `wpa_supplicant`:

```bash
wpa_supplicant -B -i wlan0 -c <(wpa_passphrase "TuSSID" "TuContraseña")

dhcpcd wlan0
```

Como alternativa, puedes usar la utilidad de gestión de conexiones (ConnMan) incluida en la ISO de Artix:

```bash
connmanctl
> enable wifi
> scan wifi
> agent on
> connect wifi_<autocompletar-con-tab>

> quit
```

Verificar la conectividad:

```bash
ping -c 3 artixlinux.org
```

* * *

## 3\. Configurar teclado y hora

Establecer la distribución del teclado:

```bash
loadkeys us
```

* * *

## 4\. Particionar la unidad NVMe

Si usas una ranura NVMe M.2 2280, la unidad objetivo aparecerá como `/dev/nvme0n1`.

Verifica tu unidad:

```bash
lsblk
```

Si la unidad tiene una firma de swap antigua, desactívala primero:

```bash
swapoff /dev/nvme0n1p2 2>/dev/null
```

Si la unidad tiene una instalación anterior, límpiala primero:

```bash
wipefs -af /dev/nvme0n1
```

Lanza cfdisk para particionar la unidad:

```bash
cfdisk /dev/nvme0n1
```

**Cuando cfdisk se abra:**

1. Si te pide un tipo de etiqueta, selecciona **gpt**. Si la unidad ya tiene una tabla MBR/DOS, borra primero todas las particiones existentes, después ve al menú inferior y selecciona **\[ Write \]** para aplicar, luego sal y vuelve a ejecutar `cfdisk /dev/nvme0n1` — te pedirá un nuevo tipo de etiqueta y podrás elegir **gpt**.
2. Selecciona **\[ New \]** en el espacio libre, escribe **1G** y pulsa Enter. Esto crea la partición EFI.
3. Con esa partición resaltada, selecciona **\[ Type \]** y elige **EFI System**.
4. Baja con la flecha al espacio libre restante, selecciona **\[ New \]** y pulsa Enter para aceptar todo el tamaño restante. Esto crea la partición raíz (el tipo por defecto es **Linux filesystem**, que es el correcto).
5. Selecciona **\[ Write \]**, escribe **yes** para confirmar y luego **\[ Quit \]**.

**Disposición final:**

| Partición | Tamaño | Tipo | Propósito |
| --- | --- | --- | --- |
| `/dev/nvme0n1p1` | 1 GB | EFI System | Arranque EFI |
| `/dev/nvme0n1p2` | Resto | Linux filesystem | Raíz Btrfs |

* * *

## 5\. Formatear las particiones

Formatear la partición EFI como FAT32:

```bash
mkfs.fat -F32 -n EFI /dev/nvme0n1p1
```

Formatear la partición raíz como Btrfs:

```bash
mkfs.btrfs -L ARTIX /dev/nvme0n1p2
```

* * *

## 6\. Crear los subvolúmenes Btrfs

Montar el subvolumen de nivel superior:

```bash
mount /dev/nvme0n1p2 /mnt
```

Crear los subvolúmenes:

```bash
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@var
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@snapshots
```

Desmontar el nivel superior:

```bash
umount /mnt
```

* * *

## 7\. Montar los subvolúmenes

Usa opciones de montaje optimizadas para SSD.

Define las opciones de montaje una sola vez:

```bash
BTRFS_OPTS="noatime,ssd,compress=zstd,space_cache=v2,discard=async"
```

Montar el subvolumen raíz:

```bash
mount -o ${BTRFS_OPTS},subvol=@ /dev/nvme0n1p2 /mnt
```

Crear los directorios de los puntos de montaje:

```bash
mkdir -p /mnt/boot/efi
mkdir -p /mnt/home
mkdir -p /mnt/var
mkdir -p /mnt/var/log
mkdir -p /mnt/var/cache
mkdir -p /mnt/.snapshots
```

Montar el resto de subvolúmenes:

```bash
mount -o ${BTRFS_OPTS},subvol=@home     /dev/nvme0n1p2 /mnt/home
mount -o ${BTRFS_OPTS},subvol=@var      /dev/nvme0n1p2 /mnt/var
mount -o ${BTRFS_OPTS},subvol=@log      /dev/nvme0n1p2 /mnt/var/log
mount -o ${BTRFS_OPTS},subvol=@cache    /dev/nvme0n1p2 /mnt/var/cache
mount -o ${BTRFS_OPTS},subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
```

Si recibes un error de que el punto de montaje no existe, simplemente vuelve atrás y crea de nuevo el directorio del punto de montaje.

Montar la partición EFI:

```bash
mount /dev/nvme0n1p1 /mnt/boot/efi
```

**Nota: ¿por qué zram en lugar de una partición de swap?** En btrfs, los archivos de swap requieren un manejo especial \[sin compresión, sin copy-on-write (COW)\]. Zram es más simple, más rápido (RAM comprimida) y evita el desgaste del SSD. Con 16–96 GB de DDR5 en un PC, zram es la mejor opción. Nota: esto significa que la hibernación (suspensión a disco) no está disponible (la hibernación requiere swap). La suspensión normal (cerrar la tapa, suspensión a RAM) funciona sin problemas.

* * *

## 8\. Instalar el sistema base

Refrescar los mirrors de paquetes:

```bash
pacman -Sy artix-keyring
```

Instalar los paquetes base:

```bash
basestrap /mnt base base-devel openrc elogind-openrc \
  linux linux-headers linux-firmware \
  amd-ucode \
  btrfs-progs \
  grub efibootmgr \
  dbus dbus-openrc \
  sudo \
  networkmanager networkmanager-openrc \
  zramen zramen-openrc \
  os-prober \
  pipewire pipewire-openrc pipewire-alsa pipewire-pulse pipewire-pulse-openrc pipewire-jack wireplumber wireplumber-openrc alsa-utils sof-firmware \
  nano vim neovim \
  git wget curl \
  reflector \
```

Ejecuta reflector para usar los mirrors más rápidos (ayudará a acelerar la instalación de paquetes a medida que avances en la instalación):

```bash
reflector --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
```

**Algunas notas sobre los paquetes:**

- `linux` — kernel base de Linux, cabeceras y firmware
- `amd-ucode` — actualizaciones de microcódigo para procesadores AMD; NOTA: si tu procesador es Intel, sustitúyelo por `intel-ucode`
- `btrfs-progs` — herramientas de gestión de Btrfs
- `elogind-openrc` — gestión de sesiones (necesario para sesiones de escritorio sin systemd)
- `networkmanager-openrc` — script de servicio OpenRC para NetworkManager (el widget de red de KDE Plasma requiere NetworkManager)
- `zramen-openrc` — script de servicio OpenRC para zram (compresión de RAM)
- `os-prober` — utilidad para buscar particiones arrancables de otros SO (por si haces arranque dual en el futuro)
- `pipewire-openrc` — scripts de servicio OpenRC para el audio con PipeWire (necesario para el audio del PC)
- `nano` — editores de texto (nano, VIM y NeoVIM)
- `git` — utilidades esenciales (git, wget, curl) para instalar otros paquetes y scripts

* * *

## 9\. Generar el fstab

```bash
fstabgen -U /mnt >> /mnt/etc/fstab
```

Verifica que cada montaje sea correcto:

```bash
cat /mnt/etc/fstab
```

Deberías ver seis entradas Btrfs (una por subvolumen) más la entrada FAT32 de EFI.

* * *

## 10\. Hacer chroot al nuevo sistema

```bash
artix-chroot /mnt
```

* * *

## 11\. Zona horaria y locale

Establecer la zona horaria (/usr/share/Región/Ciudad, con America/New_York como ejemplo seleccionado):

```bash
ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
hwclock --systohc
```

Establecer el locale:

```bash
nano /etc/locale.gen
# Descomenta: en_US.UTF-8 UTF-8

locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

* * *

## 12\. Hostname y hosts

Establecer el hostname (nombre del sistema Linux):

```bash
# Pon como hostname el nombre que quieras darle al sistema
echo "hostname" > /etc/hostname
echo "hostname" > /etc/conf.d/hostname
```

Edita `/etc/hosts`:

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   hostname.localdomain hostname
```

* * *

## 13\. Mapa de teclado de consola (opcional)

```bash
# Solo si usas una distribución de teclado distinta de la de EE. UU.
echo "KEYMAP=us" > /etc/vconsole.conf
```

* * *

## 14\. Establecer la contraseña de root y crear tu usuario

Establecer la contraseña de root:

```bash
passwd
```

Crear tu usuario y su contraseña:

```bash
useradd -m -G wheel -s /bin/bash tunombredeusuario
passwd tunombredeusuario
```

Añadir el usuario al grupo sudo:

```bash
EDITOR=nano visudo
# Descomenta (en la sección wheel): %wheel ALL=(ALL:ALL) ALL
```

* * *

## 15\. Configurar mkinitcpio

Comprueba que `/etc/mkinitcpio.conf` incluye el módulo de Btrfs:

```bash
cat /etc/mkinitcpio.conf | grep "^HOOKS"
```

Comprueba que los módulos aparecen en la salida. Asegúrate de tener:

```bash
MODULES=(btrfs)
HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)
```

Después regenera el initramfs:

```bash
mkinitcpio -P
```

* * *

## 16\. Instalar y configurar GRUB

Instalar GRUB:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Artix
```

Generar la configuración:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

* * *

## 17\. Habilitar los servicios esenciales

Habilitar la red, el inicio de sesión al arranque y zram:

```bash
rc-update add NetworkManager default
rc-update add elogind boot
rc-update add dbus default
rc-update add zramen default
```

* * *

## 18\. Configurar zram (en lugar de swap)

Configura zram editando `/etc/conf.d/zramen`:

```bash
nano /etc/conf.d/zramen
```

Establece el contenido (ajusta el tamaño según corresponda — una elección habitual es el 50 % de tu RAM):

```bash
ZRAM_SIZE=50%
ZRAM_ALGO=lz4
# o zstd
```

* * *

## 19\. Instalar KDE Plasma (instalación mínima)

Instala solo el escritorio Plasma mínimo — sin la suite de aplicaciones de KDE:

```bash
pacman -S plasma-desktop sddm sddm-openrc
# Elige todos los valores por defecto (1)
```

Esto te da el shell de Plasma, Preferencias del sistema y el gestor de inicio de sesión SDDM — nada más. Coincide con el enfoque minimalista de la ISO de Artix con KDE, que solo incluye un gestor de archivos, un reproductor multimedia, un navegador y un visor de documentos.

**Añadir las apps esenciales**:

```bash
pacman -S dolphin          # Gestor de archivos
pacman -S konsole          # Terminal
pacman -S firefox          # Navegador web
pacman -S okular           # Visor de documentos
pacman -S vlc              # Reproductor multimedia
```

**Para que los widgets de Plasma funcionen**:

```bash
pacman -S \
  plasma-pa \
  plasma-nm \
  kscreen \
  powerdevil \
  bluedevil \
  kio-extras \
  udisks2 udisks2-openrc \
  polkit-kde-agent \
  kde-cli-tools \
  kde-gtk-config \
  xdg-user-dirs \
```

**Habilitar SDDM:**

```bash
rc-update add sddm default
```

* * *

## 20\. Driver de GPU AMD

Para GPUs AMD, instala los drivers Mesa:

```bash
pacman -S mesa vulkan-radeon libva-mesa-driver
```

Para GPUs NVIDIA, se necesita el repositorio lib32. Consulta la sección **Habilitar repositorios adicionales (para acceder a más paquetes)** y después ejecuta:

```bash
pacman -S nvidia nvidia-utils lib32-nvidia-utils nvidia-settings
```

* * *

## 21\. Bluetooth

Instala las utilidades de Bluetooth (bluez) y luego habilítalo.

```bash
pacman -S bluez bluez-utils bluez-openrc
rc-update add bluetoothd default
```

* * *

## 22\. Gestión de energía (para portátiles)

Añade una herramienta de gestión de batería (TLP) para portátiles con Linux.

```bash
pacman -S tlp
```

* * *

## 23\. Actualizaciones de firmware de los dispositivos

Instala el gestor de actualizaciones de firmware. Consulta la **Lista de comprobación post-instalación** para los comandos a ejecutar tras reiniciar y actualizar el firmware de los dispositivos.

```bash
pacman -S fwupd
```

* * *

## 24\. Salir, desmontar y reiniciar

```bash
exit                      # Salir del chroot
umount -R /mnt            # Desmontar recursivamente
reboot
```

Retira la unidad USB cuando el sistema se reinicie. Deberías ver GRUB y, después, la pantalla de inicio de sesión de SDDM.

* * *

## Lista de comprobación post-instalación

Tras iniciar sesión en Plasma, abre la terminal (Konsole):

**Verificar la Wi-Fi**

```bash
ping -c 3 artixlinux.org
```

Si hay errores y no estás conectado a internet:

```bash
nmcli device wifi list
```

Eso debería escanear y mostrar las redes disponibles. Después conéctate:

```bash
nmcli device wifi connect "TuSSID" password "TuContraseña"
```

**Verificar que zram está activo:**

```bash
swapon --show
# Debería mostrar /dev/zram0 con el tamaño que configuraste
```

**Verificar los subvolúmenes Btrfs:**

```bash
btrfs subvolume list /
```

```
/dev/nvme0n1
├── nvme0n1p1   1 GB    FAT32   /boot/efi     (partición de sistema EFI)
└── nvme0n1p2   Resto   Btrfs   (subvolúmenes)
    ├── @                       /
    ├── @home                   /home
    ├── @var                    /var
    ├── @log                    /var/log
    ├── @cache                  /var/cache
    └── @snapshots              /.snapshots
```

**Sin partición de swap** — zram se encarga de la compresión de memoria en RAM.

**Verificar el driver de la GPU:**

```bash
glxinfo | grep "OpenGL renderer"
# Debería mostrar AMD Radeon / RDNA
```

**Comprobar y ejecutar actualizaciones de firmware:**

```bash
fwupdmgr refresh
fwupdmgr get-updates
fwupdmgr update
# después reinicia para instalar las actualizaciones de firmware
```

**Iniciar los dispositivos de audio:**

Añade y ejecuta los servicios de PipeWire (audio) para OpenRC.

```bash
rc-update add -U pipewire default
rc-update add -U pipewire-pulse default
rc-update add -U wireplumber default
rc-service -U pipewire start
rc-service -U pipewire-pulse start
rc-service -U wireplumber start
```

**Habilitar repositorios adicionales (para acceder a más paquetes):**

Edita `/etc/pacman.conf` y añade mirrors adicionales:

```bash
sudo nano /etc/pacman.conf
```

Añade la biblioteca lib32 para poder instalar Steam:

```bash
[lib32]
Include = /etc/pacman.d/mirrorlist
# Descomenta esta sección
```

**Actualizar todos los paquetes:**

```bash
sudo pacman -Syu
```

Ejecútalo una vez tras la instalación limpia y al menos una vez a la semana para actualizar todos los paquetes y mantener el sistema.

* * *

## Paquetes adicionales y actualizaciones

**Paquetes adicionales recomendados (algunos ya instalados en los pasos iniciales, pero se incluyen de nuevo por si acaso):**

Respaldos

```bash
sudo pacman -S timeshift snapper
```

- **Timeshift** servicio de respaldo
- **Snapper** servicio de respaldo alternativo

Servicios esenciales

```bash
sudo pacman -S \
  networkmanager networkmanager-openrc \
  elogind elogind-openrc \
  acpid acpid-openrc \
  cronie cronie-openrc \
  bluez bluez-openrc bluez-utils \
  cups cups-openrc system-config-printer \
  ntp ntp-openrc \
  pipewire-pulse pipewire-pulse-openrc pipewire-jack wireplumber wireplumber-openrc alsa-utils sof-firmware \
  pavucontrol \
  sof-firmware alsa-firmware \
  fwupd bolt upower \
  sudo nano vim git curl wget rsync unzip zip \
  reflector pacman-contrib pkgfile \
  bash-completion man-db man-pages \
  ufw \
```

- **NetworkManager**: la red de escritorio más sencilla
- **elogind**: integración de sesiones/energía
- **acpid / upower**: comportamiento de tapa, batería y suspensión
- **cronie** planificador de tareas por tiempo, usado para ejecutar programas o scripts a horas programadas
- **bluez**: Bluetooth
- **cups**: solo si imprimes
- **fwupd**: importante en Framework para las actualizaciones de firmware
- **bolt**: útil para autorizar dispositivos Thunderbolt/USB4
- **firmware SOF / ALSA**: ayuda con el hardware de audio de portátiles modernos
- **ufw**: Uncomplicated Firewall para gestionar el cortafuegos

ufw requiere añadirse a OpenRC:

```bash
rc-update add ufw default
```

KDE y utilidades:

```bash
sudo pacman -S \
  ark filelight gwenview okular \
  kate konsole \
  spectacle flameshot \
  partitionmanager \
  kdeconnect \
```

- **Okular** para PDFs
- **Kate** como editor gráfico por defecto
- **Spectacle** o **Flameshot** (preferido) para capturas de pantalla
- **Filelight** para limpiar el almacenamiento
- **KDE Connect** si quieres conectar tu sistema Linux con el teléfono

Desarrollo / programación:

```bash
sudo pacman -S \
  base-devel \
  github-cli \
  openssh openssh-openrc \
  jq yq \
  python python-pip \
  nodejs npm \
  docker docker-openrc docker-compose \
  podman podman-compose
```

Utilidad para el AUR:

```bash
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```

- **Paru**, el ayudante de AUR recomendado

Apps / paquetes del AUR:

```bash
paru -S brave-bin \
  signal-desktop \
  vscodium-bin \
  onlyoffice-bin \
  timeshift-autosnap \
```

- Navegador **Brave** (preferido frente a Firefox)
- Mensajería **Signal** (preferida frente a WhatsApp, Telegram, etc.)
- **VSCodium**, editor de texto de código abierto (preferido frente al Visual Studio Code de Microsoft)
- **ONLYOFFICE**, suite ofimática de código abierto (preferida por su mayor compatibilidad con Microsoft Office)
- **Timeshift Autosnapper** instala un hook para que Timeshift cree automáticamente un respaldo cada vez que se actualizan paquetes con pacman

Flatpak:

```bash
pacman -S flatpak
```

- Flatpak, si quieres instalar apps gráficas con facilidad

Apps multimedia:

```bash
sudo pacman -S \
  vlc mpv \
  ffmpeg \
  libreoffice-fresh \
  obs-studio \
```

- Reproductor multimedia **VLC** (preferido) **o MPV**
- **FFmpeg**, herramientas de conversión multimedia
- **OBS Studio**, grabación y screencasting de código abierto

Apps de oficina:

```bash
sudo pacman -S \
  libreoffice-fresh \
  thunderbird \
  okular \
  evince
```

- Suite ofimática **LibreOffice** (alternativa a ONLYOFFICE)
- **Thunderbird**, cliente de correo de código abierto (reemplazo de Microsoft Outlook)
- **Okular**, visor de documentos (el predeterminado de KDE)
- **Evince**, visor de documentos (alternativa a Okular incluida con GNOME)

Juegos:

```bash
sudo pacman -S \
  steam \
  gamemode \
  mangohud \
  vulkan-radeon \
  lib32-mesa \
  lib32-vulkan-radeon \
  mesa vulkan-tools
```

- **Steam**, tienda y servicio de distribución de videojuegos, incluyendo las herramientas y drivers adicionales necesarios (GPU AMD)
- Steam requiere soporte para aplicaciones de 32 bits — necesita la biblioteca lib32

Tailscale:

```bash
sudo pacman -S tailscale tailscale-openrc
sudo rc-update add tailscaled default
sudo rc-service tailscaled start
```

- **Tailscale**, servicio de VPN de malla de código abierto

Fuentes:

```bash
sudo pacman -S ttf-jetbrains-mono
paru -S ttf-jetbrains-mono-nerd
```

- Fuentes **JetBrains** y sus nerd fonts

Utilidades de monitorización:

```bash
sudo pacman -S \
  btop fastfetch \
  ncdu \
  ripgrep fd fzf \
  htop \
  usbutils pciutils \
  smartmontools lm_sensors
```

- **Btop**, panorama general del sistema
- **Fastfetch** (el obligatorio "neofetch")
- `ripgrep`, `fd`, `fzf` para mejorar la calidad de vida en la terminal
- `smartmontools` para la salud del SSD
- `lm_sensors` para las temperaturas

Almacenamiento en la nube:

```bash
paru -S \
  nextcloud-client-git \
  pcloud-drive
```

- Cliente de **Nextcloud**, almacenamiento en la nube autoalojado (requiere un servidor Nextcloud)
- Cliente de **pCloud**, almacenamiento en la nube orientado a la privacidad (requiere una cuenta de pCloud)

* * *

## Notas

- **Kernel:** esta guía usa el kernel `linux`. Artix también ofrece `linux-lts` y `linux-zen`.
- **Snapshots:** con esta disposición de subvolúmenes, puedes usar `snapper` o `timeshift` para hacer snapshots de `@` sin capturar `/home`, `/var` ni las cachés.
