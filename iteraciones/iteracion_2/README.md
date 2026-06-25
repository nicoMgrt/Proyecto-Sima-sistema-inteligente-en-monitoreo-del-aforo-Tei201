# Iteración 2 — Primera Mejora Significativa

## Información General

**Fecha:** Mayo 2026
**Versión:** v2.0
**Estado:** Prototipo funcional mejorado
**Avance asociado:** Avance #2

---

## Descripción

Segunda versión del prototipo, incorporando el feedback del Avance #1 y las lecciones aprendidas en la iteración 1. Esta versión resuelve los problemas críticos de compatibilidad de voltaje en hardware, introduce autonomía energética con batería Li-ion, y despliega el primer dashboard web embebido con indicador visual de aforo.

**Objetivos de esta iteración:**
- Corregir incompatibilidad de voltaje entre sensores HC-SR04 (5V) y ESP32-S3 (3.3V)
- Agregar autonomía energética con Battery Shield + celda NCR18650B
- Implementar dashboard web HTML/CSS embebido con semáforo de aforo
- Resolver falsos positivos del algoritmo de conteo de v1
- Iniciar diseño 3D del encapsulado en Fusion 360

---

## Cambios Respecto a v1

### Hardware

**Agregado:**
- Battery Shield V3 con cargador micro-USB integrado
- Celda Li-ion NCR18650B Panasonic 3400mAh — autonomía >6 horas continuas

**Modificado:**
- Sensores HC-SR04 de 4 pines reemplazados por sensores ultrasónicos Seeed Grove de 3 pines
  - Razón: Los HC-SR04 operan a 5V, incompatible con los 3.3V de los GPIO del ESP32-S3. Riesgo real de quemar el microcontrolador
  - Impacto: Compatibilidad nativa con 3.3V sin resistencias adicionales, cableado más limpio con un solo pin SIG compartido
- Umbral de detección ajustado de 30cm a 40cm
  - Razón: Feedback de testing — personas delgadas o que cruzaban por el centro del marco no eran detectadas con 30cm
  - Impacto: Mejora en la tasa de detección de personas en condiciones reales

**Removido:**
- Dependencia de conexión USB al computador
  - Razón: El sistema ahora es energéticamente autónomo gracias al Battery Shield

**Nota importante para v3:** En esta versión los sensores seguían en GPIO1 y GPIO10. Este problema sería identificado y resuelto en la iteración 3 (boot loop crítico).

### Software

**Nuevas funcionalidades:**
- Dashboard web HTML/CSS embebido con actualización automática cada 2 segundos — semáforo dinámico de aforo (verde/naranja/rojo)
- Migración de red institucional UAI (portal cautivo bloqueaba IoT) a modo Access Point autónomo (`WiFi.softAP`) — red independiente `"Proyecto SIMA - UAI"`

**Optimizaciones:**
- Algoritmo de detección secuencial estricta: A→B para entrada, B→A para salida

**Bugs corregidos:**
- Falso positivo de v1 (doble conteo): El algoritmo ahora exige que ambos sensores confirmen que el obstáculo ya no está presente (`!detectaA && !detectaB`) antes de consolidar el conteo. Previene que una persona parada en el marco sea contada múltiples veces

### Diseño Mecánico

- Primer diseño de encapsulado en Fusion 360
- Distribución interna tipo "sándwich": Battery Shield y celda NCR18650B en la base, ESP32-S3 en la parte superior
- Aberturas calculadas en el panel frontal para los transductores Seeed (prevención de cross-talk ultrasónico entre sensores)
- Cara posterior plana para montaje no invasivo con adhesivo 3M en el marco de la puerta

---

## Fotos del Prototipo v2

| Archivo | Descripción |
|---|---|
| `v2_prototipo_completo.jpg` | Vista completa del prototipo sobre protoboard con Battery Shield |
| `v2_circuito_integrado.jpg` | Detalle de las conexiones entre ESP32-S3, sensores Seeed y shield |
| `v2_encapsulado_inicial.jpg` | Primer modelo 3D impreso o render desde Fusion 360 |
| `v2_dashboard_web.png` | Captura del dashboard web con semáforo de aforo |

---

## Resultados de Testing

### Funcionalidad Lograda

✅ Autonomía energética: >6 horas continuas con celda NCR18650B
✅ Conteo bidireccional sin falsos positivos por rebote
✅ Dashboard web con semáforo dinámico (verde/naranja/rojo)
✅ Red autónoma independiente — sin dependencia del WiFi institucional
✅ Estabilidad eléctrica: lógica 3.3V nativa, sin riesgo de quema de GPIO

⚠️ Pendiente: Almacenamiento persistente — el contador se reinicia si la placa pierde energía

### Testing con Usuarios (Entorno Controlado)

**Resultados cuantitativos:**

| Métrica | Resultado |
|---|---|
| Precisión de conteo (paso normal) | 4.8/5 |
| Estabilidad de la red local AP | 5/5 |

