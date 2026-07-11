---
description: >-
  RebelRx 的完整技术栈——硬件、操作系统、自托管服务，以及每天真正在用的隐私工具。
---

<!--
Source: privacy/my-setup.md
Last translated: 2026-07
-->

# 🧪 RebelRx 的隐私配置

这是我自己真实使用的隐私优先技术栈，为**掌控、性能与可用性**而设计。

我每天都在使用这套配置，并且会随着更新更好的服务出现，持续更新这些章节。

---

## 🧭 理念

这套配置围绕以下原则构建：

- **所有权** → 数据始终在你的掌控之下  
- **实用性** → 工具必须在日常中真正好用  
- **可扩展性** → 能随需求增长  
- **安全性** → 不做多余的暴露  

> 💡 这不是一套"极限隐私"配置，而是一套**平衡、可用的系统**。

---

## 🖥️ 硬件概览

我的配置由**自托管基础设施、专用设备和个人设备**组成，各司其职。

---

## 🧱 核心基础设施

### Minisforum MS-A2（主服务器）

- **系统：**Devuan（裸机）
- **角色：**核心 Docker 主机

> 💡 这是我整个系统的中枢

👉 看看 Minisforum Work Station Mini 系列：<https://www.minisforum.com/collections/station-mini-series>

---

### QNAP TS-h1277AXU-RP（NAS）

- **角色：**大容量存储 + 备份
- 存放：
  - 媒体库
  - Nextcloud 数据
  - 备份
  - 归档（ROM、文档等）

> 💡 把计算（服务器）与存储（NAS）分开，能提升灵活性与韧性

👉 看看 QNAP 的 NAS 解决方案：<https://www.qnap.com/en-us/product>

---

## 💻 个人设备

### 自组 Ryzen 9 PC（Windows 11）

- **角色：**游戏 + 仅限 Windows 的应用
- 用于：
  - 高性能工作负载
  - 兼容非 Linux 软件

👉 看看电脑零售商 Micro Center，查找离你最近的门店：<https://www.microcenter.com/>

---

### Framework Desktop（Artix Linux）

- **角色：**第二工作站
- 用于：
  - 日常生产力
  - Linux 优先的工作流

👉 看看 Framework Desktop：<https://frame.work/marketplace/desktops>

---

### Framework 13 笔记本（Artix Linux）

- **角色：**旅行 + 开发机
- 用于：
  - 远程访问（通过 Tailscale）
  - 管理家中的基础设施
  - 轻量生产力

👉 看看 Framework 笔记本系列：<https://frame.work/marketplace/laptops>

---

## 🎮 游戏与模拟

### Raspberry Pi 5（8GB）

- **系统：**Batocera
- **附加：**MiSTer FPGA
- **角色：**复古游戏 / 模拟器

👉 看看科技供应商 Vilros 的 Raspberry Pi 主板与配件：<https://vilros.com/>

---

## 🏠 专用设备

### Beelink Mini S13

- **角色：**智能家居自动化
- **系统：**Home Assistant OS

👉 看看 Beelink 迷你主机系列：<https://www.bee-link.com/collections/product>

---

### Umbrel Home

- **角色：**比特币节点
- 运行：
  - BTC 全节点
  - Lightning

👉 看看 Umbrel 设备系列：<https://umbrel.com/>

---

### Intel NUC 13

- **角色：**音频服务器
- **系统：**Roon ROCK

👉 看看 B&H 的 NUC 及专业科技装备：<https://www.bhphotovideo.com/>

---

## 🧠 设计哲学

每台设备都有**清晰、单一的职责**：

- 服务器 → 计算（Docker 工作负载）
- NAS → 存储
- 客户端 → 交互（台式机/笔记本）
- 专用设备 → 特定任务

> 💡 这种分离让系统：
>
> - 更容易维护  
> - 更有韧性  
> - 更容易扩展  

---

## ✅ 为什么这套配置适合我的需求

- 没有牵一发而动全身的单点故障  
- 职责划分清晰  
- 每台设备的性能各自优化  
- 可以灵活升级单个组件  

---

## 🚀 关于硬件的最后一点

不过，你并不需要这么多硬件才能开始。

这套配置是随时间演化而来的——  
从小处着手，随需求增长再扩展。

---

## 🧱 核心架构

- **基于 Docker 的部署**
- **NAS 支撑的存储**
- **反向代理（Nginx Proxy Manager）**
- **通过 Tailscale 私密访问（无端口转发）**

