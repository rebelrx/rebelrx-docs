---
description: >-
  面向私有家庭实验室的 Nginx Proxy Manager——为每个服务提供干净的 HTTPS 域名，通过 DNS-01 在零暴露端口下签发通配符证书，并用 AdGuard Home 提供本地 DNS。
---

<!--
Source: homelab/nginx-proxy-manager.md
Last translated: 2026-07
-->

# 🔀 Nginx Proxy Manager（为一切提供私有 HTTPS）

端口和 IP 地址无法规模化。`http://192.168.1.20:8096` 对一个服务还行；到第十个服务时就成了记忆力游戏，每个浏览器都在喊"不安全"，而且没有任何东西启用了 TLS。

反向代理一次性解决所有问题：**一个入口、真正的 HTTPS、干净的域名**——`https://jellyfin.home.example.com`——且无需向互联网暴露任何端口。

---

## ⚠️ 核心原则

> 每个服务都得到一个域名和一张证书。地址栏里不该出现任何端口号。

---

## 🧭 各部分如何协作

```text
浏览器（在 tailnet 内）
   │
   ▼  "jellyfin.home.example.com?"
本地 DNS（AdGuard Home）  →  返回你服务器的 IP
   │
   ▼  HTTPS :443
Nginx Proxy Manager  →  终结 TLS，按主机名路由
   │
   ▼  Docker 网络
Jellyfin 容器 :8096
```

三个组件：

1. **一个你真正拥有的域名**（每年几美元）——只用于域名解析和证书；不在上面公开托管任何内容
2. **本地 DNS**，在你的网络内部为它提供解析——本指南使用 AdGuard Home
3. **NPM**，负责终结 TLS 并按主机名路由

> 正是因为域名是真实的，才可能签发真实的证书；没有自签名警告，也不必在每台设备上导入 CA。

---

## 📦 NPM 栈

按照[家庭实验室的目录结构](docker-home-lab.md)，`/opt/stacks/npm/compose.yaml`：

```yaml
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - "80:80"            # HTTP（重定向到 HTTPS）
      - "443:443"          # HTTPS
      - "127.0.0.1:81:81"  # 管理界面——仅限 localhost，通过 Tailscale/SSH 访问
    volumes:
      - /opt/data/npm/data:/data
      - /opt/data/npm/letsencrypt:/etc/letsencrypt
    networks:
      - proxy

networks:
  proxy:
    external: true
```

先创建一次共享网络，然后启动：

```bash
docker network create proxy
cd /opt/stacks/npm
docker compose up -d
```

!!! warning "把管理界面绑定到 localhost"

    `127.0.0.1:81:81` 这条绑定意味着管理面板从网络上无法访问——
    即使在你的局域网内也不行。请通过 Tailscale 或 SSH 隧道访问它。
    控制着你全部路由的面板，应该是最难触及的东西，而不是最容易的。

### 首次登录

（通过 Tailscale）访问 `http://server:81`，使用 NPM 的默认凭据登录：

```text
Email:    admin@example.com
Password: changeme
```

它会立即强制你修改凭据。请使用密码管理器。

---

## 🕸️ 共享代理网络

NPM 通过**共享 Docker 网络按容器名**路由到各个容器——服务本身无需发布任何端口。

把 `proxy` 网络加到所有 NPM 需要访问的栈中：

```yaml
services:
  jellyfin:
    # ...现有配置...
    networks:
      - default
      - proxy

networks:
  proxy:
    external: true
```

现在 NPM 可以直接访问 `jellyfin:8096`，你也可以**彻底移除服务发布的端口**——代理成为唯一的门。

> 发布的端口越少 = 攻击面越小 = 需要操心的事越少。代理网络是家庭实验室的走廊；只有 NPM 握着大门的钥匙。

---

## 🔐 零暴露端口的通配符证书（DNS-01）

Let's Encrypt 的常规流程（HTTP-01）要求向互联网开放 80 端口——恰恰是这套方案拒绝做的事。**DNS-01 验证**改为通过一条 DNS 记录证明域名所有权，因此可以在所有端口关闭的情况下工作。额外好处：它是唯一能签发**通配符**证书的验证方式。

### 1. 创建 DNS API 令牌

在你的 DNS 提供商处（此处以 Cloudflare 为例；NPM 支持数十家）：

- **My Profile → API Tokens → Create Token**
- 模板：*Edit zone DNS*
- 将权限范围限定为**仅**那一个区域（例如 `example.com`）

### 2. 在 NPM 中申请证书

**SSL Certificates → Add SSL Certificate → Let's Encrypt**

- 域名：`*.home.example.com` 和 `home.example.com`
- ✅ *Use a DNS Challenge* → 提供商：Cloudflare → 粘贴令牌
- 同意并保存。签发需要一两分钟。

现在，一张通配符证书就覆盖了你未来在 `*.home.example.com` 下添加的所有服务——无需逐服务签发，续期也是自动的。

!!! tip "让私有域名远离公共 DNS"

    有了通配符证书，各个具体主机名（`jellyfin.`、`paperless.`、`vault.`）
    不会出现在任何公开场合——既不在你的 DNS 区域里，也不在证书透明度日志中
    （后者记录每一张签发的证书，任何人都可以搜索）。为
    `vault.home.example.com` 单独签发证书，等于向全世界宣布你运行着一个
    密码保险库。而通配符证书什么都不会泄露。