**Feedback cualitativo:**
- Positivo: El dashboard es claro e intuitivo — el cambio de colores ayuda a entender la saturación del espacio rápidamente
- A mejorar: El sistema no detectaba personas muy delgadas o que cruzaban estrictamente por el centro del marco → solución aplicada: umbral elevado de 30cm a 40cm

---

## Feedback del Avance #2

**Comentario del evaluador 1:** *"El sistema demuestra el flujo, pero los datos generados se pierden y no se aprovechan para tomar decisiones a largo plazo."*
- Acción para v3: Integrar peticiones HTTPS hacia Google Apps Script + Google Sheets para registrar cada evento (ENTRADA/SALIDA) con timestamp, generando base de datos histórica persistente

**Comentario del evaluador 2:** *"¿Qué ocurre si un estudiante bloquea el marco de la puerta intencionalmente por varios segundos?"*
- Acción para v3: Sistema anti-bloqueo con timeout de 2.000ms — si un sensor permanece activo más de ese tiempo sin completar la secuencia, se cancela el conteo y se reinician los estados

---

## Problemas Pendientes → Resueltos en v4 Final

**Problema 1: Congelamiento del servidor web al enviar datos a la nube**
- Descripción: Al intentar enviar datos por HTTP, la placa se congelaba 1–2 segundos interrumpiendo la lectura de sensores
- Prioridad: Alta
- Solución implementada en v4: Arquitectura FreeRTOS dual-core — Core 1 detecta personas, Core 0 maneja exclusivamente las peticiones HTTP. Latencia de envío: 1.5–3 segundos sin bloquear la detección

**Problema 2: Pérdida de contador por corte de energía**
- Descripción: Si el Battery Shield se descarga, el contador vuelve a cero al reiniciar
- Prioridad: Alta
- Solución implementada en v4: Librería `Preferences.h` guarda el contador en la memoria NVS (flash no volátil) en cada cambio de estado. Recuperación confirmada en 5/5 reinicios forzados

**Problema 3: GPIO1 y GPIO10 conflictivos (identificado en v3)**
- Descripción: No identificado en v2 pero latente — GPIO1 (TX UART0) y GPIO10 (SPI interno) causarían boot loop crítico al añadir HTTPS en v3
- Solución implementada en v4: Migración a GPIO4 y GPIO5

---

## Comparación v1 vs v2

| Aspecto | v1 | v2 | Mejora |
|---|---|---|---|
| Estabilidad eléctrica | Inestable (5V vs 3.3V) | Estable (3.3V nativa Seeed) | ✅ Hardware seguro |
| Conectividad | Bloqueada (portal cautivo UAI) | Red autónoma AP/Hotspot | ✅ 100% operativa |
| Falsos positivos | Sí (persona parada = conteo múltiple) | No (secuencia estricta A→B / B→A) | ✅ Resuelto |
| Autonomía energética | Dependiente de USB | Independiente (NCR18650B ~6h) | ✅ Autónomo |
| Dashboard visual | Sin interfaz | Semáforo web verde/naranja/rojo | ✅ Nuevo |
| Almacenamiento datos | Sin persistencia | Sin persistencia | ⚠️ Pendiente v4 |
| Umbral detección | 30cm | 40cm | ✅ Ajustado por feedback |
| Encapsulado | Inexistente | Primer boceto Fusion 360 | ✅ Iniciado |

---

## Plan para Iteración 3 (Versión Final — Ejecutado en v4)

**Hardware:**
- Cambio de GPIO1/GPIO10 a GPIO4/GPIO5 → ✅ Ejecutado
- Finalizar ensamble físico dentro de la carcasa impresa en 3D → ✅ Ejecutado

**Software:**
- Implementar conexión HTTPS con Google Apps Script + Google Sheets → ✅ Ejecutado (HTTP 200 OK)
- Habilitar persistencia con `Preferences.h` en memoria NVS → ✅ Ejecutado (5/5 reinicios ✓)
- Timeout anti-bloqueo de 2.000ms → ✅ Ejecutado (100% en pruebas)
- FreeRTOS dual-core para HTTP no bloqueante → ✅ Ejecutado (latencia 1.5–3s)

**Diseño:**
- Refinar tolerancias del encapsulado para puerto micro-USB → ✅ Ejecutado en Fusion 360
- Finalizar sistema de montaje → ✅ Ejecutado

---

## Archivos en esta Carpeta

| Archivo | Descripción |
|---|---|
| `v2_codigo.ino` | Firmware v2 con servidor web, modo AP y anti-rebote |
| `v2_esquema.png` | Esquema del circuito actualizado a 3.3V con sensores Seeed |
| `v2_modelo3d.f3d` | Primer diseño de encapsulado en Fusion 360 (distribución sándwich) |
| `v2_fotos/` | Fotografías del prototipo sobre protoboard con Battery Shield |

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
