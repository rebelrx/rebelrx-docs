---
description: >-
  Instalación paso a paso de un servidor Devuan Linux con sysvinit, ext4 y una configuración solo de terminal/SSH — sin systemd.
---

<!--
Source: linux/devuan-server-install.md
Last translated: 2026-07
-->

# Guía de instalación de servidor Devuan

**Equipo objetivo:** PC / estación de trabajo (procesador AMD Ryzen y GPU NVIDIA)

**Sistema de archivos:** ext4

**Escritorio:** ninguno. Terminal / SSH

**Init:** sysvinit

**Unidad:** SSD NVMe (`/dev/nvme0n1`)

La wiki de Devuan requiere registro; sin embargo, las instrucciones oficiales de instalación están disponibles en: [devuan.org/os/documentation/install-guides/excalibur/install-devuan](https://www.devuan.org/os/documentation/install-guides/excalibur/install-devuan)

La instalación es sencilla usando la opción netinstall.

Antes de empezar:

\- Asegúrate de que la BIOS esté en modo UEFI (no legacy)
\- Habilita la virtualización (SVM/VT-x)
\- Desactiva el arranque seguro (secure boot) (necesario para los drivers de NVIDIA)

* * *

## 1\. Descargar Devuan y crear una ISO arrancable

Descarga Devuan Excalibur desde [files.devuan.org/](https://files.devuan.org/)

- Elige devuan_excalibur (la última versión estable)
- Elige installer_iso
- Descarga la ISO \_netinstall

Crea una ISO arrancable en USB. PulsarTECH tiene una excelente guía de USB multiarranque con Ventoy + Linux en: [youtube.com/watch?v=BfjLJ0CqWsY](https://www.youtube.com/watch?v=BfjLJ0CqWsY)

En Linux:

```bash
sudo dd if=devuan_excalibur_6.1.0_amd64_netinstall.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

## 2\. Arrancar la ISO live

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

* * *

## 3\. Instalar Devuan

Una vez en la pantalla gráfica de Devuan, elige la opción `Install`:

1. **Idioma, ubicación, teclado**
    - Selecciona el idioma del sistema, la ubicación y la distribución del teclado.
2. **Configuración de red**
    - El instalador configurará la red automáticamente si estás conectado por ethernet. Si usas Wi-Fi, proporciona el SSID y la contraseña de tu red.
    - Introduce un hostname. Ejemplo: `devuanserver` (cualquier nombre vale, siempre que no tenga espacios ni caracteres especiales)
    - Deja el nombre de dominio en blanco (a menos que quieras añadir uno)
3. **Contraseña de root y cuenta de usuario**
    - Establece una contraseña de root
    - Añade una cuenta de usuario. Ejemplo: `nombredepila`
    - Establece una contraseña para el usuario
4. **Reloj y zona horaria**
    - Introduce tu zona horaria. Ejemplo: Eastern
5. **Particionado del disco**
    - Si vas a configurar cifrado (opcional), sigue estas instrucciones para particionar los discos: [devuan.org/os/documentation/install-guides/excalibur/full-disk-encryption.html](https://www.devuan.org/os/documentation/install-guides/excalibur/full-disk-encryption.html)
    - Si no vas a configurar cifrado (recomendado para una instalación simplificada), puedes seleccionar `Guided - use entire disk` o `Manual`
    - Si seleccionas `Manual`, más abajo tienes esquemas de particiones según tu preferencia (se recomienda el esquema de particiones simple)
    - Tras crear las particiones, escribe los cambios en los discos — selecciona `<Yes>`
6. **Configuración del gestor de paquetes**
    - Selecciona un mirror del archivo de Devuan. La opción preferida es `deb.devuan.org`
    - Si necesitas usar un proxy HTTP, puedes introducir sus datos. En caso contrario, déjalo en blanco y selecciona `<Continue>`
7. **Configuración de popularity-contest**
    - Participar en la encuesta de uso de paquetes es opcional. Selecciona `<Yes>` o `<No>` según tu preferencia (se recomienda `<No>`)
8. **Selección de software**
    1. Usa la barra espaciadora para deseleccionar todo y luego selecciona solo `SSH server` y `standard system utilities`. No selecciones ningún entorno de escritorio. Esto produce una instalación mínima de servidor headless
9. Selección del sistema de init
    - Las opciones de init son:
        - `sysvinit` (predeterminado, clásico, bien documentado)
        - `OpenRC` (moderno, basado en dependencias, popular en Gentoo)
        - `runit` (minimalista, arranque rápido)
    - Para el init por defecto (el más estable y compatible), selecciona `sysvinit`
    - Si prefieres un sistema de init más moderno y basado en dependencias (más cercano a systemd en su uso), elige `openrc`
10. Gestor de arranque
    - Instala GRUB en la partición EFI. Cuando pregunte si instalar el gestor de arranque GRUB en la unidad principal, selecciona `<Yes>`
    - Selecciona `/dev/nvme0n1` como dispositivo para la instalación del gestor de arranque
11. **Finalizar y reiniciar**
    - La instalación ha terminado. Selecciona &lt;Continue&gt; para reiniciar
    - Retira la unidad USB inmediatamente después de reiniciar

* * *

### Esquema de particiones simple (recomendado)

| #   | Montaje | Tamaño | Sistema de archivos | Propósito |
| --- | --- | --- | --- | --- |
| 1   | /boot/efi | 1 GB | FAT32 | Necesario para el arranque UEFI |
| 2   | /boot   | 2 GB | ext4 | Partición del SO para simplificar la recuperación |
| 3   | / | Resto | ext4 | Almacenamiento de datos |

**Justificación del diseño**

- Mantener el servidor simple
- Reducir la fragmentación y los problemas con los tamaños de partición
- Optimizar para ejecutar Docker y aplicaciones en contenedores
- Usar el NAS como almacenamiento masivo para descargar los medios

Si prefieres un estilo de particiones aislado y de nivel empresarial, puedes optar por:

### Esquema de particiones empresarial

| #   | Punto de montaje | Tamaño | Sistema de archivos | Propósito |
| --- | --- | --- | --- | --- |
| 1   | `/boot/efi` | 512 MiB | FAT32 (partición de sistema EFI) | Gestor de arranque UEFI |
| 2   | `/boot` | 2 GiB | ext4 | Imágenes del kernel y del initramfs |
| 3   | `/` | 50 GiB | ext4 | Sistema de archivos raíz |
| 4   | `/var` | 100 GiB | ext4 | Logs, bases de datos, capas de contenedores, caché de paquetes |
| 5   | `/tmp` | 10 GiB | ext4 (montado con noexec,nosuid,nodev) | Archivos temporales |
| 6   | `swap` | 32 GiB | Linux swap | Espacio de intercambio (aprox. 1/3 de la RAM máxima) |
| 7   | `/home` | 50 GiB | ext4 | Directorios home de los usuarios |
| 8   | `/srv` | Resto | ext4 o XFS | Datos del servidor, VMs, contenedores, datasets |

### Justificación del diseño

- **`/var` separado**: unos logs descontrolados o la acumulación de imágenes de contenedores no pueden llenar el sistema de archivos raíz y tumbar el sistema.
- **`/tmp` separado**: montado con `noexec,nosuid,nodev` como medida de endurecimiento; evita que afecte a otras particiones.
- **Swap de 32 GiB**: con hasta 96 GB de RAM y posibles cargas de trabajo con GPU/CUDA, 32 GiB dan un margen cómodo ante presión de memoria sin resultar un desperdicio. Si ejecutas inferencia de IA/ML intensiva en memoria, plantéate aumentarlo.
- **`/srv` grande**: el grueso del almacenamiento de un servidor headless va aquí — imágenes de disco de VMs, volúmenes de contenedores, datasets, exportaciones NFS y similares.
- **`/boot` separado**: garantiza que el gestor de arranque y las imágenes del kernel sean siempre accesibles, con independencia de los problemas del sistema de archivos raíz.

* * *

## 4\. Post-instalación: configuración del primer arranque

Inicia sesión como root en la terminal (con la contraseña establecida en los pasos anteriores).

- **Verificar la conectividad de red:**

```bash
ip addr show
ping -c 3 devuan.org
```

- **Actualizar el sistema:**

```bash
apt update && apt upgrade -y
```

- **Configurar las fuentes de APT:**

```bash
nano /etc/apt/sources.list
# Asegúrate de que contrib, non-free y non-free-firmware estén habilitados

deb http://deb.devuan.org/merged excalibur main contrib non-free non-free-firmware
deb http://deb.devuan.org/merged excalibur-updates main contrib non-free non-free-firmware
deb http://deb.devuan.org/merged excalibur-security main contrib non-free non-free-firmware
```

- **Actualizar de nuevo:**

```bash
apt update
```

- **Instalar los paquetes esenciales de servidor:**

```bash
apt install -y \
  sudo vim neovim htop tmux curl wget git \
  lm-sensors smartmontools nvme-cli \
  unattended-upgrades apt-listchanges \
  ufw \
  build-essential dkms linux-headers-amd64
```

- **Añadir el usuario a sudo:**

```bash
usermod -aG sudo tunombredeusuario
```

- **Crear un archivo de swap (si usaste el esquema de particiones simple)**

El swap proporciona un colchón de seguridad cuando la memoria del sistema se agota. Aunque los sistemas modernos con mucha RAM pueden funcionar sin swap, sigue siendo recomendable para la estabilidad, especialmente al ejecutar contenedores Docker, bases de datos o cargas de trabajo con GPU.

En lugar de crear una partición de swap dedicada con el esquema de particiones simple, se prefiere usar un **archivo de swap**. Es más fácil de gestionar, redimensionar y eliminar sin reparticionar los discos.

El ejemplo siguiente crea un archivo de swap de 16 GB. Ajusta el tamaño según tu sistema:

```bash
fallocate -l 16G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

Añade la siguiente línea a `/etc/fstab` para que el archivo de swap se active al arrancar:

```bash
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

Verificar el swap

```bash
swapon --show
free -h
```

### Guía de tamaños

- **16 GB de RAM o menos:** se recomiendan 8–16 GB de swap
- **32–64 GB de RAM:** 8–16 GB de swap son suficientes
- **64 GB+ de RAM:** 4–16 GB de swap como colchón de seguridad
- **Cargas pesadas de GPU / IA:** plantéate 16–32 GB de swap

* * *

## 5\. Instalar y configurar Tailscale

Tailscale ofrece una red de malla segura basada en WireGuard y sin configuración. Todo el acceso SSH pasará por la red de Tailscale, lo que significa que no habrá ningún puerto SSH expuesto al internet público.

- **Devuan Excalibur está basado en Debian Trixie, así que usa el repositorio de Trixie:**

```bash
mkdir -p --mode=0755 /usr/share/keyrings

curl -fsSL https://pkgs.tailscale.com/stable/debian/trixie.noarmor.gpg \
  | tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null

curl -fsSL https://pkgs.tailscale.com/stable/debian/trixie.tailscale-keyring.list \
  | tee /etc/apt/sources.list.d/tailscale.list

apt update
```

- **Instalar Tailscale:**

```bash
apt install -y tailscale
```

**Importante:** los paquetes oficiales de Tailscale incluyen un archivo de servicio de systemd, pero ningún script de sysvinit. Como Devuan usa sysvinit, tendrás que crear uno manualmente después de instalar.

- **El paquete solo incluye una unit de systemd, así que debes crear un script de init manualmente:**

```bash
cat > /etc/init.d/tailscaled << 'INITEOF'
#!/bin/sh
### BEGIN INIT INFO
# Provides:          tailscale
# Required-Start:    $local_fs $network $all
# Required-Stop:     $local_fs $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Tailscale daemon
# Description:       Runs the tailscaled mesh VPN daemon.
### END INIT INFO

. /lib/lsb/init-functions

PIDFILE=/var/run/tailscale.pid
LOGFILE=/var/log/tailscale.log
TAILSCALED=/usr/sbin/tailscaled

fail_unless_root() {
    if [ "$(id -u)" != '0' ]; then
        log_failure_msg "must be run as root"
        exit 1
    fi
}

case "$1" in
    start)
        fail_unless_root
        log_daemon_msg "Starting Tailscale daemon" "tailscaled"
        $TAILSCALED --cleanup
        start-stop-daemon --start --background --no-close \
            --exec $TAILSCALED \
            --pidfile "$PIDFILE" \
            --make-pidfile \
            -- \
            --state=/var/lib/tailscale/tailscaled.state \
            --socket=/run/tailscale/tailscaled.sock >> $LOGFILE 2>&1
        status=$?
        log_end_msg $status
        ;;

    stop)
        fail_unless_root
        log_daemon_msg "Stopping Tailscale daemon" "tailscaled"
        start-stop-daemon --stop --pidfile "$PIDFILE" \
            --remove-pidfile --retry 10
        status=$?
        log_end_msg $status
        ;;

    restart)
        $0 stop
        sleep 1
        $0 start
        ;;

    status)
        status_of_proc -p "$PIDFILE" "$TAILSCALED" "tailscaled"
        ;;

    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
