---
description: >-
  Безопасный удалённый доступ к homelab через Tailscale на Linux без systemd — настройка sysvinit на Devuan, маршрутизация подсетей, exit-ноды и ACL без единого открытого порта.
---

<!--
Source: homelab/tailscale.md
Last translated: 2026-07
-->

# 🔒 VPN Tailscale (Devuan + без systemd)

Практическое руководство по доступу к вашему homelab откуда угодно **без единого открытого порта** в интернет.

Большинство руководств по Tailscale предполагают systemd. Это руководство охватывает **Devuan (sysvinit)** для вашего сервера — где Tailscale вообще не поставляет поддержку init — плюс настройку OpenRC для десктопов на Artix.

---

## ⚠️ Основной принцип

> Экспонирование — враг. Доступ должен быть приватным по умолчанию и предоставляться осознанно.

Проброс портов открывает ваши сервисы всему интернету в надежде, что страницы входа выдержат. Mesh-VPN переворачивает эту модель: ничто не доступно, если устройство не **ваше и не аутентифицировано**.

---

## 🧭 Что такое Tailscale

Tailscale строит приватную mesh-сеть («tailnet») между вашими устройствами на основе **WireGuard** — современного, прошедшего аудит VPN-протокола, встроенного в ядро Linux.

- Каждое устройство получает стабильный приватный IP (`100.x.y.z`)
- Устройства соединяются **напрямую друг с другом** (peer-to-peer), когда это возможно
- Трафик между вашими устройствами зашифрован сквозным шифрованием
- Работает через NAT и файрволы без какой-либо настройки роутера

> Ваш сервер, ноутбук и телефон ведут себя так, будто находятся в одной локальной сети — откуда угодно.

---

## ⚖️ Честный компромисс

По умолчанию Tailscale не является полностью self-hosted. Понимайте, кому вы доверяете:

| Компонент | Статус |
| :--- | :--- |
| Клиенты (`tailscaled`) | Открытый исходный код |
| Шифрование (WireGuard) | Открытый код, сквозное шифрование |
| Координационный сервер | **Размещён у Tailscale Inc. (закрытый код)** |

Координационный сервер лишь обменивается публичными ключами и метаданными соединений — он **не может расшифровать ваш трафик**. Но он видит, какие устройства существуют и когда они подключаются.

