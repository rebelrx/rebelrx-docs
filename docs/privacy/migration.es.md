---
description: >-
  Un plan realista y por fases para dejar las plataformas de las Big Tech — pasos ordenados, sin cortes en frío y sin migraciones abandonadas.
---

<!--
Source: privacy/migration.md
Last translated: 2026-07
-->

# 🏗️ Guía de migración

Cómo alejarte de las Big Tech **sin romper tu forma de trabajar**.

---

## ⚠️ Regla de oro

> NO migres todo a la vez.

Es la razón n.º 1 por la que la gente fracasa.

---

## 🧭 Visión general de la estrategia de migración

Seguirás un enfoque por fases:

1. Configuración en paralelo  
2. Migración gradual  
3. Uso dual  
4. Cambio definitivo  
5. Desmantelamiento  

---

## 🪜 Fase 1: configuración en paralelo

Configura tus nuevas herramientas **sin borrar nada**.

### Apps esenciales para instalar primero

- Navegador → **Brave**
- Gestor de contraseñas → **Bitwarden / Proton Pass**
- Correo → **Proton Mail / Tuta**
- Mensajería → **Signal**

---

### Opcional (infraestructura temprana)

Si vas a autoalojar:

- **Nextcloud (o Sync.com si prefieres alojado)**
- **Immich (fotos)**
- **AdGuard Home / Pi-hole**
- **Tailscale**

> 💡 No construyas de más al principio — primero haz que las cosas funcionen.

---

## 🔄 Fase 2: migración gradual

Mueve una categoría cada vez.

---

### 🌐 Navegador

- Instala Brave
- Añade extensiones (mínimas):
  - uBlock Origin (opcional — Brave ya bloquea anuncios)
- Importa los marcadores desde Chrome

---

### 🔐 Contraseñas

- Exporta desde Chrome
- Importa en:
  - Bitwarden **o**
  - Proton Pass
- Activa el 2FA

> ⚠️ Haz esto pronto — todo lo demás depende de ello

---

### 📧 Correo

- Crea una cuenta de Proton Mail o Tuta
- Configura:
  - Reenvío de Gmail → nueva bandeja de entrada
  - Empieza a usar el nuevo correo para los inicios de sesión

> 💡 Opcional: configura un dominio propio

---

### ☁️ Archivos

**Opción A (autoalojado):**

- Sube los datos → Nextcloud

**Opción B (alojado):**

- Sube los datos → Sync.com

Empieza por:

- Documentos
- Archivos personales
- Datos no críticos

---

### 📸 Fotos

- Exporta desde Google Photos
- Sube a Immich

> ⚠️ Esto puede llevar tiempo — hazlo por lotes

---

### 📝 Notas

- Exporta Google Keep / Apple Notes
- Importa en Joplin

---

### 📆 Calendario y contactos

- Exporta los datos de Google
- Importa en:
  - Nextcloud  
  - o Baikal / Radicale

---

### 🌍 Red

- Despliega AdGuard Home o Pi-hole
- Actualiza el DNS en:
  - El router (opcional)
  - Los dispositivos

---

### 🔒 VPN / Acceso

- Instala Mullvad (VPN de privacidad)
- Configura Tailscale (acceso remoto al homelab)

---

### 📚 Documentos

- Escanea/importa en Paperless-ngx
- Sustituye los flujos de trabajo de PDF:
  - Sumatra PDF
  - BentoPDF

---

### 🎥 Medios

- Configura Jellyfin
- (Avanzado opcional):
  - Despliega el stack Arr (Sonarr, Radarr, Prowlarr)

---

### 📚 Libros y audio

- Importa ebooks → Calibre-Web / Kavita
- Importa audiolibros/pódcast → Audiobookshelf

---

### 💰 Finanzas

- Configura Actual Budget
- Importa/exporta los datos financieros (si aplica)

---

### 🧰 Desarrollo

- Sustituye los flujos de GitHub por Forgejo (opcional)
- Cámbiate a VSCodium

---

## 🔁 Fase 3: uso dual

Ejecuta ambos sistemas simultáneamente.

Valida:

- La fiabilidad de la sincronización
- El acceso desde el móvil
- Los respaldos
- El rendimiento

---

## 🔌 Fase 4: cambio definitivo

Empieza a cambiar del todo:

- Actualiza los correos de inicio de sesión en todas las cuentas
- Mueve los flujos de trabajo diarios:
  - Notas → Joplin
  - Archivos → Nextcloud / Sync.com
  - Fotos → Immich
- Reduce la dependencia de Google/Apple

---

## 🧹 Fase 5: desmantelamiento

Cuando tengas confianza:

- Exporta los respaldos finales de los servicios antiguos
- Desactiva las cuentas sin uso
- Elimina las apps que ya no uses

> ⚠️ Conserva respaldos antes de borrar nada

---

## ⚠️ Errores comunes

- Migrar demasiado rápido  
- Romper los flujos de trabajo familiares/compartidos  
- No tener estrategia de respaldo  
- Sobreingeniería temprana  
- Perseguir la "privacidad perfecta" en lugar de sistemas usables  

---

## 🔐 Buenas prácticas de seguridad

- Usa un gestor de contraseñas (Bitwarden / Proton Pass)
- Activa el 2FA en todas partes
- Mantén los sistemas actualizados
- Mantén **respaldos fuera de casa**
- Prueba las restauraciones (no solo los respaldos)

---

## 🧠 Consejos pro

### 1. Prioriza las victorias de alto impacto

Empieza por:

- El navegador
- El gestor de contraseñas
- El correo

---

### 2. Separa alojado vs autoalojado

| Tipo | Cuándo usarlo |
|------|------------|
| Alojado (Proton, Sync.com) | Configuración más simple y rápida |
| Autoalojado (Nextcloud, Immich) | Máximo control |

---

### 3. Construye por capas

- Capa 1 → Apps (Brave, Signal, Bitwarden)
- Capa 2 → Servicios (correo, almacenamiento)
- Capa 3 → Infraestructura (Nextcloud, DNS, VPN)

---

## 🚀 Reflexión final

> La migración es un proceso, no un evento.

No necesitas reemplazarlo todo de la noche a la mañana.

Céntrate en:

- El progreso
- La estabilidad
- El control
