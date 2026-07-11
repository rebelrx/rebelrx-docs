---
description: >-
  Por qué Linux es el último gran sistema operativo que te permite desvincularte — control, transparencia, conceptos fundamentales, hardware y cómo es realmente una instalación.
---

<!--
Source: linux/why-linux.md
Last translated: 2026-07
-->

# 🐧 Por qué Linux

La mayoría de la gente no elige un sistema operativo.

Se lo eligen *por ellos*.

* Windows → atado a la identidad y la telemetría de Microsoft
* macOS → atado al ecosistema y al hardware de Apple
* SO móviles → plataformas totalmente cerradas y vinculadas a la identidad

Linux es diferente.

> Linux es el único gran sistema operativo que todavía te permite desvincularte.

---

## 🧭 Qué cubre esta página

* Por qué Linux existe como alternativa
* Qué ganas (y a qué renuncias)
* Términos clave que necesitas entender
* Cómo es realmente una instalación de Linux

---

## ⚔️ Por qué elegir Linux

### 🔐 Control

Linux te da plena autoridad sobre:

* La configuración de tu sistema
* El software instalado
* El comportamiento de la red
* Las políticas de actualización

Sin actualizaciones forzadas.
Sin cuentas obligatorias.
Sin procesos ocultos que no puedas inspeccionar.

> El administrador eres tú, no el fabricante.

---

### 🧱 Transparencia

Linux es de código abierto.

Eso significa:

* El código puede inspeccionarse
* El comportamiento puede verificarse
* Las puertas traseras son más difíciles de ocultar a gran escala

Compáralo con:

* Windows → cerrado, opaco
* macOS → parcialmente abierto, mayormente controlado

---

### 🚫 Sin ecosistema forzado

Las plataformas de sistemas operativos (SO) modernas son cada vez más:

* Dependientes de cuentas
* Dependientes de la nube
* Vinculadas a la identidad

Linux te permite:

* Ejecutar sistemas completamente sin conexión
* Evitar el bloqueo por cuentas
* Usar tu máquina sin validación externa

---

### ⚡ Eficiencia

Los sistemas Linux suelen ser:

* Más rápidos
* Menos exigentes en recursos
* Más estables a largo plazo

Especialmente cuando evitas los valores por defecto sobrecargados.

---

## ⚖️ El compromiso

Linux no es "gratis" en el sentido en que la gente cree.

Estás intercambiando:

| Comodidad | Control |
| ------------------ | -------------- |
| Apps plug-and-play | Configuración manual |
| Soporte del fabricante | Autosuficiencia |
| Experiencia de usuario familiar | Curva de aprendizaje |

> Linux recompensa a quienes están dispuestos a entender su sistema.

---

## 🧠 Conceptos fundamentales (glosario)

Antes de instalar Linux, deberías entender unos cuantos términos clave.

---

### 🧬 El kernel

El kernel es el núcleo del sistema operativo.

Se sitúa entre:

* Tu hardware (CPU, RAM, disco, GPU)
* Tu software (aplicaciones, servicios)

El kernel de Linux es responsable de:

* La comunicación con el hardware
* La gestión de procesos
* La gestión de la memoria
* Los límites de seguridad del sistema

> Todo en Linux pasa, en última instancia, por el kernel.

Esto es importante porque:

* Todas las distribuciones de Linux comparten la **misma base de kernel**
* La diferencia entre distros está en todo lo que rodea *al* kernel

---

### 💻 CLI (interfaz de línea de comandos)

Una forma de interactuar con tu sistema basada en texto. También se la conoce como la "terminal".

En lugar de hacer clic:

* Escribes comandos
* Controlas el sistema directamente desde una terminal de texto

Ejemplo:

```bash
ls
cd /home
cp file.txt backup.txt
```

Todos los SO modernos incluyen una CLI integrada. ¡Puede que ni supieras que estaba ahí en Windows o macOS!

En Windows, la CLI recibe varios nombres, entre ellos:

* Terminal
* Símbolo del sistema (Command Prompt)
* Windows PowerShell

En macOS, la CLI se llama Terminal.

Sin embargo, Windows y macOS tratan la CLI como algo opcional y reservado para configuraciones avanzadas.

Windows y macOS favorecen en cambio la interfaz gráfica de usuario (GUI) para el usuario cotidiano (simplemente **apuntar y hacer clic**)

> La CLI es fundamental en Linux y, en realidad, no es opcional.

!!! tip "Curva de aprendizaje de la CLI (qué esperar)"

    Si vienes de Windows o macOS, la CLI te resultará incómoda al principio.

    Es normal.

    Al principio, puede que te encuentres:

    - Copiando y pegando comandos que no entiendes del todo  
    - Buscando cómo hacer tareas básicas (instalar apps, navegar por archivos)  
    - Cometiendo pequeños errores que rompen cosas temporalmente  

    Esta fase pasa rápido.

    En poco tiempo, la CLI se convierte en:

    - Algo más rápido que los flujos de apuntar y hacer clic  
    - Más precisa y repetible  
    - Una ventaja central de usar Linux  

    > El objetivo no es memorizar comandos, sino entender cómo funciona el sistema.