!!! tip "Вариант полного суверенитета: Headscale"

    [Headscale](https://github.com/juanfont/headscale) — это открытая, самостоятельно
    размещаемая замена координационного сервера Tailscale. Официальные клиенты Tailscale
    подключаются к нему напрямую.

    Рекомендуемый путь: начните с облачного Tailscale, чтобы освоить модель, и переходите
    на Headscale, когда ваша tailnet стабилизируется. Клиентские команды из этого
    руководства идентичны в обоих случаях.

---

## ⚙️ Требования

- Сервер на Devuan (см. [Руководство по установке сервера Devuan](../linux/devuan-server-install.md))
- Бесплатная учётная запись Tailscale → <https://login.tailscale.com/start>
  - Вход выполняется через провайдера идентификации. Чтобы не подпускать Big Tech, используйте регистрацию через **Passkey** или аккаунт GitHub вместо входа через Google/Microsoft.

Бесплатный план даёт доступ почти ко всем возможностям Tailscale и покрывает неограниченное число устройств и 6 пользователей — более чем достаточно для homelab.

---

## 📦 Установка на Devuan (сервер)

### 1. Добавьте репозиторий Tailscale

`.deb`-репозиторий Tailscale привязан к кодовым именам **Debian** — тот же трюк с сопоставлением, что и в [руководстве по Docker Homelab](docker-home-lab.md).

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# У Devuan свои кодовые имена; Debian-репозиторию Tailscale нужна база Debian:
#   Devuan 5  "Daedalus"  → bookworm
#   Devuan 6  "Excalibur" → trixie
DEBIAN_CODENAME=trixie   # укажите базу Debian, соответствующую вашему выпуску Devuan

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

!!! info "Ожидайте безобидную ошибку"

    Пост-установочный шаг пакета пытается обратиться к systemd и выдаст жалобу.
    Игнорируйте её — бинарники (`/usr/sbin/tailscaled` и `/usr/bin/tailscale`)
    устанавливаются корректно. Init-скрипт мы создадим сами на следующем шаге.

---

### 2. Создайте init-скрипт для sysvinit

Tailscale не предоставляет init-скрипт для sysvinit. Создайте его:

```bash
sudo nano /etc/init.d/tailscaled
```

Вставьте следующее:

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

Сделайте его исполняемым и зарегистрируйте в уровнях запуска по умолчанию:

```bash
sudo chmod +x /etc/init.d/tailscaled
sudo update-rc.d tailscaled defaults
sudo /etc/init.d/tailscaled start
```

Убедитесь, что демон работает:

```bash
sudo /etc/init.d/tailscaled status
```

---

### 3. Подключитесь к своей tailnet

```bash
sudo tailscale up
```

Откройте напечатанный URL в браузере, чтобы аутентифицировать машину.

> Это делается один раз. Демон хранит аутентифицированное состояние в
> `/var/lib/tailscale/` и автоматически переподключается при каждой загрузке.

Убедитесь, что узел активен, и запишите его адрес:

```bash
tailscale status
tailscale ip -4
```

---

## 🖥️ Установка на Artix (десктоп / ноутбук)

В Artix init-скрипты упакованы отдельно для каждой системы инициализации:

```bash
sudo pacman -S tailscale tailscale-openrc
sudo rc-update add tailscaled default
sudo rc-service tailscaled start
sudo tailscale up
```

(Для runit или s6 установите `tailscale-runit` или `tailscale-s6` соответственно.)

Это соответствует разделу пост-установки в [Руководстве по установке десктопа Artix](../linux/artix-kde-openrc-install.md).

---

## 📱 Прочие устройства

Установите приложение Tailscale на телефон или планшет и войдите в ту же учётную запись:

- Android → F-Droid или Play Store
- iOS → App Store

Теперь каждое подключённое устройство может достучаться до любого другого по его адресу `100.x.y.z`.

---

## 🌐 MagicDNS (имена вместо IP-адресов)

В консоли администратора Tailscale → **DNS** включите **MagicDNS**.

Теперь вместо запоминания адресов:

```bash
ssh user@100.xx.xx.xx
```

Вы используете имена машин:

```bash
ssh user@имямашины
```

Переименовывайте машины в консоли администратора (**Machines** → меню `…`), чтобы имена оставались чистыми и предсказуемыми.

---

## 🔑 Отключите истечение ключей на серверах

По умолчанию ключи каждого узла истекают примерно через 6 месяцев, и узел **выпадает из tailnet, пока вы не переаутентифицируете его вручную**.

Для ноутбуков — нормально. Для «безголовых» серверов — мучительно.

В консоли администратора → **Machines** → ваш сервер → `…` → **Disable key expiry**.

---

## 🏠 Subnet router (доступ ко всей локальной сети)

Некоторые устройства не могут запускать Tailscale — NAS, принтеры, IoT-устройства, интерфейсы IPMI. **Subnet router** позволяет одному узлу Tailscale стать мостом ко всей вашей LAN.

### 1. Включите переадресацию IP (на сервере Devuan)

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 2. Анонсируйте подсеть вашей LAN

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24
```

(Замените на фактическую подсеть вашей локальной сети.)

### 3. Одобрите маршрут

Консоль администратора → **Machines** → ваш сервер → **Edit route settings** → одобрите подсеть.

Теперь ваш телефон в мобильной сети достаёт до устройств `192.168.1.x`, как будто вы дома — веб-интерфейс NAS, принтер, всё.

> Один subnet router заменяет установку Tailscale на каждое ваше устройство.

---

## 🚪 Exit-нода (опционально)

Exit-нода направляет **весь** интернет-трафик устройства через ваше домашнее подключение — полезно в Wi-Fi отелей и аэропортов.

На сервере:

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24 --advertise-exit-node
```

Одобрите её в консоли администратора (там же, где маршруты). Затем на ноутбуке или телефоне выберите сервер в качестве exit-ноды, когда находитесь в недоверенных сетях.

!!! tip

    Флаги `tailscale up` не аддитивны — каждый запуск заменяет предыдущую
    конфигурацию. Всегда передавайте полный набор флагов, которые должны быть активны.

---

## 🐳 Доступ к сервисам homelab

С работающим Tailscale ваши [Docker-сервисы](docker-home-lab.md) доступны при **нуле открытых портов**:

```text
http://homemachine:8096    → Jellyfin
http://homemachine:2283    → Immich
http://homemachine:81      → Панель администратора Nginx Proxy Manager
```

Чистая схема:

- Docker-сервисы привязываются к хосту (или только к IP Tailscale)
- Nginx Proxy Manager маршрутизирует внутренние hostname
- Tailscale — **единственный** путь внутрь
- Проброс портов на роутере: **отсутствует**

> Если порт не проброшен, сканеры всего интернета его даже не увидят.

---

## 🛡️ ACL (ограничьте, кто к чему имеет доступ)

По умолчанию каждое устройство в tailnet может достучаться до любого другого. Для homelab одного пользователя это приемлемо — но ужесточайте правила по мере роста.

В консоли администратора → **Access Controls** ACL описываются в JSON. Простой пример: пометьте серверы тегом, затем ограничьте телефоны/ноутбуки конкретными сервисами:

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

!!! warning "SSH-режим check и автоматизация"

    Правила Tailscale SSH поддерживают `"action": "check"`, который принудительно требует
    периодической повторной аутентификации через браузер. Сильная защита для интерактивных
    человеческих сессий — но любая **автоматизация** (CI-раннеры, скрипты деплоя, cron-задачи),
    попавшая под это правило, молча зависнет в ожидании браузера, который никогда не откроется.

    Оставьте `check` для людей. Для всего автономного используйте отдельное правило `accept`,
    жёстко ограниченное низкопривилегированным пользователем.

---

## 🔐 Лучшие практики безопасности

- Защитите вход в Tailscale надёжной двухфакторной аутентификацией; теперь это ключ ко всей вашей инфраструктуре
- Удаляйте старые устройства из консоли администратора при списании оборудования
- Используйте теги и ACL, как только в tailnet появится больше одного человека
- Обновляйте клиенты: достаточно `apt upgrade` / `pacman -Syu`

---

## 🧰 Устранение неполадок

```bash
tailscale status        # кто подключён и как (напрямую или через релей)
tailscale netcheck      # тип NAT, ближайший DERP-релей, информация о пробросе портов
tailscale ping имямашины   # проверка связности и пути до узла
```

Типичные проблемы:

- **`failed to connect to local tailscaled`** → демон не запущен. `sudo /etc/init.d/tailscaled start` (Devuan) или `sudo rc-service tailscaled start` (Artix).
- **Соединения показывают `relay` вместо `direct`** → трафик идёт через DERP-релеи Tailscale. Он всё ещё зашифрован, просто медленнее. Разрешение исходящего UDP 41641 обычно восстанавливает прямые пути.
- **Маршруты подсетей не работают** → маршрут не одобрен в консоли администратора или не включена переадресация IP. Проверьте и то и другое.
- **Узел исчез из tailnet** → истечение ключа. Переаутентифицируйтесь через `sudo tailscale up`, затем отключите истечение, чтобы это не повторялось.

---

## 🧠 Финальная мысль

Каждый проброшенный порт — постоянное приглашение для всего интернета.

Приватная mesh-сеть переворачивает модель:

> Ничего не выставлено наружу. Всё доступно вам — и только вам.