---

## 🧭 用 AdGuard Home 提供本地 DNS

互联网根本不知道 `jellyfin.home.example.com` 是什么——也只有你的网络应该知道。在 AdGuard Home 中：

**Filters → DNS rewrites → Add DNS rewrite**

```text
Domain: *.home.example.com
Answer: 192.168.1.10        # 你服务器的局域网 IP
```

一条通配符重写规则就覆盖了当前和未来的所有服务。

!!! tip "让它在 Tailscale 上同样可用"

    把重写规则指向服务器的 **Tailscale IP**（`100.x.y.z`）而非局域网 IP，
    并把 AdGuard Home 设为你 tailnet 的 DNS 服务器（Tailscale 管理控制台 →
    DNS → 添加你的 AdGuard 实例，启用 *Override local DNS*）。现在
    `https://jellyfin.home.example.com` 在家里和通过 [tailnet](tailscale.md)
    的表现完全一致——处处同一个 URL，还附带广告拦截。

---

## 🔀 创建 Proxy Host

到了逐服务收获成果的环节。**Hosts → Proxy Hosts → Add Proxy Host**：

**Details 选项卡**

```text
Domain Names:         jellyfin.home.example.com
Scheme:               http
Forward Hostname/IP:  jellyfin        ← 代理网络上的容器名
Forward Port:         8096
Websockets Support:   ✅
Block Common Exploits: ✅
```

**SSL 选项卡**

```text
SSL Certificate:  *.home.example.com   ← 前面申请的通配符证书
Force SSL:        ✅
HTTP/2 Support:   ✅
```

保存。`https://jellyfin.home.example.com` 就此上线——连小锁图标都有了。

每个服务重复一次，每次只要三十秒：

| 服务 | 转发到 |
| :--- | :--- |
| Immich | `immich-server:2283` |
| Paperless | `paperless-webserver:8000` |
| Forgejo | `forgejo:3000` |
| Uptime Kuma | `uptime-kuma:3001` |

!!! tip "Websockets：直接保持开启"

    家庭实验室里一半的应用（Jellyfin、Uptime Kuma、Dozzle、Home Assistant，
    任何界面会实时刷新的东西）都需要 websockets，而它被关闭时的症状又模糊得
    令人抓狂——页面能加载，但什么都不更新。默认开启它，从此再也不用调试这个问题。

---

## 🛡️ 加固清单

- 管理界面绑定到 `127.0.0.1`，仅通过 [Tailscale](tailscale.md) 访问
- 每个 proxy host 都启用 **Force SSL**；对证书自动续期有信心后再加上 **HSTS**
- 逐主机启用 **Block Common Exploits**
- 默认站点（Settings → Default Site）设为 404——打到代理上的未知主机名什么也探不到
- 服务的 proxy host 一旦正常工作，就移除其发布的端口
- NPM 的数据位于 `/opt/data/npm`——已被[备份策略](backup-and-recovery.md)覆盖；丢失它意味着要手工重建每一个 host

---

## 🧰 故障排查

- **502 Bad Gateway** → NPM 无法访问目标。服务在 `proxy` 网络上吗？转发主机名是不是精确的**容器名**？端口是不是*内部*端口（容器自身的端口，而非发布的映射）？
- **DNS 验证失败** → API 令牌权限范围不对，或 DNS 传播延迟——先重试一次，再核对令牌权限。NPM 的日志（`docker logs nginx-proxy-manager`）会显示 certbot 的实际报错。
- **域名无法解析** → 客户端没有使用 AdGuard 作为 DNS。检查设备实际使用的解析器（`nslookup jellyfin.home.example.com`）；使用蜂窝数据的手机需要上文的 Tailscale DNS 配置。
- **重定向循环** → 后端应用也在强制 HTTPS。把应用的基础 URL 设为代理后的 `https://` 地址；如果应用自身提供 TLS，则用 `https` scheme 转发。
- **局域网正常，Tailscale 上失灵** → DNS 重写指向了局域网 IP。改用 Tailscale IP（两种场景下都可达）作为重写答案。

---

## 🚫 不要做的事

- 不要为了"让证书省事"在路由器上转发 80/443——DNS-01 的存在正是为了让你不必这样做
- 不要使用 `.local`、`.lan` 或自造的顶级域——你会与 mDNS 冲突缠斗不休，且永远拿不到真实证书；一个真域名每月的花费还不到一杯咖啡
- 不要把管理界面（`:81`）暴露到 localhost/Tailscale 之外
- 不要像代理媒体应用那样随意地代理基础设施的管理面板（Portainer、Dockge、NPM 自身）——基础设施的控制平面需要的是更严格的访问控制，而不是更好看的 URL

---

## 🧠 最后的思考

反向代理是一堆容器开始变得像**基础设施**的地方：有名字、有加密、风格统一，无论在沙发上还是在另一个大洲，都以同样的方式触手可及。

> 一扇门。所有服务都在门后。什么都不暴露。
