---
description: >-
  Пошаговая установка сервера Devuan Linux с sysvinit, ext4 и работой только через терминал/SSH — без systemd.
---

<!--
Source: linux/devuan-server-install.md
Last translated: 2026-07
-->

# Руководство по установке сервера Devuan

**Целевая система:** ПК / рабочая станция (процессор AMD Ryzen и GPU NVIDIA)

**Файловая система:** ext4

**Рабочий стол:** отсутствует. Терминал / SSH

**Init:** sysvinit

**Накопитель:** NVMe SSD (`/dev/nvme0n1`)

Вики Devuan требует регистрации, однако официальные инструкции по установке доступны здесь: [devuan.org/os/documentation/install-guides/excalibur/install-devuan](https://www.devuan.org/os/documentation/install-guides/excalibur/install-devuan)

Установка через вариант netinstall не представляет сложности.

Перед началом:

\- Убедитесь, что BIOS переведён в режим UEFI (не legacy)
\- Включите виртуализацию (SVM/VT-x)
\- Отключите secure boot (для драйверов NVIDIA)

* * *

## 1\. Скачивание Devuan и создание загрузочного ISO

Скачайте Devuan Excalibur с [files.devuan.org/](https://files.devuan.org/)

- Выберите devuan_excalibur (последний стабильный выпуск)
- Выберите installer_iso
- Скачайте ISO с суффиксом \_netinstall

Создайте загрузочный USB из ISO. У PulsarTECH есть отличное руководство по мультизагрузочной флешке Ventoy + Linux: [youtube.com/watch?v=BfjLJ0CqWsY](https://www.youtube.com/watch?v=BfjLJ0CqWsY)

На Linux:

```bash
sudo dd if=devuan_excalibur_6.1.0_amd64_netinstall.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

## 2\. Загрузка с live-ISO

Загрузите ПК с USB. Вот справочная таблица клавиш меню загрузки для разных производителей:

| Производитель | Клавиша меню загрузки | Клавиша BIOS/UEFI |
| :--- | :--- | :--- |
| **Acer** | F12, Esc, F9 | F2, Delete |
| **Asus** | F8, Esc | F2, Delete |
| **Dell** | F12 | F2  |
| **HP** | F9, Esc | F10 |
| **Lenovo** | F12, F10, F8 (или кнопка Novo) | F2, F1 |
| **MSI** | F11 | Delete |
| **Toshiba** | F12 | F2, F1, Esc |
| **Samsung** | F12, F2, Esc | F2  |
| **Sony VAIO** | F11, F10, Esc (или кнопка Assist) | F2, F1, F3 |
| **Gigabyte** | F12 | Delete, F2 |
| **Intel NUC** | F10 | F2  |

* * *

## 3\. Установка Devuan

Оказавшись на графическом экране Devuan, выберите пункт `Install`:

1. **Язык, местоположение, клавиатура**
    - Выберите язык системы, местоположение и раскладку клавиатуры.
2. **Настройка сети**
    - Если подключение по ethernet, установщик настроит сеть автоматически. Если используете Wi-Fi, укажите SSID и пароль своей сети.
    - Введите hostname. Пример: `devuanserver` (подойдёт любое имя без пробелов и спецсимволов)
    - Оставьте имя домена пустым (если не хотите его добавить)
3. **Пароль root и учётная запись пользователя**
    - Задайте пароль root
    - Добавьте учётную запись пользователя. Пример: `имя`
    - Задайте пароль для пользователя
4. **Часы и часовой пояс**
    - Укажите свой часовой пояс. Пример: Eastern
5. **Разбиение диска**
    - Если настраиваете шифрование (опционально), следуйте этим инструкциям по разбиению дисков: [devuan.org/os/documentation/install-guides/excalibur/full-disk-encryption.html](https://www.devuan.org/os/documentation/install-guides/excalibur/full-disk-encryption.html)
    - Если шифрование не нужно (рекомендуется для упрощённой установки), можно выбрать `Guided - use entire disk` или `Manual`
    - Если выбрали `Manual`, ниже приведены схемы разбиения на выбор (рекомендуется простая схема разбиения)
    - После создания разделов запишите изменения на диски — выберите `<Yes>`
6. **Настройка менеджера пакетов**
    - Выберите зеркало архива Devuan. Предпочтительный вариант — `deb.devuan.org`
    - Если нужен HTTP-прокси, введите его данные. Иначе оставьте поле пустым и выберите `<Continue>`
7. **Настройка popularity-contest**
    - Участие в опросе об использовании пакетов необязательно. Выберите `<Yes>` или `<No>` по своему усмотрению (рекомендуется `<No>`)
8. **Выбор программного обеспечения**
    1. Клавишей пробела снимите все отметки, затем выберите только `SSH server` и `standard system utilities`. Не выбирайте никакое окружение рабочего стола. Это даст минимальную установку headless-сервера
9. Выбор системы инициализации
    - Варианты init:
        - `sysvinit` (по умолчанию, классический, хорошо документированный)
        - `OpenRC` (современный, на основе зависимостей, популярен в Gentoo)
        - `runit` (минималистичный, быстрая загрузка)
    - Для init по умолчанию (самый стабильный и совместимый) выберите `sysvinit`
    - Если предпочитаете более современный init на основе зависимостей (по использованию ближе к systemd), выберите `openrc`
10. Загрузчик
    - Установите GRUB на EFI-раздел. Когда спросят, устанавливать ли загрузчик GRUB на основной накопитель, выберите `<Yes>`
    - Выберите `/dev/nvme0n1` в качестве устройства для установки загрузчика
11. **Завершение и перезагрузка**
    - Установка завершена. Выберите &lt;Continue&gt; для перезагрузки
    - Сразу после перезагрузки извлеките USB-накопитель

* * *

### Простая схема разбиения (рекомендуется)

| #   | Точка монтирования | Размер | Файловая система | Назначение |
| --- | --- | --- | --- | --- |
| 1   | /boot/efi | 1 ГБ | FAT32 | Необходим для загрузки UEFI |
| 2   | /boot   | 2 ГБ | ext4 | Раздел ОС для упрощения восстановления |
| 3   | / | Остальное | ext4 | Хранение данных |

**Обоснование дизайна**

- Сохранить сервер простым
- Уменьшить фрагментацию и проблемы с размерами разделов
- Оптимизировать под Docker и контейнеризованные приложения
- Использовать NAS как массовое хранилище, чтобы вынести туда медиа

Если предпочитаете корпоративный, изолированный стиль разбиения, можно выбрать:

### Корпоративная схема разбиения

| #   | Точка монтирования | Размер | Файловая система | Назначение |
| --- | --- | --- | --- | --- |
| 1   | `/boot/efi` | 512 МиБ | FAT32 (системный раздел EFI) | Загрузчик UEFI |
| 2   | `/boot` | 2 ГиБ | ext4 | Образы ядра и initramfs |
| 3   | `/` | 50 ГиБ | ext4 | Корневая файловая система |
| 4   | `/var` | 100 ГиБ | ext4 | Логи, базы данных, слои контейнеров, кэш пакетов |
| 5   | `/tmp` | 10 ГиБ | ext4 (монтируется с noexec,nosuid,nodev) | Временные файлы |
| 6   | `swap` | 32 ГиБ | Linux swap | Пространство подкачки (примерно 1/3 максимума RAM) |
| 7   | `/home` | 50 ГиБ | ext4 | Домашние каталоги пользователей |
| 8   | `/srv` | Остальное | ext4 или XFS | Данные сервера, VM, контейнеры, датасеты |

### Обоснование дизайна

- **Отдельный `/var`**: разросшиеся логи или накопление образов контейнеров не смогут заполнить корневую файловую систему и уронить систему.
- **Отдельный `/tmp`**: монтируется с `noexec,nosuid,nodev` для усиления защиты; не влияет на другие разделы.
- **32 ГиБ swap**: при объёме до 96 ГБ RAM и потенциальных GPU/CUDA-нагрузках 32 ГиБ дают комфортный запас при нехватке памяти без излишеств. Если запускаете требовательный к памяти AI/ML-инференс — увеличьте.
- **Большой `/srv`**: основной объём хранилища headless-сервера располагается здесь — образы дисков VM, тома контейнеров, датасеты, экспорты NFS и подобное.
- **Отдельный `/boot`**: гарантирует, что загрузчик и образы ядра всегда доступны независимо от проблем корневой файловой системы.

* * *

## 4\. После установки: первая загрузка

Войдите в терминал как root (с паролем, заданным на предыдущих шагах).

- **Проверьте сетевое подключение:**

```bash
ip addr show
ping -c 3 devuan.org
```

- **Обновите систему:**

```bash
apt update && apt upgrade -y
```

- **Настройте источники APT:**

```bash
nano /etc/apt/sources.list
# Убедитесь, что contrib, non-free и non-free-firmware включены

deb http://deb.devuan.org/merged excalibur main contrib non-free non-free-firmware
deb http://deb.devuan.org/merged excalibur-updates main contrib non-free non-free-firmware
deb http://deb.devuan.org/merged excalibur-security main contrib non-free non-free-firmware
```

- **Снова обновите:**

```bash
apt update
```

- **Установите основные серверные пакеты:**

```bash
apt install -y \
  sudo vim neovim htop tmux curl wget git \
  lm-sensors smartmontools nvme-cli \
  unattended-upgrades apt-listchanges \
  ufw \
  build-essential dkms linux-headers-amd64
```

- **Добавьте пользователя в sudo:**

```bash
usermod -aG sudo вашпользователь
```

- **Создайте файл подкачки (если использовали простую схему разбиения)**

Swap даёт страховочный буфер при исчерпании оперативной памяти. Хотя современные системы с большим объёмом RAM могут работать без swap, он всё же рекомендуется для стабильности — особенно при запуске Docker-контейнеров, баз данных или GPU-нагрузок.

Вместо выделенного раздела подкачки при простой схеме разбиения предпочтителен **файл подкачки**. Им проще управлять, его проще изменять в размере и удалять без переразбиения дисков.

Пример ниже создаёт файл подкачки на 16 ГБ. Подберите размер под свою систему:

```bash
fallocate -l 16G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

Добавьте следующую строку в `/etc/fstab`, чтобы файл подкачки включался при загрузке:

```bash
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

Проверка swap

```bash
swapon --show
free -h
```

### Рекомендации по размеру

- **16 ГБ RAM или меньше:** рекомендуется 8–16 ГБ swap
- **32–64 ГБ RAM:** достаточно 8–16 ГБ swap
- **64 ГБ+ RAM:** 4–16 ГБ swap как страховочный буфер
- **Тяжёлые GPU / AI-нагрузки:** рассмотрите 16–32 ГБ swap

* * *

## 5\. Установка и настройка Tailscale

Tailscale обеспечивает безопасную mesh-сеть на базе WireGuard без сложной настройки. Весь SSH-доступ будет идти через сеть Tailscale — то есть SSH-порт вообще не будет открыт в публичный интернет.

- **Devuan Excalibur основан на Debian Trixie, поэтому используйте репозиторий Trixie:**

```bash
mkdir -p --mode=0755 /usr/share/keyrings

curl -fsSL https://pkgs.tailscale.com/stable/debian/trixie.noarmor.gpg \
  | tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null

curl -fsSL https://pkgs.tailscale.com/stable/debian/trixie.tailscale-keyring.list \
  | tee /etc/apt/sources.list.d/tailscale.list

apt update
```

- **Установите Tailscale:**

```bash
apt install -y tailscale
```

**Важно:** официальные пакеты Tailscale поставляются с сервисным файлом systemd, но без скрипта sysvinit. Поскольку Devuan использует sysvinit, после установки скрипт придётся создать вручную.

- **Пакет содержит только systemd-юнит, поэтому init-скрипт нужно создать вручную:**

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

- **Включите и запустите Tailscale:**

```bash
update-rc.d tailscaled defaults
mkdir -p /run/tailscale
service tailscaled start
service tailscaled status
```

- **Аутентифицируйтесь в своей tailnet:**

```bash
tailscale up
```

Команда выведет URL. Откройте его в браузере на другом устройстве, войдите в учётную запись Tailscale и авторизуйте машину. После аутентификации проверьте подключение:

```bash
tailscale status
tailscale ip -4
```

Запишите IP-адрес Tailscale (обычно `100.x.y.z`). Именно этот адрес вы будете использовать для SSH.

- **Включите Tailscale SSH:**

Tailscale SSH позволяет аутентифицировать SSH-сессии через провайдера идентификации Tailscale, полностью избавляя от SSH-ключей и паролей:

```bash
tailscale set --ssh
```

При использовании Tailscale SSH подключения аутентифицируются вашими ACL Tailscale вместо локальных SSH-ключей. Настроить, у кого есть доступ, можно в консоли администратора Tailscale в разделе **Access Controls**.

**Примечание:** если вы включили Tailscale SSH на шаге 5.6.6, локальный SSH-демон можно при желании полностью отключить, поскольку SSH теперь обслуживает сам Tailscale:

```bash
service ssh stop
update-rc.d ssh disable
```

- **Проверьте SSH через Tailscale с другой машины, прежде чем отключать монитор:**

```bash
ssh вашпользователь@100.x.y.z
```

Вместо Tailscale-IP можно использовать имя машины в вашей tailnet, если включён Tailscale MagicDNS.

- **Настройте файрвол:**

Поскольку SSH доступен только через Tailscale, файрвол должен полностью блокировать SSH на публичных интерфейсах. Открывайте порты только для сервисов, которые явно нужны в локальной сети.

```bash
ufw default deny incoming
ufw default allow outgoing

# Разрешить весь трафик на интерфейсе Tailscale (доверенная mesh-сеть)
ufw allow in on tailscale0

# НЕ разрешать SSH на публичных интерфейсах — только через Tailscale
# ufw allow ssh  <-- намеренно опущено

ufw enable
```

Если позже появятся сервисы, которым нужен доступ из LAN (например, NFS, веб-сервер, Samba), добавьте правила для этих конкретных портов на конкретных LAN-интерфейсах:

```bash
# Пример: разрешить HTTP только на LAN-интерфейсе 2.5G
ufw allow in on enp2s0 to any port 80 proto tcp
```

* * *

## 6\. Установка драйверов GPU NVIDIA

Перед установкой проприетарного драйвера NVIDIA необходимо отключить открытый драйвер Nouveau:

```bash
cat > /etc/modprobe.d/blacklist-nouveau.conf << 'EOF'
blacklist nouveau
options nouveau modeset=0
EOF

update-initramfs -u
reboot
```

После перезагрузки войдите снова и установите:

```bash
sudo apt update
sudo apt install linux-headers-$(uname -r) build-essential libglvnd-dev pkg-config dkms
```

Определите и установите драйвер:

```bash
sudo nvidia-detect
sudo apt install nvidia-driver nvidia-kernel-dkms nvidia-smi nvidia-settings
```

Используйте backports только если драйвер по умолчанию не подходит вашей GPU:

```bash
sudo apt install nvidia-driver firmware-misc-nonfree
```

Проверьте сборку DKMS:

```bash
dkms status
```

Вы должны увидеть строку вида:

```
nvidia/550.163.01, 6.12.x-amd64, x86_64: installed
```

Обязательная перезагрузка и проверка:

```bash
reboot
```

После перезагрузки убедитесь, что драйвер загружен:

```bash
nvidia-smi
```

Ожидаемый вывод покажет GPU с версией драйвера, версией CUDA, температурой и использованием памяти. Для headless-сервера без подключённого дисплея nvidia-smi — основной способ убедиться, что GPU работает.

- **Включите режим персистентности (headless):**

Без запущенного X-сервера драйвер NVIDIA может выгружаться между GPU-задачами, добавляя задержку. Включите режим персистентности:

```bash
nvidia-smi -pm 1
```

Чтобы сохранить его между перезагрузками, создайте init-скрипт. Для sysvinit:

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

- **Установите CUDA Toolkit:**

Если CUDA нужна для вычислительных нагрузок (AI-инференс, приложения с GPU-ускорением):

```bash
apt install -y nvidia-cuda-toolkit
```

Проверьте:

```bash
nvcc --version
```

* * *

## 7\. Установка Docker

Официальные Debian-пакеты Docker CE включают init-скрипт sysvinit (`/etc/init.d/docker`), поэтому Docker работает на Devuan нативно, без systemd.

- Удалите конфликтующие пакеты

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
    apt-get remove -y $pkg 2>/dev/null
done
```

- Добавьте репозиторий Docker

Поскольку Devuan Excalibur основан на Debian Trixie, используйте репозиторий Trixie:

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

- Установите Docker Engine

```bash
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- Включите и запустите Docker (sysvinit)

Пакет Docker поставляется с `/etc/init.d/docker`. Включите его при загрузке и запустите:

```bash
update-rc.d docker defaults
service docker start
```

Убедитесь, что Docker работает:

```bash
service docker status
docker info
```

- Разрешите своему пользователю запускать Docker

Добавьте своего обычного пользователя в группу `docker`, чтобы не набирать `sudo` для каждой команды Docker:

```bash
usermod -aG docker вашпользователь
```

Выйдите из системы и войдите снова (или выполните `newgrp docker`), чтобы изменение группы вступило в силу.

**Замечание о безопасности:** членство в группе `docker` даёт доступ к хосту, эквивалентный root. Добавляйте только доверенных пользователей.

- Проверьте установку

```bash
docker run --rm hello-world
```

- NVIDIA Container Toolkit (GPU в контейнерах)

Чтобы использовать GPU NVIDIA внутри Docker-контейнеров (для CUDA, AI-инференса и т. п.), установите NVIDIA Container Toolkit:

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
  | gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
  | tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

apt update
apt install -y nvidia-container-toolkit
```

Настройте runtime Docker:

```bash
nvidia-ctk runtime configure --runtime=docker
service docker restart
```

Проверьте доступ к GPU из контейнера:

```bash
docker run --rm --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
```

Вы должны увидеть GPU NVIDIA с версией драйвера и версией CUDA.

### Краткая справка по Docker Compose

Docker Compose устанавливается как CLI-плагин. Использование:

```bash
docker compose up -d          # Запустить сервисы в фоне

docker compose down            # Остановить и удалить сервисы

docker compose logs -f         # Следить за логами

docker compose ps              # Список работающих сервисов
```

* * *

## 8\. NFS-монтирования NAS

Если вы монтируете внешние NAS-накопители (например, Synology, QNAP, TrueNAS), следуйте инструкциям ниже.

- Установите клиентские пакеты NFS

```bash
apt install -y nfs-common
```

- Создайте точки монтирования

Создайте каталоги для каждого NFS-ресурса, который хотите смонтировать. Аккуратная конвенция — монтировать их под `/mnt/nas/`:

```bash
mkdir -p /mnt/nas/media
mkdir -p /mnt/nas/backups
mkdir -p /mnt/nas/shared
```

Подгоните имена под структуру экспортов вашего NAS.

- Проверьте монтирования вручную

Прежде чем делать их постоянными, убедитесь, что каждое работает:

```bash
mount -t nfs4 nas.local:/volume1/media /mnt/nas/media
ls /mnt/nas/media
```

Замените `nas.local` на IP или hostname вашего NAS, а `/volume1/media` — на фактический путь NFS-экспорта. Если используете NFSv3:

```bash
mount -t nfs -o vers=3 nas.local:/volume1/media /mnt/nas/media
```

- Настройте постоянные монтирования в /etc/fstab

После успешной ручной проверки добавьте записи в `/etc/fstab` для автоматического монтирования при загрузке:

```bash
nano /etc/fstab

# NFS-монтирования NAS
nas.local:/volume1/media    /mnt/nas/media    nfs4  rw,soft,intr,timeo=30,retrans=3,_netdev  0  0
nas.local:/volume1/backups  /mnt/nas/backups  nfs4  rw,soft,intr,timeo=30,retrans=3,_netdev  0  0
nas.local:/volume1/shared   /mnt/nas/shared   nfs4  rw,soft,intr,timeo=30,retrans=3,_netdev  0  0
```

**Пояснение опций монтирования:**

- **`rw`**: чтение-запись. Для ресурсов только на чтение, вроде медиабиблиотек, используйте `ro`.
- **`soft`**: если NAS недоступен, возвращается ошибка вместо бесконечного зависания. Критично для headless-сервера — при `hard`-монтировании процессы замрут и станут неубиваемыми, если NAS уйдёт в офлайн.
- **`intr`**: позволяет прерывать операции NFS сигналами (например, Ctrl+C).
- **`timeo=30`**: тайм-аут в децисекундах (3 секунды) перед повтором.
- **`retrans=3`**: число повторов перед сообщением об ошибке.
- **`_netdev`**: сообщает системе инициализации, что этому монтированию сначала нужна сеть, предотвращая зависание загрузки при недоступном NAS.

Смонтируйте все новые записи fstab:

```bash
mount -a
```

Проверьте:

```bash
df -h | grep nas
```

- Права доступа NFS-монтирований

Права NFS зависят от того, как настроены экспорты на вашем NAS. Распространённые подходы:

**Сопоставление UID/GID (рекомендуется):** убедитесь, что UID и GID вашего пользователя Devuan совпадают с UID/GID, ожидаемыми NFS-экспортом. Проверьте командой `id вашпользователь` на Devuan и сравните с настройками NAS.

**all_squash с anonuid/anongid:** если экспорт NAS использует `all_squash`, весь доступ сопоставляется с единым UID/GID, заданным на стороне NAS. Это самый простой вариант для совместного доступа.

**Без root squash:** включайте `no_root_squash` на NAS только если вам действительно нужна запись от root с этого сервера. Это риск безопасности, и обычно в этом нет необходимости.

- Использование NFS-монтирований в Docker-контейнерах

Пробросьте NFS-путь хоста через bind-mount

В вашем `compose.yaml`:

yaml

```yaml
services:
  myapp:
    image: myapp:latest
    volumes:
      - /mnt/nas/media:/data/media
      - /mnt/nas/shared:/data/shared
```

Это самый простой подход. NFS-соединением занимается хост, а контейнеры видят данные как обычный каталог.

- Мониторинг здоровья NFS-монтирований

NFS-монтирования могут незаметно «протухнуть» (stale), если NAS перезагрузится или сеть моргнёт. Создайте простую cron-задачу проверки здоровья:

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

Скрипт каждые 5 минут проверяет, что каждое NFS-монтирование отвечает, и пытается перемонтировать «протухший» ресурс. Результаты смотрите в `/var/log/syslog` командой `grep nfs-health /var/log/syslog`.

## 9\. Fastfetch

«Обязательный neofetch» (fastfetch):

```bash
sudo apt update
sudo apt install fastfetch
```

* * *

## Результат

- Без systemd
- Полная поддержка Docker
- Работающее GPU-ускорение
- Монтирования NAS
