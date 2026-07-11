---
description: >-
  在 Devuan 服务器上以正确方式挂载 NAS 存储——NFS 与 SMB 的取舍、无 systemd 也能跨重启存活的 fstab 条目、Docker 启动顺序的陷阱，以及会毁掉家庭实验室数据库的"SQLite 跑在 NFS 上"错误。
---

<!--
Source: homelab/nas-mounting.md
Last translated: 2026-07
-->

# 🗂️ 挂载 NAS 存储（Devuan）

NAS 负责大容量存储；服务器负责运行服务。本指南教你**可靠地**把两者连起来：挂载能跨重启存活、在 sysvinit 下行为正确、并且不会悄无声息地把空目录喂给你的 Docker 容器。

---

## ⚠️ 核心原则

> 存储应该是无聊的。一个需要你惦记的挂载，就是一个终将坑你的挂载。

---

## 🧭 NFS 还是 SMB？

两者都能用。根据"谁和谁通信"来选择：

| | NFS | SMB / CIFS |
| :--- | :--- | :--- |
| 最适合 | Linux ↔ Linux / NAS | 混合网络（Windows、打印机、扫描仪） |
| 性能 | 更快、开销更低 | 略重 |
| 权限 | Unix UID/GID，原生支持 | 挂载时映射 |
| 认证模型 | 基于主机/IP（v3/v4） | 用户名 + 密码 |
| 结论 | **Linux 家庭实验室的默认选择** | NFS 不可用或数据由 Windows 共享时使用 |

本指南两者都涉及——以 NFS 为主线。

---

## 🖥️ NAS 侧的准备

在 NAS 上（QNAP、Synology、TrueNAS——界面各异，概念相同）：

1. 创建或选定共享文件夹（例如 `media`、`backups`）
2. 启用 **NFS 服务**并为该共享添加一条 NFS 规则：
    - 允许的客户端 → 你服务器的 IP（若通过 tailnet 挂载则用其 Tailscale IP）
    - 访问权限 → 读/写
    - Squash → 仅当 Docker 容器必须以 root 写入时才选 *no root squash*；否则映射到特定的 UID/GID
3. 使用 SMB 时 → 为服务器创建一个专用的低权限 NAS 用户；绝不要用 NAS 管理员账号挂载

!!! tip "每个用途一个共享"

    为 `media`、`backups`、`archive` 建立各自独立的共享，意味着各自独立的权限、
    各自独立的 NFS 规则，并且卸载其中一个不会影响其他。抵制住建一个包罗万象的
    巨型"万物共享"的诱惑。

---

## 📦 安装客户端软件包（Devuan）

```bash
# NFS
sudo apt update
sudo apt install -y nfs-common

# SMB（仅在需要时安装）
sudo apt install -y cifs-utils
```

---

## 🧪 先做测试挂载（永远如此）

绝不要直奔 `fstab`。先交互式地证明挂载可用：

```bash
# 查看 NAS 导出了哪些共享
showmount -e 192.168.1.xx

# 创建挂载点并测试
sudo mkdir -p /mnt/nas/media
sudo mount -t nfs -o vers=4.1 192.168.1.xx:/media /mnt/nas/media

# 验证：列目录、写入、再读回
ls /mnt/nas/media
touch /mnt/nas/media/.write-test && rm /mnt/nas/media/.write-test
```

如果写入测试失败，**现在**就修复权限（见"故障排查"）——写进 `fstab` 的坏挂载，到开机时再调试可就没这么轻松了。

在设为永久挂载之前先卸载：

```bash
sudo umount /mnt/nas/media
```

---

## 📌 通过 /etc/fstab 永久挂载

### NFS

```bash
sudo nano /etc/fstab
```

每个共享添加一行：

```text
192.168.1.xx:/media    /mnt/nas/media    nfs    rw,hard,vers=4.1,_netdev,nofail    0    0
192.168.1.xx:/backups  /mnt/nas/backups  nfs    rw,hard,vers=4.1,_netdev,nofail    0    0
```

各选项的作用：

- `hard` → NAS 掉线时，I/O 会**等待**它恢复，而不是悄悄返回错误、写坏数据。对你在乎的数据来说这是正确选择。
- `vers=4.1` → 锁定协议版本；NAS 固件更新后不会有协商上的意外
- `_netdev` → 告诉 init 脚本此挂载依赖网络——**在 sysvinit 上必不可少**
- `nofail` → NAS 没开机也不会卡死服务器的整个启动过程

应用并验证：

```bash
sudo mount -a
df -h | grep nas
```

### SMB

凭据绝不能写进 `fstab`（它对所有人可读）。使用凭据文件：

```bash
sudo nano /root/.smb-credentials
```

```text
username=serversvc
password=你的强密码
```

```bash
sudo chmod 600 /root/.smb-credentials
```

然后在 `/etc/fstab` 中：

```text
//192.168.1.xx/media  /mnt/nas/media  cifs  credentials=/root/.smb-credentials,uid=1000,gid=1000,vers=3.0,_netdev,nofail  0  0
```

`uid=1000,gid=1000` 会把所有文件归属到你的用户——请将其设置为与你的 [Docker 栈](docker-home-lab.md)运行所用的 `PUID`/`PGID` 一致。

---

## 🔁 没有 systemd 时的启动行为