---

## ☁️ 数据与生产力

### Nextcloud AIO

- 文件
- 日历（CalDAV）
- 联系人（CardDAV）
- NAS 支撑的存储

### 云存储（托管）

- pCloud.com

---

## 📧 邮箱

- Proton Mail

---

## 📸 照片

### Immich

- Google 相册的替代品
- 快速、现代的界面
- 完全自托管

---

## 📝 办公

### ONLYOFFICE

- Microsoft Office 365 的替代品
- 完整生产力套件（文档、表格、演示、PDF 和表单编辑器）
- 免费且开源

---

## 📄 PDF 与文档

- Sumatra PDF（轻量阅读器）
- BentoPDF（PDF 工具集）
- Paperless-ngx（文档归档）

---

## 📝 笔记与知识

### Joplin

- 基于 Markdown
- 跨平台
- 通过 Nextcloud 同步

### Paperless-ngx

- 文档管理系统
- OCR + 标签
- 取代成堆的纸质文件

---

## 🔐 安全与身份

### 密码管理器

- Proton Pass（托管方案）

### 2FA

- 所有服务全部启用
- 支持时优先使用 Passkey

---

## 🌍 网络与隐私层

### DNS 拦截

- AdGuard Home
- 全网范围的广告 + 跟踪器拦截

### VPN

- Mullvad（隐私优先的外部 VPN）

### 私密访问

- Tailscale
- 安全远程访问各项服务
- 零暴露端口

---

## 🌐 浏览器

- Brave
  - 内置广告/跟踪器拦截
  - 几乎不需要额外扩展

---

## 🎥 媒体与娱乐

### Jellyfin

- 自托管流媒体
- 取代 Netflix / HBO / 各类流媒体服务

### Arr 全家桶

- Sonarr
- Radarr
- Prowlarr
- 自动化媒体管理

---

## 📚 图书与音频

### Calibre-Web / Kavita

- 电子书库

### Audiobookshelf

- 有声书 + 播客
- 完全自托管

---

## 💰 财务

### Actual Budget

- 自托管预算工具
- Mint/YNAB 的隐私优先替代品

---

## 🧰 开发与基础设施

### Git

- Forgejo（自托管 Git 服务）

### 编辑器

- VSCodium（无遥测的 VS Code）

---

## 🌐 网络工具

- LibreSpeed
- Speedtest-tracker

自托管的网络性能测试，没有任何追踪。

---

## 🔁 这套配置替换了什么

| 大型科技公司 | 替代品 |
|---------|------------|
| Google Drive | Nextcloud |
| Google 相册 | Immich |
| Google 日历 | Nextcloud |
| Google 通讯录 | Nextcloud |
| Gmail | Proton Mail / Tuta |
| Chrome 密码 | Proton Pass |
| Chrome | Brave |
| Google 文档（部分） | Nextcloud + Joplin |
| Netflix / HBO | Jellyfin + Arr 全家桶 |
| Kindle / Audible | Calibre-Web / Audiobookshelf |
| Adobe Acrobat | Sumatra PDF / BentoPDF |
| Mint / YNAB | Actual Budget |
| GitHub | Forgejo |
| 运营商 DNS | AdGuard Home / Pi-hole |
| Speedtest.net | LibreSpeed / Speedtest-tracker |

---

## 🧠 设计原则

### 1. 尽可能本地优先

数据存放在：

- 你的服务器上
- 你的 NAS 上

---

### 2. 有价值时才自托管

不是所有东西都需要自托管。

平衡之道：

- 自托管 → 核心数据（文件、照片、密码）
- 托管 → 图省事的部分（如果偏好，邮箱可以托管）

---

### 3. 默认安全

- 不暴露任何端口
- 仅通过 Tailscale 访问
- 内部路由走反向代理

---

### 4. 保持可维护

- 基于 Docker 的服务
- 清晰的目录结构
- 配置纳入版本控制（Forgejo）

---

## ⚖️ 为什么这套配置行得通

- 对数据的高度掌控  
- 极低的持续成本  
- 可扩展的架构  
- 安全的远程访问  
- 在所有设备上通用  

---

## 🚀 关于基础设施的最后思考

这不是唯一的做法，也绝对算不上完美！

但它是一套**久经实战的真实配置**，在以下三者间取得了平衡：

- 隐私  
- 可用性  
- 可靠性  

> 🧠 目标不是完美，而是**没有摩擦的掌控**。
