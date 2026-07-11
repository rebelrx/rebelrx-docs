---
description: >-
  El stack completo de RebelRx — hardware, sistemas operativos, servicios autoalojados y las herramientas de privacidad que se usan realmente a diario.
---

<!--
Source: privacy/my-setup.md
Last translated: 2026-07
-->

# 🧪 La configuración de privacidad de RebelRx

Este es mi propio stack real, centrado en la privacidad y diseñado para el **control, el rendimiento y la usabilidad**.

Uso esta configuración a diario e iré actualizando estas secciones continuamente a medida que aparezcan servicios más nuevos y mejores.

---

## 🧭 Filosofía

Esta configuración se construye en torno a:

- **Propiedad** → Tus datos permanecen bajo tu control  
- **Practicidad** → Las herramientas deben funcionar de verdad en el día a día  
- **Escalabilidad** → Puede crecer con tus necesidades  
- **Seguridad** → Sin exposición innecesaria  

> 💡 Esta no es una configuración de "privacidad máxima". Es un **sistema equilibrado y usable**.

---

## 🖥️ Panorama del hardware

Mi configuración se reparte entre una mezcla de **infraestructura autoalojada, dispositivos dedicados y equipos personales**, cada uno con un papel específico.

---

## 🧱 Infraestructura central

### Minisforum MS-A2 (servidor principal)

- **SO:** Devuan (sobre hardware físico)
- **Rol:** Host principal de Docker

> 💡 Esta es la columna vertebral de todo mi sistema

👉 Echa un vistazo a la selección de Minisforum Work Station Mini: <https://www.minisforum.com/collections/station-mini-series>

---

### QNAP TS-h1277AXU-RP (NAS)

- **Rol:** Almacenamiento masivo + respaldos
- Almacena:
  - Bibliotecas de medios
  - Datos de Nextcloud
  - Respaldos
  - Archivos (ROMs, documentos, etc.)

> 💡 Separar el cómputo (servidor) del almacenamiento (NAS) mejora la flexibilidad y la resiliencia

👉 Echa un vistazo a la selección de soluciones NAS de QNAP: <https://www.qnap.com/en-us/product>

---

## 💻 Equipos personales

### PC Ryzen 9 a medida (Windows 11)

- **Rol:** Juegos + aplicaciones exclusivas de Windows
- Se usa para:
  - Cargas de trabajo de alto rendimiento
  - Compatibilidad con software que no es de Linux

👉 Echa un vistazo a Micro Center, tienda de informática, para ver las ubicaciones más cercanas: <https://www.microcenter.com/>

---

### Framework Desktop (Artix Linux)

- **Rol:** Estación de trabajo secundaria
- Se usa para:
  - Productividad general
  - Flujos de trabajo Linux-first

👉 Echa un vistazo al Framework Desktop: <https://frame.work/marketplace/desktops>

---

### Portátil Framework 13 (Artix Linux)

- **Rol:** Máquina de viaje + desarrollo
- Se usa para:
  - Acceso remoto (vía Tailscale)
  - Administrar mi infraestructura doméstica
  - Productividad ligera

👉 Echa un vistazo a la selección de portátiles Framework: <https://frame.work/marketplace/laptops>

---

## 🎮 Juegos y emulación

### Raspberry Pi 5 (8 GB)

- **SO:** Batocera
- **Adicional:** MiSTer FPGA
- **Rol:** Retro gaming / emulación

👉 Echa un vistazo a la selección de placas y accesorios Raspberry Pi de Vilros, proveedor de tecnología: <https://vilros.com/>

---

## 🏠 Dispositivos dedicados

### Beelink Mini S13

- **Rol:** Automatización del hogar inteligente
- **SO:** Home Assistant OS

👉 Echa un vistazo a la selección de mini PC de Beelink: <https://www.bee-link.com/collections/product>

---

### Umbrel Home

- **Rol:** Nodo de Bitcoin
- Ejecuta:
  - Nodo BTC completo
  - Lightning

👉 Echa un vistazo a la selección de dispositivos Umbrel: <https://umbrel.com/>

---

### Intel NUC 13

- **Rol:** Servidor de audio
- **SO:** Roon ROCK

👉 Echa un vistazo a B&H y su selección de NUCs y equipo tecnológico profesional: <https://www.bhphotovideo.com/>

---

## 🧠 Filosofía de diseño

Cada dispositivo tiene una **responsabilidad única y clara**:

- Servidor → Cómputo (cargas de Docker)
- NAS → Almacenamiento
- Clientes → Interacción (escritorio/portátil)
- Dispositivos dedicados → Tareas especializadas

> 💡 Esta separación mantiene el sistema:
>
> - Más fácil de mantener  
> - Más resiliente  
> - Más fácil de escalar  

---

## ✅ Por qué esta configuración encaja con mis necesidades

- Sin un único punto de fallo para todo  
- Clara separación de responsabilidades  
- Rendimiento optimizado por dispositivo  
- Flexibilidad para actualizar componentes individuales  

---

## 🚀 Última reflexión sobre el hardware

Pero no necesitas tanto hardware para empezar.

Esta configuración evolucionó con el tiempo —  
empieza pequeño y expándete a medida que crezcan tus necesidades.

