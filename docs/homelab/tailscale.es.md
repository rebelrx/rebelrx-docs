---
description: >-
  Acceso remoto seguro a tu homelab con Tailscale en Linux sin systemd — configuración sysvinit en Devuan, enrutamiento de subredes, nodos de salida y ACLs sin abrir un solo puerto.
---

<!--
Source: homelab/tailscale.md
Last translated: 2026-07
-->

# 🔒 VPN Tailscale (Devuan + sin systemd)

Una guía práctica para acceder a tu homelab desde cualquier lugar **sin abrir un solo puerto** a internet.

La mayoría de las guías de Tailscale asumen systemd. Esta cubre **Devuan (sysvinit)** para tu servidor — donde Tailscale no incluye soporte de init alguno — además de la configuración con OpenRC para escritorios Artix.

---

## ⚠️ Principio fundamental

> La exposición es el enemigo. El acceso debe ser privado por defecto y concederse de forma deliberada.

El reenvío de puertos expone tus servicios a todo internet y confía en que tus páginas de inicio de sesión aguanten. Una VPN de malla invierte ese modelo: nada es alcanzable a menos que el dispositivo sea **tuyo y esté autenticado**.

---

## 🧭 Qué es Tailscale

Tailscale construye una red de malla privada (una "tailnet") entre tus dispositivos usando **WireGuard**, el protocolo VPN moderno y auditado integrado en el kernel de Linux.

- Cada dispositivo recibe una IP privada estable (`100.x.y.z`)
- Los dispositivos se conectan **directamente entre sí** (peer-to-peer) siempre que es posible
- El tráfico está cifrado de extremo a extremo entre tus dispositivos
- Funciona a través de NAT y firewalls sin configurar nada en el router

> Tu servidor, portátil y teléfono se comportan como si estuvieran en la misma LAN — desde cualquier lugar.

---

## ⚖️ El compromiso, con honestidad

Tailscale no es totalmente autoalojado por defecto. Entiende en qué estás confiando:

| Componente | Estado |
| :--- | :--- |
| Clientes (`tailscaled`) | Código abierto |
| Cifrado (WireGuard) | Código abierto, de extremo a extremo |
| Servidor de coordinación | **Alojado por Tailscale Inc. (código cerrado)** |

El servidor de coordinación solo intercambia claves públicas y metadatos de conexión — **no puede descifrar tu tráfico**. Pero sí ve qué dispositivos existen y cuándo se conectan.

