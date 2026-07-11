---
description: >-
  Una escalera de tres niveles para la privacidad móvil — endurece el teléfono que ya tienes, reemplaza las apps que te rastrean y, cuando estés listo, reemplaza el propio SO con GrapheneOS.
---

<!--
Source: privacy/mobile.md
Last translated: 2026-07
-->

# 📱 Privacidad móvil

Tu teléfono es el dispositivo de vigilancia más personal jamás construido. Sabe dónde duermes, con quién hablas, qué lees y cuánto tiempo lo miras.

Esta guía es una **escalera, no un precipicio**: tres niveles, cada uno una mejora real, cada uno opcional.

---

## ⚠️ Principio fundamental

> El dispositivo que llevas a todas partes debería responder ante ti; no informar sobre ti.

---

## 🧭 La realidad de partida

- **Android de fábrica** → Google Play Services corre con privilegios de sistema: ubicación, sensores, uso de apps, red, sin importar los interruptores de tus ajustes. Un teléfono Android típico contacta con los servidores de Google cientos de veces al día.
- **iPhone** → Valores por defecto sensiblemente mejores y una seguridad de hardware sólida, pero un sistema cerrado donde Apple guarda los metadatos y las llaves del ecosistema.
- **Android desgooglizado** → El único camino donde el propio SO trabaja para ti.

No tienes que saltar al final. Empieza donde estás.

---

## 🪜 Nivel 1 — Endurece lo que ya tienes (esta noche, gratis)

Sin teléfono nuevo, sin flashear nada. Treinta minutos:

- **Audita los permisos de las apps** → Ajustes → Privacidad. Revoca ubicación, micrófono y cámara a todo lo que no los necesite claramente. Prefiere "Solo mientras se usa".
- **Mata el ID de publicidad** → Android: Ajustes → Privacidad → Anuncios → *Eliminar ID de publicidad*. iPhone: Ajustes → Privacidad → Rastreo → desactiva *Permitir que las apps soliciten rastrearte*.
- **Borra las apps que no usas** → Cada app instalada es superficie de ataque permanente y un recolector de datos.
- **Usa el navegador en lugar de la app** siempre que puedas (redes sociales especialmente) — las webs obtienen mucho menos acceso que las apps instaladas.
- **Desactiva el escaneo de Wi-Fi y Bluetooth** → Android lo entierra en los ajustes de Ubicación; emiten balizas con tu presencia incluso con la Wi-Fi "apagada".
- **Pon un código de acceso robusto** → Mínimo 6+ dígitos; la biometría es comodidad, el código es el cerrojo de verdad.
- **Extra para iPhone** → Activa la **Protección de datos avanzada** (Ajustes → iCloud) para respaldos cifrados de extremo a extremo.

> El nivel 1 no derrota la telemetría de la plataforma. Reduce lo que recopila la capa de apps — y eso es la mayor parte del sangrado diario.

---

## 🪜 Nivel 2 — Reemplaza las apps que te rastrean

El mismo teléfono, mejor software. Las versiones móviles de las [Recomendaciones de apps](app-recommendations.md):

