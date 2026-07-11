---
description: >-
  用注重隐私的高价值替代品换掉大型科技公司的应用和服务，按类别整理，并分为从入门到进阶的层级。
---

<!--
Source: privacy/app-recommendations.md
Last translated: 2026-07
-->

# 🔁 注重隐私的应用推荐

针对常见大型科技公司服务的高价值替代品。

---

## 🌐 浏览器与搜索

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Chrome | [Brave](https://brave.com/) | 内置广告/跟踪器拦截，基于 Chromium |
| Google 搜索 | [DuckDuckGo](https://duckduckgo.com/) / [Startpage](https://www.startpage.com/) | 减少追踪 |

> 💡 进阶：自托管 [SearxNG](https://docs.searxng.org/)，获得完全私有的搜索引擎

---

## 📧 电子邮件

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Gmail | [Proton Mail](https://proton.me/mail) / [Tuta](https://tuta.com/) | 端到端加密，隐私优先 |

> 💡 提示：使用**自有域名**，获得长期的灵活性

---

## ☁️ 云存储

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Google Drive | [Nextcloud](https://nextcloud.com/) | 完全掌控，可自托管 |
| Google Drive / Dropbox | [Sync.com](https://www.sync.com/) | 加密云存储（托管方案） |
| iCloud Drive | [Nextcloud](https://nextcloud.com/) / [Syncthing](https://syncthing.net/) | 去中心化的同步选项 |

---

## 📝 笔记与文档

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Google Keep | [Joplin](https://joplinapp.org/) | 基于 Markdown，开放 |
| Apple 备忘录 | [Joplin](https://joplinapp.org/) / [Standard Notes](https://standardnotes.com/) | 跨平台 |
| VS Code | [VSCodium](https://vscodium.com/) | 开源、无遥测 |

---

## 📆 日历与联系人

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Google 日历 | [Nextcloud](https://nextcloud.com/) / [Baikal](https://sabre.io/baikal/) | 支持 CalDAV |
| Google 通讯录 | [Nextcloud](https://nextcloud.com/) / [Radicale](https://radicale.org/) | 完全掌控 |

---

## 🏢 办公套件

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Microsoft Office 365 | [ONLYOFFICE](https://www.onlyoffice.com/) / [LibreOffice](https://www.libreoffice.org/) | 开源 |
| Outlook | [Thunderbird](https://www.thunderbird.net/) | 开源 |

---

## 💬 即时通讯

| 替换 | 换成 | 原因 |
|--------|------|-----|
| 短信 / Messenger | [Signal](https://signal.org/) | 端到端加密 |

> 💡 进阶：
>
> - [SimpleX](https://simplex.chat/)
> - [Element（Matrix）](https://element.io/)

---

## 📸 照片

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Google 相册 | [Immich](https://immich.app/) | 自托管、现代化 |
| iCloud 照片 | [Immich](https://immich.app/) / [PhotoPrism](https://www.photoprism.app/) | 完全归你所有 |

---

## 🔐 密码管理器

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Chrome 密码 | [Bitwarden](https://bitwarden.com/) / [Vaultwarden](https://github.com/dani-garcia/vaultwarden) | 安全、跨平台 |
| Chrome 密码 | [Proton Pass](https://proton.me/pass) | 隐私优先的非自托管方案 |

---

## 🔑 双因素认证

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Google Authenticator / Authy | [Aegis](https://getaegis.app/)（Android）/ [Ente Auth](https://ente.io/auth/)（跨平台） | 开源、加密且可导出的备份、不被云端锁定 |

> 💡 Google Authenticator 会把你的 2FA 种子同步到 Google 账号；Authy 闭源且在 2024 年发生数据泄露。你的第二道防线值得更好的选择。详见[移动端隐私指南](mobile.md)。

---

## 🌍 DNS 与广告拦截

| 替换 | 换成 | 原因 |
|--------|------|-----|
| 运营商 DNS | [AdGuard Home](https://adguard.com/en/adguard-home/overview.html) / [Pi-hole](https://pi-hole.net/) | 全网范围拦截广告与追踪 |

---

## 🔒 VPN 与安全访问

| 替换 | 换成 | 原因 |
|--------|------|-----|
| 不用 VPN / 免费 VPN | [Mullvad](https://mullvad.net/) | 隐私优先，无需账号 |
| 传统 VPN | [Tailscale](https://tailscale.com/) / [Headscale](https://github.com/juanfont/headscale) | 私有 mesh VPN，安全远程访问 |

---

## 🎥 媒体与流媒体

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Netflix / HBO / 流媒体 | [Jellyfin](https://jellyfin.org/) | 完全自托管的媒体流服务 |
| 流媒体订阅 | Arr 全家桶（[Sonarr](https://sonarr.tv/)、[Radarr](https://radarr.video/)、[Prowlarr](https://prowlarr.com/)） | 自动化媒体管理 |

---

## 📚 图书、PDF 与知识

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Kindle 生态 | [Calibre-Web](https://github.com/janeczku/calibre-web) / [Kavita](https://www.kavitareader.com/) | 自托管电子书库 |
| Audible / 播客应用 | [Audiobookshelf](https://www.audiobookshelf.org/) | 自托管有声书 + 播客 |
| Adobe Reader | [Sumatra PDF](https://www.sumatrapdfreader.org/free-pdf-reader) | 轻量、隐私友好 |
| Adobe Acrobat / CC | [BentoPDF](https://github.com/alam00000/bentopdf) | 开放/自托管的 PDF 工具 |
| 成堆的纸质文件 | [Paperless-ngx](https://docs.paperless-ngx.com/) | 文档管理系统 |

---

## 💰 个人财务

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Mint / YNAB | [Actual Budget](https://actualbudget.org/) | 自托管、隐私优先的预算工具 |

---

## 🧰 开发与 Git

| 替换 | 换成 | 原因 |
|--------|------|-----|
| GitHub | [Forgejo](https://forgejo.org/) | 自托管 Git，完全掌控 |

---

## 🌐 网络工具

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Speedtest.net | [LibreSpeed](https://librespeed.org/) / [Speedtest Tracker](https://github.com/alexjustesen/speedtest-tracker) | 自托管测速，无追踪 |

---

## 🧱 隐私技术栈层级

### 🟢 入门

- Brave
- DuckDuckGo
- Signal
- Bitwarden / Proton Pass
- Aegis / Ente Auth（2FA）
- Proton Mail / Tuta
- ONLYOFFICE

---

### 🟡 进阶

- Nextcloud 或 Sync.com
- Joplin
- AdGuard Home
- Mullvad VPN
- 自有域名邮箱

---

### 🔴 高级

- Nextcloud AIO
- Immich
- Vaultwarden
- SearxNG
- Jellyfin
- Arr 全家桶
- Paperless-ngx
- Audiobookshelf
- Nginx Proxy Manager
- Tailscale / Headscale
- Forgejo
- GrapheneOS（见[移动端隐私](mobile.md)）
