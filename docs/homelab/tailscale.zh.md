---
description: >-
  在非 systemd 的 Linux 上使用 Tailscale 安全远程访问你的家庭实验室——Devuan sysvinit 配置、子网路由、出口节点与 ACL，全程无需开放任何端口。
---

<!--
Source: homelab/tailscale.md
Last translated: 2026-07
-->

# 🔒 Tailscale VPN（Devuan + 非 systemd）

这是一份实用指南，教你如何在**不向互联网开放任何端口**的情况下，从任何地方访问你的家庭实验室。

大多数 Tailscale 指南都默认使用 systemd。本指南面向服务器上的 **Devuan（sysvinit）**——Tailscale 官方完全不提供其 init 支持——同时也涵盖 Artix 桌面的 OpenRC 配置。

---

## ⚠️ 核心原则

> 暴露即是敌人。访问应默认私有，并经过深思熟虑后才授予。

端口转发把服务暴露给整个互联网，只能寄希望于登录页面足够牢靠。而 mesh VPN 反转了这个模型：除非设备**属于你且已通过认证**，否则任何东西都无法访问。

---

## 🧭 Tailscale 是什么

Tailscale 使用 **WireGuard**（内置于 Linux 内核、经过审计的现代 VPN 协议）在你的设备之间构建一个私有 mesh 网络（称为"tailnet"）。

- 每台设备获得一个稳定的私有 IP（`100.x.y.z`）
- 设备之间尽可能**直接互连**（点对点）
- 你的设备之间的流量全程端到端加密
- 可穿透 NAT 和防火墙，无需任何路由器配置

> 你的服务器、笔记本和手机就像处于同一个局域网中——无论身在何处。

---

## ⚖️ 坦诚的权衡

Tailscale 默认并非完全自托管。要清楚你在信任什么：

| 组件 | 状态 |
| :--- | :--- |
| 客户端（`tailscaled`） | 开源 |
| 加密（WireGuard） | 开源，端到端 |
| 协调服务器 | **由 Tailscale Inc. 托管（闭源）** |

协调服务器只交换公钥和连接元数据——它**无法解密你的流量**。但它确实能看到哪些设备存在，以及它们何时连接。

