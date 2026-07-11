---
description: >-
  Una estrategia de respaldo por capas y probada para tu homelab con Docker — Kopia al NAS local, Borg cifrado fuera de casa, volcados de bases de datos y los simulacros de restauración que la hacen real.
---

<!--
Source: homelab/backup-and-recovery.md
Last translated: 2026-07
-->

# 💾 Respaldo y recuperación (homelab con Docker)

Una guía práctica para construir un **sistema de respaldo por capas y probado** para el [homelab con Docker](docker-home-lab.md), y demostrar que funciona antes de que lo necesites.

---

## ⚠️ Principio fundamental

> Si tu estrategia de respaldo no está probada, no existe.

Los respaldos no son un producto que instalas. Son una **disciplina**: copias en los lugares correctos, automatizadas sin ti, y restauradas con regularidad para demostrar que son reales.

---

## 🧭 La regla 3-2-1

La base que toda estrategia debería cumplir:

- **3** copias de tus datos (el original + dos respaldos)
- **2** medios o sistemas de almacenamiento diferentes
- **1** copia fuera de casa

Para este homelab, el mapeo es directo:

| Copia | Dónde | Herramienta |
| :--- | :--- | :--- |
| 1 (en vivo) | Servidor → `/opt/data` | — |
| 2 (local) | NAS | Kopia |
| 3 (fuera de casa) | Repositorio remoto cifrado | Borg (borgmatic) |

> Los respaldos locales protegen contra errores y discos muertos. El respaldo externo protege contra incendios, robos y ransomware.

---

## 📦 Qué respaldar (y qué no)

No todos los datos merecen el mismo trato. Clasifícalos en niveles:

### 🔴 Irreemplazables (respaldar en todos los niveles)

- Datos de las apps → `/opt/data/<stack>` (biblioteca de Immich, documentos de Paperless, archivos de Nextcloud, bóveda de Vaultwarden)
- Volcados de bases de datos (ver más abajo)
- Fotos, documentos, archivos personales

### 🟡 Recuperables con esfuerzo (respaldar localmente)

- Configuración de servicios aún no capturada en Git
- Bibliotecas de metadatos que tardaron años en curarse

### 🟢 Readquiribles (no respaldar)

- Bibliotecas de medios gestionadas por el stack Arr
- Imágenes de contenedores (están a un `docker compose pull` de distancia)
- Cachés, transcodificaciones, miniaturas

!!! tip "Tu árbol de compose ya está respaldado"

    Siguiendo la [estructura del homelab](docker-home-lab.md), todo lo que hay en
    `/opt/stacks/` vive en Git (Forgejo). Ese es tu respaldo de infraestructura —
    y precisamente por eso los secretos se quedan en archivos `.env` y fuera del repositorio.

    Esta guía cubre la otra mitad: `/opt/data/`.

---

## 🗄️ Bases de datos: el asesino silencioso de respaldos

Copiar los archivos de una base de datos **en ejecución** produce un respaldo corrupto con la frecuencia suficiente para arruinar la única restauración que importa. Postgres, MySQL y MariaDB escriben constantemente; una instantánea a nivel de archivo en mitad de una escritura es un volado.

La solución es simple: **volcar primero y respaldar después los volcados.**

Crea `/opt/scripts/db-dumps.sh`:

```bash
#!/bin/sh
# Volcados nocturnos de bases de datos — copias seguras a nivel de archivo para que las recojan los trabajos de respaldo
set -eu

DUMP_ROOT=/opt/data/db-dumps
mkdir -p "$DUMP_ROOT"

# Contenedores Postgres: docker exec + pg_dumpall
for c in immich-postgres paperless-db joplin-db forgejo-db; do
  docker exec "$c" sh -c 'pg_dumpall -U "$POSTGRES_USER"' | \
    gzip > "$DUMP_ROOT/${c}.sql.gz.tmp" && \
    mv "$DUMP_ROOT/${c}.sql.gz.tmp" "$DUMP_ROOT/${c}.sql.gz"
done

# Contenedores MariaDB
for c in seafile-mysql romm-db; do
  docker exec "$c" sh -c 'mariadb-dump -uroot -p"$MYSQL_ROOT_PASSWORD" --all-databases' | \
    gzip > "$DUMP_ROOT/${c}.sql.gz.tmp" && \
    mv "$DUMP_ROOT/${c}.sql.gz.tmp" "$DUMP_ROOT/${c}.sql.gz"
done
```

Ajusta las listas de contenedores a tus stacks y prográmalo cada noche **antes** de que corran los trabajos de respaldo:

