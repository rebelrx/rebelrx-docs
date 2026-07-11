---
description: >-
  Monta el almacenamiento del NAS en un servidor Devuan como es debido — NFS vs SMB, entradas de fstab que sobreviven a los reinicios sin systemd, trampas de orden con Docker y el error de SQLite-sobre-NFS que corrompe las bases de datos del homelab.
---

<!--
Source: homelab/nas-mounting.md
Last translated: 2026-07
-->

# 🗂️ Montar el almacenamiento del NAS (Devuan)

Tu NAS guarda el almacenamiento masivo; tu servidor ejecuta los servicios. Esta guía los conecta de forma **fiable**: montajes que sobreviven a los reinicios, se comportan bien bajo sysvinit y no alimentan silenciosamente directorios vacíos a tus contenedores de Docker.

---

## ⚠️ Principio fundamental

> El almacenamiento debe ser aburrido. Un montaje en el que tienes que pensar es un montaje que te fallará.

---

## 🧭 ¿NFS o SMB?

Ambos funcionan. Elige según qué habla con qué:

| | NFS | SMB / CIFS |
| :--- | :--- | :--- |
| Ideal para | Linux ↔ Linux / NAS | Redes mixtas (Windows, impresoras, escáneres) |
| Rendimiento | Más rápido, menos sobrecarga | Algo más pesado |
| Permisos | UID/GID de Unix, nativo | Mapeados al montar |
| Modelo de autenticación | Basado en host/IP (v3/v4) | Usuario + contraseña |
| Veredicto | **Opción por defecto para un homelab Linux** | Úsalo cuando NFS no esté disponible o Windows comparta los datos |

Esta guía cubre ambos — NFS como vía principal.

---

## 🖥️ Preparación en el lado del NAS

En el NAS (QNAP, Synology, TrueNAS — la interfaz cambia, los conceptos no):

1. Crea o elige la carpeta compartida (p. ej., `media`, `backups`)
2. Habilita el **servicio NFS** y añade una regla NFS para el recurso compartido:
    - Cliente permitido → la IP de tu servidor (o su IP de Tailscale si montas a través de la tailnet)
    - Acceso → lectura/escritura
    - Squash → *no root squash* solo si los contenedores de Docker deben escribir como root; de lo contrario, mapea a un UID/GID concreto
3. Para SMB → crea un usuario del NAS dedicado y de bajos privilegios para el servidor; nunca montes con la cuenta de administrador del NAS

!!! tip "Un recurso compartido por propósito"

    Compartidos separados para `media`, `backups` y `archive` significan permisos
    separados, reglas NFS separadas y la posibilidad de desmontar uno sin
    perturbar a los demás. Resiste la tentación del gigantesco `share-de-todo`.

---

## 📦 Instalar los paquetes cliente (Devuan)

```bash
# NFS
sudo apt update
sudo apt install -y nfs-common

# SMB (solo si lo vas a usar)
sudo apt install -y cifs-utils
```

---

## 🧪 Prueba el montaje primero (siempre)

Nunca vayas directo a `fstab`. Demuestra que el montaje funciona de forma interactiva:

```bash
# Ver qué exporta el NAS
showmount -e 192.168.1.xx

# Crear el punto de montaje y probar
sudo mkdir -p /mnt/nas/media
sudo mount -t nfs -o vers=4.1 192.168.1.xx:/media /mnt/nas/media

# Verificar: listarlo, escribir en él, leerlo de vuelta
ls /mnt/nas/media
touch /mnt/nas/media/.write-test && rm /mnt/nas/media/.write-test
```

Si la prueba de escritura falla, arregla los permisos **ahora** (mira Solución de problemas) — un montaje roto en `fstab` es mucho menos divertido de depurar en el arranque.

Desmonta antes de hacerlo permanente:

```bash
sudo umount /mnt/nas/media
```

---

## 📌 Montajes permanentes vía /etc/fstab

### NFS

```bash
sudo nano /etc/fstab
```

Añade una línea por recurso compartido:

```text
192.168.1.xx:/media    /mnt/nas/media    nfs    rw,hard,vers=4.1,_netdev,nofail    0    0
192.168.1.xx:/backups  /mnt/nas/backups  nfs    rw,hard,vers=4.1,_netdev,nofail    0    0
```

Qué te aporta cada opción:

- `hard` → si el NAS se cae, la E/S **espera** a que vuelva en lugar de devolver errores silenciosamente y corromper escrituras. Lo correcto para datos que te importan.
- `vers=4.1` → fija la versión del protocolo; sin sorpresas de negociación tras actualizaciones de firmware del NAS
- `_netdev` → indica a los scripts de init que esto necesita red primero — **esencial en sysvinit**
- `nofail` → un NAS apagado no colgará todo el arranque del servidor

Aplica y verifica:

```bash
sudo mount -a
df -h | grep nas
```

### SMB

Las credenciales nunca van en `fstab` (es legible por todo el mundo). Usa un archivo de credenciales:

```bash
sudo nano /root/.smb-credentials
```

```text
username=serversvc
password=tu-contraseña-robusta
```

```bash
sudo chmod 600 /root/.smb-credentials
```

Después, en `/etc/fstab`:

```text
//192.168.1.xx/media  /mnt/nas/media  cifs  credentials=/root/.smb-credentials,uid=1000,gid=1000,vers=3.0,_netdev,nofail  0  0
```

El `uid=1000,gid=1000` asigna cada archivo a tu usuario — ajústalo para que coincida con el `PUID`/`PGID` con el que corren tus [stacks de Docker](docker-home-lab.md).

---

## 🔁 Comportamiento en el arranque sin systemd

