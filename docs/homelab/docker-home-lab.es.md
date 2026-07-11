---
description: >-
  Construye un homelab limpio y reproducible con Docker Compose sobre Devuan en hardware físico — sin systemd, sin la sobrecarga de un hipervisor.
---

<!--
Source: homelab/docker-home-lab.md
Last translated: 2026-07
-->

# 🐳 Homelab con Docker (Devuan + Compose)

Una guía práctica para ejecutar un **homelab limpio y reproducible con Docker** sobre hardware físico usando Devuan y Docker Compose.

La mayoría de las guías de homelab asumen Debian, Ubuntu u otra distribución con systemd. Esta va en la dirección contraria: **Devuan**, Debian *sin* systemd, para una base más ligera, con un init simple y totalmente bajo tu control. Los stacks de Compose provienen del [repositorio homelab de RebelRx](https://github.com/rebelrx/rebelrx-homelab) y funcionan igual en cualquier host Docker; aquí los configuramos a la manera de Devuan.

---

## 🔥 Por qué Docker (y no Proxmox)

*Puedes* usar Proxmox. Mucha gente lo hace.

Pero para la mayoría de los homelabs, introduce complejidad innecesaria.

### ❌ Desafíos de Proxmox

- Sobrecarga de las VMs (desperdicio de CPU y RAM)
- Más capas que depurar
- Complejidad en los respaldos
- La red se vuelve más difícil de lo que debería
- Fomenta la fragmentación (muchas VMs pequeñas en lugar de un sistema unificado)

---

### ✅ Por qué preferir Docker sobre hardware físico

- **Ligero** → sin sobrecarga de virtualización  
- **Simple** → un solo SO, un solo sistema que administrar  
- **Despliegues rápidos** → levanta servicios en segundos  
- **Reproducible** → todo está definido en Compose  
- **Portable** → mueve todo tu stack con Git  

> 💡 Si no ejecutas cargas de trabajo empresariales multiinquilino, probablemente no necesitas un hipervisor.

---

## ⚙️ Requisitos

- Un PC o portátil
- Acceso a internet
- Devuan

Si no tienes un PC dedicado, puedes comprar una de estas mini PC económicas que incluyen RAM y SSD para empezar de inmediato: [Beelink Mini S12](https://www.amazon.com/dp/B0BW8JSQCH)

Si no tienes Devuan instalado, consulta la [Guía de instalación de servidor Devuan Linux](../linux/devuan-server-install.md)

---

## 📦 Instalar dependencias

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg git
```

---

## 🔑 Añadir el repositorio de Docker

El repositorio `.deb` de Docker se indexa por el nombre en clave de **Debian** — pero Devuan usa
los suyos propios (`daedalus`, `excalibur`, …), que el repositorio de Docker no reconoce.
Mapéalo a la base Debian sobre la que está construida tu versión de Devuan.

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/debian/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Devuan usa su propio nombre en clave; el repo Debian de Docker necesita la base Debian:
#   Devuan 5  "Daedalus"  → bookworm
#   Devuan 6  "Excalibur" → trixie
DEBIAN_CODENAME=trixie   # ajústalo a la base Debian de tu versión de Devuan

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian \
  $DEBIAN_CODENAME stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## 🐳 Instalar Docker + Compose

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

## 🚀 Habilitar e iniciar Docker (SysVinit)

```bash
sudo service docker start
sudo update-rc.d docker defaults
```

Verifica:

```bash
docker --version
docker compose version
```

---

## 🔐 Añadir tu usuario al grupo docker

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Prueba:

```bash
docker run hello-world
```

---

## 📁 Estructura de directorios

Mantén la **configuración y los datos en árboles separados** — es la decisión de
layout más importante de todas, y cada stack del repositorio la sigue:

```bash
/opt/stacks/<stack>/     # Stacks de Docker Compose (compose.yaml, .env, README)
/opt/data/<stack>/       # Datos persistentes de las apps (nunca se suben a Git)
```

Ejemplo:

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

### Por qué importa

- Separación entre **configuración y datos**
- Respaldos más fáciles (haz snapshot de `/opt/data`; el árbol de compose vive en Git)
- Repositorios Git más limpios (nunca se versionan datos de ejecución)
- Evita el "desorden" de Docker

> 💡 `/opt/stacks` es además el `DOCKGE_STACKS_DIR` por defecto de Dockge, así que
> el gestor de stacks lo detecta todo sin configuración adicional.

---

## 🚀 Clonar el repositorio homelab de RebelRx

```bash
cd ~
git clone https://github.com/rebelrx/rebelrx-homelab.git
```

> Este repositorio ofrece **plantillas de Compose del mundo real** usadas en producción,
> un directorio por stack, cada uno con su `README.md` completo.

Para desplegar un stack, cópialo a `/opt/stacks/` y rellena su `.env`:

```bash
sudo mkdir -p /opt/stacks
sudo cp -a ~/rebelrx-homelab/stacks/npm /opt/stacks/
```

---

## 📦 Estructura de las plantillas

Dentro del repositorio:

```bash
rebelrx-homelab/
└── stacks/
   ├── arr/
   ├── audiobooks/
   ├── authentik/
   └── ...
```

Cada stack incluye como mínimo:

- `compose.yaml`
- `.env.example`
- `README.md` (servicios, variables de entorno, puertos, despliegue, respaldo)

Algunos stacks incluyen archivos extra (p. ej. `paperless` tiene un `docker-compose.env.example`, `monitor` un `prometheus.yml.example`) — el README de cada stack lo indica.

---

## 🧠 Conceptos clave (lee esto primero)

### 1. Mentalidad Compose-first

Todo se define en YAML.

- Nada de hacer clics en interfaces gráficas
- Nada de crear contenedores manualmente
- Git = fuente de verdad

---

### 2. Archivos `.env`

Cada stack usa variables de entorno:

```env
PUID=1000
PGID=1000
TZ=America/New_York
```

**Por qué importa:**

- Permisos consistentes
- Configuraciones portables
- Fácil de sobrescribir

Los secretos (contraseñas, claves de API) también viven en `.env` — que está **ignorado por Git**.
Solo se suben plantillas `.env.example` con los secretos en blanco.

---

### 3. Mapeo de volúmenes

```yaml
volumes:
  - /opt/data/plex:/config
```

Esto garantiza:

- Los datos persisten entre reinicios del contenedor
- Respaldos fáciles (todo está bajo `/opt/data`)
- Control total sobre el almacenamiento

---

### 4. Publicación de puertos

```yaml
ports:
  - 8080:8080              # publicado en todas las interfaces — accesible en tu LAN
  # - 127.0.0.1:8080:8080  # solo loopback — con proxy inverso delante
```

**Cómo lo maneja el repositorio:**

- Las interfaces web se publican en **todas las interfaces por defecto**, para que
  funcionen en tu LAN sin configuración extra.
- Antepon `127.0.0.1:` a un mapeo para dejar un servicio **solo en loopback** y
  acceder a él exclusivamente a través del proxy inverso.
- Los **servicios internos** (bases de datos, Redis/Valkey, brokers) **no publican
  ningún puerto del host** — solo son accesibles en la red interna del stack.

---

## 🌐 Proxy inverso (Nginx Proxy Manager)

Enfoque recomendado — el stack `npm` del repositorio:

- Ejecuta **Nginx Proxy Manager (NPM)**
- Expón los servicios mediante subdominios
- Gestiona el SSL automáticamente

Los servicios se conectan a una red compartida `proxy_net`; NPM alcanza cada uno por
el nombre del contenedor, de modo que el proxy inverso funciona haya o no un puerto
publicado en el host.

Ejemplo:

```text
https://plex.tudominio.com
```

Ventajas:

- Sin malabares con puertos
- URLs limpias
- Control de acceso centralizado

---

## ▶️ Ejecutar tu primer stack

Empieza por el proxy inverso, para que todo lo demás tenga algo detrás de lo que colocarse:

```bash
cd /opt/stacks/npm

cp .env.example .env
nano .env
```

Después:

```bash
docker compose up -d
```

---

## 🔄 Actualizar contenedores

```bash
docker compose pull
docker compose up -d
```

## 🔄 Detener e iniciar contenedores

```bash
docker compose down
docker compose up -d
```

## 🔄 Reiniciar contenedores

```bash
docker compose restart [nombre_del_servicio]
```

💡 Descarga la "Ultimate Docker Compose Cheat Sheet" de DevOps Cycle: [Docker Compose Cheat Sheet](https://devopscycle.com/pdfs/the-ultimate-docker-compose-cheat-sheet.pdf)

---

## 🧼 Limpieza

Elimina recursos sin usar:

```bash
docker system prune -a
```

---

## ⚠️ Errores comunes

### ❌ Problemas de permisos

Los archivos de Compose pueden ser propiedad de tu usuario; los directorios de **datos**
suelen ser propiedad del `PUID:PGID` del contenedor (habitualmente `1000:1000`):

```bash
sudo chown -R $USER:$USER /opt/stacks/<stack>
sudo chown -R 1000:1000 /opt/data/<stack>   # ajústalo al PUID/PGID del stack
```

---

### ❌ Puertos ya en uso

```bash
ss -tulnp | grep :PUERTO
```

---

### ❌ Contenedores que no se actualizan

```bash
docker compose pull
```

---

### ❌ Editar contenedores en ejecución

No lo hagas.

Edita el **archivo de Compose** y vuelve a desplegar.

---

## 🧠 Reflexiones finales

Esta configuración se basa en unos pocos principios:

- **Mantenlo simple**
- **Define todo como código**
- **Sé dueño de tu infraestructura**
- **Evita capas innecesarias**

Solo Docker + Compose + disciplina.

---

## 🔗 Próximos pasos sugeridos

- Añade más stacks del repositorio de RebelRx
- Configura el proxy inverso con el stack `npm` (Nginx Proxy Manager)
- Implementa respaldos con el stack `backup` (Kopia)
- Usa un gestor de stacks de Docker Compose con el stack `dockge` (Dockge)
- Añade un explorador de archivos con el stack `filebrowser` (Filebrowser Quantum)

---

## ⚠️ Aviso legal

Esta guía tiene fines educativos.

Eres responsable de proteger y mantener tu propio sistema.