```bash
sudo chmod +x /opt/scripts/db-dumps.sh
sudo crontab -e
```

```text
# Volcados a la 01:00, respaldo local a las 02:00, externo a las 03:00
0 1 * * * /opt/scripts/db-dumps.sh
```

> El patrón `.tmp` + `mv` garantiza que un respaldo nunca recoja un volcado a medio escribir.

---

## 🟢 Nivel 2 — Respaldos locales con Kopia (servidor → NAS)

[Kopia](https://kopia.io/) ofrece instantáneas rápidas, deduplicadas y cifradas con una interfaz web limpia — ideal para el nivel local de alta frecuencia.

**Prerequisito:** tu NAS montado en el servidor (p. ej., en `/mnt/nas/backups` vía NFS).

### Stack de Compose

`/opt/stacks/backup/compose.yaml`:

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
      - --insecure                  # la UI se accede solo vía Tailscale; TLS opcional
      - --address=0.0.0.0:51515
      - --server-username=${KOPIA_UI_USER}
      - --server-password=${KOPIA_UI_PASSWORD}
    ports:
      - "51515:51515"
    volumes:
      - /opt/data/kopia/config:/app/config
      - /opt/data/kopia/cache:/app/cache
      - /opt/data/kopia/logs:/app/logs
      - /opt/data:/source/opt-data:ro        # lo que respaldamos (solo lectura)
      - /mnt/nas/backups/kopia:/repository   # adónde va
```

!!! warning "Monta los orígenes en solo lectura"

    El `:ro` en el montaje de origen significa que un contenedor de respaldo
    comprometido o con mal comportamiento **no puede alterar los datos que protege**.
    Las herramientas de respaldo necesitan acceso de lectura — nunca de escritura —
    a tus datos en vivo.

### Inicializar y programar

Abre `http://server:51515` (a través de Tailscale), crea el repositorio en `/repository` y define una política de instantáneas para `/source/opt-data`:

- **Programación** → diaria a las 02:00
- **Retención** → 7 diarias, 4 semanales, 6 mensuales
- **Exclusiones** → directorios de caché, carpetas de transcodificación, cualquier cosa del nivel 🟢

La contraseña del repositorio (`KOPIA_REPO_PASSWORD`) cifra todo en reposo.

!!! danger "Guarda la contraseña del repositorio fuera del sistema que protege"

    Un respaldo cifrado con una contraseña que solo existe en el servidor muerto
    no es un respaldo. Guarda las contraseñas y claves de los repositorios en tu
    gestor de contraseñas **y** en una copia offline (impresa, en un lugar seguro).

---

## 🔴 Nivel 3 — Fuera de casa con Borg (borgmatic)

[Borg](https://www.borgbackup.org/) proporciona archivos cifrados, deduplicados y con capacidad de solo-añadir; [borgmatic](https://torsion.org/borgmatic/) lo envuelve en un único archivo de configuración declarativo.

Para el extremo remoto, elige un host nativo de Borg — [BorgBase](https://www.borgbase.com/) y [rsync.net](https://rsync.net/) son las opciones consolidadas, o bien otra máquina que controles (la casa de un familiar, un segundo emplazamiento) funciona igual de bien y lo mantiene todo en casa.

> Todo se cifra **en el lado del cliente** antes de salir de tu servidor. El host remoto almacena texto cifrado que no puede leer.

### Stack de Compose

Añade al mismo stack `backup`:

```yaml
  borgmatic:
    image: ghcr.io/borgmatic-collective/borgmatic:latest
    container_name: borgmatic
    restart: unless-stopped
    environment:
      - BORG_PASSPHRASE=${BORG_PASSPHRASE}
      - TZ=${TZ}
    volumes:
      - /opt/data:/mnt/source:ro                          # origen (solo lectura)
      - /opt/data/borgmatic/config:/etc/borgmatic.d       # config + crontab
      - /opt/data/borgmatic/borg-config:/root/.config/borg
      - /opt/data/borgmatic/cache:/root/.cache/borg
      - /opt/data/borgmatic/ssh:/root/.ssh                # clave para el repo remoto
```

### Configuración

`/opt/data/borgmatic/config/config.yaml`:

```yaml
source_directories:
  - /mnt/source

exclude_patterns:
  - '*/cache/*'
  - '*/transcodes/*'
  - /mnt/source/kopia          # no respaldes la caché de la herramienta de respaldo local

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

`/opt/data/borgmatic/config/crontab.txt`:

```text
0 3 * * * PATH=$PATH:/usr/local/bin borgmatic --verbosity -1 --syslog-verbosity 1
```

Inicializa el repositorio una sola vez:

```bash
docker exec borgmatic borgmatic repo-create --encryption repokey-blake2
docker exec borgmatic borgmatic key export --paper   # imprímelo. en serio.
```

!!! tip "¿Por qué Kopia y Borg a la vez?"

    Dos herramientas independientes significan que una mala actualización, un bug o
    la corrupción de un repositorio no pueden acabar con todas las copias de tus
    datos a la vez. El nivel local optimiza restauraciones rápidas y frecuentes; el
    externo optimiza la supervivencia ante desastres.

    ¿Solo vas a usar una? Borg + borgmatic hacia el NAS **y** hacia el exterior
    cubre el 3-2-1 con una sola cadena de herramientas.

---

## 🔁 Pruebas de restauración (la parte que todos se saltan)

Un respaldo sin probar es una esperanza, no un plan. Pon un **simulacro de restauración trimestral** en tu calendario. Treinta minutos, cuatro comprobaciones:

### 1. Restaurar un archivo (Kopia)

Elige un archivo real, restáuralo en otro sitio y verifica su contenido:

```bash
docker exec kopia kopia snapshot list /source/opt-data
docker exec kopia kopia restore <snapshot-id>/immich/library/some-photo.jpg /tmp/restore-test/
```

### 2. Restaurar un archivo (Borg)

```bash
docker exec borgmatic borgmatic list
docker exec borgmatic borgmatic extract --archive latest \
  --path mnt/source/paperless/media/documents \
  --destination /tmp/restore-test/
```

### 3. Restaurar una base de datos

La que de verdad importa:

```bash
# En un contenedor desechable — nunca tu BD en producción
docker run -d --name pg-restore-test -e POSTGRES_PASSWORD=test postgres:17
zcat /opt/data/db-dumps/paperless-db.sql.gz | \
  docker exec -i pg-restore-test psql -U postgres
docker exec pg-restore-test psql -U postgres -c '\l'   # ¿existen las bases de datos? ¿los recuentos de filas son razonables?
docker rm -f pg-restore-test
```

### 4. Ensaya el desastre sobre el papel

Pregúntate: *el servidor ha desaparecido; ¿cuál es la secuencia?* Deberías poder escribirla de memoria:

1. Instalar Devuan → [guía del servidor](../linux/devuan-server-install.md)
2. Instalar Docker → [guía del homelab](docker-home-lab.md)
3. Clonar el repositorio del homelab desde Forgejo → `/opt/stacks` restaurado
4. Restaurar `/opt/data` desde el NAS (Kopia) — o desde el exterior (Borg) si el NAS también murió
5. Restaurar los volcados de bases de datos en contenedores de BD nuevos
6. `docker compose up -d`, stack por stack

Si algún paso te hace decir "eso tendría que averiguarlo", avérigualo **ahora** y déjalo por escrito.

---

## 📟 Monitorización (el fallo silencioso es lo predeterminado)

Los respaldos fallan en silencio — un disco lleno, una clave caducada, un NAS sin montar — y te enteras a la hora de restaurar. Conéctalos a tu monitorización existente:

- Añade un monitor push en **Uptime Kuma** y hazle ping al final de cada ejecución de respaldo; alerta si no llega el ping a su hora
- borgmatic lo soporta de forma nativa — añade a `config.yaml`:

```yaml
uptime_kuma:
  push_url: https://uptime-kuma.local/api/push/<token>
```

- Revisa `docker logs kopia` y `docker logs borgmatic` tras cualquier cambio de infraestructura

> Monitoriza la **ausencia de éxito**, no solo la presencia de errores.

---

## 🚫 Qué no hacer

- No respaldes archivos de bases de datos en vivo; vuelca primero
- No des a los contenedores de respaldo acceso de escritura a los datos de origen
- No guardes las claves de cifrado solo en la máquina que se está respaldando
- No dejes que la redundancia del RAID o del NAS se haga pasar por respaldo — protege contra el fallo de discos, no contra el borrado, la corrupción o el ransomware
- No lo "configures y olvides" — así es como descubres en el año tres que los respaldos se detuvieron en el año uno

---

## 🧠 Reflexión final

Nadie lamenta la hora invertida en probar una restauración.

Mucha gente lamenta los años de fotos, documentos e infraestructura que se esfumaron con un solo disco.

> Tus datos son tan soberanos como tu capacidad de recuperarlos.