En Devuan, la secuencia de arranque de sysvinit gestiona esto correctamente **gracias a `_netdev`**: la etapa `mountnfs` se ejecuta después de que la red esté levantada y monta todo lo que fstab marca como dependiente de la red. Sin units, sin configuraciones de automontaje — el mecanismo de hace 30 años simplemente funciona.

Verifícalo tras tu próximo reinicio:

```bash
df -h | grep nas
```

!!! tip "Montaje bajo demanda con autofs (opcional)"

    Si el NAS está a veces apagado, `autofs` monta los recursos compartidos solo
    cuando se accede a ellos y los libera tras un periodo de inactividad — sin
    dependencia alguna en el arranque:

    ```
    sudo apt install -y autofs
    ```

    Después mapea tus recursos compartidos en `/etc/auto.master.d/`. Para un NAS
    siempre encendido, el fstab plano es más simple y mejor; recurre a autofs solo
    cuando el NAS sea genuinamente intermitente.

---

## 🐳 La trampa de Docker

Este es el modo de fallo que muerde a todo el mundo una vez:

**Si Docker arranca antes de que el montaje del NAS aterrice, los contenedores bind-montan un directorio vacío** — y algunas apps inicializan alegremente una biblioteca nueva y vacía dentro. Jellyfin escaneando una carpeta de medios vacía borra sus metadatos; un trabajo de respaldo que ve un origen vacío "tiene éxito" respaldando nada.

Defensas, por orden de valor:

**1. Deja que el orden de arranque haga su trabajo.** En sysvinit, `mountnfs` corre antes que el script de init de Docker — con `_netdev` configurado, un NAS *en línea* se monta antes de que arranquen los contenedores. El riesgo es que el NAS esté lento o apagado (lo que `nofail` permite deliberadamente).

**2. Protege los stacks que dependen del montaje.** Añade una comprobación a la secuencia de init de Docker — crea `/etc/init.d/wait-for-nas`:

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          wait-for-nas
# Required-Start:    $network $remote_fs
# Required-Stop:
# Default-Start:     2 3 4 5
# Default-Stop:
# X-Start-Before:    docker
# Short-Description: Delay boot until NAS shares are mounted
### END INIT INFO

case "$1" in
  start)
    for i in $(seq 1 30); do
      mountpoint -q /mnt/nas/media && exit 0
      sleep 2
    done
    echo "wait-for-nas: NAS not mounted after 60s, continuing anyway"
    ;;
  *) ;;
esac
exit 0
```

```bash
sudo chmod +x /etc/init.d/wait-for-nas
sudo update-rc.d wait-for-nas defaults
```

**3. Haz que el vacío sea detectable.** Coloca un archivo marcador en el recurso compartido del NAS (`touch /mnt/nas/media/.nas-mounted` mientras está montado). Cualquier script — especialmente los de respaldo — puede entonces negarse a ejecutarse contra una ruta sin montar:

```bash
[ -f /mnt/nas/media/.nas-mounted ] || { echo "NAS no montado, abortando"; exit 1; }
```

> El truco del archivo marcador no cuesta nada y ha salvado más datos de homelab que ninguna otra terna de líneas de esta guía.

---

## 🚫 Qué no hacer

!!! danger "Nunca ejecutes bases de datos SQLite sobre NFS o SMB"

    El stack Arr (Sonarr, Radarr, Prowlarr…), Jellyfin y muchas apps de homelab
    usan **SQLite**, que depende de un bloqueo de archivos que los sistemas de
    archivos en red implementan de forma poco fiable. Las bases de datos en
    montajes de NAS se corrompen — no es cuestión de *si*, sino de *cuándo*.

    La regla de la [guía de Docker](docker-home-lab.md) ya lo resuelve:

    - Datos de apps y bases de datos → `/opt/data/<stack>` en **disco local**
    - Medios masivos, documentos, respaldos → montaje del NAS

- No montes con la cuenta de administrador del NAS — usuario dedicado de bajos privilegios, por recurso compartido
- No uses montajes `soft` para nada que escriba — las escrituras parciales silenciosas son la manera en que los archivos se corrompen
- No pongas credenciales en `fstab` — archivo de credenciales, `chmod 600`
- No te saltes la prueba de montaje interactiva — fstab es donde *registras* un montaje que funciona, no donde *descubres* uno roto

---

## 🧰 Solución de problemas

- **`access denied by server`** → la regla NFS del NAS no incluye la IP de tu servidor, o la ruta de exportación es errónea. Revísalo con `showmount -e <ip-del-nas>`.
- **Archivos propiedad de `nobody:nogroup`** → desajuste del mapeo de IDs de NFSv4. La solución más simple en una LAN doméstica: haz que el UID/GID del recurso compartido del NAS coincida con tu usuario del servidor (1000:1000), o configura la opción de squash del recurso para mapear todo acceso a ese UID.
- **`Stale file handle`** → la exportación cambió en el lado del NAS mientras estaba montada. `sudo umount -l /mnt/nas/media && sudo mount -a`.
- **Los comandos se cuelgan para siempre en el punto de montaje** → el NAS está caído y el montaje es `hard` (funcionando según lo diseñado — seguridad de los datos por encima de la capacidad de respuesta). Levanta el NAS de nuevo, o `sudo umount -f -l` para liberarlo a la fuerza.
- **Transferencias SMB lentas** → confirma `vers=3.0` o superior en las opciones de montaje; la negociación SMB1 todavía acecha en los valores por defecto de NAS antiguos y es tanto lenta como insegura.

---

## 🧠 Reflexión final

Un NAS al que no puedes acceder de forma fiable no es más que un calefactor muy caro.

Haz que los montajes sean aburridos — probados, fijados, ordenados, protegidos — y el resto del homelab podrá construirse sobre un almacenamiento en el que nunca tiene que pensar.

> El almacenamiento aburrido es el cimiento sobre el que se sostiene todo lo demás.
