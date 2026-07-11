---
description: >-
  一套经过验证的分层备份策略，面向你的 Docker 家庭实验室——Kopia 备份到本地 NAS、加密 Borg 异地备份、数据库转储，以及让这一切真正可靠的恢复演练。
---

<!--
Source: homelab/backup-and-recovery.md
Last translated: 2026-07
-->

# 💾 备份与恢复（Docker 家庭实验室）

这是一份实用指南，教你为 [Docker 家庭实验室](docker-home-lab.md)构建一套**分层、经过测试的备份系统**，并在真正需要之前证明它确实有效。

---

## ⚠️ 核心原则

> 未经测试的备份策略，等于不存在。

备份不是一个装上就完事的产品。它是一种**纪律**：副本放在正确的位置、无需你操心地自动运行，并按计划执行恢复以证明它们是真实可用的。

---

## 🧭 3-2-1 法则

任何策略都应满足的底线：

- **3** 份数据副本（原始数据 + 两份备份）
- **2** 种不同的存储介质或系统
- **1** 份异地副本

对这个家庭实验室来说，映射得非常清晰：

| 副本 | 位置 | 工具 |
| :--- | :--- | :--- |
| 1（在线） | 服务器 → `/opt/data` | — |
| 2（本地） | NAS | Kopia |
| 3（异地） | 加密的远程仓库 | Borg（borgmatic） |

> 本地备份防的是误操作和硬盘损坏。异地备份防的是火灾、盗窃和勒索软件。

---

## 📦 备份什么（以及不备份什么）

不是所有数据都值得同等对待。分级处理：

### 🔴 不可替代（所有层级都备份）

- 应用数据 → `/opt/data/<stack>`（Immich 图库、Paperless 文档、Nextcloud 文件、Vaultwarden 密码库）
- 数据库转储（见下文）
- 照片、文档、个人档案

### 🟡 费力可恢复（本地备份）

- 尚未纳入 Git 的服务配置
- 花了多年心血整理的元数据库

### 🟢 可重新获取（不备份）

- 由 Arr 全家桶管理的媒体库
- 容器镜像（一条 `docker compose pull` 就能拿回来）
- 缓存、转码文件、缩略图

!!! tip "你的 compose 目录树已经有备份了"

    按照[家庭实验室的结构](docker-home-lab.md)，`/opt/stacks/` 里的一切都
    存放在 Git（Forgejo）中。那就是你的基础设施备份——这也正是机密信息要
    留在 `.env` 文件里、不进仓库的原因。

    本指南覆盖的是另一半：`/opt/data/`。

---

## 🗄️ 数据库：无声的备份杀手

直接复制**运行中**数据库的文件，产生损坏备份的概率高到足以毁掉你唯一一次真正需要的恢复。Postgres、MySQL 和 MariaDB 无时无刻不在写入；在写入中途做文件级快照就是掷硬币。

解决办法很简单：**先转储，再备份转储文件。**

创建 `/opt/scripts/db-dumps.sh`：

```bash
#!/bin/sh
# 每晚数据库转储——生成文件级安全的副本，供备份任务拾取
set -eu

DUMP_ROOT=/opt/data/db-dumps
mkdir -p "$DUMP_ROOT"

# Postgres 容器：docker exec + pg_dumpall
for c in immich-postgres paperless-db joplin-db forgejo-db; do
  docker exec "$c" sh -c 'pg_dumpall -U "$POSTGRES_USER"' | \
    gzip > "$DUMP_ROOT/${c}.sql.gz.tmp" && \
    mv "$DUMP_ROOT/${c}.sql.gz.tmp" "$DUMP_ROOT/${c}.sql.gz"
done

# MariaDB 容器
for c in seafile-mysql romm-db; do
  docker exec "$c" sh -c 'mariadb-dump -uroot -p"$MYSQL_ROOT_PASSWORD" --all-databases' | \
    gzip > "$DUMP_ROOT/${c}.sql.gz.tmp" && \
    mv "$DUMP_ROOT/${c}.sql.gz.tmp" "$DUMP_ROOT/${c}.sql.gz"
done
```

根据你的栈调整容器列表，然后安排在每晚备份任务运行**之前**执行：

```bash
sudo chmod +x /opt/scripts/db-dumps.sh
sudo crontab -e
```

```text
# 01:00 转储，02:00 本地备份，03:00 异地备份
0 1 * * * /opt/scripts/db-dumps.sh
```

