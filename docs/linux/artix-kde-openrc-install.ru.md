---
description: >-
  Пошаговая ручная установка Artix Linux с OpenRC, подтомами Btrfs, zram и KDE Plasma на оборудовании AMD.
---

<!--
Source: linux/artix-kde-openrc-install.md
Last translated: 2026-07
-->

# Руководство по ручной установке Artix Linux

**Целевая система:** ПК / ноутбук (процессор AMD Ryzen и GPU AMD)

**Файловая система:** Btrfs с подтомами

**Swap:** zram (без раздела подкачки)

**Рабочий стол:** KDE Plasma (самый минимум)

**Init:** OpenRC

**Накопитель:** NVMe SSD (`/dev/nvme0n1`)

Вики Artix — отличный дополнительный ресурс для справки и устранения неполадок: [wiki.artixlinux.org/](https://wiki.artixlinux.org/)

Общие шаги установки Artix (сходные с большинством дистрибутивов Linux) включают: подключение к интернету, разбиение диска, форматирование разделов, монтирование разделов, установку ядра Linux и пакетов базовой системы, настройку локализации, установку загрузчика, добавление пользователей и установку паролей, настройку сети, установку окружения рабочего стола и экранного менеджера входа, активацию основных служб и постустановочную настройку.

* * *

## 1\. Загрузка с live-ISO

Скачайте базовый ISO-образ с OpenRC с [artixlinux.org/download.php](https://artixlinux.org/download.php). Запишите его на USB-накопитель

- У PulsarTECH есть отличное руководство по мультизагрузочной флешке Ventoy + Linux: [youtube.com/watch?v=BfjLJ0CqWsY](https://www.youtube.com/watch?v=BfjLJ0CqWsY)

Команда Linux для записи на USB:

```bash
sudo dd bs=4M if=artix-base-openrc-YYYYMMDD-x86_64.iso of=/dev/sdX status=progress oflag=sync
```

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

Оказавшись в терминале: логин по умолчанию — `root`, пароль — `artix`.

* * *

## 2\. Подключение к интернету

**Ethernet**:

Если используете ethernet (рекомендуется), переходите сразу к шагу «Проверьте подключение» ниже.

**Wi-Fi**:

Беспроводной интерфейс может быть по умолчанию программно заблокирован (soft-block). Сначала разблокируйте его:

```bash
rfkill unblock wifi
ip link set wlan0 up
```

Затем подключитесь через `wpa_supplicant`:

```bash
wpa_supplicant -B -i wlan0 -c <(wpa_passphrase "ВашSSID" "ВашПароль")

dhcpcd wlan0
```

Либо можно использовать менеджер подключений (ConnMan), входящий в ISO Artix:

```bash
connmanctl
> enable wifi
> scan wifi
> agent on
> connect wifi_<дополнение-по-tab>

> quit
```

Проверьте подключение:

```bash
ping -c 3 artixlinux.org
```

* * *

## 3\. Настройка клавиатуры и времени

Установите раскладку клавиатуры:

```bash
loadkeys us
```

* * *

## 4\. Разбиение NVMe-накопителя

Если используется слот NVMe M.2 2280, целевой накопитель будет виден как `/dev/nvme0n1`.

Проверьте свой накопитель:

```bash
lsblk
```

Если на накопителе осталась старая сигнатура swap, сначала отключите её:

```bash
swapoff /dev/nvme0n1p2 2>/dev/null
```

Если на накопителе есть предыдущая установка, сначала сотрите её:

```bash
wipefs -af /dev/nvme0n1
```

Запустите cfdisk для разбиения накопителя:

```bash
cfdisk /dev/nvme0n1
```

**Когда cfdisk откроется:**

1. Если появится запрос типа таблицы разделов, выберите **gpt**. Если на накопителе уже есть таблица MBR/DOS, сначала удалите все существующие разделы, затем в нижнем меню выберите **\[ Write \]** для применения, выйдите и снова запустите `cfdisk /dev/nvme0n1` — он предложит выбрать новый тип таблицы, и вы сможете выбрать **gpt**.
2. Выберите **\[ New \]** на свободном месте, введите **1G**, нажмите Enter. Это создаст EFI-раздел.
3. С выделенным этим разделом выберите **\[ Type \]** и укажите **EFI System**.
4. Стрелкой вниз перейдите к оставшемуся свободному месту, выберите **\[ New \]**, нажмите Enter, чтобы принять весь оставшийся объём. Это создаст корневой раздел (тип по умолчанию — **Linux filesystem**, что и нужно).
5. Выберите **\[ Write \]**, введите **yes** для подтверждения, затем **\[ Quit \]**.

**Итоговая разметка:**

| Раздел | Размер | Тип | Назначение |
| --- | --- | --- | --- |
| `/dev/nvme0n1p1` | 1 ГБ | EFI System | Загрузка EFI |
| `/dev/nvme0n1p2` | Остальное | Linux filesystem | Корень Btrfs |

* * *

## 5\. Форматирование разделов

Отформатируйте EFI-раздел в FAT32:

```bash
mkfs.fat -F32 -n EFI /dev/nvme0n1p1
```

Отформатируйте корневой раздел в Btrfs:

```bash
mkfs.btrfs -L ARTIX /dev/nvme0n1p2
```

* * *

## 6\. Создание подтомов Btrfs

Смонтируйте подтом верхнего уровня:

```bash
mount /dev/nvme0n1p2 /mnt
```

Создайте подтома:

```bash
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@var
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@snapshots
```

Размонтируйте верхний уровень:

```bash
umount /mnt
```

* * *

## 7\. Монтирование подтомов

Используйте опции монтирования, оптимизированные для SSD.

Задайте опции монтирования один раз:

```bash
BTRFS_OPTS="noatime,ssd,compress=zstd,space_cache=v2,discard=async"
```

Смонтируйте корневой подтом:

```bash
mount -o ${BTRFS_OPTS},subvol=@ /dev/nvme0n1p2 /mnt
```

Создайте каталоги точек монтирования:

```bash
mkdir -p /mnt/boot/efi
mkdir -p /mnt/home
mkdir -p /mnt/var
mkdir -p /mnt/var/log
mkdir -p /mnt/var/cache
mkdir -p /mnt/.snapshots
```

Смонтируйте остальные подтома:

```bash
mount -o ${BTRFS_OPTS},subvol=@home     /dev/nvme0n1p2 /mnt/home
mount -o ${BTRFS_OPTS},subvol=@var      /dev/nvme0n1p2 /mnt/var
mount -o ${BTRFS_OPTS},subvol=@log      /dev/nvme0n1p2 /mnt/var/log
mount -o ${BTRFS_OPTS},subvol=@cache    /dev/nvme0n1p2 /mnt/var/cache
mount -o ${BTRFS_OPTS},subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
```

Если получаете ошибку, что точка монтирования не существует, просто вернитесь и создайте каталог точки монтирования снова.

Смонтируйте EFI-раздел:

```bash
mount /dev/nvme0n1p1 /mnt/boot/efi
```

**Примечание: почему zram вместо раздела подкачки?** На btrfs файлы подкачки требуют особого обращения \[без сжатия, без copy-on-write (COW)\]. Zram проще, быстрее (сжатая RAM) и не изнашивает SSD. С 16–96 ГБ DDR5 в ПК zram — лучший выбор. Учтите: это означает, что гибернация (suspend-to-disk) недоступна (гибернация требует swap). Обычный спящий режим (закрытие крышки, сон в RAM) работает нормально.

* * *

## 8\. Установка базовой системы

Обновите зеркала пакетов:

```bash
pacman -Sy artix-keyring
```

Установите базовые пакеты:

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

Запустите reflector, чтобы использовать самые быстрые зеркала (это ускорит установку пакетов по мере продвижения по установке):

```bash
reflector --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
```

**Несколько примечаний о пакетах:**

- `linux` — базовое ядро Linux, заголовки и прошивки
- `amd-ucode` — обновления микрокода для процессоров AMD; ПРИМЕЧАНИЕ: если у вас процессор Intel, замените на `intel-ucode`
- `btrfs-progs` — инструменты управления Btrfs
- `elogind-openrc` — управление сеансами (необходимо для десктопных сессий без systemd)
- `networkmanager-openrc` — сервисный скрипт OpenRC для NetworkManager (сетевому виджету KDE Plasma нужен NetworkManager)
- `zramen-openrc` — сервисный скрипт OpenRC для zram (сжатие RAM)
- `os-prober` — утилита поиска загрузочных разделов других ОС (на случай будущей двойной загрузки)
- `pipewire-openrc` — сервисные скрипты OpenRC для аудио PipeWire (необходимо для звука на ПК)
- `nano` — текстовые редакторы (nano, VIM и NeoVIM)
- `git` — важнейшие утилиты (git, wget, curl) для установки других пакетов и скриптов

* * *

## 9\. Генерация fstab

```bash
fstabgen -U /mnt >> /mnt/etc/fstab
```

Проверьте, что каждое монтирование выглядит корректно:

```bash
cat /mnt/etc/fstab
```

Вы должны увидеть шесть записей Btrfs (по одной на подтом) плюс запись FAT32 для EFI.

* * *

## 10\. Chroot в новую систему

```bash
artix-chroot /mnt
```

* * *

## 11\. Часовой пояс и локаль

Установите часовой пояс (в /usr/share выбираются Регион/Город; в примере — America/New_York):

```bash
ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
hwclock --systohc
```

Установите локаль:

```bash
nano /etc/locale.gen
# Раскомментируйте: en_US.UTF-8 UTF-8

locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

* * *

## 12\. Hostname и hosts

Задайте hostname (имя Linux-системы):

```bash
# Впишите вместо hostname любое имя, которое хотите дать системе
echo "hostname" > /etc/hostname
echo "hostname" > /etc/conf.d/hostname
```

Отредактируйте `/etc/hosts`:

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   hostname.localdomain hostname
```

* * *

## 13\. Раскладка консоли (опционально)

```bash
# Только если используете раскладку, отличную от US
echo "KEYMAP=us" > /etc/vconsole.conf
```

* * *

## 14\. Установка пароля root и создание пользователя

Задайте пароль root:

```bash
passwd
```

Создайте своего пользователя и его пароль:

```bash
useradd -m -G wheel -s /bin/bash вашпользователь
passwd вашпользователь
```

Добавьте пользователя в группу sudo:

```bash
EDITOR=nano visudo
# Раскомментируйте (в секции wheel): %wheel ALL=(ALL:ALL) ALL
```

* * *

## 15\. Настройка mkinitcpio

Проверьте, что `/etc/mkinitcpio.conf` включает модуль Btrfs:

```bash
cat /etc/mkinitcpio.conf | grep "^HOOKS"
```

Убедитесь, что модули выводятся. Должно быть:

```bash
MODULES=(btrfs)
HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)
```

Затем перегенерируйте initramfs:

```bash
mkinitcpio -P
```

* * *

## 16\. Установка и настройка GRUB

Установите GRUB:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Artix
```

Сгенерируйте конфигурацию:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

* * *

## 17\. Включение основных служб

Включите сеть, вход при загрузке и zram:

```bash
rc-update add NetworkManager default
rc-update add elogind boot
rc-update add dbus default
rc-update add zramen default
```

* * *

## 18\. Настройка zram (вместо swap)

Настройте zram, отредактировав `/etc/conf.d/zramen`:

```bash
nano /etc/conf.d/zramen
```

Задайте содержимое (подберите размер по необходимости — распространённый выбор: 50 % вашей RAM):

```bash
ZRAM_SIZE=50%
ZRAM_ALGO=lz4
# или zstd
```

* * *

## 19\. Установка KDE Plasma (минимальная)

Установите только минимальный рабочий стол Plasma — без набора приложений KDE:

```bash
pacman -S plasma-desktop sddm sddm-openrc
# Везде выбирайте значения по умолчанию (1)
```

Это даёт оболочку Plasma, «Параметры системы» и экранный менеджер SDDM — и ничего больше. Это соответствует минималистичному подходу KDE-ISO самого Artix, где есть лишь файловый менеджер, медиаплеер, браузер и просмотрщик документов.

**Добавьте необходимые приложения**:

```bash
pacman -S dolphin          # Файловый менеджер
pacman -S konsole          # Терминал
pacman -S firefox          # Веб-браузер
pacman -S okular           # Просмотрщик документов
pacman -S vlc              # Медиаплеер
```

**Чтобы работали виджеты Plasma**:

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

**Включите SDDM:**

```bash
rc-update add sddm default
```

* * *

## 20\. Драйвер GPU AMD

Для GPU AMD установите драйверы Mesa:

```bash
pacman -S mesa vulkan-radeon libva-mesa-driver
```

Для GPU NVIDIA требуется репозиторий lib32. См. раздел **Включите дополнительные репозитории (для более широкого доступа к пакетам)**, затем выполните:

```bash
pacman -S nvidia nvidia-utils lib32-nvidia-utils nvidia-settings
```

* * *

## 21\. Bluetooth

Установите утилиты Bluetooth (bluez), затем включите службу.

```bash
pacman -S bluez bluez-utils bluez-openrc
rc-update add bluetoothd default
```

* * *

## 22\. Управление питанием (для ноутбуков)

Добавьте инструмент управления батареей (TLP) для Linux-ноутбуков.

```bash
pacman -S tlp
```

* * *

## 23\. Обновления прошивок устройств

Установите менеджер обновлений прошивок. Команды для запуска после перезагрузки см. в **Чек-листе после установки**.

```bash
pacman -S fwupd
```

* * *

## 24\. Выход, размонтирование и перезагрузка

```bash
exit                      # Выйти из chroot
umount -R /mnt            # Рекурсивное размонтирование
reboot
```

Извлеките USB-накопитель при перезапуске системы. Вас должен встретить GRUB, а затем экран входа SDDM.

* * *

## Чек-лист после установки

Войдя в Plasma, откройте терминал (Konsole):

**Проверьте Wi-Fi**

```bash
ping -c 3 artixlinux.org
```

Если есть ошибки и интернет не подключён:

```bash
nmcli device wifi list
```

Команда просканирует и покажет доступные сети. Затем подключитесь:

```bash
nmcli device wifi connect "ВашSSID" password "ВашПароль"
```

**Проверьте, что zram активен:**

```bash
swapon --show
# Должен показать /dev/zram0 с настроенным вами размером
```

**Проверьте подтома Btrfs:**

```bash
btrfs subvolume list /
```

```
/dev/nvme0n1
├── nvme0n1p1   1 ГБ    FAT32   /boot/efi     (системный раздел EFI)
└── nvme0n1p2   Остальное  Btrfs   (подтома)
    ├── @                       /
    ├── @home                   /home
    ├── @var                    /var
    ├── @log                    /var/log
    ├── @cache                  /var/cache
    └── @snapshots              /.snapshots
```

**Раздела подкачки нет** — сжатием памяти в RAM занимается zram.

**Проверьте драйвер GPU:**

```bash
glxinfo | grep "OpenGL renderer"
# Должен показать AMD Radeon / RDNA
```

**Проверьте и установите обновления прошивок:**

```bash
fwupdmgr refresh
fwupdmgr get-updates
fwupdmgr update
# затем перезагрузитесь для установки обновлений прошивок
```

**Запустите аудиоустройства:**

Добавьте и запустите службы PipeWire (аудио) для OpenRC.

```bash
rc-update add -U pipewire default
rc-update add -U pipewire-pulse default
rc-update add -U wireplumber default
rc-service -U pipewire start
rc-service -U pipewire-pulse start
rc-service -U wireplumber start
```

**Включите дополнительные репозитории (для более широкого доступа к пакетам):**

Отредактируйте `/etc/pacman.conf` и добавьте дополнительные зеркала:

```bash
sudo nano /etc/pacman.conf
```

Добавьте библиотеку lib32, чтобы можно было установить Steam:

```bash
[lib32]
Include = /etc/pacman.d/mirrorlist
# Раскомментируйте эту секцию
```

**Обновите все пакеты:**

```bash
sudo pacman -Syu
```

Выполните один раз после чистой установки и минимум раз в неделю, чтобы обновлять все пакеты и поддерживать систему.

* * *

## Дополнительные пакеты и обновления

**Рекомендуемые дополнительные пакеты (некоторые уже установлены на предыдущих шагах, но включены снова на всякий случай):**

Резервное копирование

```bash
sudo pacman -S timeshift snapper
```

- **Timeshift** — служба резервного копирования
- **Snapper** — альтернативная служба резервного копирования

Основные службы

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

- **NetworkManager**: самая простая сеть для десктопа
- **elogind**: интеграция сеансов/питания
- **acpid / upower**: поведение крышки, батареи, сна
- **cronie** — планировщик задач по времени, запускающий программы или скрипты по расписанию
- **bluez**: Bluetooth
- **cups**: только если печатаете
- **fwupd**: важно на Framework для обновлений прошивки
- **bolt**: полезно для авторизации устройств Thunderbolt/USB4
- **Прошивки SOF / ALSA**: помогают с аудиооборудованием современных ноутбуков
- **ufw**: Uncomplicated Firewall для управления файрволом

ufw требуется добавить в OpenRC:

```bash
rc-update add ufw default
```

KDE и утилиты:

```bash
sudo pacman -S \
  ark filelight gwenview okular \
  kate konsole \
  spectacle flameshot \
  partitionmanager \
  kdeconnect \
```

- **Okular** для PDF
- **Kate** как графический редактор по умолчанию
- **Spectacle** или **Flameshot** (предпочтителен) для скриншотов
- **Filelight** для очистки хранилища
- **KDE Connect**, если хотите связать Linux-систему с телефоном

Разработка / программирование:

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

Утилита для AUR:

```bash
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```

- **Paru** — рекомендуемый AUR-помощник

Приложения / пакеты из AUR:

```bash
paru -S brave-bin \
  signal-desktop \
  vscodium-bin \
  onlyoffice-bin \
  timeshift-autosnap \
```

- Браузер **Brave** (предпочтительнее Firefox)
- Мессенджер **Signal** (предпочтительнее WhatsApp, Telegram и т. п.)
- **VSCodium** — открытый текстовый редактор (предпочтительнее Visual Studio Code от Microsoft)
- **ONLYOFFICE** — открытый офисный пакет (предпочтителен из-за наилучшей совместимости с Microsoft Office)
- **Timeshift Autosnapper** устанавливает хук, чтобы Timeshift автоматически создавал резервную копию при каждом обновлении пакетов через pacman

Flatpak:

```bash
pacman -S flatpak
```

- Flatpak, если хотите легко устанавливать GUI-приложения

Медиаприложения:

```bash
sudo pacman -S \
  vlc mpv \
  ffmpeg \
  libreoffice-fresh \
  obs-studio \
```

- Медиаплеер **VLC** (предпочтителен) **или MPV**
- **FFmpeg** — инструменты конвертации медиа
- **OBS Studio** — открытая запись экрана / скринкастинг

Офисные приложения:

```bash
sudo pacman -S \
  libreoffice-fresh \
  thunderbird \
  okular \
  evince
```

- Офисный пакет **LibreOffice** (альтернатива ONLYOFFICE)
- **Thunderbird** — открытый почтовый клиент (замена Microsoft Outlook)
- **Okular** — просмотрщик документов (стандартный для KDE)
- **Evince** — просмотрщик документов (альтернатива Okular, поставляется с GNOME)

Игры:

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

- **Steam** — магазин и сервис дистрибуции видеоигр, включая дополнительные необходимые инструменты / драйверы (GPU AMD)
- Steam требует поддержки 32-битных приложений — нужна библиотека lib32

Tailscale:

```bash
sudo pacman -S tailscale tailscale-openrc
sudo rc-update add tailscaled default
sudo rc-service tailscaled start
```

- **Tailscale** — открытый сервис mesh-VPN

Шрифты:

```bash
sudo pacman -S ttf-jetbrains-mono
paru -S ttf-jetbrains-mono-nerd
```

- Шрифты **JetBrains** и их nerd-варианты

Утилиты мониторинга:

```bash
sudo pacman -S \
  btop fastfetch \
  ncdu \
  ripgrep fd fzf \
  htop \
  usbutils pciutils \
  smartmontools lm_sensors
```

- **Btop** — обзор системы
- **Fastfetch** (обязательный «neofetch»)
- `ripgrep`, `fd`, `fzf` — для комфорта в терминале
- `smartmontools` — здоровье SSD
- `lm_sensors` — температуры

Облачные хранилища:

```bash
paru -S \
  nextcloud-client-git \
  pcloud-drive
```

- Клиент **Nextcloud** — самостоятельно размещаемое облачное хранилище (требуется сервер Nextcloud)
- Клиент **pCloud** — ориентированное на приватность облачное хранилище (требуется учётная запись pCloud)

* * *

## Примечания

- **Ядро:** в этом руководстве используется ядро `linux`. Artix также предлагает `linux-lts` и `linux-zen`.
- **Снимки:** с такой схемой подтомов можно использовать `snapper` или `timeshift` для снимков `@` без захвата `/home`, `/var` и кэшей.
