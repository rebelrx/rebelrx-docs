---
description: >-
  Nginx Proxy Manager para un homelab privado — nombres HTTPS limpios para cada servicio, certificados wildcard sin puertos expuestos mediante DNS-01, y DNS local con AdGuard Home.
---

<!--
Source: homelab/nginx-proxy-manager.md
Last translated: 2026-07
-->

# 🔀 Nginx Proxy Manager (HTTPS privado para todo)

Los puertos y las direcciones IP no escalan. `http://192.168.1.20:8096` funciona para un servicio; al llegar al décimo es un juego de memoria, todos los navegadores gritan "No seguro" y nada tiene TLS.

Un proxy inverso lo arregla todo: **un solo punto de entrada, HTTPS de verdad, nombres limpios** — `https://jellyfin.home.example.com` — sin exponer un solo puerto a internet.

---

## ⚠️ Principio fundamental

> Cada servicio recibe un nombre y un certificado. Nada lleva un número de puerto en la barra de direcciones.

---

## 🧭 Cómo encajan las piezas

```text
Navegador (en la tailnet)
   │
   ▼  "¿jellyfin.home.example.com?"
DNS local (AdGuard Home)  →  responde con la IP de tu servidor
   │
   ▼  HTTPS :443
Nginx Proxy Manager  →  termina TLS, enruta por hostname
   │
   ▼  Red de Docker
Contenedor de Jellyfin :8096
```

Tres componentes:

1. **Un dominio real de tu propiedad** (unos pocos dólares al año) — usado únicamente para nombres y certificados; nada se aloja públicamente en él
2. **DNS local** que responde por él dentro de tu red — AdGuard Home en esta guía
3. **NPM** terminando TLS y enrutando por hostname

> Que el dominio sea real es lo que hace posibles los certificados reales; sin avisos de certificado autofirmado ni importar CAs en cada dispositivo.

---

## 📦 El stack de NPM

Siguiendo la [estructura del homelab](docker-home-lab.md), `/opt/stacks/npm/compose.yaml`:

```yaml
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - "80:80"            # HTTP (redirige a HTTPS)
      - "443:443"          # HTTPS
      - "127.0.0.1:81:81"  # panel de administración — solo localhost, se accede vía Tailscale/SSH
    volumes:
      - /opt/data/npm/data:/data
      - /opt/data/npm/letsencrypt:/etc/letsencrypt
    networks:
      - proxy

networks:
  proxy:
    external: true
```

Crea la red compartida una sola vez y levántalo:

```bash
docker network create proxy
cd /opt/stacks/npm
docker compose up -d
```

!!! warning "Enlaza el panel de administración a localhost"

    El enlace `127.0.0.1:81:81` hace que el panel de administración sea inalcanzable
    desde la red — incluso desde tu LAN. Accede a él mediante Tailscale o un túnel SSH.
    El panel que controla todo tu enrutamiento debería ser lo más difícil de
    alcanzar, no lo más fácil.

### Primer inicio de sesión

Navega a `http://server:81` (a través de Tailscale) e inicia sesión con las credenciales por defecto de NPM:

```text
Email:      admin@example.com
Contraseña: changeme
```

Te obliga a cambiar las credenciales de inmediato. Usa el gestor de contraseñas.

---

## 🕸️ La red proxy compartida

NPM enruta a los contenedores **por nombre a través de una red Docker compartida** — no hacen falta puertos publicados en los propios servicios.

Añade la red `proxy` a cualquier stack que NPM deba alcanzar:

```yaml
services:
  jellyfin:
    # ...configuración existente...
    networks:
      - default
      - proxy

networks:
  proxy:
    external: true
```

Ahora NPM puede alcanzar `jellyfin:8096` directamente, y puedes **eliminar por completo los puertos publicados del servicio** — el proxy se convierte en la única puerta.

> Menos puertos publicados = menor superficie de ataque = menos cosas sobre las que razonar. La red proxy es el pasillo del homelab; NPM es el único con llaves de la entrada.

---

## 🔐 Certificado wildcard sin puertos expuestos (DNS-01)

El flujo habitual de Let's Encrypt (HTTP-01) exige el puerto 80 abierto a internet — exactamente lo que esta configuración se niega a hacer. El desafío **DNS-01** demuestra la propiedad del dominio mediante un registro DNS, así que funciona con todos los puertos cerrados. Bonus: es el único tipo de desafío que puede emitir certificados **wildcard**.

### 1. Crea un token de API de DNS

En tu proveedor de DNS (se muestra Cloudflare; NPM admite docenas):

- **My Profile → API Tokens → Create Token**
- Plantilla: *Edit zone DNS*
- Limítalo a **solo** esa zona (p. ej., `example.com`)

### 2. Solicita el certificado en NPM

**SSL Certificates → Add SSL Certificate → Let's Encrypt**

- Nombres de dominio: `*.home.example.com` y `home.example.com`
- ✅ *Use a DNS Challenge* → proveedor: Cloudflare → pega el token
- Acepta y guarda. La emisión tarda uno o dos minutos.

Un solo certificado wildcard cubre ahora cada servicio que añadas jamás bajo `*.home.example.com` — sin emisión por servicio, y con renovaciones automáticas.

!!! tip "Mantén los nombres privados fuera del DNS público"

    Con el wildcard, los hostnames individuales (`jellyfin.`, `paperless.`,
    `vault.`) nunca aparecen en ningún lugar público — ni en tu zona DNS ni en
    los registros de transparencia de certificados, que registran cada certificado
    emitido y son consultables por cualquiera. Un certificado individual para
    `vault.home.example.com` le anuncia al mundo que gestionas una bóveda de
    contraseñas. El wildcard no anuncia nada.