INITEOF

chmod +x /etc/init.d/tailscaled
```

- **Habilitar e iniciar Tailscale:**

```bash
update-rc.d tailscaled defaults
mkdir -p /run/tailscale
service tailscaled start
service tailscaled status
```

- **Autenticarte en tu tailnet:**

```bash
tailscale up
```

Esto imprimirá una URL. Ábrela en un navegador en otro dispositivo, inicia sesión en tu cuenta de Tailscale y autoriza la máquina. Una vez autenticado, verifica la conectividad:

```bash
tailscale status
tailscale ip -4
```

Anota la IP de Tailscale (normalmente `100.x.y.z`). Esta es la dirección que usarás para SSH.

- **Habilitar Tailscale SSH:**

Tailscale SSH te permite autenticar las sesiones SSH a través del proveedor de identidad de Tailscale, eliminando por completo la necesidad de claves SSH o contraseñas:

```bash
tailscale set --ssh
```

Al usar Tailscale SSH, las conexiones se autentican mediante tus ACLs de Tailscale en lugar de claves SSH locales. Puedes configurar quién tiene acceso en la consola de administración de Tailscale, en **Access Controls**.

**Nota:** si habilitaste Tailscale SSH más arriba, puedes desactivar opcionalmente el demonio SSH local por completo, ya que Tailscale gestiona SSH directamente:

```bash
service ssh stop
update-rc.d ssh disable
```

- **Prueba SSH sobre Tailscale desde otra máquina antes de desconectar el monitor:**

```bash
ssh tunombredeusuario@100.x.y.z
```

También puedes usar el nombre de tu máquina en la tailnet en lugar de la IP de Tailscale, si tienes MagicDNS de Tailscale habilitado.

- **Configurar el cortafuegos:**

Como SSH solo es accesible vía Tailscale, el cortafuegos debe bloquear SSH desde las interfaces públicas por completo. Abre puertos únicamente para los servicios que necesites explícitamente en la LAN.

```bash
ufw default deny incoming
ufw default allow outgoing