| Reemplaza | Por | Por qué |
|--------|------|-----|
| Chrome / Safari | [Brave](https://brave.com/) | Bloqueo de anuncios/rastreadores integrado |
| Google Search | [DuckDuckGo](https://duckduckgo.com/) / [Startpage](https://www.startpage.com/) | Rastreo reducido |
| SMS / Messenger / WhatsApp | [Signal](https://signal.org/) | Cifrado de extremo a extremo |
| App de Gmail | [Proton Mail](https://proton.me/mail) / [Tuta](https://tuta.com/) | Correo cifrado |
| Google Maps | [Organic Maps](https://organicmaps.app/) | Sin conexión, código abierto, cero rastreo |
| Google Photos | App móvil de [Immich](https://immich.app/) | Auto-respaldo a **tu** servidor |
| Gboard | [HeliBoard](https://github.com/Helium314/HeliBoard) (Android) | Tu teclado no debería necesitar acceso a la red |
| Google Authenticator / Authy | [Aegis](https://getaegis.app/) (Android) / [Ente Auth](https://ente.io/auth/) | 2FA de código abierto, respaldos cifrados y exportables |
| App de YouTube | [NewPipe](https://newpipe.net/) (Android) | Sin cuenta, sin anuncios, sin perfilado de visionado |
| App de notas | [Joplin](https://joplinapp.org/) | Se sincroniza con tu configuración existente |

### 🔑 Unas palabras sobre las apps de 2FA

El doble factor no es negociable, pero la *app* importa:

- **Google Authenticator** sincroniza tus semillas de 2FA con tu cuenta de Google (sin cifrado de extremo a extremo durante años) — las llaves de todo, en manos de la empresa que estás dejando
- **Authy** es de código cerrado y sufrió una brecha en 2024 (se expusieron los números de teléfono de 33 millones de usuarios)
- **Aegis** y **Ente Auth** son de código abierto, con respaldos cifrados y bajo control local

!!! warning "Respalda tu bóveda de 2FA antes de necesitarla"

    Un teléfono perdido con un autenticador sin respaldo te deja fuera de todas
    las cuentas que protegía. Exporta la bóveda cifrada (tanto Aegis como Ente Auth
    lo permiten) y guárdala con los respaldos de tu gestor de contraseñas. Prueba
    la restauración una vez.

### 📦 De dónde sacar las apps (Android)

- **[F-Droid](https://f-droid.org/)** → Apps de código abierto, sin cuenta
- **[Obtainium](https://github.com/ImranR98/Obtainium)** → Instala y actualiza directamente desde las releases de los desarrolladores
- **[Aurora Store](https://auroraoss.com/)** → Interfaz anónima al catálogo de la Play Store para las apps que no puedas evitar

---

## 🪜 Nivel 3 — Reemplaza el SO (GrapheneOS)

Los cambios de apps no pueden arreglar un sistema operativo que informa a casa. Cuando estés listo para sacar a Google de la propia plataforma, [**GrapheneOS**](https://grapheneos.org/) es la opción más sólida disponible: un Android endurecido y centrado en la seguridad, sin servicios de Google y sin compromisos añadidos.

### La ironía del Pixel

Sí: la mejor forma de escapar del software de Google es el hardware de Google.

GrapheneOS solo es compatible con **dispositivos Pixel** (actualmente del Pixel 6 a la serie Pixel 10) porque los Pixel son los únicos teléfonos convencionales que permiten **volver a bloquear el bootloader con tus propias claves de firma**, lo que significa que el arranque verificado protege *tu* SO, no el del fabricante. Ningún otro hardware de consumo ofrece eso.

!!! tip "Consejos prácticos de compra"

    - Compra un modelo **libre de fábrica (factory unlocked)**, directamente a
      Google o a un vendedor reputado — los modelos bloqueados por operadora
      (especialmente las variantes de Verizon/AT&T en EE. UU.) a menudo no
      permiten desbloquear el bootloader en absoluto
    - Un Pixel 8/8a usado es el punto dulce en precio: bastante por debajo de
      los 350 $ y con soporte de seguridad para años
    - Los modelos más nuevos simplemente alargan la ventana de soporte

### Instalación

El [instalador web oficial](https://grapheneos.org/install/web) se ejecuta desde un navegador. Conecta el teléfono, sigue los pasos y vuelve a bloquear el bootloader al final. 15–20 minutos, sin herramientas que instalar.

### Vivir con él

- **Google Play en sandbox** → Si una app requiere de verdad los Play Services, GrapheneOS puede ejecutar los auténticos como una app corriente, sin privilegios y totalmente aislada; sin acceso a nivel de sistema, instalada solo en el perfil que elijas. La mayoría lo pone en un perfil aparte y mantiene su perfil principal libre de Google.
- **Perfiles de usuario** → Aislamiento total entre contextos (personal / trabajo / "apps que exigen Google")
- **El día a día** → Sigue siendo Android. Todas tus apps del nivel 2 funcionan.

!!! warning "Comprueba primero tus apps bancarias"

    Algunas apps bancarias exigen la atestación de hardware de Google y se
    niegan a funcionar en cualquier SO alternativo — aproximadamente la mitad
    funcionan en GrapheneOS con Play en sandbox, y varía según el banco y el año.
    Consulta las listas de compatibilidad de la comunidad para tus apps concretas
    **antes** de flashear. El plan B que siempre funciona: la web de tu banco en Brave.

### Alternativa: CalyxOS

[CalyxOS](https://calyxos.org/) cambia parte del endurecimiento de GrapheneOS por comodidad: **microG** (una reimplementación de código abierto de los Play Services) viene preinstalado, y es compatible con Fairphone y algunos dispositivos Motorola además de los Pixel. Un camino intermedio razonable — pero si tienes un Pixel compatible, GrapheneOS es la opción más fuerte.

---

## 🚫 Qué no hacer

- No rootees tu teléfono por privacidad. El root **rompe** el modelo de seguridad y el arranque verificado que te protegen
- No instales apps de "privacidad" aleatorias de la Play Store. La mayoría son adware con gabardina
- No flashees ROMs abandonadas o de un solo mantenedor. Un SO sin actualizaciones de seguridad es peor que el de fábrica
- No intentes hacer los tres niveles en un fin de semana. Así es como las migraciones acaban abandonadas

---

## 🧠 Reflexión final

Tu teléfono ve más de tu vida que cualquier otro objeto que poseas.

Cada nivel de esta escalera lo acerca a ser **tuyo**:

> El mismo bolsillo. Distinta lealtad.
