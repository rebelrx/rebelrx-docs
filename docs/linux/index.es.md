<!--
Source: linux/index.md
Last translated: 2026-04
-->

# 🐧 Linux

Guías para construir sistemas Linux limpios, estables y soberanos en escritorios, portátiles y servidores.

Esta sección se enfoca en:

- Instalaciones prácticas de Linux  
- Configuración posterior a la instalación  
- Resolución de problemas del mundo real  
- Sistemas que maximizan el **control e independencia del usuario**  

---

## 🧭 Qué Encontrarás Aquí

- **Guías de Instalación** → Configuraciones paso a paso para distribuciones Linux sin systemd seleccionadas  
- **Configuración Post-Instalación** → Endurecimiento, usabilidad, rendimiento  
- **Resolución de Problemas** → Soluciones reales  
- **Filosofía del Sistema** → Cómo pensar correctamente sobre Linux  

---

## ⚠️ Principio Fundamental

> Tu sistema operativo debe servirte a **ti**, y no a sistemas externos, plataformas o marcos de identidad.

---

## 🚫 Por Qué Evitar systemd

Las distribuciones modernas de Linux se han estandarizado en gran medida alrededor de **systemd** como su sistema de inicio.

RebelRx evita explícitamente systemd.

### ¿Por qué?

Porque systemd representa un cambio hacia:

- Centralización del control central del sistema  
- Integración profunda entre los componentes del sistema  
- Menor transparencia en comparación con los sistemas init tradicionales estilo UNIX  

Más importante aún:

> Introduce una capa donde pueden integrarse mecanismos de control externo a gran escala.

Esto incluye el creciente impulso global hacia:

- Sistemas de verificación de edad  
- Controles de acceso vinculados a identidad  
- Mecanismos de aplicación a nivel de dispositivo  

RebelRx asume lo contrario:

> Los sistemas operativos se están convirtiendo en capas de control.

Y systemd es el punto de inserción más probable dentro de Linux.

---

## 🔐 Filosofía de Privacidad y Control

En todas las plataformas principales (Windows, macOS, iOS, Android), ya estamos viendo:

- Integración obligatoria de cuentas  
- Aumento de telemetría y seguimiento  
- Restricciones a nivel de plataforma vinculadas a la identidad  

Linux suele presentarse como la alternativa.

Pero:

> No todo Linux es igual.

Tu nivel de control depende en gran medida de:

- Tu distribución  
- Tu sistema de inicio  
- Tu disposición a evitar configuraciones por defecto orientadas a la conveniencia  

---

## 🧱 Enfoque Recomendado (Sin systemd)

Para minimizar la exposición a capas de control centralizadas, esta guía se enfoca en:

### 🔹 Artix Linux (basado en Arch)

- Sin systemd  
- Rolling release  
- Múltiples opciones de init (OpenRC, runit, s6)  
- Máxima flexibilidad y control  

### 🔹 Devuan (basado en Debian)

- Sin systemd  
- Modelo de versiones estables  
- Ecosistema familiar de Debian  
- Ideal para servidores y despliegues a largo plazo  

👉 Lista completa de distribuciones Linux sin systemd: <https://nosystemd.org/> y <https://systemdfree.com/>

---

## 📚 Guías

- [Instalación manual de escritorio Artix](artix-kde-openrc-install.md)  
- [Instalación de servidor Devuan](devuan-server-install.md)  

---

## 🚧 Guías en Desarrollo

- Referencia de terminal y comandos de Linux  
- Guía de endurecimiento del sistema  
- Estrategia de copias de seguridad y recuperación  

---

## 🧠 Reflexión Final

La conveniencia y el control suelen ser inversamente proporcionales.

Esta sección prioriza el control, incluso cuando requiere más esfuerzo.