---

## 🧱 Arquitectura central

- **Despliegue basado en Docker**
- **Almacenamiento respaldado por el NAS**
- **Proxy inverso (Nginx Proxy Manager)**
- **Acceso privado vía Tailscale (sin reenvío de puertos)**

---

## ☁️ Datos y productividad

### Nextcloud AIO

- Archivos
- Calendario (CalDAV)
- Contactos (CardDAV)
- Almacenamiento respaldado por el NAS

### Almacenamiento en la nube (alojado)

- pCloud.com

---

## 📧 Correo

- Proton Mail

---

## 📸 Fotos

### Immich

- Reemplazo de Google Photos
- Interfaz rápida y moderna
- Totalmente autoalojado

---

## 📝 Ofimática

### ONLYOFFICE

- Reemplazo de Microsoft Office 365
- Suite de productividad completa (editores de documentos, hojas de cálculo, presentaciones, PDF y formularios)
- Gratuito y de código abierto

---

## 📄 PDFs y documentos

- Sumatra PDF (lector ligero)
- BentoPDF (herramientas de PDF)
- Paperless-ngx (archivo documental)

---

## 📝 Notas y conocimiento

### Joplin

- Basado en Markdown
- Multiplataforma
- Sincronización vía Nextcloud

### Paperless-ngx

- Sistema de gestión documental
- OCR + etiquetado
- Reemplaza las montañas de papel

---

## 🔐 Seguridad e identidad

### Gestor de contraseñas

- Proton Pass (alternativa alojada)

### 2FA

- Habilitado en todos los servicios
- Passkeys cuando están disponibles

---

## 🌍 Capa de red y privacidad

### Bloqueo de DNS

- AdGuard Home
- Bloqueo de anuncios y rastreadores en toda la red

### VPN

- Mullvad (VPN externa centrada en la privacidad)

### Acceso privado

- Tailscale
- Acceso remoto seguro a los servicios
- Sin puertos expuestos

---

## 🌐 Navegador

- Brave
  - Bloqueo de anuncios/rastreadores integrado
  - Mínimas extensiones necesarias

---

## 🎥 Medios y entretenimiento

### Jellyfin

- Streaming autoalojado
- Reemplaza Netflix / HBO / servicios de streaming

### Stack Arr

- Sonarr
- Radarr
- Prowlarr
- Gestión automatizada de medios

---

## 📚 Libros y audio

### Calibre-Web / Kavita

- Bibliotecas de ebooks

### Audiobookshelf

- Audiolibros + pódcast
- Totalmente autoalojado

---

## 💰 Finanzas

### Actual Budget

- Presupuestos autoalojados
- Alternativa centrada en la privacidad a Mint/YNAB

---

## 🧰 Desarrollo e infraestructura

### Git

- Forgejo (servicio Git autoalojado)

### Editor

- VSCodium (VS Code sin telemetría)

---

## 🌐 Herramientas de red

- LibreSpeed
- Speedtest-tracker

Pruebas de rendimiento de red autoalojadas y sin rastreo.

---

## 🔁 Qué reemplaza esta configuración

| Big Tech | Reemplazo |
|---------|------------|
| Google Drive | Nextcloud |
| Google Photos | Immich |
| Google Calendar | Nextcloud |
| Google Contacts | Nextcloud |
| Gmail | Proton Mail / Tuta |
| Contraseñas de Chrome | Proton Pass |
| Chrome | Brave |
| Google Docs (parcial) | Nextcloud + Joplin |
| Netflix / HBO | Jellyfin + stack Arr |
| Kindle / Audible | Calibre-Web / Audiobookshelf |
| Adobe Acrobat | Sumatra PDF / BentoPDF |
| Mint / YNAB | Actual Budget |
| GitHub | Forgejo |
| DNS del ISP | AdGuard Home / Pi-hole |
| Speedtest.net | LibreSpeed / Speedtest-tracker |

---

## 🧠 Principios de diseño

### 1. Local primero, siempre que sea posible

Los datos viven:

- En tu servidor
- En tu NAS

---

### 2. Autoaloja cuando aporte valor

No todo necesita ser autoalojado.

Enfoque equilibrado:

- Autoalojado → los datos centrales (archivos, fotos, contraseñas)
- Alojado → la comodidad (correo, si lo prefieres)

---

### 3. Seguro por defecto

- Sin puertos expuestos
- Acceso solo vía Tailscale
- Proxy inverso para el enrutamiento interno

---

### 4. Que sea mantenible

- Servicios basados en Docker
- Estructura de directorios clara
- Configuraciones bajo control de versiones (Forgejo)

---

## ⚖️ Por qué funciona esta configuración

- Alto control sobre los datos  
- Coste recurrente mínimo  
- Arquitectura escalable  
- Acceso remoto seguro  
- Funciona en todos los dispositivos  

---

## 🚀 Reflexión final sobre la infraestructura

¡Esta no es la única manera de hacerlo, y desde luego no es perfecta!

Pero es una **configuración real y curtida en batalla** que equilibra:

- Privacidad  
- Usabilidad  
- Fiabilidad  

> 🧠 El objetivo no es la perfección; es el **control sin fricción**.
