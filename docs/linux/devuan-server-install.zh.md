---
description: >-
  Devuan Linux 服务器分步安装指南：sysvinit、ext4、纯终端/SSH 配置——没有 systemd。
---

<!--
Source: linux/devuan-server-install.md
Last translated: 2026-07
-->

# Devuan 服务器安装指南

**目标设备：**PC / 工作站（AMD Ryzen 处理器 + NVIDIA GPU）

**文件系统：**ext4

**桌面：**无。终端 / SSH

**Init：**sysvinit

**硬盘：**NVMe SSD（`/dev/nvme0n1`）

Devuan Wiki 需要注册才能访问，不过官方安装说明可在此查看：[devuan.org/os/documentation/install-guides/excalibur/install-devuan](https://www.devuan.org/os/documentation/install-guides/excalibur/install-devuan)

使用 netinstall 方式安装非常直接。

开始之前：

\- 确保 BIOS 设置为 UEFI（而非 legacy）
\- 启用虚拟化（SVM/VT-x）
\- 禁用 secure boot（安装 NVIDIA 驱动需要）

* * *

## 1\. 下载 Devuan 并制作可启动 ISO

从 [files.devuan.org/](https://files.devuan.org/) 下载 Devuan Excalibur

- 选择 devuan_excalibur（最新稳定版）
- 选择 installer_iso
- 下载 \_netinstall ISO

用 U 盘制作可启动镜像。PulsarTECH 有一个很棒的 Ventoy + Linux 多系统启动盘教程：[youtube.com/watch?v=BfjLJ0CqWsY](https://www.youtube.com/watch?v=BfjLJ0CqWsY)

在 Linux 上：

```bash
sudo dd if=devuan_excalibur_6.1.0_amd64_netinstall.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

## 2\. 启动 Live ISO

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

* * *

## 3\. 安装 Devuan

进入 Devuan 图形界面后，选择 `Install` 选项：

1. **语言、位置、键盘**
    - 选择系统语言、所在位置和键盘布局。
2. **网络配置**
    - 若通过有线连接，安装器会自动配置网络。若使用 WiFi，请提供网络的 SSID 和密码。
    - 输入主机名。示例：`devuanserver`（任意名字都行，只要没有空格和特殊字符）
    - 域名留空（除非你想添加）
3. **Root 密码与用户账户**
    - 设置 root 密码
    - 添加一个用户账户。示例：`firstname`
    - 为该用户设置密码
4. **时钟与时区**
    - 输入你的时区。示例：Eastern
5. **磁盘分区**
    - 若要配置加密（可选），按此说明为磁盘分区：[devuan.org/os/documentation/install-guides/excalibur/full-disk-encryption.html](https://www.devuan.org/os/documentation/install-guides/excalibur/full-disk-encryption.html)
    - 若不配置加密（推荐，安装更简单），可以选择 `Guided - use entire disk` 或 `Manual`
    - 若选择 `Manual`，下方按偏好提供了分区布局（推荐"简单分区布局"）
    - 创建好分区后，把变更写入磁盘——选择 `<Yes>`
6. **软件包管理器配置**
    - 选择一个 Devuan 归档镜像。首选 `deb.devuan.org`
    - 若需要 HTTP 代理，可填写代理信息。否则留空并选择 `<Continue>`
7. **Popularity-Contest 配置**
    - 是否参加软件包使用调查是可选的。按偏好选择 `<Yes>` 或 `<No>`（推荐 `<No>`）
8. **软件选择**
    1. 用空格键取消所有已选项，然后只选 `SSH server` 和 `standard system utilities`。不要选择任何桌面环境。这样会得到一个最小化的无头（headless）服务器安装
9. Init 系统选择
    - init 的选项包括：
        - `sysvinit`（默认、经典、文档完善）
        - `OpenRC`（现代、基于依赖、在 Gentoo 中流行）
        - `runit`（极简、启动快）
    - 想要默认 init（最稳定、兼容性最好），选择 `sysvinit`
    - 若偏好更现代、基于依赖的 init（用法上更接近 systemd），选择 `openrc`
10. 引导加载器
    - 将 GRUB 安装到 EFI 分区。当询问是否将 GRUB 引导加载器安装到主硬盘时，选择 `<Yes>`
    - 选择 `/dev/nvme0n1` 作为引导加载器的安装设备
11. **完成并重启**
    - 安装到此完成。选择 &lt;Continue&gt; 重启
    - 重启后立即拔掉 U 盘

* * *

### 简单分区布局（推荐）

| #   | 挂载点 | 大小 | 文件系统 | 用途 |
| --- | --- | --- | --- | --- |
| 1   | /boot/efi | 1GB | FAT32 | UEFI 引导所必需 |
| 2   | /boot   | 2GB | ext4 | 系统分区，便于恢复 |
| 3   | / | 其余全部 | ext4 | 数据存储 |

**设计理由**

- 让服务器保持简单
- 减少碎片化以及分区大小带来的问题
- 为运行 Docker 和容器化应用而优化
- 用 NAS 做大容量存储，把媒体文件卸载出去

如果你偏好企业级、彼此隔离的分区风格，可以选择：

### 企业级分区布局

| #   | 挂载点 | 大小 | 文件系统 | 用途 |
| --- | --- | --- | --- | --- |
| 1   | `/boot/efi` | 512 MiB | FAT32（EFI 系统分区） | UEFI 引导加载器 |
| 2   | `/boot` | 2 GiB | ext4 | 内核与 initramfs 镜像 |
| 3   | `/` | 50 GiB | ext4 | 根文件系统 |
| 4   | `/var` | 100 GiB | ext4 | 日志、数据库、容器层、软件包缓存 |
| 5   | `/tmp` | 10 GiB | ext4（以 noexec,nosuid,nodev 挂载） | 临时文件 |
| 6   | `swap` | 32 GiB | Linux swap | 交换空间（约为最大内存的 1/3） |
| 7   | `/home` | 50 GiB | ext4 | 用户家目录 |
| 8   | `/srv` | 其余全部 | ext4 或 XFS | 服务器数据、虚拟机、容器、数据集 |

### 设计理由

- **独立的 `/var`**：日志失控增长或容器镜像堆积不会填满根文件系统、拖垮整个系统。
- **独立的 `/tmp`**：以 `noexec,nosuid,nodev` 挂载以加固安全；防止它影响其他分区。
- **32 GiB swap**：在最高 96 GB 内存及潜在 GPU/CUDA 工作负载的情况下，32 GiB 为内存压力提供了充裕又不浪费的余量。如果要跑内存密集型 AI/ML 推理，可考虑加大。
- **大容量 `/srv`**：无头服务器的大部分存储都放在这里——虚拟机磁盘镜像、容器卷、数据集、NFS 导出等等。
- **独立的 `/boot`**：确保无论根文件系统出什么问题，引导加载器和内核镜像始终可访问。

* * *

## 4\. 安装后：首次开机设置

在终端以 root 登录（使用前面步骤设置的密码）。

- **验证网络连通性：**

```bash
ip addr show
ping -c 3 devuan.org
```

- **更新系统：**

```bash
apt update && apt upgrade -y
```

- **配置 APT 软件源：**

```bash
nano /etc/apt/sources.list
# 确保 contrib、non-free 和 non-free-firmware 都已启用

deb http://deb.devuan.org/merged excalibur main contrib non-free non-free-firmware
deb http://deb.devuan.org/merged excalibur-updates main contrib non-free non-free-firmware
deb http://deb.devuan.org/merged excalibur-security main contrib non-free non-free-firmware
```

- **再次更新：**

```bash
apt update
```

- **安装服务器必备软件包：**

```bash
apt install -y \
  sudo vim neovim htop tmux curl wget git \
  lm-sensors smartmontools nvme-cli \
  unattended-upgrades apt-listchanges \
  ufw \
  build-essential dkms linux-headers-amd64
```

- **将用户加入 sudo：**

```bash
usermod -aG sudo 你的用户名
```

- **创建 Swap 文件（如果你采用了简单分区布局）**

Swap 在系统内存耗尽时提供安全缓冲。虽然大内存的现代系统可以不用 swap，但为了稳定性仍然推荐配置——尤其是在运行 Docker 容器、数据库或 GPU 工作负载时。

在简单分区布局下，与其创建专用 swap 分区，不如使用 **swap 文件**。它更容易管理、调整大小和移除，无需重新分区。

下面的示例创建一个 16 GB 的 swap 文件。请按你的系统调整大小：

```bash
fallocate -l 16G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

将以下行加入 `/etc/fstab`，让 swap 文件开机自动启用：

```bash
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

验证 Swap

```bash
swapon --show
free -h
```

### 大小建议

- **16 GB 内存及以下：**推荐 8–16 GB swap
- **32–64 GB 内存：**8–16 GB swap 足够
- **64 GB 以上内存：**4–16 GB swap 作为安全缓冲
- **重度 GPU / AI 工作负载：**可考虑 16–32 GB swap

* * *

## 5\. 安装并配置 Tailscale

Tailscale 提供基于 WireGuard 的安全、零配置 mesh 网络。所有 SSH 访问都将经由 Tailscale 网络，这意味着没有任何 SSH 端口暴露在公网上。

- **Devuan Excalibur 基于 Debian Trixie，因此使用 Trixie 软件源：**

```bash
mkdir -p --mode=0755 /usr/share/keyrings

curl -fsSL https://pkgs.tailscale.com/stable/debian/trixie.noarmor.gpg \
  | tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null

curl -fsSL https://pkgs.tailscale.com/stable/debian/trixie.tailscale-keyring.list \
  | tee /etc/apt/sources.list.d/tailscale.list

apt update
```

- **安装 Tailscale：**

```bash
apt install -y tailscale
```

**重要：**Tailscale 官方包附带的是 systemd 服务文件，没有 sysvinit 脚本。由于 Devuan 使用 sysvinit，安装后需要手动创建一个。

- **软件包只包含 systemd unit，因此必须手动创建 init 脚本：**

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

- **启用并启动 Tailscale：**

```bash
update-rc.d tailscaled defaults
mkdir -p /run/tailscale
service tailscaled start
service tailscaled status
```

- **认证加入你的 Tailnet：**

```bash
tailscale up
```

命令会输出一个 URL。在另一台设备的浏览器中打开它，登录你的 Tailscale 账号并授权这台机器。认证完成后，验证连通性：

```bash
tailscale status
tailscale ip -4
```

记下 Tailscale IP（通常是 `100.x.y.z`）。这就是你之后 SSH 要用的地址。

- **启用 Tailscale SSH：**

Tailscale SSH 让你通过 Tailscale 的身份提供商来认证 SSH 会话，彻底省去 SSH 密钥和密码：

```bash
tailscale set --ssh
```

使用 Tailscale SSH 时，连接由你的 Tailscale ACL 认证，而非本地 SSH 密钥。可以在 Tailscale 管理控制台的 **Access Controls** 中配置谁拥有访问权限。

**注意：**如果你在上文启用了 Tailscale SSH，可以选择完全禁用本地 SSH 守护进程，因为 SSH 已由 Tailscale 直接处理：

```bash
service ssh stop
update-rc.d ssh disable
```

- **在拔掉显示器之前，先从另一台机器测试经由 Tailscale 的 SSH：**

```bash
ssh 你的用户名@100.x.y.z
```

如果启用了 Tailscale MagicDNS，也可以用机器在 Tailnet 中的名字代替 Tailscale IP。

- **配置防火墙：**

由于 SSH 只能通过 Tailscale 访问，防火墙应彻底阻断来自公共接口的 SSH。只为局域网中确实需要的服务开放端口。

```bash
ufw default deny incoming
ufw default allow outgoing

# 放行 Tailscale 接口上的全部流量（可信的 mesh 网络）
ufw allow in on tailscale0

# 不要在公共接口上放行 SSH——它只走 Tailscale
# ufw allow ssh  <-- 有意省略

ufw enable
```

如果之后要运行需要局域网访问的服务（如 NFS、Web 服务器、Samba），请在特定的局域网接口上为特定端口添加规则：

```bash
# 示例：仅在 2.5G 局域网接口上放行 HTTP
ufw allow in on enp2s0 to any port 80 proto tcp
```

* * *

## 6\. 安装 NVIDIA GPU 驱动

安装 NVIDIA 专有驱动之前，必须先禁用开源的 Nouveau 驱动：

```bash
cat > /etc/modprobe.d/blacklist-nouveau.conf << 'EOF'
blacklist nouveau
options nouveau modeset=0
EOF

update-initramfs -u
reboot
```

重启后重新登录并安装：

```bash
sudo apt update
sudo apt install linux-headers-$(uname -r) build-essential libglvnd-dev pkg-config dkms
```

检测并安装驱动：

```bash
sudo nvidia-detect
sudo apt install nvidia-driver nvidia-kernel-dkms nvidia-smi nvidia-settings
```

只有当默认驱动不支持你的 GPU 时才使用 backports：

```bash
sudo apt install nvidia-driver firmware-misc-nonfree
```

验证 DKMS 编译：

```bash
dkms status
```

你应该会看到类似这样的一行：

```
nvidia/550.163.01, 6.12.x-amd64, x86_64: installed
```

必须重启并验证：

```bash
reboot
```

重启后，确认驱动已加载：

```bash
nvidia-smi
```

预期输出会显示 GPU 及驱动版本、CUDA 版本、温度和显存占用。对于没有接显示器的无头服务器，nvidia-smi 是验证 GPU 正常工作的主要手段。

- **启用持久化模式（无头服务器）：**

在没有运行 X 服务器的情况下，NVIDIA 驱动可能会在 GPU 任务之间卸载，增加延迟。启用持久化模式：

```bash
nvidia-smi -pm 1
```

要让它跨重启保持生效，创建一个 init 脚本。sysvinit 版本如下：

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

- **安装 CUDA Toolkit：**

如果计算工作负载需要 CUDA（AI 推理、GPU 加速应用）：

```bash
apt install -y nvidia-cuda-toolkit
```

验证：

```bash
nvcc --version
```

* * *

## 7\. 安装 Docker

Docker CE 的官方 Debian 软件包自带 sysvinit 的 init 脚本（`/etc/init.d/docker`），因此 Docker 无需 systemd 即可在 Devuan 上原生运行。

- 移除冲突的软件包

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
    apt-get remove -y $pkg 2>/dev/null
done
```

- 添加 Docker 软件源

由于 Devuan Excalibur 基于 Debian Trixie，使用 Trixie 软件源：

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

- 安装 Docker Engine

```bash
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- 启用并启动 Docker（sysvinit）

Docker 软件包附带 `/etc/init.d/docker`。设为开机启动并立即启动：

```bash
update-rc.d docker defaults
service docker start
```

验证 Docker 已运行：

```bash
service docker status
docker info
```

- 允许你的用户运行 Docker

把你的日常用户加入 `docker` 组，这样每条 Docker 命令就不必再加 `sudo`：

```bash
usermod -aG docker 你的用户名
```

注销并重新登录（或执行 `newgrp docker`）使组变更生效。

**安全提示：**`docker` 组的成员身份等同于拥有主机的 root 权限。只添加可信用户。

- 测试安装

```bash
docker run --rm hello-world
```

- NVIDIA Container Toolkit（GPU 容器）

要在 Docker 容器内使用 NVIDIA GPU（用于 CUDA、AI 推理等），安装 NVIDIA Container Toolkit：

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
  | gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -fsSL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
  | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
  | tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

apt update
apt install -y nvidia-container-toolkit
```

配置 Docker 运行时：

```bash
nvidia-ctk runtime configure --runtime=docker
service docker restart
```

从容器内测试 GPU 访问：

```bash
docker run --rm --gpus all nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
```

你应该能看到列出的 NVIDIA GPU 及驱动版本和 CUDA 版本。

### Docker Compose 速查

Docker Compose 以 CLI 插件的形式安装。用法如下：

```bash
docker compose up -d          # 在后台启动服务

docker compose down            # 停止并移除服务

docker compose logs -f         # 跟踪日志

docker compose ps              # 列出运行中的服务
```

* * *

## 8\. NFS NAS 挂载

如果要挂载外部 NAS 存储（如 Synology、QNAP、TrueNAS），按以下说明操作。

- 安装 NFS 客户端软件包

```bash
apt install -y nfs-common
```

- 创建挂载点

为每个要挂载的 NFS 共享创建目录。一个干净的惯例是把它们统一挂在 `/mnt/nas/` 之下：

```bash
mkdir -p /mnt/nas/media
mkdir -p /mnt/nas/backups
mkdir -p /mnt/nas/shared
```

按你 NAS 的导出结构调整名称。

- 先手动测试挂载

在设为永久之前，先验证每个挂载可用：

```bash
mount -t nfs4 nas.local:/volume1/media /mnt/nas/media
ls /mnt/nas/media
```

把 `nas.local` 替换为你 NAS 的 IP 或主机名，把 `/volume1/media` 替换为实际的 NFS 导出路径。如果使用 NFSv3：

```bash
mount -t nfs -o vers=3 nas.local:/volume1/media /mnt/nas/media
```

- 在 /etc/fstab 中配置永久挂载

手动测试成功后，把条目加入 `/etc/fstab` 以便开机自动挂载：

```bash
nano /etc/fstab

# NAS NFS 挂载
nas.local:/volume1/media    /mnt/nas/media    nfs4  rw,soft,timeo=30,retrans=3,_netdev  0  0
nas.local:/volume1/backups  /mnt/nas/backups  nfs4  rw,soft,timeo=30,retrans=3,_netdev  0  0
nas.local:/volume1/shared   /mnt/nas/shared   nfs4  rw,soft,timeo=30,retrans=3,_netdev  0  0
```

**挂载选项说明：**

- **`rw`**：读写访问。像媒体库这样的只读共享请用 `ro`。
- **`soft`**：NAS 不可达时返回错误，而不是无限期挂起。对无头服务器至关重要——若使用 `hard` 挂载，NAS 掉线时进程会冻结且无法杀死。
- **`timeo=30`**：重试前的超时时间，单位是十分之一秒（即 3 秒）。
- **`retrans=3`**：报告失败前的重试次数。
- **`_netdev`**：告诉 init 系统此挂载需要先有网络，防止 NAS 不可达时开机卡死。

> ⚠️ **`soft` 与 `hard` 是一个有意为之的取舍。**本指南使用 `soft`，以便 NAS 掉线时无头服务器仍能保持响应。[NAS 挂载指南](../homelab/nas-mounting.md)则为有写入的共享（尤其是备份）推荐 `hard`，因为 `soft` 可能悄无声息地截断被中断的写入。按共享分别选择：以读为主的媒体用 `soft`，写入关键的数据用 `hard`。

挂载 fstab 中的全部新条目：

```bash
mount -a
```

验证：

```bash
df -h | grep nas
```

- NFS 挂载权限

NFS 权限取决于 NAS 导出的配置方式。常见做法：

**UID/GID 映射（推荐）：**确保 Devuan 用户的 UID 和 GID 与 NFS 导出预期的 UID/GID 一致。在 Devuan 上用 `id 你的用户名` 查看，并与 NAS 的设置比对。

**all_squash 加 anonuid/anongid：**如果 NAS 导出使用 `all_squash`，所有访问都会映射到 NAS 侧定义的单一 UID/GID。这是共享访问最简单的方案。

**No root squash：**只有当你确实需要从这台服务器以 root 写入时才在 NAS 上启用 `no_root_squash`。这有安全风险，且通常没有必要。

- 在 Docker 容器中使用 NFS 挂载

绑定挂载主机上的 NFS 路径

在你的 `compose.yaml` 中：

```yaml
services:
  myapp:
    image: myapp:latest
    volumes:
      - /mnt/nas/media:/data/media
      - /mnt/nas/shared:/data/shared
```

这是最简单的方式。NFS 连接由主机处理，容器把数据当作普通目录来看待。

- 监控 NFS 挂载健康状态

如果 NAS 重启或网络抖动，NFS 挂载可能会悄悄变成过期（stale）状态。创建一个简单的健康检查 cron 任务：

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

该脚本每 5 分钟检查一次每个 NFS 挂载是否有响应，并在共享变为过期状态时尝试重新挂载。可用 `grep nfs-health /var/log/syslog` 在 `/var/log/syslog` 中查看结果。

## 9\. Fastfetch

"必备的 neofetch"（fastfetch）：

```bash
sudo apt update
sudo apt install fastfetch
```

* * *

## 成果

- 没有 systemd
- 完整的 Docker 支持
- GPU 加速正常工作
- NAS 挂载就绪