!!! tip "Opción de soberanía total: Headscale"

    [Headscale](https://github.com/juanfont/headscale) es un reemplazo de código abierto
    y autoalojable del servidor de coordinación de Tailscale. Los clientes oficiales de
    Tailscale se conectan a él directamente.

    Ruta recomendada: empieza con el Tailscale alojado para aprender el modelo y migra a
    Headscale cuando tu tailnet sea estable. Los comandos del lado del cliente de esta
    guía son idénticos en ambos casos.

---

## ⚙️ Requisitos

- Un servidor Devuan (consulta la [Guía de instalación de servidor Devuan](../linux/devuan-server-install.md))
- Una cuenta gratuita de Tailscale → <https://login.tailscale.com/start>
  - El inicio de sesión se realiza mediante un proveedor de identidad. Para mantener a las Big Tech fuera del circuito, usa el registro con **Passkey** o una cuenta de GitHub en lugar de un inicio de sesión de Google/Microsoft.

El plan gratuito da acceso a casi todas las funciones de Tailscale y cubre dispositivos ilimitados y 6 usuarios — más que suficiente para un homelab.

---

## 📦 Instalación en Devuan (servidor)

### 1. Añadir el repositorio de Tailscale

El repositorio `.deb` de Tailscale se indexa por el nombre en clave de **Debian** — el mismo truco de mapeo usado en la [guía del Homelab con Docker](docker-home-lab.md).

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# Devuan usa su propio nombre en clave; el repo Debian de Tailscale necesita la base Debian:
#   Devuan 5  "Daedalus"  → bookworm
#   Devuan 6  "Excalibur" → trixie
DEBIAN_CODENAME=trixie   # ajústalo a la base Debian de tu versión de Devuan

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

!!! info "Espera un error inofensivo"

    El paso post-instalación del paquete intenta comunicarse con systemd y se quejará.
    Ignóralo — los binarios (`/usr/sbin/tailscaled` y `/usr/bin/tailscale`)
    se instalan correctamente. Nosotros aportamos el script de init en el paso siguiente.

---

### 2. Crear el script de sysvinit

Tailscale no proporciona ningún script de init para sysvinit. Crea uno:

```bash
sudo nano /etc/init.d/tailscaled
```

Pega lo siguiente:

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

Hazlo ejecutable y regístralo en los runlevels por defecto:

```bash
sudo chmod +x /etc/init.d/tailscaled
sudo update-rc.d tailscaled defaults
sudo /etc/init.d/tailscaled start
```

Verifica que está en ejecución:

```bash
sudo /etc/init.d/tailscaled status
```

---

### 3. Únete a tu tailnet

```bash
sudo tailscale up
```

Sigue la URL impresa en un navegador para autenticar la máquina.

> Solo lo haces una vez. El demonio guarda su estado autenticado en
> `/var/lib/tailscale/` y se reconecta automáticamente en cada arranque.

Confirma que el nodo está activo y anota su dirección:

```bash
tailscale status
tailscale ip -4
```

---

## 🖥️ Instalación en Artix (escritorio / portátil)

Artix empaqueta los scripts de init por separado para cada sistema de init:

```bash
sudo pacman -S tailscale tailscale-openrc
sudo rc-update add tailscaled default
sudo rc-service tailscaled start
sudo tailscale up
```

(Para runit o s6, instala `tailscale-runit` o `tailscale-s6` en su lugar.)

Esto coincide con la sección de post-instalación de la [Guía de instalación de escritorio Artix](../linux/artix-kde-openrc-install.md).

---

## 📱 Otros dispositivos

Instala la app de Tailscale en tu teléfono o tablet e inicia sesión con la misma cuenta:

- Android → F-Droid o Play Store
- iOS → App Store

Cada dispositivo que se une puede ahora alcanzar a todos los demás en su dirección `100.x.y.z`.

---

## 🌐 MagicDNS (nombres en lugar de IPs)

En la consola de administración de Tailscale → **DNS**, habilita **MagicDNS**.

Ahora, en lugar de memorizar direcciones:

```bash
ssh user@100.xx.xx.xx
```

Usas nombres de máquina:

```bash
ssh user@nombremaquina
```

Renombra las máquinas en la consola de administración (**Machines** → el menú `…`) para mantener nombres limpios y predecibles.

---

## 🔑 Desactivar la expiración de claves en servidores

Por defecto, las claves de cada nodo expiran tras ~6 meses y el nodo **desaparece de tu tailnet hasta que lo reautentiques de forma interactiva**.

Aceptable para portátiles. Frustrante para servidores sin monitor.

En la consola de administración → **Machines** → tu servidor → `…` → **Disable key expiry**.

---

## 🏠 Subnet router (alcanza toda tu LAN)

Algunos dispositivos no pueden ejecutar Tailscale — un NAS, impresoras, aparatos IoT, interfaces IPMI. Un **subnet router** permite que un solo nodo Tailscale haga de puente hacia toda tu LAN.

### 1. Habilitar el reenvío de IP (en el servidor Devuan)

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 2. Anunciar la subred de tu LAN

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24
```

(Sustitúyela por la subred real de tu LAN.)

### 3. Aprobar la ruta

Consola de administración → **Machines** → tu servidor → **Edit route settings** → aprueba la subred.

Ahora tu teléfono con datos móviles puede alcanzar dispositivos `192.168.1.x` como si estuvieras en casa — la interfaz web del NAS, la impresora, todo.

> Un solo subnet router sustituye a instalar Tailscale en cada dispositivo que tienes.

---

## 🚪 Nodo de salida (opcional)

Un nodo de salida enruta **todo** el tráfico de internet de un dispositivo a través de tu conexión doméstica — útil en el Wi-Fi de hoteles o aeropuertos.

En el servidor:

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24 --advertise-exit-node
```

Apruébalo en la consola de administración (en el mismo sitio que las rutas). Luego, en tu portátil o teléfono, selecciona el servidor como nodo de salida cuando estés en redes no confiables.

!!! tip

    Las opciones de `tailscale up` no son acumulativas — cada ejecución reemplaza la
    configuración anterior. Pasa siempre el conjunto completo de opciones que quieras activas.

---

## 🐳 Acceder a los servicios del homelab

Con Tailscale activo, tus [servicios de Docker](docker-home-lab.md) son accesibles con **cero puertos expuestos**:

```text
http://homemachine:8096    → Jellyfin
http://homemachine:2283    → Immich
http://homemachine:81      → Panel de Nginx Proxy Manager
```

El patrón limpio:

- Los servicios de Docker se enlazan al host (o solo a la IP de Tailscale)
- Nginx Proxy Manager enruta los hostnames internos
- Tailscale es la **única** vía de entrada
- Reenvío de puertos en el router: **ninguno**

> Si un puerto no está reenviado, los escáneres de todo internet ni siquiera pueden verlo.

---

## 🛡️ ACLs (controla quién alcanza qué)

Por defecto, cada dispositivo de tu tailnet puede alcanzar a todos los demás. Para un homelab de un solo usuario es aceptable — pero endurécelo a medida que creces.

En la consola de administración → **Access Controls**, las ACLs se definen en JSON. Un ejemplo simple: etiqueta tus servidores y restringe teléfonos/portátiles a servicios específicos:

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

!!! warning "El modo check de SSH y la automatización"

    Las reglas SSH de Tailscale admiten `"action": "check"`, que fuerza una
    reautenticación periódica por navegador. Es una protección fuerte para sesiones
    humanas interactivas — pero cualquier **automatización** (runners de CI, scripts de
    despliegue, tareas cron) que use esa regla se quedará colgada en silencio esperando
    un navegador que nunca llega.

    Reserva `check` para humanos. Usa una regla `accept` separada — con alcance muy
    limitado a un usuario de bajos privilegios — para todo lo desatendido.

---

## 🔐 Buenas prácticas de seguridad

- Protege tu inicio de sesión de Tailscale con 2FA robusto; ahora es la llave de toda tu infraestructura
- Elimina los dispositivos antiguos de la consola de administración cuando retires hardware
- Usa etiquetas y ACLs en cuanto más de una persona se una a tu tailnet
- Mantén los clientes actualizados: `apt upgrade` / `pacman -Syu` es suficiente

---

## 🧰 Solución de problemas

```bash
tailscale status        # quién está conectado, y cómo (directo vs. con relé)
tailscale netcheck      # tipo de NAT, relé DERP más cercano, info de mapeo de puertos
tailscale ping nombremaquina   # verifica la conectividad y la ruta hacia un nodo
```

Problemas comunes:

- **`failed to connect to local tailscaled`** → el demonio no está corriendo. `sudo /etc/init.d/tailscaled start` (Devuan) o `sudo rc-service tailscaled start` (Artix).
- **Las conexiones muestran `relay` en lugar de `direct`** → el tráfico está rebotando por los relés DERP de Tailscale. Sigue cifrado, solo es más lento. Permitir UDP 41641 saliente suele restaurar las rutas directas.
- **Las rutas de subred no funcionan** → la ruta no está aprobada en la consola de administración, o el reenvío de IP no está habilitado. Comprueba ambas cosas.
- **Un nodo desapareció de la tailnet** → expiración de claves. Reautentica con `sudo tailscale up` y luego desactiva la expiración para que no se repita.

---

## 🧠 Reflexión final

Cada puerto reenviado es una invitación permanente a todo internet.

Una malla privada invierte el modelo:

> Nada está expuesto. Todo es accesible para ti, y solo para ti.