!!! tip "完全自主的选项：Headscale"

    [Headscale](https://github.com/juanfont/headscale) 是 Tailscale 协调服务器的
    开源、可自托管替代品。官方 Tailscale 客户端可以直接连接它。

    推荐路径：先使用托管版 Tailscale 熟悉这套模型，等你的 tailnet 稳定后再迁移到
    Headscale。无论哪种方式，本指南中客户端侧的命令完全相同。

---

## ⚙️ 前提条件

- 一台 Devuan 服务器（参见 [Devuan 服务器安装指南](../linux/devuan-server-install.md)）
- 一个免费的 Tailscale 账号 → <https://login.tailscale.com/start>
  - 登录通过身份提供商完成。为了把大型科技公司排除在外，请使用 **Passkey** 注册或 GitHub 账号，而不是 Google/Microsoft 登录。

免费套餐几乎涵盖 Tailscale 的全部功能，支持不限数量的设备和 6 个用户——对家庭实验室绰绰有余。

---

## 📦 在 Devuan 上安装（服务器）

### 1. 添加 Tailscale 软件源

Tailscale 的 `.deb` 软件源按 **Debian** 代号索引——与 [Docker 家庭实验室指南](docker-home-lab.md) 中使用的映射技巧相同。

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# Devuan 有自己的代号；Tailscale 的 Debian 源需要对应的 Debian 基础版本：
#   Devuan 5  "Daedalus"  → bookworm
#   Devuan 6  "Excalibur" → trixie
DEBIAN_CODENAME=trixie   # 请根据你的 Devuan 版本对应的 Debian 基础版本设置

curl -fsSL https://pkgs.tailscale.com/stable/debian/${DEBIAN_CODENAME}.noarmor.gpg | \
  sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg > /dev/null

echo \
  "deb [signed-by=/usr/share/keyrings/tailscale-archive-keyring.gpg] \
  https://pkgs.tailscale.com/stable/debian \
  ${DEBIAN_CODENAME} main" | \
  sudo tee /etc/apt/sources.list.d/tailscale.list > /dev/null

sudo apt update
sudo apt install -y tailscale
```

!!! info "会出现一个无害的报错"

    软件包的安装后脚本会尝试与 systemd 通信并因此报错。
    忽略它即可——二进制文件（`/usr/sbin/tailscaled` 和 `/usr/bin/tailscale`）
    已正确安装。下一步我们将自己提供 init 脚本。

---

### 2. 创建 sysvinit 脚本

Tailscale 不为 sysvinit 提供 init 脚本。自己创建一个：

```bash
sudo nano /etc/init.d/tailscaled
```

粘贴以下内容：

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          tailscaled
# Required-Start:    $local_fs $network $syslog
# Required-Stop:     $local_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Tailscale node agent
# Description:       Tailscale mesh VPN daemon
### END INIT INFO

PATH=/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/sbin/tailscaled
PIDFILE=/var/run/tailscaled.pid
NAME=tailscaled
DESC="Tailscale daemon"

DAEMON_ARGS="--state=/var/lib/tailscale/tailscaled.state --socket=/run/tailscale/tailscaled.sock --port=41641"

test -x $DAEMON || exit 0

. /lib/lsb/init-functions

case "$1" in
  start)
    log_daemon_msg "Starting $DESC" "$NAME"
    mkdir -p /run/tailscale /var/lib/tailscale
    start-stop-daemon --start --quiet --background \
      --make-pidfile --pidfile $PIDFILE \
      --exec $DAEMON -- $DAEMON_ARGS
    log_end_msg $?
    ;;
  stop)
    log_daemon_msg "Stopping $DESC" "$NAME"
    start-stop-daemon --stop --quiet --oknodo --retry 10 --pidfile $PIDFILE
    $DAEMON --cleanup
    rm -f $PIDFILE
    log_end_msg $?
    ;;
  restart|force-reload)
    $0 stop
    sleep 1
    $0 start
    ;;
  status)
    status_of_proc -p $PIDFILE $DAEMON $NAME
    ;;
  *)
    echo "Usage: /etc/init.d/$NAME {start|stop|restart|status}"
    exit 1
    ;;
esac

exit 0
```

赋予可执行权限并注册到默认运行级别：

```bash
sudo chmod +x /etc/init.d/tailscaled
sudo update-rc.d tailscaled defaults
sudo /etc/init.d/tailscaled start
```

确认它正在运行：

```bash
sudo /etc/init.d/tailscaled status
```

---

### 3. 加入你的 Tailnet

```bash
sudo tailscale up
```

在浏览器中打开输出的 URL，为这台机器完成认证。

> 这一步只需做一次。守护进程会把认证状态保存在
> `/var/lib/tailscale/` 中，每次开机自动重连。

确认节点已上线并记下其地址：

```bash
tailscale status
tailscale ip -4
```

---

## 🖥️ 在 Artix 上安装（台式机 / 笔记本）

Artix 按 init 系统分别打包 init 脚本：

```bash
sudo pacman -S tailscale tailscale-openrc
sudo rc-update add tailscaled default
sudo rc-service tailscaled start
sudo tailscale up
```

（如使用 runit 或 s6，请改为安装 `tailscale-runit` 或 `tailscale-s6`。）

这与 [Artix 桌面安装指南](../linux/artix-kde-openrc-install.md) 的安装后配置部分相对应。

---

## 📱 其他设备

在手机或平板上安装 Tailscale 应用并登录同一账号：

- Android → F-Droid 或 Play Store
- iOS → App Store

加入的每台设备现在都能通过 `100.x.y.z` 地址访问其他所有设备。

---

## 🌐 MagicDNS（用名称替代 IP）

在 Tailscale 管理控制台 → **DNS** 中启用 **MagicDNS**。

现在无需再记忆地址：

```bash
ssh user@100.xx.xx.xx
```

改用机器名：

```bash
ssh user@machinename
```

在管理控制台中重命名机器（**Machines** → `…` 菜单），保持名称简洁且可预测。

---

## 🔑 为服务器禁用密钥过期

默认情况下，每个节点的密钥约 6 个月后过期，节点会**从 tailnet 中掉线，直到你手动交互式重新认证**。

对笔记本来说无所谓，但对无头（headless）服务器来说很折磨人。

在管理控制台 → **Machines** → 你的服务器 → `…` → **Disable key expiry**。

---

## 🏠 子网路由器（访问整个局域网）

有些设备无法运行 Tailscale——NAS、打印机、IoT 设备、IPMI 接口等。**子网路由器**让一个 Tailscale 节点为你的整个局域网充当桥梁。

### 1. 启用 IP 转发（在 Devuan 服务器上）

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 2. 通告你的局域网子网

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24
```

（替换为你实际的局域网子网。）

### 3. 批准路由

管理控制台 → **Machines** → 你的服务器 → **Edit route settings** → 批准该子网。

现在你的手机在蜂窝网络下也能访问 `192.168.1.x` 的设备，就像在家一样——NAS 的 Web 界面、打印机，一切都能访问。

> 一个子网路由器，省去了在每台设备上安装 Tailscale 的麻烦。

---

## 🚪 出口节点（可选）

出口节点会将某台设备的**全部**互联网流量经由你的家庭网络转发——在酒店或机场 Wi-Fi 上非常有用。

在服务器上：

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24 --advertise-exit-node
```

在管理控制台批准它（与批准路由的位置相同）。然后在笔记本或手机上，处于不可信网络时选择该服务器作为出口节点。

!!! tip

    `tailscale up` 的参数不会叠加——每次运行都会替换之前的配置。
    请始终传入你希望生效的完整参数集。

---

## 🐳 访问家庭实验室服务

Tailscale 运行后，你的 [Docker 服务](docker-home-lab.md) 在**零暴露端口**的情况下即可访问：

```text
http://homemachine:8096    → Jellyfin
http://homemachine:2283    → Immich
http://homemachine:81      → Nginx Proxy Manager 管理界面
```

清晰的模式：

- Docker 服务绑定到主机（或仅绑定 Tailscale IP）
- Nginx Proxy Manager 负责路由内部主机名
- Tailscale 是**唯一**入口
- 路由器端口转发：**零**

> 只要端口没有转发，整个互联网的扫描器连它的存在都看不到。

---

## 🛡️ ACL（精确控制谁能访问什么）

默认情况下，tailnet 中的每台设备都能访问其他所有设备。对单人使用的家庭实验室来说可以接受——但随着规模扩大应逐步收紧。

在管理控制台 → **Access Controls** 中，ACL 以 JSON 定义。一个简单示例：给服务器打上标签，再把手机/笔记本限制到特定服务：

```json
{
  "tagOwners": {
    "tag:server": ["autogroup:admin"]
  },
  "acls": [
    {
      "action": "accept",
      "src": ["autogroup:member"],
      "dst": ["tag:server:8096,2283,443"]
    }
  ]
}
```

!!! warning "SSH 的 check 模式与自动化"

    Tailscale SSH 规则支持 `"action": "check"`，它会强制周期性地通过浏览器重新认证。
    对交互式的人工会话来说，这是很强的保护——但任何使用该规则的**自动化任务**
    （CI runner、部署脚本、cron 任务）都会静默挂起，永远等不来那个浏览器。

    `check` 只留给人用。对所有无人值守的任务，使用单独的 `accept` 规则——
    并严格限定到一个低权限用户。

---

## 🔐 安全最佳实践

- 用强双因素认证保护你的 Tailscale 登录；它现在是你整个基础设施的钥匙
- 淘汰硬件时，记得在管理控制台中移除旧设备
- 一旦 tailnet 中不止你一个人，就开始使用标签和 ACL
- 保持客户端更新：`apt upgrade` / `pacman -Syu` 就够了

---

## 🧰 故障排查

```bash
tailscale status        # 谁在线，以及连接方式（直连 vs 中继）
tailscale netcheck      # NAT 类型、最近的 DERP 中继、端口映射信息
tailscale ping machinename   # 验证到某节点的连通性与路径
```

常见问题：

- **`failed to connect to local tailscaled`** → 守护进程未运行。`sudo /etc/init.d/tailscaled start`（Devuan）或 `sudo rc-service tailscaled start`（Artix）。
- **连接显示 `relay` 而非 `direct`** → 流量正经由 Tailscale 的 DERP 中继转发。仍然是加密的，只是更慢。放行出站 UDP 41641 通常能恢复直连。
- **子网路由不生效** → 路由未在管理控制台批准，或 IP 转发未启用。两者都要检查。
- **节点从 tailnet 消失** → 密钥过期。用 `sudo tailscale up` 重新认证，然后禁用密钥过期以免再次发生。

---

## 🧠 最后的思考

每一个转发的端口，都是向整个互联网发出的常驻邀请函。

私有 mesh 网络颠覆了这个模型：

> 什么都不暴露。一切只有你——且仅有你——可以访问。
