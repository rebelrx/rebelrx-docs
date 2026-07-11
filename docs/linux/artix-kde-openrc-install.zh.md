---
description: >-
  在 AMD 硬件上手动安装 Artix Linux 的分步指南：OpenRC、Btrfs 子卷、zram 与 KDE Plasma。
---

<!--
Source: linux/artix-kde-openrc-install.md
Last translated: 2026-07
-->

# Artix Linux 手动安装指南

**目标设备：**PC / 笔记本（AMD Ryzen 处理器 + AMD GPU）

**文件系统：**Btrfs（使用子卷）

**Swap：**zram（不设 swap 分区）

**桌面：**KDE Plasma（最精简安装）

**Init：**OpenRC

**硬盘：**NVMe SSD（`/dev/nvme0n1`）

Artix Wiki 也是很好的补充参考和排错资源：[wiki.artixlinux.org/](https://wiki.artixlinux.org/)

安装 Artix 的大致步骤（与大多数 Linux 发行版类似）包括：连接互联网、为磁盘分区、格式化分区、挂载分区、安装 Linux 内核和基础系统软件包、配置本地化、安装引导加载器、添加用户并设置密码、配置网络、安装桌面环境和登录管理器、启用核心服务，以及安装后配置。

* * *

## 1\. 启动 Live ISO

从 [artixlinux.org/download.php](https://artixlinux.org/download.php) 下载 OpenRC 版基础 ISO，并将其烧录到 U 盘

- PulsarTECH 有一个很棒的 Ventoy + Linux 多系统启动盘教程：[youtube.com/watch?v=BfjLJ0CqWsY](https://www.youtube.com/watch?v=BfjLJ0CqWsY)

在 Linux 上烧录 U 盘的命令：

```bash
sudo dd bs=4M if=artix-base-openrc-YYYYMMDD-x86_64.iso of=/dev/sdX status=progress oflag=sync
```

从 U 盘启动电脑。下面是各厂商启动菜单按键的参考列表：

| 厂商 | 启动菜单键 | BIOS/UEFI 键 |
| :--- | :--- | :--- |
| **Acer** | F12、Esc、F9 | F2、Delete |
| **Asus** | F8、Esc | F2、Delete |
| **Dell** | F12 | F2  |
| **HP** | F9、Esc | F10 |
| **Lenovo** | F12、F10、F8（或 Novo 键） | F2、F1 |
| **MSI** | F11 | Delete |
| **Toshiba** | F12 | F2、F1、Esc |
| **Samsung** | F12、F2、Esc | F2  |
| **Sony VAIO** | F11、F10、Esc（或 Assist 键） | F2、F1、F3 |
| **Gigabyte** | F12 | Delete、F2 |
| **Intel NUC** | F10 | F2  |

进入终端后，默认登录名为 `root`，密码为 `artix`。

* * *

## 2\. 连接互联网

**有线网络**：

如果使用有线网络（推荐），直接跳到下面的"验证连通性"步骤。

**Wi-Fi**：

无线网卡默认可能被软锁定（soft-block）。先解除锁定：

```bash
rfkill unblock wifi
ip link set wlan0 up
```

然后用 `wpa_supplicant` 连接：

```bash
wpa_supplicant -B -i wlan0 -c <(wpa_passphrase "你的SSID" "你的密码")

dhcpcd wlan0
```

也可以使用 Artix ISO 自带的连接管理工具（ConnMan）：

```bash
connmanctl
> enable wifi
> scan wifi
> agent on
> connect wifi_<按tab补全>

> quit
```

验证连通性：

```bash
ping -c 3 artixlinux.org
```

* * *

## 3\. 设置键盘与时间

设置键盘布局：

```bash
loadkeys us
```

* * *

## 4\. 为 NVMe 硬盘分区

如果使用的是 NVMe M.2 2280 插槽，目标硬盘会显示为 `/dev/nvme0n1`。

确认你的硬盘：

```bash
lsblk
```

如果硬盘上有旧的 swap 签名，先将其禁用：

```bash
swapoff /dev/nvme0n1p2 2>/dev/null
```

如果硬盘上有之前的系统安装，先将其抹除：

```bash
wipefs -af /dev/nvme0n1
```

启动 cfdisk 为硬盘分区：

```bash
cfdisk /dev/nvme0n1
```

**cfdisk 打开后：**

1. 如果提示选择分区表类型，选择 **gpt**。如果硬盘上已有 MBR/DOS 分区表，先删除所有现有分区，然后到底部菜单选择 **\[ Write \]** 应用，退出后重新运行 `cfdisk /dev/nvme0n1`——它会再次提示选择新的分区表类型，此时选择 **gpt**。
2. 在空闲空间上选择 **\[ New \]**，输入 **1G**，按回车。这将创建 EFI 分区。
3. 保持该分区高亮，选择 **\[ Type \]**，选择 **EFI System**。
4. 方向键下移到剩余空闲空间，选择 **\[ New \]**，按回车接受全部剩余容量。这将创建根分区（类型默认为 **Linux filesystem**，正是我们要的）。
5. 选择 **\[ Write \]**，输入 **yes** 确认，然后 **\[ Quit \]** 退出。

**最终布局：**

| 分区 | 大小 | 类型 | 用途 |
| --- | --- | --- | --- |
| `/dev/nvme0n1p1` | 1 GB | EFI System | EFI 引导 |
| `/dev/nvme0n1p2` | 其余全部 | Linux filesystem | Btrfs 根 |

* * *

## 5\. 格式化分区

将 EFI 分区格式化为 FAT32：

```bash
mkfs.fat -F32 -n EFI /dev/nvme0n1p1
```

将根分区格式化为 Btrfs：

```bash
mkfs.btrfs -L ARTIX /dev/nvme0n1p2
```

* * *

## 6\. 创建 Btrfs 子卷

挂载顶层子卷：

```bash
mount /dev/nvme0n1p2 /mnt
```

创建子卷：

```bash
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@var
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@snapshots
```

卸载顶层：

```bash
umount /mnt
```

* * *

## 7\. 挂载子卷

使用为 SSD 优化的挂载选项。

先一次性定义好挂载选项：

```bash
BTRFS_OPTS="noatime,ssd,compress=zstd,space_cache=v2,discard=async"
```

挂载根子卷：

```bash
mount -o ${BTRFS_OPTS},subvol=@ /dev/nvme0n1p2 /mnt
```

创建各挂载点目录：

```bash
mkdir -p /mnt/boot/efi
mkdir -p /mnt/home
mkdir -p /mnt/var
mkdir -p /mnt/var/log
mkdir -p /mnt/var/cache
mkdir -p /mnt/.snapshots
```

挂载其余子卷：

```bash
mount -o ${BTRFS_OPTS},subvol=@home     /dev/nvme0n1p2 /mnt/home
mount -o ${BTRFS_OPTS},subvol=@var      /dev/nvme0n1p2 /mnt/var
mount -o ${BTRFS_OPTS},subvol=@log      /dev/nvme0n1p2 /mnt/var/log
mount -o ${BTRFS_OPTS},subvol=@cache    /dev/nvme0n1p2 /mnt/var/cache
mount -o ${BTRFS_OPTS},subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
```

如果报错说挂载点不存在，回头把对应的挂载点目录再创建一次即可。

挂载 EFI 分区：

```bash
mount /dev/nvme0n1p1 /mnt/boot/efi
```

**注：为什么用 zram 而不是 swap 分区？**在 btrfs 上，swap 文件需要特殊处理\[禁用压缩、禁用写时复制（COW）\]。zram 更简单、更快（压缩内存），也避免了 SSD 磨损。PC 上有 16–96GB 的 DDR5 内存时，zram 是更好的选择。注意：这意味着休眠（挂起到磁盘）不可用（休眠需要 swap）。普通睡眠（合盖、挂起到内存）工作正常。

* * *

## 8\. 安装基础系统

刷新软件包镜像：

```bash
pacman -Sy artix-keyring
```

安装基础软件包：

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

运行 reflector 以使用最快的镜像（能加快后续安装过程中软件包的下载速度）：

```bash
reflector --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
```

**几点软件包说明：**

- `linux` — Linux 基础内核、头文件和固件
- `amd-ucode` — AMD 处理器的微码更新；注意：如果你用的是 Intel 处理器，请改为 `intel-ucode`
- `btrfs-progs` — Btrfs 管理工具
- `elogind-openrc` — 会话管理（在没有 systemd 的情况下运行桌面会话所必需）
- `networkmanager-openrc` — NetworkManager 的 OpenRC 服务脚本（KDE Plasma 的网络小部件需要 NetworkManager）
- `zramen-openrc` — zram 的 OpenRC 服务脚本（内存压缩）
- `os-prober` — 扫描其他操作系统可引导分区的工具（方便将来做双系统）
- `pipewire-openrc` — PipeWire 音频的 OpenRC 服务脚本（PC 音频所必需）
- `nano` — 文本编辑器（nano、VIM 和 NeoVIM）
- `git` — 基础工具（git、wget、curl），用于安装其他软件包和脚本

* * *

## 9\. 生成 fstab

```bash
fstabgen -U /mnt >> /mnt/etc/fstab
```

核对每条挂载是否正确：

```bash
cat /mnt/etc/fstab
```

你应该能看到六条 Btrfs 条目（每个子卷一条），外加 EFI 的 FAT32 条目。

* * *

## 10\. Chroot 进入新系统

```bash
artix-chroot /mnt
```

* * *

## 11\. 时区与 Locale

设置时区（在 /usr/share 下按"地区/城市"选择，此处以 America/New_York 为例）：

```bash
ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
hwclock --systohc
```

设置 locale：

```bash
nano /etc/locale.gen
# 取消注释：en_US.UTF-8 UTF-8

locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

* * *

## 12\. 主机名与 hosts

设置主机名（Linux 系统名称）：

```bash
# 把 hostname 换成你想给系统起的任意名字
echo "hostname" > /etc/hostname
echo "hostname" > /etc/conf.d/hostname
```

编辑 `/etc/hosts`：

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   hostname.localdomain hostname
```

* * *

## 13\. 控制台键盘映射（可选）

```bash
# 仅当使用非美式键盘布局时需要
echo "KEYMAP=us" > /etc/vconsole.conf
```

* * *

## 14\. 设置 root 密码并创建用户

设置 root 密码：

```bash
passwd
```

创建你的用户及其密码：

```bash
useradd -m -G wheel -s /bin/bash 你的用户名
passwd 你的用户名
```

将用户加入 sudo 组：

```bash
EDITOR=nano visudo
# 取消注释（在 wheel 部分）：%wheel ALL=(ALL:ALL) ALL
```

* * *

## 15\. 配置 mkinitcpio

检查 `/etc/mkinitcpio.conf` 是否包含 Btrfs 模块：

```bash
cat /etc/mkinitcpio.conf | grep "^HOOKS"
```

确认模块输出无误。确保有：

```bash
MODULES=(btrfs)
HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)
```

然后重新生成 initramfs：

```bash
mkinitcpio -P
```

* * *

## 16\. 安装并配置 GRUB

安装 GRUB：

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Artix
```

生成配置：

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

* * *

## 17\. 启用核心服务

启用网络、开机登录和 zram：

```bash
rc-update add NetworkManager default
rc-update add elogind boot
rc-update add dbus default
rc-update add zramen default
```

* * *

## 18\. 配置 zram（取代 swap）

编辑 `/etc/conf.d/zramen` 来配置 zram：

```bash
nano /etc/conf.d/zramen
```

设置内容（大小按需调整——常见做法是内存的 50%）：

```bash
ZRAM_SIZE=50%
ZRAM_ALGO=lz4
# 或 zstd
```

* * *

## 19\. 安装 KDE Plasma（最小安装）

只安装最精简的 Plasma 桌面——不装 KDE 应用套件：

```bash
pacman -S plasma-desktop sddm sddm-openrc
# 全部选择默认值（1）
```

这只会给你 Plasma 外壳、系统设置和 SDDM 登录管理器——仅此而已。这与 Artix 官方 KDE ISO 的极简思路一致，后者也只附带文件管理器、媒体播放器、浏览器和文档查看器。

**添加必备应用**：

```bash
pacman -S dolphin          # 文件管理器
pacman -S konsole          # 终端
pacman -S firefox          # 网页浏览器
pacman -S okular           # 文档查看器
pacman -S vlc              # 媒体播放器
```

**让 Plasma 的小部件正常工作**：

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

**启用 SDDM：**

```bash
rc-update add sddm default
```

* * *

## 20\. AMD GPU 驱动

AMD GPU 请安装 Mesa 驱动：

```bash
pacman -S mesa vulkan-radeon libva-mesa-driver
```

NVIDIA GPU 需要 lib32 仓库。参见 **启用额外软件仓库（获取更多软件包）** 一节，然后运行：

```bash
pacman -S nvidia nvidia-utils lib32-nvidia-utils nvidia-settings
```

* * *

## 21\. 蓝牙

安装蓝牙工具（bluez），然后启用。

```bash
pacman -S bluez bluez-utils bluez-openrc
rc-update add bluetoothd default
```

* * *

## 22\. 电源管理（笔记本）

为 Linux 笔记本添加电池管理工具（TLP）。

```bash
pacman -S tlp
```

* * *

## 23\. 设备固件更新

安装设备固件更新管理器。重启后运行的固件更新命令，参见**安装后检查清单**。

```bash
pacman -S fwupd
```

* * *

## 24\. 退出、卸载并重启

```bash
exit                      # 退出 chroot
umount -R /mnt            # 递归卸载
reboot
```

系统重启时拔掉 U 盘。你应该会先看到 GRUB，然后是 SDDM 登录界面。

* * *

## 安装后检查清单

登录 Plasma 后，打开终端（Konsole）：

**验证 WiFi**

```bash
ping -c 3 artixlinux.org
```

如果报错、未连接互联网：

```bash
nmcli device wifi list
```

这会扫描并显示可用网络。然后连接：

```bash
nmcli device wifi connect "你的SSID" password "你的密码"
```

**验证 zram 已启用：**

```bash
swapon --show
# 应显示 /dev/zram0 及你配置的大小
```

**验证 Btrfs 子卷：**

```bash
btrfs subvolume list /
```

```
/dev/nvme0n1
├── nvme0n1p1   1 GB    FAT32   /boot/efi     （EFI 系统分区）
└── nvme0n1p2   其余    Btrfs   （子卷）
    ├── @                       /
    ├── @home                   /home
    ├── @var                    /var
    ├── @log                    /var/log
    ├── @cache                  /var/cache
    └── @snapshots              /.snapshots
```

**没有 swap 分区**——由 zram 在内存中负责压缩。

**验证 GPU 驱动：**

```bash
glxinfo | grep "OpenGL renderer"
# 应显示 AMD Radeon / RDNA
```

**检查并执行固件更新：**

```bash
fwupdmgr refresh
fwupdmgr get-updates
fwupdmgr update
# 然后重启以安装固件更新
```

**启动音频设备：**

为 OpenRC 添加并启动 PipeWire（音频）服务。

```bash
rc-update add -U pipewire default
rc-update add -U pipewire-pulse default
rc-update add -U wireplumber default
rc-service -U pipewire start
rc-service -U pipewire-pulse start
rc-service -U wireplumber start
```

**启用额外软件仓库（获取更多软件包）：**

编辑 `/etc/pacman.conf` 添加额外镜像：

```bash
sudo nano /etc/pacman.conf
```

添加 lib32 库，这样才能安装 Steam：

```bash
[lib32]
Include = /etc/pacman.d/mirrorlist
# 取消注释这一部分
```

**更新所有软件包：**

```bash
sudo pacman -Syu
```

全新安装后运行一次，之后每周至少运行一次以更新全部软件包、维护系统。

* * *

## 额外软件包与更新

**推荐的额外软件包（部分已在前面步骤中安装，此处再列一遍以防遗漏）：**

备份

```bash
sudo pacman -S timeshift snapper
```

- **Timeshift** 备份服务
- **Snapper** 备份服务的替代方案

核心服务

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

- **NetworkManager**：最省心的桌面网络方案
- **elogind**：会话/电源集成
- **acpid / upower**：合盖、电池、睡眠行为
- **cronie**：基于时间的任务调度器，用于在预定时间运行指定程序或脚本
- **bluez**：蓝牙
- **cups**：仅在需要打印时安装
- **fwupd**：在 Framework 机器上对固件更新很重要
- **bolt**：用于 Thunderbolt/USB4 设备授权
- **SOF / ALSA 固件**：帮助支持现代笔记本的音频硬件
- **ufw**：Uncomplicated Firewall，用于管理防火墙

ufw 需要加入 OpenRC：

```bash
rc-update add ufw default
```

KDE 与实用工具：

```bash
sudo pacman -S \
  ark filelight gwenview okular \
  kate konsole \
  spectacle flameshot \
  partitionmanager \
  kdeconnect \
```

- **Okular** 看 PDF
- **Kate** 作为默认图形编辑器
- **Spectacle** 或 **Flameshot**（更推荐）截图
- **Filelight** 清理存储空间
- **KDE Connect**：想把 Linux 系统和手机联动时使用

开发 / 编程：

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

AUR 工具：

```bash
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```

- **Paru**：推荐的 AUR 助手

AUR 应用 / 软件包：

```bash
paru -S brave-bin \
  signal-desktop \
  vscodium-bin \
  onlyoffice-bin \
  timeshift-autosnap \
```

- **Brave** 浏览器（比 Firefox 更推荐）
- **Signal** 通讯软件（比 WhatsApp、Telegram 等更推荐）
- **VSCodium** 开源文本编辑器（比微软的 Visual Studio Code 更推荐）
- **ONLYOFFICE** 开源办公套件（推荐理由：与 Microsoft Office 兼容性最好）
- **Timeshift Autosnapper**：安装一个钩子，让 Timeshift 在每次通过 pacman 更新软件包时自动创建备份

Flatpak：

```bash
pacman -S flatpak
```

- 想方便地安装图形应用就装 Flatpak

媒体应用：

```bash
sudo pacman -S \
  vlc mpv \
  ffmpeg \
  libreoffice-fresh \
  obs-studio \
```

- **VLC**（更推荐）**或 MPV** 媒体播放器
- **FFmpeg** 媒体转换工具
- **OBS Studio** 开源录屏 / 直播工具

办公应用：

```bash
sudo pacman -S \
  libreoffice-fresh \
  thunderbird \
  okular \
  evince
```

- **LibreOffice** 办公套件（ONLYOFFICE 的替代方案）
- **Thunderbird** 开源邮件客户端（替代 Microsoft Outlook）
- **Okular** 文档查看器（KDE 默认）
- **Evince** 文档查看器（Okular 的替代品，随 GNOME 打包）

游戏：

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

- **Steam** 游戏商店与分发平台，含所需的额外工具/驱动（AMD GPU）
- Steam 需要 32 位应用支持——依赖 lib32 库

Tailscale：

```bash
sudo pacman -S tailscale tailscale-openrc
sudo rc-update add tailscaled default
sudo rc-service tailscaled start
```

- **Tailscale** 开源 mesh VPN 服务

字体：

```bash
sudo pacman -S ttf-jetbrains-mono
paru -S ttf-jetbrains-mono-nerd
```

- **JetBrains** 字体及其 nerd 字体

监控工具：

```bash
sudo pacman -S \
  btop fastfetch \
  ncdu \
  ripgrep fd fzf \
  htop \
  usbutils pciutils \
  smartmontools lm_sensors
```

- **Btop** 系统概览
- **Fastfetch**（必备的"neofetch"）
- `ripgrep`、`fd`、`fzf`：提升终端使用体验
- `smartmontools`：SSD 健康状态
- `lm_sensors`：温度监测

云存储：

```bash
paru -S \
  nextcloud-client-git \
  pcloud-drive
```

- **Nextcloud** 自托管云存储客户端（需要 Nextcloud 服务器）
- **pCloud** 注重隐私的云存储客户端（需要 pCloud 账号）

* * *

## 备注

- **内核：**本指南使用 `linux` 内核。Artix 还提供 `linux-lts` 和 `linux-zen`。
- **快照：**采用这套子卷布局后，你可以用 `snapper` 或 `timeshift` 对 `@` 做快照，而不会把 `/home`、`/var` 或缓存一并收入快照。