---

## 🧭 DNS local con AdGuard Home

Internet no tiene ni idea de qué es `jellyfin.home.example.com` — solo tu red debe saberlo. En AdGuard Home:

**Filters → DNS rewrites → Add DNS rewrite**

```text
Dominio:   *.home.example.com
Respuesta: 192.168.1.10        # la IP LAN de tu servidor
```

Una sola reescritura wildcard cubre cada servicio actual y futuro.

!!! tip "Haz que funcione también sobre Tailscale"

    Apunta la reescritura a la **IP de Tailscale** del servidor (`100.x.y.z`) en
    lugar de la IP LAN, y establece AdGuard Home como servidor DNS de tu tailnet
    (consola de administración de Tailscale → DNS → añade tu instancia de AdGuard,
    activa *Override local DNS*). Ahora `https://jellyfin.home.example.com` funciona
    igual en casa y a través de la [tailnet](tailscale.md) — una URL en todas
    partes, con bloqueo de anuncios incluido.

---

## 🔀 Crear proxy hosts

La recompensa por servicio. **Hosts → Proxy Hosts → Add Proxy Host**:

**Pestaña Details**

```text
Domain Names:         jellyfin.home.example.com
Scheme:               http
Forward Hostname/IP:  jellyfin        ← nombre del contenedor en la red proxy
Forward Port:         8096
Websockets Support:   ✅
Block Common Exploits: ✅
```

**Pestaña SSL**

```text
SSL Certificate:  *.home.example.com   ← el wildcard de antes
Force SSL:        ✅
HTTP/2 Support:   ✅
```

Guarda. `https://jellyfin.home.example.com` ya está en marcha — candado incluido.

Repite por servicio; cada uno son treinta segundos:

| Servicio | Reenviar a |
| :--- | :--- |
| Immich | `immich-server:2283` |
| Paperless | `paperless-webserver:8000` |
| Forgejo | `forgejo:3000` |
| Uptime Kuma | `uptime-kuma:3001` |

!!! tip "Websockets: déjalo activado y punto"

    La mitad del homelab (Jellyfin, Uptime Kuma, Dozzle, Home Assistant, cualquier
    cosa con una interfaz que se actualiza en vivo) necesita websockets, y el síntoma
    cuando está desactivado es exasperantemente vago — las páginas cargan pero nada
    se actualiza. Actívalo por defecto y no vuelvas a depurarlo jamás.

---

## 🛡️ Lista de endurecimiento

- Panel de administración enlazado a `127.0.0.1`, accesible solo vía [Tailscale](tailscale.md)
- **Force SSL** en cada proxy host; añade **HSTS** cuando confíes en la renovación del certificado
- **Block Common Exploits** habilitado por host
- Sitio por defecto (Settings → Default Site) configurado a un 404 — los hostnames desconocidos que golpeen el proxy no aprenden nada
- Elimina los puertos publicados de los servicios en cuanto su proxy host funcione
- Los datos de NPM viven en `/opt/data/npm` — ya cubiertos por la [estrategia de respaldo](backup-and-recovery.md); perderlos significa recrear cada host a mano

---

## 🧰 Solución de problemas

- **502 Bad Gateway** → NPM no puede alcanzar el destino. ¿Está el servicio en la red `proxy`? ¿Es el hostname de reenvío el **nombre exacto del contenedor**? ¿Es el puerto el *interno* (el propio puerto del contenedor, no un mapeo publicado)?
- **Falla el desafío DNS** → alcance del token de API incorrecto, o retraso de propagación — reintenta una vez y luego revisa los permisos del token. Los logs de NPM (`docker logs nginx-proxy-manager`) muestran el error real de certbot.
- **El nombre no resuelve** → el cliente no está usando AdGuard para DNS. Comprueba qué resolutor usa realmente el dispositivo (`nslookup jellyfin.home.example.com`); los teléfonos con datos móviles necesitan la configuración de DNS de Tailscale de arriba.
- **Bucle de redirección** → la app del backend también fuerza HTTPS. Configura la URL base de la app a la dirección `https://` del proxy, o reenvía con esquema `https` si la propia app sirve TLS.
- **Funciona en la LAN, muerto sobre Tailscale** → la reescritura DNS apunta a la IP LAN. Usa la IP de Tailscale (accesible desde ambos contextos) como respuesta de la reescritura.

---

## 🚫 Qué no hacer

- No reenvíes los puertos 80/443 en tu router "solo para facilitar los certificados" — DNS-01 existe precisamente para que no tengas que hacerlo
- No uses `.local`, `.lan` ni un TLD inventado — pelearás con conflictos de mDNS y nunca podrás obtener certificados reales; un dominio real cuesta menos que un café al mes
- No expongas el panel de administración (`:81`) más allá de localhost/Tailscale
- No hagas proxy de los paneles de administración de la infraestructura (Portainer, Dockge, el propio NPM) con la misma ligereza que las apps de medios — los planos de control de la infraestructura merecen un acceso más estricto, no URLs más bonitas

---

## 🧠 Reflexión final

El proxy inverso es donde un montón de contenedores empieza a sentirse como **infraestructura**: con nombre, cifrada, consistente, accesible de la misma forma desde tu sofá o desde otro continente.

> Una sola puerta. Todos los servicios detrás. Nada expuesto.
