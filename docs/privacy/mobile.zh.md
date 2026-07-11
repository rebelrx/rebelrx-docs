---
description: >-
  移动端隐私的三级阶梯——先加固你手头的手机，再替换那些追踪你的应用，等你准备好了，用 GrapheneOS 把操作系统本身也换掉。
---

<!--
Source: privacy/mobile.md
Last translated: 2026-07
-->

# 📱 移动端隐私

你的手机是有史以来最贴身的监控设备。它知道你在哪里睡觉、和谁交谈、读些什么，以及盯着它看了多久。

本指南是一架**阶梯，而不是悬崖**：三个级别，每一级都是实实在在的改进，每一级都可自由选择。

---

## ⚠️ 核心原则

> 你随身携带的设备应该听命于你，而不是汇报关于你的一切。

---

## 🧭 起点的现实

- **原生 Android** → Google Play 服务以系统级权限运行：位置、传感器、应用使用情况、网络——无论你在设置里怎么开关。一部典型的 Android 手机每天与 Google 服务器通信数百次。
- **iPhone** → 默认设置明显更好，硬件安全也很强，但它是一个封闭系统，元数据和整个生态的钥匙都握在 Apple 手里。
- **去谷歌化的 Android** → 唯一一条让操作系统本身为你服务的路。

你不必一步跳到终点。从你所在的位置开始。

---

## 🪜 第 1 级——加固你手头的设备（今晚就能做，免费）

不用换手机，不用刷机。三十分钟：

- **审查应用权限** → 设置 → 隐私。撤销所有并非明确需要的应用的位置、麦克风和相机权限。优先选择"仅在使用时允许"。
- **干掉广告 ID** → Android：设置 → 隐私 → 广告 → *删除广告 ID*。iPhone：设置 → 隐私 → 跟踪 → 关闭*允许 App 请求跟踪*。
- **删除不用的应用** → 每个已安装的应用都是常驻的攻击面和数据收集器。
- **能用浏览器就别用应用**（尤其是社交媒体）——网站能拿到的权限远少于已安装的应用。
- **关闭 Wi-Fi 和蓝牙扫描** → Android 把这两项埋在位置设置里；即便 Wi-Fi"已关闭"，它们仍在广播你的存在。
- **设置强密码** → 至少 6 位以上；生物识别只是便利，密码才是真正的锁。
- **iPhone 加分项** → 启用**高级数据保护**（设置 → iCloud），获得端到端加密的备份。

> 第 1 级无法击败平台层面的遥测。它缩小的是应用层收集的范围——而那正是日常数据流失的大头。

---

## 🪜 第 2 级——替换追踪你的应用

手机不变，软件换好的。这是[应用推荐](app-recommendations.md)的移动版：