# Permitir todo el tráfico en la interfaz de Tailscale (red de malla de confianza)
ufw allow in on tailscale0

# NO permitir SSH en las interfaces públicas — es solo por Tailscale
# ufw allow ssh  <-- omitido intencionadamente

ufw enable
```

Si más adelante ejecutas servicios que necesiten acceso desde la LAN (p. ej., NFS, un servidor web, Samba), añade reglas para esos puertos concretos en las interfaces LAN concretas:

```bash
# Ejemplo: permitir HTTP solo en la interfaz LAN de 2.5G
ufw allow in on enp2s0 to any port 80 proto tcp
```

* * *

## 6\. Instalar los drivers de GPU NVIDIA

El driver de código abierto Nouveau debe desactivarse antes de instalar el driver propietario de NVIDIA:

```bash
cat > /etc/modprobe.d/blacklist-nouveau.conf << 'EOF'
blacklist nouveau
options nouveau modeset=0
EOF

update-initramfs -u
reboot
```

Tras reiniciar, vuelve a iniciar sesión e instala:

```bash
sudo apt update
sudo apt install linux-headers-$(uname -r) build-essential libglvnd-dev pkg-config dkms
```

Detectar e instalar el driver:

```bash
sudo nvidia-detect
sudo apt install nvidia-driver nvidia-kernel-dkms nvidia-smi nvidia-settings
```

Usa backports solo si el driver por defecto falla con tu GPU:

```bash
sudo apt install nvidia-driver firmware-misc-nonfree
```

Verificar la compilación de DKMS:

```bash
dkms status
```

Deberías ver una línea como:

```
nvidia/550.163.01, 6.12.x-amd64, x86_64: installed
```

Reinicio obligatorio y verificación:

```bash
reboot
```

Tras el reinicio, confirma que el driver está cargado:

```bash
nvidia-smi
```

La salida esperada mostrará la GPU con la versión del driver, la versión de CUDA, la temperatura y el uso de memoria. En un servidor headless sin pantalla conectada, nvidia-smi es la manera principal de verificar que la GPU está operativa.

- **Habilitar el modo de persistencia (headless):**

Sin un servidor X en ejecución, el driver de NVIDIA puede descargarse entre tareas de GPU, añadiendo latencia. Habilita el modo de persistencia:

```bash
nvidia-smi -pm 1
```

Para hacerlo persistente entre reinicios, crea un script de init. Para sysvinit:

```bash
cat > /etc/init.d/nvidia-persistenced << 'INITEOF'
#!/bin/sh
### BEGIN INIT INFO
# Provides:          nvidia-persistenced
# Required-Start:    $local_fs
# Required-Stop:     $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: NVIDIA Persistence Daemon
### END INIT INFO