> `.tmp` + `mv` 的模式确保备份任务永远不会拾取到写了一半的转储文件。

---

## 🟢 第 2 层——用 Kopia 做本地备份（服务器 → NAS）

[Kopia](https://kopia.io/) 提供快速、去重、加密的快照，还带有清爽的 Web 界面——非常适合高频率的本地备份层。

**前提条件：**NAS 已挂载到服务器上（例如通过 NFS 挂载在 `/mnt/nas/backups`）。

### Compose 栈

`/opt/stacks/backup/compose.yaml`：

```yaml
services:
  kopia:
    image: kopia/kopia:latest
    container_name: kopia
    restart: unless-stopped
    hostname: server-backup
    environment:
      - KOPIA_PASSWORD=${KOPIA_REPO_PASSWORD}
      - TZ=${TZ}
    command:
      - server
      - start
      - --insecure                  # UI 仅通过 Tailscale 访问；TLS 可选
      - --address=0.0.0.0:51515
      - --server-username=${KOPIA_UI_USER}
      - --server-password=${KOPIA_UI_PASSWORD}
    ports:
      - "51515:51515"
    volumes:
      - /opt/data/kopia/config:/app/config
      - /opt/data/kopia/cache:/app/cache
      - /opt/data/kopia/logs:/app/logs
      - /opt/data:/source/opt-data:ro        # 备份的对象（只读）
      - /mnt/nas/backups/kopia:/repository   # 备份的去处
```

!!! warning "以只读方式挂载源数据"

    源挂载上的 `:ro` 意味着，即使备份容器被入侵或行为异常，也**无法篡改
    它所保护的数据**。备份工具对你的在线数据只需要读权限——永远不需要写权限。

### 初始化与排程

（通过 Tailscale）打开 `http://server:51515`，在 `/repository` 中创建仓库，并为 `/source/opt-data` 定义快照策略：

- **排程** → 每天 02:00
- **保留** → 7 份每日、4 份每周、6 份每月
- **排除项** → 缓存目录、转码文件夹，以及一切 🟢 层的东西

仓库密码（`KOPIA_REPO_PASSWORD`）对所有静态数据加密。

!!! danger "把仓库密码保存在它所保护的系统之外"

    如果备份的加密密码只存在于那台已经死掉的服务器上，那它就不是备份。
    请把仓库密码和密钥保存在密码管理器里，**并且**再留一份离线副本
    （打印出来，放在安全的地方）。

---

## 🔴 第 3 层——用 Borg 做异地备份（borgmatic）

[Borg](https://www.borgbackup.org/) 提供加密、去重、支持追加模式的归档；[borgmatic](https://torsion.org/borgmatic/) 把它包装成一个声明式配置文件。

远端方面，选择一个原生支持 Borg 的托管方——[BorgBase](https://www.borgbase.com/) 和 [rsync.net](https://rsync.net/) 是成熟的选项；或者用另一台你自己控制的机器（家人的住处、第二个站点）也同样好用，还能让一切都留在自己手里。

> 一切数据在离开你的服务器之前都已在**客户端加密**。远端主机存储的是它自己也无法读取的密文。

### Compose 栈

添加到同一个 `backup` 栈中：

```yaml
  borgmatic:
    image: ghcr.io/borgmatic-collective/borgmatic:latest
    container_name: borgmatic
    restart: unless-stopped
    environment:
      - BORG_PASSPHRASE=${BORG_PASSPHRASE}
      - TZ=${TZ}
    volumes:
      - /opt/data:/mnt/source:ro                          # 源数据（只读）
      - /opt/data/borgmatic/config:/etc/borgmatic.d       # 配置 + crontab
      - /opt/data/borgmatic/borg-config:/root/.config/borg
      - /opt/data/borgmatic/cache:/root/.cache/borg
      - /opt/data/borgmatic/ssh:/root/.ssh                # 远程仓库的密钥
```

### 配置

`/opt/data/borgmatic/config/config.yaml`：

```yaml
source_directories:
  - /mnt/source

exclude_patterns:
  - '*/cache/*'
  - '*/transcodes/*'
  - /mnt/source/kopia          # 不备份本地备份工具自己的缓存

repositories:
  - path: ssh://xxxxxxxx@xxxxxxxx.repo.borgbase.com/./repo
    label: offsite

keep_daily: 7
keep_weekly: 4
keep_monthly: 12

checks:
  - name: repository
    frequency: 2 weeks
  - name: archives
    frequency: 1 month
```

`/opt/data/borgmatic/config/crontab.txt`：

```text
0 3 * * * PATH=$PATH:/usr/local/bin borgmatic --verbosity -1 --syslog-verbosity 1
```

初始化仓库（只需一次）：

```bash
docker exec borgmatic borgmatic repo-create --encryption repokey-blake2
docker exec borgmatic borgmatic key export --paper   # 把这个打印出来。说真的。
```

!!! tip "为什么 Kopia 和 Borg 两个都用？"

    两个彼此独立的工具意味着：一次糟糕的更新、一个 bug 或一次仓库损坏，
    不可能同时毁掉你数据的所有副本。本地层为快速、频繁的恢复而优化；
    异地层为灾难生存而优化。

    只想用一个？Borg + borgmatic 同时备份到 NAS **和**异地，
    用一条工具链就能满足 3-2-1。

---

## 🔁 恢复测试（人人都跳过的环节）

未经测试的备份是一种希望，而不是一个计划。在日历上安排**每季度一次的恢复演练**。三十分钟，四项检查：

### 1. 恢复一个文件（Kopia）

挑一个真实的文件，恢复到别的位置，核对内容：

```bash
docker exec kopia kopia snapshot list /source/opt-data
docker exec kopia kopia restore <snapshot-id>/immich/library/some-photo.jpg /tmp/restore-test/
```

### 2. 恢复一个文件（Borg）

```bash
docker exec borgmatic borgmatic list
docker exec borgmatic borgmatic extract --archive latest \
  --path mnt/source/paperless/media/documents \
  --destination /tmp/restore-test/
```

### 3. 恢复一个数据库

真正要命的那一项：

```bash
# 恢复到一个临时容器里——绝不要碰你的生产数据库
docker run -d --name pg-restore-test -e POSTGRES_PASSWORD=test postgres:17
zcat /opt/data/db-dumps/paperless-db.sql.gz | \
  docker exec -i pg-restore-test psql -U postgres
docker exec pg-restore-test psql -U postgres -c '\l'   # 数据库都在吗？行数合理吗？
docker rm -f pg-restore-test
```

### 4. 在纸面上预演灾难

问自己：*服务器没了，接下来的顺序是什么？*你应该能凭记忆写出来：

1. 安装 Devuan → [服务器指南](../linux/devuan-server-install.md)
2. 安装 Docker → [家庭实验室指南](docker-home-lab.md)
3. 从 Forgejo 克隆 homelab 仓库 → `/opt/stacks` 恢复完毕
4. 从 NAS 恢复 `/opt/data`（Kopia）——若 NAS 也阵亡了，则从异地恢复（Borg）
5. 把数据库转储恢复到全新的数据库容器中
6. `docker compose up -d`，一个栈一个栈来

如果任何一步让你说出"这个我到时候得研究一下"，那就**现在**研究清楚，并写下来。

---

## 📟 监控（无声失败是默认状态）

备份总是悄无声息地失败——磁盘满了、密钥过期了、NAS 没挂载——而你要到恢复时才会发现。把它们接入你现有的监控体系：

- 在 **Uptime Kuma** 中添加一个 push 监控项，在每次备份运行结束时 ping 它一次；如果预定时间没收到 ping 就告警
- borgmatic 原生支持这一点——在 `config.yaml` 中添加：

```yaml
uptime_kuma:
  push_url: https://uptime-kuma.local/api/push/<token>
```

- 任何基础设施变更之后，检查 `docker logs kopia` 和 `docker logs borgmatic`

> 要监控的是**成功的缺席**，而不仅仅是错误的出现。

---

## 🚫 不要做的事

- 不要直接备份在线的数据库文件；先转储
- 不要给备份容器对源数据的写权限
- 不要把加密密钥只保存在被备份的那台机器上
- 不要让 RAID 或 NAS 的冗余冒充备份——它防的是硬盘故障，防不了删除、损坏或勒索软件
- 不要"配好就忘"——那正是你在第三年才发现备份在第一年就停了的原因

---

## 🧠 最后的思考

没有人后悔花一小时测试恢复。

倒是有很多人，为随着一块硬盘一起消失的多年照片、文档和基础设施而悔恨不已。

> 你的数据有多自主，取决于你把它找回来的能力有多强。