| 替换 | 换成 | 原因 |
|--------|------|-----|
| Chrome / Safari | [Brave](https://brave.com/) | 内置广告/跟踪器拦截 |
| Google 搜索 | [DuckDuckGo](https://duckduckgo.com/) / [Startpage](https://www.startpage.com/) | 减少追踪 |
| 短信 / Messenger / WhatsApp | [Signal](https://signal.org/) | 端到端加密 |
| Gmail 应用 | [Proton Mail](https://proton.me/mail) / [Tuta](https://tuta.com/) | 加密邮件 |
| Google 地图 | [Organic Maps](https://organicmaps.app/) | 离线、开源、零追踪 |
| Google 相册 | [Immich](https://immich.app/) 移动应用 | 自动备份到**你自己的**服务器 |
| Gboard | [HeliBoard](https://github.com/Helium314/HeliBoard)（Android） | 你的键盘不应该需要联网 |
| Google Authenticator / Authy | [Aegis](https://getaegis.app/)（Android）/ [Ente Auth](https://ente.io/auth/) | 开源 2FA，加密且可导出的备份 |
| YouTube 应用 | [NewPipe](https://newpipe.net/)（Android） | 无需账号、没有广告、不做观看画像 |
| 备忘录应用 | [Joplin](https://joplinapp.org/) | 可与你现有的方案同步 |

### 🔑 关于 2FA 应用的几句话

双因素认证没有商量余地，但用哪个*应用*很有讲究：

- **Google Authenticator** 会把你的 2FA 种子同步到 Google 账号（多年来都没有端到端加密）——一切的钥匙，握在你正要离开的那家公司手里
- **Authy** 闭源，且在 2024 年被攻破（3300 万用户的手机号被泄露）
- **Aegis** 和 **Ente Auth** 开源，备份加密且完全由本地掌控

!!! warning "在需要之前就备份好你的 2FA 保险库"

    手机丢了、验证器又没备份，你就会被锁在它所保护的每一个账户之外。
    导出加密保险库（Aegis 和 Ente Auth 都支持），与密码管理器的备份放在
    一起保存。并且实际测试一次恢复流程。

### 📦 从哪里获取应用（Android）

- **[F-Droid](https://f-droid.org/)** → 开源应用，无需账号
- **[Obtainium](https://github.com/ImranR98/Obtainium)** → 直接从开发者的发布页安装和更新
- **[Aurora Store](https://auroraoss.com/)** → Play 商店目录的匿名前端，用于那些实在避不开的应用

---

## 🪜 第 3 级——替换操作系统（GrapheneOS）

换应用治不了一个不停向老家汇报的操作系统。当你准备好把 Google 从平台本身中移除时，[**GrapheneOS**](https://grapheneos.org/) 是现有最强的选择：一个经过加固、安全至上的 Android，没有 Google 服务，也没有任何拼凑上去的妥协。

### Pixel 的讽刺

没错：逃离 Google 软件的最佳方式，是 Google 的硬件。

GrapheneOS **只支持 Pixel 设备**（目前是 Pixel 6 到 Pixel 10 系列），因为 Pixel 是唯一允许你**用自己的签名密钥重新锁定引导加载程序**的主流手机——这意味着验证启动保护的是*你的*系统，而不是厂商的。没有任何其他消费级硬件提供这一点。

!!! tip "实用购买建议"

    - 购买**无锁版（factory unlocked）**，直接从 Google 或信誉良好的零售商处购买——
      运营商锁定的机型（尤其是美国 Verizon/AT&T 版本）往往根本无法解锁
      引导加载程序
    - 二手 Pixel 8/8a 是预算甜点：远低于 350 美元，还有多年的安全支持
    - 更新的机型只是把支持窗口拉得更长

### 安装

[官方网页安装器](https://grapheneos.org/install/web)直接在浏览器里运行。连接手机、按步骤操作、最后重新锁定引导加载程序。15–20 分钟，无需安装任何工具。

### 日常使用

- **沙盒化的 Google Play** → 如果某个应用确实离不开 Play 服务，GrapheneOS 可以把货真价实的 Play 服务当作一个普通的、无特权的、完全沙盒化的应用来运行；没有系统级访问权限，只安装在你选定的配置文件里。大多数人把它放进单独的配置文件，让主配置文件保持无 Google 状态。
- **用户配置文件** → 不同情境之间完全隔离（个人 / 工作 / "非要 Google 不可的应用"）
- **日常体验** → 它仍然是 Android。你第 2 级的所有应用都能用。

!!! warning "先检查你的银行应用"

    有些银行应用要求 Google 的硬件认证（attestation），拒绝在任何第三方系统上
    运行——大约一半能在带沙盒 Play 的 GrapheneOS 上工作，且因银行和年份而异。
    刷机**之前**先查社区兼容性列表，确认你的具体应用。永远行得通的退路：
    在 Brave 里用银行的网页版。

### 替代方案：CalyxOS

[CalyxOS](https://calyxos.org/) 用 GrapheneOS 的部分加固换取了便利：**microG**（Play 服务的开源重新实现）预装其中，且除 Pixel 外还支持 Fairphone 和部分 Motorola 设备。这是一条合理的中间路线——但如果你有受支持的 Pixel，GrapheneOS 是更强的选择。

---

## 🚫 不要做的事

- 不要为了隐私去 root 手机。root 会**破坏**保护你的安全模型和验证启动
- 不要从 Play 商店随便装"隐私"应用。它们大多是披着风衣的广告软件
- 不要刷已被弃坑或只有一个维护者的 ROM。没有安全更新的系统比原生系统更糟
- 不要试图在一个周末完成全部三级。迁移就是这样半途而废的

---

## 🧠 最后的思考

你的手机看到的你的生活，比你拥有的任何其他物品都多。

这架阶梯的每一级，都让它离真正**属于你**更近一步：

> 同一个口袋，不同的效忠对象。