case "$1" in
  start)
    /usr/bin/nvidia-smi -pm 1
    ;;
  stop)
    /usr/bin/nvidia-smi -pm 0
    ;;
  *)
    echo "Usage: $0 {start|stop}"
    exit 1
    ;;
esac
exit 0
INITEOF


chmod +x /etc/init.d/nvidia-persistenced
update-rc.d nvidia-persistenced defaults
```

- **Instalar el CUDA Toolkit:**

Si necesitas CUDA para cargas de trabajo de cómputo (inferencia de IA, aplicaciones aceleradas por GPU):

```bash
apt install -y nvidia-cuda-toolkit
```

Verificar:

```bash
nvcc --version
```

* * *

## 7\. Instalar Docker

Los paquetes oficiales de Docker CE para Debian incluyen un script de init de sysvinit (`/etc/init.d/docker`), así que Docker funciona de forma nativa en Devuan sin systemd.

- Eliminar los paquetes en conflicto

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
    apt-get remove -y $pkg 2>/dev/null
done
```

- Añadir el repositorio de Docker

Como Devuan Excalibur está basado en Debian Trixie, usa el repositorio de Trixie:

```bash
apt install -y ca-certificates curl gnupg

install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

cat > /etc/apt/sources.list.d/docker.list << EOF
deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian trixie stable
EOF


apt update
```