---

### 🐚 Shell (bash, zsh, etc.)

El shell es lo que interpreta tus comandos.

Shells comunes:

* `bash` → el predeterminado en la mayoría de sistemas
* `zsh` → más personalizable

Piensa en él como:

> El idioma que usas para hablar con tu sistema dentro de la CLI.

---

### 📦 Gestor de paquetes

El gestor de paquetes es la forma en que Linux instala software.

En lugar de descargar archivos `.exe` (como en Windows):

* Instalas desde repositorios

Ejemplos:

* `apt` → Debian / Devuan
* `pacman` → Arch / Artix

Comando de ejemplo en Devuan para instalar la app Nginx (usada para servir webs):

```bash
sudo apt install nginx
```

---

### 🧩 Distribución (distro)

Las distribuciones (o distros) son distintos "sabores" de Linux.

Cada distro incluye:

* Un gestor de paquetes
* Herramientas por defecto
* Una filosofía de sistema

Ejemplos:

* Devuan → estable, orientada a servidores, sin systemd
* Artix → basada en Arch, flexible, sin systemd

> Linux no es un solo SO; es una familia de sistemas.

---

### ⚙️ Sistema de init

El sistema que arranca todo durante el inicio.

Ejemplos:

* `systemd` → dominante, centralizado
* `OpenRC`, `runit`, `s6` → más simples, modulares

Las guías de Linux de RebelRx priorizan:

> Sistemas sin systemd para una mayor transparencia y control.

---

### 🗂️ Estructura del sistema de archivos

