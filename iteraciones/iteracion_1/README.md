# Iteración 1 — Concepto Inicial

## Información General

**Fecha:** Abril 2026
**Versión:** v1.0
**Estado:** Prototipo Alpha — Prueba de concepto en laboratorio
**Avance asociado:** Laboratorio Inicial / Avance #2

---

## Descripción

Primera versión del prototipo SIMA, enfocada en validar la factibilidad técnica del concepto de conteo de aforo mediante sensores de proximidad. Esta iteración corresponde al punto de partida del proyecto: circuito básico en protoboard, sin autonomía energética, sin almacenamiento persistente, y con los primeros problemas técnicos identificados que guiarían las iteraciones siguientes.

**Contexto del problema que motiva esta iteración (Avance #1):**
El levantamiento de datos previo identificó que el 74.2% de los estudiantes de la UAI ha tenido que abandonar la Biblioteca de Pregrado por falta de espacio, y el 39.7% pierde entre 5 y más de 10 minutos buscando asiento en hora peak (bloques M3–M5). La v1 fue el primer intento de responder a ese problema con hardware real.

**Objetivo de esta iteración:**
- Probar conexión de sensores ultrasónicos con el ESP32-S3
- Validar comunicación WiFi y servidor web local embebido
- Demostrar concepto de dashboard de aforo en navegador
- Identificar los problemas técnicos reales para planificar la v2

---

## Componentes Utilizados

### Hardware

| Componente | Especificación | Problema identificado |
|---|---|---|
| ESP32-S3 N16R8 | MCU principal, 3.3V GPIO | — |
| 2× HC-SR04 (4 pines) | Sensores ultrasónicos | Incompatibilidad 5V vs 3.3V |
| Protoboard | Conexión sin soldadura | — |
| Jumper wires M-M y M-H | Cableado | — |
| Cable USB | Alimentación desde computador | Sin autonomía energética |

### Software

| Elemento | Detalle |
|---|---|
| IDE | Arduino IDE v2.x |
| Librerías | `WiFi.h`, `WebServer.h` |
| Versión firmware | v1.0 — lógica básica sin anti-rebote |
| Conectividad | Red WiFi institucional UAI (con portal cautivo) |

---

## Fotos del Prototipo v1

| Archivo | Descripción |
|---|---|
| `v1_circuito_protoboard.jpg` | Vista del circuito armado sobre protoboard |
| `v1_vista_general.jpg` | Vista general del prototipo en mesa de laboratorio |
| `v1_detalle_conexiones.jpg` | Detalle de las conexiones entre ESP32-S3 y sensores HC-SR04 |
| `v1_notas.txt` | Bitácora de laboratorio con observaciones del equipo |

---

## Resultados de Testing Inicial

### Funcionalidad Lograda

✅ Servidor web básico embebido en la placa operativo
✅ Detección cruda de presencia frente a los sensores

### Funciones Fallidas

❌ Conexión estable a la red WiFi institucional UAI (bloqueada por portal cautivo)
❌ Lectura confiable y simultánea de distancias de ambos sensores
❌ Lógica de conteo estricto — se registraban múltiples ingresos por una sola persona (falsos positivos)
❌ Autonomía energética — dependencia total del cable USB al computador

---

## Problemas Identificados

### Problema 1 — Incompatibilidad de Niveles Lógicos (Voltaje)
- **Descripción:** Los sensores fallaban al leer los ecos o enviaban señales potencialmente peligrosas para los GPIO de la placa
- **Causa:** El HC-SR04 opera con lógica de 5V en su pin ECHO, mientras que los GPIO del ESP32-S3 soportan máximo 3.3V. Riesgo real de daño permanente al microcontrolador
- **Solución propuesta → Ejecutada en v2:** Migración a sensores Seeed Grove de 3 pines que operan de forma nativa a 3.3V — compatibilidad directa sin resistencias ni divisores de voltaje

### Problema 2 — Bloqueo de Red IoT por Portal Cautivo
- **Descripción:** El ESP32-S3 no lograba establecer conexión estable a internet a través de la red WiFi de la universidad
- **Causa:** La red UAI usa un portal cautivo (página de login) que los microcontroladores no pueden autenticar automáticamente — mecanismo de seguridad que bloquea dispositivos IoT
- **Solución propuesta → Ejecutada en v2/v4:** Modo Access Point autónomo (`WiFi.softAP`) en v2, luego integración con router propio y Google Sheets en v4

### Problema 3 — Doble Conteo (Falsos Positivos)
- **Descripción:** Si una persona se detenía a conversar en el marco de la puerta, el sistema registraba múltiples entradas repetidamente
- **Causa:** El código consolidaba el conteo al detectar la activación del segundo sensor, sin esperar a que la persona liberara completamente el espacio
- **Solución propuesta → Ejecutada en v2:** Máquina de estados con filtro anti-rebote — el conteo se consolida únicamente cuando ambos sensores vuelven a estado de reposo (`!detectaA && !detectaB`)

### Problema 4 — Sin Autonomía Energética
- **Descripción:** El sistema dependía del cable USB conectado a un computador — no podía instalarse en la puerta de la biblioteca de forma autónoma
- **Causa:** Sin batería ni circuito de gestión de energía
- **Solución propuesta → Ejecutada en v2:** Battery Shield V3 + celda Li-ion NCR18650B Panasonic 3400mAh — autonomía de >6h en v2, escalada a 16–20h en v4 con modo modem sleep

---

## Aprendizajes

### Técnicos
1. Verificar siempre los niveles lógicos de voltaje en los datasheets antes de interconectar módulos — la diferencia entre 3.3V y 5V puede dañar hardware irreversiblemente
2. Las redes institucionales no están preparadas para el despliegue directo de dispositivos IoT autónomos — planificar la conectividad desde el diseño inicial

### De Diseño
1. El comportamiento real del usuario es impredecible — el sistema debe manejar oclusiones prolongadas (personas paradas en la puerta) sin corromper los datos de aforo
2. La prueba de concepto mínima (v1) es indispensable para identificar problemas reales que no son predecibles en papel

---

## Comparación con Versiones Posteriores

| Aspecto | v1 Alpha | v2 Beta | v4 Final |
|---|---|---|---|
| Sensores | HC-SR04 5V ❌ | Seeed 3.3V ✅ | Seeed 3.3V ✅ |
| GPIO sensores | Sin definir | GPIO1/GPIO10 ⚠️ | GPIO4/GPIO5 ✅ |
| Conectividad | Red UAI ❌ | AP autónomo ✅ | Router + Sheets ✅ |
| Falsos positivos | Sí ❌ | No ✅ | No (<5%) ✅ |
| Autonomía energética | USB ❌ | ~6h ✅ | ~16–20h ✅ |
| Almacenamiento datos | Sin persistencia ❌ | Sin persistencia ❌ | NVS + Sheets ✅ |
| Dashboard | Web básico | Semáforo dinámico | Semáforo + estadísticas |
| Encapsulado | Sin encapsulado ❌ | Boceto Fusion 360 | Gemelo 3D completo ✅ |
| Precisión global | Sin medir | 4.8/5 (subjetivo) | 91.4% (35 pruebas) ✅ |

---

## Plan para Iteración 2 — Estado de Ejecución

| Mejora planificada | Estado |
|---|---|
| Reemplazar HC-SR04 por Seeed Grove 3 pines | ✅ Ejecutado en v2 |
| Integrar Battery Shield V3 + celda NCR18650B | ✅ Ejecutado en v2 |
| Implementar secuencia estricta A→B / B→A | ✅ Ejecutado en v2 |
| Migrar a red independiente (AP/Hotspot) | ✅ Ejecutado en v2 |
| Iniciar diseño de encapsulado en Fusion 360 | ✅ Ejecutado en v2 |

---

## Archivos en esta Carpeta

| Archivo | Descripción |
|---|---|
| `v1_codigo.ino` | Firmware inicial con lógica básica y errores de rebote documentados |
| `v1_esquema.png` | Esquema del circuito inicial con HC-SR04 a 5V |
| `v1_fotos/` | Fotografías del prototipo sobre protoboard en laboratorio |
| `v1_notas.txt` | Bitácora de laboratorio con observaciones del equipo |

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