- Instalar Docker Engine

```bash
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- Habilitar e iniciar Docker (sysvinit)

El paquete de Docker incluye `/etc/init.d/docker`. Habilítalo al arranque e inícialo:

```bash
update-rc.d docker defaults
service docker start
```

Verificar que Docker está en ejecución:

```bash
service docker status
docker info
```

- Permitir que tu usuario ejecute Docker

Añade tu usuario habitual al grupo `docker` para no necesitar `sudo` en cada comando de Docker:

```bash
usermod -aG docker tunombredeusuario
```

Cierra la sesión y vuelve a entrar (o usa `newgrp docker`) para que el cambio de grupo surta efecto.

**Nota de seguridad:** pertenecer al grupo `docker` otorga un acceso equivalente a root sobre el host. Añade solo usuarios de confianza.

- Probar la instalación

```bash
docker run --rm hello-world
```

- NVIDIA Container Toolkit (contenedores con GPU)

Para usar la GPU NVIDIA dentro de contenedores Docker (para CUDA, inferencia de IA, etc.), instala el NVIDIA Container Toolkit:

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
  | gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
  | tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

apt update
apt install -y nvidia-container-toolkit
```

Configurar el runtime de Docker:

```bash
nvidia-ctk runtime configure --runtime=docker
service docker restart
```