在 Devuan 上，sysvinit 的启动序列**正是靠 `_netdev`** 正确处理这一切：`mountnfs` 阶段在网络就绪后运行，挂载 fstab 中所有标记为依赖网络的条目。不需要 unit，不需要 automount 配置——这套有三十年历史的机制就是好用。

下次重启后验证：

```bash
df -h | grep nas
```

!!! tip "用 autofs 按需挂载（可选）"

    如果 NAS 有时是关机的，`autofs` 只在访问时才挂载共享，空闲后自动释放——
    完全没有启动依赖：

    ```
    sudo apt install -y autofs
    ```

    然后在 `/etc/auto.master.d/` 中映射你的共享。对于常开的 NAS，
    朴素的 fstab 更简单也更好；只有当 NAS 真的时开时关时才考虑 autofs。

---

## 🐳 Docker 的陷阱

这是每个人都会被咬一次的故障模式：

**如果 Docker 在 NAS 挂载完成之前启动，容器绑定挂载到的就是一个空目录**——而有些应用会兴高采烈地在里面初始化一个全新的空库。Jellyfin 扫描到空的媒体文件夹会删掉自己的元数据；备份任务看到空的源目录会"成功地"备份一个寂寞。

防御手段，按价值排序：

**1. 让启动顺序发挥作用。** 在 sysvinit 上，`mountnfs` 先于 Docker 的 init 脚本运行——只要设置了 `_netdev`，*在线的* NAS 会在容器启动前挂载完毕。风险在于 NAS 响应缓慢或离线（这正是 `nofail` 有意允许的情况）。

**2. 为依赖挂载的栈加一道保险。** 在 Docker 的 init 序列中加入检查——创建 `/etc/init.d/wait-for-nas`：

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          wait-for-nas
# Required-Start:    $network $remote_fs
# Required-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:
# X-Start-Before:    docker
# Short-Description: Delay boot until NAS shares are mounted
### END INIT INFO

case "$1" in
  start)
    for i in $(seq 1 30); do
      mountpoint -q /mnt/nas/media && exit 0
      sleep 2
    done
    echo "wait-for-nas: NAS not mounted after 60s, continuing anyway"
    ;;
  *) ;;
esac
exit 0
```

```bash
sudo chmod +x /etc/init.d/wait-for-nas
sudo update-rc.d wait-for-nas defaults
```

**3. 让"空目录"变得可检测。** 在 NAS 共享上放一个标记文件（在挂载状态下执行 `touch /mnt/nas/media/.nas-mounted`）。之后任何脚本——尤其是备份脚本——都可以拒绝对未挂载的路径动手：

```bash
[ -f /mnt/nas/media/.nas-mounted ] || { echo "NAS 未挂载，中止"; exit 1; }
```

> 标记文件这一招零成本，但它拯救过的家庭实验室数据，比本指南里任何其他三行都多。

---

## 🚫 不要做的事

!!! danger "绝不要让 SQLite 数据库跑在 NFS 或 SMB 上"

    Arr 全家桶（Sonarr、Radarr、Prowlarr……）、Jellyfin 以及许多家庭实验室应用
    都使用 **SQLite**，它依赖文件锁，而网络文件系统对文件锁的实现并不可靠。
    放在 NAS 挂载上的数据库一定会损坏——问题不是*会不会*，而是*什么时候*。

    [Docker 指南](docker-home-lab.md)中的规则已经解决了这个问题：

    - 应用数据和数据库 → `/opt/data/<stack>`，放在**本地磁盘**
    - 大容量媒体、文档、备份 → NAS 挂载

- 不要用 NAS 管理员账号挂载——每个共享使用专用的低权限用户
- 不要对任何有写入的场景使用 `soft` 挂载——无声的部分写入正是文件损坏的来源
- 不要把凭据写进 `fstab`——用凭据文件，`chmod 600`
- 不要跳过交互式测试挂载——fstab 是用来*记录*一个已验证可用的挂载的地方，不是用来*发现*一个坏挂载的地方

---

## 🧰 故障排查

- **`access denied by server`** → NAS 上的 NFS 规则没有包含你服务器的 IP，或导出路径写错了。用 `showmount -e <nas-ip>` 重新核对。
- **文件归属为 `nobody:nogroup`** → NFSv4 ID 映射不匹配。家庭局域网里最简单的修法：让 NAS 共享上的 UID/GID 与服务器用户一致（1000:1000），或将共享的 squash 选项设置为把所有访问映射到该 UID。
- **`Stale file handle`** → 挂载期间 NAS 侧的导出发生了变化。`sudo umount -l /mnt/nas/media && sudo mount -a`。
- **命令在挂载点上永久挂起** → NAS 宕机而挂载是 `hard`（按设计工作——数据安全优先于响应速度）。恢复 NAS，或用 `sudo umount -f -l` 强制释放。
- **SMB 传输缓慢** → 确认挂载选项中是 `vers=3.0` 或更高；SMB1 协商仍潜伏在老 NAS 的默认设置中，既慢又不安全。

---

## 🧠 最后的思考

一台无法可靠访问的 NAS，不过是一台非常昂贵的取暖器。

把挂载做得无聊——测试过、版本锁定、顺序正确、有保险措施——家庭实验室的其余部分就能建立在一个永远不用操心的存储之上。

> 无聊的存储，是其他一切赖以站立的地基。