Linux no usa letras de unidad como `C:\`.

En su lugar:

* Todo está bajo `/` (la raíz)

---

### 📁 Directorios principales

Estos son los directorios principales que encontrarás:

* `/` → la raíz de todo el sistema de archivos
* `/home` → archivos de usuario (tus documentos, descargas, configuraciones)
* `/root` → directorio home del usuario root
* `/etc` → archivos de configuración de todo el sistema
* `/var` → datos variables (logs, bases de datos, cachés)
* `/usr` → software instalado por el usuario y aplicaciones del sistema
* `/bin` → binarios de comandos esenciales (comandos básicos del sistema)
* `/sbin` → binarios del sistema (comandos administrativos)
* `/lib` → bibliotecas compartidas requeridas por los binarios
* `/tmp` → archivos temporales (a menudo se vacían al reiniciar)
* `/opt` → software opcional / de terceros
* `/mnt` → puntos de montaje temporales (montajes manuales)
* `/media` → medios extraíbles (unidades USB, discos externos)
* `/boot` → archivos del gestor de arranque e imágenes del kernel
* `/dev` → archivos de dispositivo (interfaces de hardware)
* `/proc` → sistema de archivos virtual que expone información del sistema y los procesos
* `/sys` → datos de la interfaz del kernel y el hardware

---

### 🧠 Cómo pensarlo

* Hay **un único sistema de archivos unificado**
* Las unidades se "montan" en directorios (no se les asignan letras)
* Todo vive en algún lugar bajo `/`

Ejemplo:

* Una unidad USB podría aparecer en: `/media/usb-drive`
* Un NAS montado podría estar en: `/mnt/nas`

> No cambias de unidad — navegas un único árbol.

---

### 🔌 Dispositivos en `/dev`

En Linux:

> Los dispositivos de hardware se exponen como archivos.

Ejemplos comunes:

* `/dev/sda` → primer dispositivo de almacenamiento (disco)
* `/dev/sdb` → segundo dispositivo de almacenamiento
* `/dev/sda1` → primera partición del primer disco
* `/dev/nvme0n1` → SSD NVMe
* `/dev/nvme0n1p1` → partición en la unidad NVMe
* `/dev/ttyUSB0` → dispositivo serie USB
* `/dev/null` → descarta toda entrada (agujero negro)
* `/dev/random` → generador de números aleatorios

Ejemplo de uso:

```bash
lsblk
```

Muestra discos como:

* `sda`, `sdb`, `nvme0n1`, etc.

---

### ⚠️ Importante

Como todo se trata como un archivo:

* Escribir en el dispositivo equivocado (p. ej., `/dev/sda`) puede sobrescribir un disco
* Montar incorrectamente puede ocultar o reemplazar el contenido de directorios

> Esto es potente, pero exige atención.

---

### 🧠 Idea clave

Linux trata:

* Los archivos
* Los dispositivos
* Los procesos

como parte del mismo sistema unificado.

> Cuando entiendes el sistema de archivos, entiendes cómo está organizado Linux.

---

### 🔐 Root vs usuario

* `root` → control total del sistema
* `user` → acceso limitado

Elevas privilegios usando:

```bash
sudo comando
```

---

## 💻 Requisitos de hardware

Una de las mayores ventajas de Linux:

> Linux corre en casi cualquier cosa.

---

### 🧱 La realidad mínima

Linux puede correr en:

* Portátiles antiguos (de más de 10 años)
* Mini PC de bajo consumo
* Servidores
* Equipos de sobremesa y portátiles modernos

En muchos casos:

> Hardware que se siente "lento" en Windows vuelve a ser totalmente usable en Linux.

---

### ⚡ Hardware moderno

Linux también escala hacia arriba:

* CPUs multinúcleo
* GPUs de gama alta
* Configuraciones con mucha RAM

Sin embargo, la compatibilidad depende de:

* Los drivers de GPU (NVIDIA puede requerir configuración extra)
* Los chipsets Wi-Fi
* Hardware nuevo o de nicho

!!! warning "Compatibilidad de GPUs NVIDIA"

    Las GPUs NVIDIA pueden requerir configuración adicional en Linux.

    A diferencia de la mayoría del hardware, los drivers de NVIDIA no son totalmente
    de código abierto y a menudo necesitan:

    - Instalación manual del driver  
    - Hacer coincidir las versiones del driver con tu kernel  
    - Configuración extra para Wayland o entornos de escritorio  

    En cambio, las GPUs AMD suelen funcionar de fábrica con drivers de código abierto.

    > Si eres nuevo en Linux, el hardware AMD suele ser la experiencia más fluida.

---

### 🧠 Idea clave

* Windows y macOS son **ecosistemas restringidos por hardware**
* Linux es **flexible en hardware**

> El hardware lo eliges tú. No el fabricante.

---

## 🧩 Filosofía de hardware (recomendación: Framework)

Si vas a comprar hardware nuevo, elige fabricantes alineados con los principios de Linux.

### 🔧 Framework Computer

Framework es una de las pocas empresas alineadas con:

* El derecho a reparar
* El diseño de hardware modular
* La compatibilidad con Linux

Los sistemas Framework te permiten:

* Reemplazar componentes en lugar de reemplazar todo el equipo
* Actualizar con el tiempo
* Sin obsolescencia de hardware forzada

> Tu hardware debería ser tan soberano como tu software.

👉 Echa un vistazo a los portátiles y el sobremesa de Framework: <https://frame.work/>

---

## 🛠️ Cómo es realmente una instalación de Linux

Instalar Linux es fundamentalmente diferente de Windows o macOS.

---

### 🧱 Windows / macOS

* Arrancar el instalador
* Hacer clic por la interfaz gráfica
* El sistema se autoconfigura
* Cuenta obligatoria
* Listo

---

### 🐧 Linux (flujo típico)

Según la distro, espera:

1. Arrancar en un entorno live
2. Particionar los discos manualmente (o semimanualmente)
3. Instalar el sistema base
4. Configurar:

   * Usuarios
   * Red
   * Gestor de arranque
5. Instalar el entorno de escritorio (si hace falta)
6. Reiniciar en tu sistema

!!! warning "Riesgo del particionado de discos"

    Las instalaciones de Linux suelen requerir particionar el disco manualmente.

    Es uno de los pocos pasos en los que puedes, por accidente:

    - Borrar datos existentes  
    - Sobrescribir otro sistema operativo  
    - Configurar mal tu arranque  

    Siempre:

    - Haz una copia de seguridad de los datos importantes antes de instalar  
    - Comprueba dos veces los discos y particiones seleccionados  
    - Ten claro si estás reemplazando o haciendo arranque dual  

    > El particionado es potente — pero asume que sabes lo que haces.

---

### 🔧 La realidad tras la instalación

Después de instalar, no has "terminado".

Vas a:

* Instalar el software esencial
* Configurar servicios
* Configurar la red y el cortafuegos
* Ajustar el rendimiento y la usabilidad

> Linux se construye, no te lo entregan hecho.

---

## 🧭 Para quién es Linux

Linux encaja bien contigo si:

* Valoras la privacidad y la independencia
* Quieres control total sobre tu sistema
* Estás dispuesto a aprender y resolver problemas
* Prefieres la estabilidad a largo plazo antes que la comodidad

---

## 🚫 Para quién no es

Linux puede frustrarte si:

* Esperas que todo sea plug-and-play
* Dependes mucho de software propietario (Microsoft 365, Adobe, etc.)
* Quieres cero mantenimiento y cero aprendizaje

---

## 🧠 Reflexión final

La mayoría de los sistemas operativos avanzan hacia:

* La imposición de identidad
* El control de plataforma
* La reducción de la autonomía del usuario

Linux es uno de los últimos entornos donde:

> El usuario sigue estando al mando.

Pero solo si lo eliges de forma intencionada.

---

## ➡️ Siguiente paso

Si esto encaja con tus objetivos:

* Empieza con una instalación guiada
* Acepta la curva de aprendizaje
* Construye tu sistema de forma deliberada

👉 Continúa con:

* [Instalación manual de escritorio Artix](artix-kde-openrc-install.md)
* [Instalación de servidor Devuan](devuan-server-install.md)