Probar el acceso a la GPU desde un contenedor:

```bash
docker run --rm --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
```

Deberías ver la GPU NVIDIA listada con la versión del driver y la versión de CUDA.

### Referencia rápida de Docker Compose

Docker Compose se instala como un plugin de la CLI. Úsalo con:

```bash
docker compose up -d          # Iniciar los servicios en segundo plano

docker compose down            # Detener y eliminar los servicios

docker compose logs -f         # Seguir los logs

docker compose ps              # Listar los servicios en ejecución
```

* * *

## 8\. Montajes NFS del NAS

Si vas a montar unidades NAS de almacenamiento externo (p. ej., Synology, QNAP, TrueNAS), sigue las instrucciones a continuación.

- Instalar los paquetes cliente de NFS

```bash
apt install -y nfs-common
```

- Crear los puntos de montaje

Crea directorios para cada recurso NFS que quieras montar. Una convención limpia es montarlos bajo `/mnt/nas/`:

```bash
mkdir -p /mnt/nas/media
mkdir -p /mnt/nas/backups
mkdir -p /mnt/nas/shared
```

Ajusta los nombres para que coincidan con la estructura de exportaciones de tu NAS.

- Probar los montajes manualmente

Antes de hacerlos permanentes, verifica que cada montaje funciona:

```bash
mount -t nfs4 nas.local:/volume1/media /mnt/nas/media
ls /mnt/nas/media
```

Sustituye `nas.local` por la IP o el hostname de tu NAS, y `/volume1/media` por la ruta real de la exportación NFS. Si usas NFSv3:

```bash
mount -t nfs -o vers=3 nas.local:/volume1/media /mnt/nas/media
```

- Configurar montajes permanentes en /etc/fstab

Cuando la prueba manual tenga éxito, añade entradas a `/etc/fstab` para el montaje automático al arrancar:

```bash
nano /etc/fstab

# Montajes NFS del NAS
nas.local:/volume1/media    /mnt/nas/media    nfs4  rw,soft,timeo=30,retrans=3,_netdev  0  0
nas.local:/volume1/backups  /mnt/nas/backups  nfs4  rw,soft,timeo=30,retrans=3,_netdev  0  0
nas.local:/volume1/shared   /mnt/nas/shared   nfs4  rw,soft,timeo=30,retrans=3,_netdev  0  0
```

**Explicación de las opciones de montaje:**

