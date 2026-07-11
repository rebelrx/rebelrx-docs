---
description: >-
  在裸机 Devuan 上构建干净、可复现的 Docker Compose 家庭实验室（homelab）——没有 systemd，也没有虚拟化管理程序的开销。
---

<!--
Source: homelab/docker-home-lab.md
Last translated: 2026-07
-->

# 🐳 Docker 家庭实验室（Devuan + Compose）

这是一份实用指南，教你如何在裸机上使用 Devuan 和 Docker Compose 运行一个**干净、可复现的 Docker 家庭实验室**。

大多数 homelab 指南默认使用 Debian、Ubuntu 或其他基于 systemd 的发行版。本指南反其道而行之：使用 **Devuan**——即*不带* systemd 的 Debian——获得一个更精简、init 更简单、完全由你掌控的基础系统。Compose 栈来自 [RebelRx homelab 仓库](https://github.com/rebelrx/rebelrx-homelab)，可以在任何 Docker 主机上以相同方式运行；这里我们按 Devuan 的方式来搭建。

---

## 🔥 为什么选 Docker（而不是 Proxmox）

你*可以*用 Proxmox，很多人也确实在用。

但对大多数家庭实验室来说，它带来了不必要的复杂性。

### ❌ Proxmox 的问题

- 虚拟机开销（浪费 CPU 和内存）
- 需要调试的层级更多
- 备份更复杂
- 网络配置变得比应有的更困难
- 助长碎片化（许多小虚拟机，而非统一的系统）

---

### ✅ 为什么更推荐裸机 Docker

- **轻量** → 没有虚拟化开销  
- **简单** → 一个操作系统，一套需要管理的系统  
- **部署迅速** → 几秒钟就能启动服务  
- **可复现** → 一切都在 Compose 中定义  
- **可迁移** → 用 Git 就能搬走整个技术栈  

> 💡 如果你不运行企业级多租户工作负载，你很可能并不需要虚拟化管理程序。

---

## ⚙️ 前提条件

- 一台 PC 或笔记本电脑
- 互联网连接
- Devuan

如果你没有专用 PC，可以购买这类自带内存和 SSD 的廉价迷你主机，开箱即用：[Beelink Mini S12](https://www.amazon.com/dp/B0BW8JSQCH)

如果尚未安装 Devuan，请参阅 [Devuan Linux 服务器安装指南](../linux/devuan-server-install.md)

---

## 📦 安装依赖

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg git
```

---

## 🔑 添加 Docker 软件源

Docker 的 `.deb` 软件源按 **Debian** 代号索引——而 Devuan 使用自己的代号
（`daedalus`、`excalibur`……），Docker 的软件源无法识别。
需要将其映射到你的 Devuan 版本所基于的 Debian 基础版本。

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/debian/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Devuan 有自己的代号；Docker 的 Debian 源需要对应的 Debian 基础版本：
#   Devuan 5  "Daedalus"  → bookworm
#   Devuan 6  "Excalibur" → trixie
DEBIAN_CODENAME=trixie   # 请根据你的 Devuan 版本对应的 Debian 基础版本设置

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian \
  $DEBIAN_CODENAME stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## 🐳 安装 Docker + Compose

```bash
sudo apt update

sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

---

## 🚀 启用并启动 Docker（SysVinit）

```bash
sudo service docker start
sudo update-rc.d docker defaults
```

验证：

```bash
docker --version
docker compose version
```

---

## 🔐 将用户加入 docker 组

```bash
sudo usermod -aG docker $USER
newgrp docker
```

测试：

```bash
docker run hello-world
```

---

## 📁 目录结构

将**配置与数据放在各自独立的目录树中**——这是最重要的一条布局决策，
仓库中的每个栈都遵循这一原则：

```bash
/opt/stacks/<stack>/     # Docker Compose 栈（compose.yaml、.env、README）
/opt/data/<stack>/       # 应用持久化数据（绝不提交到 Git）
```

示例：

```bash
/opt/stacks/
├── npm/
│   ├── compose.yaml
│   ├── .env
│   └── README.md
├── plex/
└── ...

/opt/data/
├── npm/
├── plex/
└── ...
```

### 为什么这很重要

- **配置与数据**分离
- 备份更简单（对 `/opt/data` 做快照；compose 目录树存放在 Git 中）
- Git 仓库更干净（运行时数据永不纳入版本控制）
- 避免 Docker 环境"杂乱蔓延"

> 💡 `/opt/stacks` 同时是 Dockge 的默认 `DOCKGE_STACKS_DIR`，
> 因此栈管理器无需额外配置即可自动识别所有内容。

---

## 🚀 克隆 RebelRx Homelab 仓库

```bash
cd ~
git clone https://github.com/rebelrx/rebelrx-homelab.git
```

> 该仓库提供**在生产环境中实际使用的 Compose 模板**，
> 每个栈一个目录，每个目录都有内容完整的 `README.md`。

要部署某个栈，将其复制到 `/opt/stacks/` 并填写它的 `.env`：

```bash
sudo mkdir -p /opt/stacks
sudo cp -a ~/rebelrx-homelab/stacks/npm /opt/stacks/
```

---

## 📦 模板结构

仓库内部：

```bash
rebelrx-homelab/
└── stacks/
   ├── arr/
   ├── audiobooks/
   ├── authentik/
   └── ...
```

每个栈至少包含：

- `compose.yaml`
- `.env.example`
- `README.md`（服务、环境变量、端口、部署、备份）

某些栈附带额外文件（例如 `paperless` 有 `docker-compose.env.example`，`monitor` 有 `prometheus.yml.example`）——各栈的 README 中都有说明。

---

## 🧠 核心概念（请先阅读）

### 1. Compose 优先的思维方式

一切都用 YAML 定义。

- 不在图形界面里到处点击
- 不手动创建容器
- Git = 唯一可信来源

---

### 2. `.env` 文件

每个栈都使用环境变量：

```env
PUID=1000
PGID=1000
TZ=America/New_York
```

**为什么这很重要：**

- 权限保持一致
- 配置可迁移
- 便于覆盖设置

机密信息（密码、API 密钥）同样保存在 `.env` 中——该文件已被 **Git 忽略**。
只有机密留空的 `.env.example` 模板才会提交到仓库。

---

### 3. 卷映射

```yaml
volumes:
  - /opt/data/plex:/config
```

这样可以确保：

- 数据在容器重启后依然保留
- 备份简单（一切都在 `/opt/data` 之下）
- 对存储拥有完全控制权

---

### 4. 端口绑定

```yaml
ports:
  - 8080:8080              # 发布到所有网络接口——局域网内可访问
  # - 127.0.0.1:8080:8080  # 仅限 loopback——前面放置反向代理
```

**仓库的处理方式：**

- Web 界面默认发布到**所有网络接口**，这样在你的局域网内开箱即用。
- 在端口映射前加上 `127.0.0.1:` 前缀，可让服务**仅监听 loopback**，
  只能通过反向代理访问。
- **内部服务**（数据库、Redis/Valkey、消息代理）**完全不发布任何主机端口**——
  它们只能在栈的内部网络中访问。

---

## 🌐 反向代理（Nginx Proxy Manager）

推荐方案——使用仓库中的 `npm` 栈：

- 运行 **Nginx Proxy Manager（NPM）**
- 通过子域名暴露服务
- 自动处理 SSL

各服务连接到共享的 `proxy_net` 网络；NPM 通过容器名访问每个服务，
因此无论是否发布了主机端口，反向代理都能正常工作。

示例：

```text
https://plex.yourdomain.com
```

优点：

- 不必来回折腾端口
- URL 干净整洁
- 集中的访问控制

---

## ▶️ 运行第一个栈

先从反向代理开始，这样后续的一切服务都有了"依托"：

```bash
cd /opt/stacks/npm

cp .env.example .env
nano .env
```

然后：

```bash
docker compose up -d
```

---

## 🔄 更新容器

```bash
docker compose pull
docker compose up -d
```

## 🔄 停止与启动容器

```bash
docker compose down
docker compose up -d
```

## 🔄 重启容器

```bash
docker compose restart [服务名]
```

💡 下载 DevOps Cycle 出品的《Ultimate Docker Compose Cheat Sheet》：[Docker Compose 速查表](https://devopscycle.com/pdfs/the-ultimate-docker-compose-cheat-sheet.pdf)

---

## 🧼 清理

删除未使用的资源：

```bash
docker system prune -a
```

---

## ⚠️ 常见陷阱

### ❌ 权限问题

Compose 文件可以归你的用户所有；**数据**目录通常归容器的
`PUID:PGID` 所有（一般是 `1000:1000`）：

```bash
sudo chown -R $USER:$USER /opt/stacks/<stack>
sudo chown -R 1000:1000 /opt/data/<stack>   # 与该栈的 PUID/PGID 保持一致
```

---

### ❌ 端口已被占用

```bash
ss -tulnp | grep :端口号
```

---

### ❌ 容器无法更新

```bash
docker compose pull
```

---

### ❌ 直接修改运行中的容器

不要这样做。

修改 **Compose 文件**，然后重新部署。

---

## 🧠 最后的思考

这套方案建立在几条原则之上：

- **保持简单**
- **一切皆代码**
- **掌控自己的基础设施**
- **避免不必要的层级**

只需要 Docker + Compose + 自律。

---

## 🔗 建议的后续步骤

- 从 RebelRx 仓库添加更多栈
- 使用 `npm` 栈搭建反向代理（Nginx Proxy Manager）
- 使用 `backup` 栈实现备份（Kopia）
- 使用 `dockge` 栈管理 Docker Compose 栈（Dockge）
- 使用 `filebrowser` 栈添加文件浏览器（Filebrowser Quantum）

---

## ⚠️ 免责声明

本指南仅供学习参考。

你需要自行负责系统的安全与维护。