- **`rw`**: acceso de lectura-escritura. Usa `ro` para recursos de solo lectura como las bibliotecas de medios.
- **`soft`**: devuelve un error si el NAS es inalcanzable en lugar de colgarse indefinidamente. Crítico en un servidor headless — un montaje `hard` hará que los procesos se congelen y se vuelvan imposibles de matar si el NAS se desconecta.
- **`timeo=30`**: tiempo de espera en decisegundos (3 segundos) antes de reintentar.
- **`retrans=3`**: número de reintentos antes de reportar el fallo.
- **`_netdev`**: indica al sistema de init que este montaje requiere que la red esté levantada primero, evitando cuelgues en el arranque si el NAS es inalcanzable.

> ⚠️ **`soft` vs `hard` es un compromiso deliberado.** Esta guía usa `soft` para mantener un servidor headless con capacidad de respuesta cuando el NAS se cae. La [guía de montaje del NAS](../homelab/nas-mounting.md) recomienda `hard` para los recursos que reciben escrituras (los respaldos especialmente), porque `soft` puede truncar silenciosamente las escrituras interrumpidas. Elige por recurso compartido: `soft` para medios de solo lectura mayoritaria, `hard` para datos críticos de escritura.

Montar todas las nuevas entradas del fstab:

```bash
mount -a
```

Verificar:

```bash
df -h | grep nas
```

- Permisos de los montajes NFS

Los permisos de NFS dependen de cómo estén configuradas las exportaciones de tu NAS. Enfoques habituales:

**Mapeo de UID/GID (recomendado):** asegúrate de que el UID y el GID de tu usuario de Devuan coincidan con los UID/GID esperados por la exportación NFS. Compruébalo con `id tunombredeusuario` en Devuan y compáralo con la configuración del NAS.

**all_squash con anonuid/anongid:** si la exportación del NAS usa `all_squash`, todo acceso se mapea a un único UID/GID definido en el lado del NAS. Es lo más simple para accesos compartidos.

**Sin root squash:** habilita `no_root_squash` en el NAS solo si necesitas específicamente acceso de escritura como root desde este servidor. Es un riesgo de seguridad y generalmente es innecesario.

- Usar los montajes NFS en contenedores Docker

Haz bind-mount de la ruta NFS del host

En tu `compose.yaml`:

```yaml
services:
  myapp:
    image: myapp:latest
    volumes:
      - /mnt/nas/media:/data/media
      - /mnt/nas/shared:/data/shared
```

Es el enfoque más simple. El host gestiona la conexión NFS y los contenedores ven los datos como un directorio normal.

- Monitorizar la salud de los montajes NFS

Los montajes NFS pueden quedarse obsoletos (stale) en silencio si el NAS se reinicia o la red tiene un tropiezo. Crea una tarea cron sencilla de comprobación de salud:

```bash
cat > /etc/cron.d/nfs-health << 'EOF'
*/5 * * * * root /usr/local/bin/nfs-health-check.sh
EOF


cat > /usr/local/bin/nfs-health-check.sh << 'SCRIPT'
#!/bin/sh
for mount in /mnt/nas/media /mnt/nas/backups /mnt/nas/shared; do
    if mountpoint -q "$mount"; then
        timeout 5 ls "$mount" > /dev/null 2>&1
        if [ $? -ne 0 ]; then
            logger -t nfs-health "STALE mount detected: $mount — attempting remount"
            umount -l "$mount" 2>/dev/null
            mount "$mount"
        fi
    else
        logger -t nfs-health "Mount missing: $mount — attempting mount"
        mount "$mount"
    fi
done
SCRIPT


chmod +x /usr/local/bin/nfs-health-check.sh
```

Esto comprueba cada 5 minutos que cada montaje NFS responde e intenta un remontaje si un recurso se ha quedado obsoleto. Consulta los resultados en `/var/log/syslog` con `grep nfs-health /var/log/syslog`.

## 9\. Fastfetch

El "neofetch obligatorio" (fastfetch):

```bash
sudo apt update
sudo apt install fastfetch
```

* * *

## Resultado

- Sin systemd
- Soporte completo de Docker
- Aceleración por GPU funcionando
- Montajes del NAS
