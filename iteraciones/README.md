# Historial de Iteraciones — SIMA

## Propósito

Esta carpeta documenta la evolución del sistema SIMA a través de sus tres versiones de desarrollo. El proceso iterativo fue fundamental para resolver problemas reales de hardware, conectividad y algoritmo que no eran predecibles en el diseño inicial.

---

## Estructura de Carpetas

```
iteraciones/
├── iteracion_1/    # v1 Alpha — Prueba de concepto en laboratorio (Abril 2026)
├── iteracion_2/    # v2 Beta — Primera mejora significativa (Mayo 2026)
└── iteracion_3/    # v4 Final — Sistema completo y validado (Junio 2026)
```

Cada carpeta contiene: `README.md` con descripción completa de la versión, fotos del prototipo, esquema del circuito, código fuente y notas de laboratorio.

---

## Tabla Comparativa de Iteraciones

| Aspecto | v1 Alpha | v2 Beta | v4 Final |
|---|---|---|---|
| **Fecha** | Abril 2026 | Mayo 2026 | Junio 2026 |
| **Sensores** | HC-SR04 4 pines (5V) ❌ | Seeed Grove 3 pines (3.3V) ✅ | Seeed Grove 3.3V ✅ |
| **GPIO sensores** | Sin definir | GPIO1/GPIO10 ⚠️ | GPIO4/GPIO5 ✅ |
| **Conectividad** | Red UAI (bloqueada) ❌ | AP autónomo ✅ | Router + Google Sheets ✅ |
| **Almacenamiento** | Sin persistencia ❌ | Sin persistencia ❌ | NVS flash + Sheets ✅ |
| **Autonomía** | USB (sin batería) ❌ | ~6h (NCR18650B) ✅ | ~16–20h (modem sleep) ✅ |
| **Falsos positivos** | Sí ❌ | No ✅ | No (<5%) ✅ |
| **Dashboard** | Web básico | Semáforo dinámico | Semáforo + estadísticas diarias |
| **Encapsulado** | Sin encapsulado ❌ | Boceto Fusion 360 | Gemelo 3D completo ✅ |
| **Envío a nube** | No ❌ | No ❌ | Google Sheets (HTTP 200 OK) ✅ |
| **Precisión global** | Sin medir | 4.8/5 (subjetivo) | 91.4% (35 pruebas) ✅ |
| **Funcionalidad** | ~40% | ~70% | 100% ✅ |

---

## Evolución del Diseño

### v1 Alpha — Concepto Inicial (Abril 2026)
**Estado:** Prueba de concepto en laboratorio

El primer prototipo validó la factibilidad técnica básica: servidor web embebido en el ESP32-S3 y detección cruda de presencia con sensores ultrasónicos. Sin embargo, identificó tres problemas críticos que bloqueaban el funcionamiento real: incompatibilidad de voltaje entre sensores HC-SR04 (5V) y los GPIO del ESP32-S3 (3.3V), bloqueo de la red WiFi institucional por portal cautivo, y falsos positivos cuando una persona se detenía frente a los sensores.

### v2 Beta — Primera Mejora Significativa (Mayo 2026)
**Estado:** Prototipo funcional mejorado

Resolvió los tres problemas críticos de v1. Migró a sensores Seeed Grove de 3 pines compatibles con 3.3V, implementó modo Access Point autónomo para independizarse de la red UAI, y desarrolló el algoritmo de secuencia estricta con anti-rebote. Incorporó autonomía energética con Battery Shield + celda NCR18650B y el primer dashboard web con semáforo visual. Problema pendiente: sin persistencia de datos ante cortes de energía, y envío bloqueante que congelaba la detección.

### v4 Final — Sistema Completo (Junio 2026)
**Estado:** Sistema IoT completo y validado — listo para presentación

Implementó el ciclo completo de datos con arquitectura FreeRTOS dual-core (Core 1 detección, Core 0 HTTP), persistencia NVS con `Preferences.h`, integración HTTPS con Google Sheets y Google Looker Studio, reset automático nocturno por tiempo de inactividad, y encapsulado completo en Fusion 360. Precisión global validada en 35 pruebas: 91.4%.

---

## Justificación de Cambios

### Hardware

**De v1 a v2:**
- Cambio: HC-SR04 4 pines → Seeed Grove 3 pines
- Razón: Los HC-SR04 operan a 5V — riesgo de dañar permanentemente los GPIO del ESP32-S3 que soportan 3.3V máximo
- Impacto: Compatibilidad nativa, cableado simplificado con pin SIG único

**De v2 a v4:**
- Cambio: GPIO1/GPIO10 → GPIO4/GPIO5
- Razón: GPIO1 es TX del UART0 y GPIO10 está mapeado al SPI interno — causaban boot loop crítico al agregar HTTPS
- Impacto: Eliminación total del boot loop, sistema completamente estable

### Software

**De v1 a v2:**
- Algoritmo anti-rebote: conteo consolidado solo cuando `!detectaA && !detectaB`
- Modo Access Point autónomo (`WiFi.softAP`) para independizarse del portal cautivo UAI
- Dashboard web HTML/CSS con semáforo dinámico de aforo

**De v2 a v4:**
- FreeRTOS dual-core: detección en Core 1, HTTP en Core 0 — elimina latencia bloqueante
- Persistencia NVS con `Preferences.h` — contador sobrevive cortes de energía (5/5 reinicios ✓)
- Integración HTTPS Google Sheets con timestamp ISO 8601 — HTTP 200 OK confirmado
- Timeout anti-bloqueo de 2.000ms — previene conteos fantasma
- Reset automático nocturno si apagado >2 horas
- WiFi modem sleep — autonomía extendida de ~6h a ~16–20h

### Diseño 3D

**De v1 a v2:**
- Primer boceto de encapsulado en Fusion 360 con distribución interna "sándwich"
- Aberturas frontales para transductores de los sensores Seeed

**De v2 a v4:**
- Gemelo 3D completo con todos los componentes internos a dimensiones reales
- Tapa desmontable sin destruir el ensamble (diseño para reparación en campo)
- Orificios laterales para micro USB y switch de encendido
- Postes espaciadores M3 internos

---

## Incorporación de Feedback

### Feedback del Avance #1 → Cambios en v2
- *"El sistema debe ser autónomo energéticamente para instalarse en terreno"*
  - Acción: Battery Shield V3 + celda NCR18650B Panasonic 3400mAh

### Feedback del Avance #2 → Cambios en v4
- *"Los datos generados se pierden y no se aprovechan para decisiones a largo plazo"*
  - Acción: Integración Google Sheets con registro histórico de cada evento con timestamp
- *"¿Qué ocurre si alguien bloquea la puerta intencionalmente varios segundos?"*
  - Acción: Timeout anti-bloqueo de 2.000ms implementado y validado en pruebas

### Feedback de Testing → Cambios en v4
- *"El sistema no detectaba personas delgadas o que cruzaban por el centro del marco"*
  - Acción: Umbral de detección elevado de 30cm a 40cm

---

## Métricas de Mejora

| Métrica | v1 Alpha | v2 Beta | v4 Final | Mejora total |
|---|---|---|---|---|
| Compatibilidad eléctrica | Riesgo de quema | Estable 3.3V | Estable 3.3V | ✅ Resuelto |
| Latencia envío datos | Sin envío | Sin envío | 1.5–3 seg (no bloqueante) | ✅ Nuevo |
| Autonomía batería | Sin batería | ~6h | ~16–20h | ↑ +300% |
| Tasa de falsos positivos | Alta | <5% | <5% | ↓ ~90% |
| Persistencia ante corte energía | No | No | 100% (5/5) | ✅ Nuevo |
| Precisión detección global | Sin medir | 4.8/5 (subjetivo) | 91.4% (35 pruebas) | ✅ Validado |
| Almacenamiento histórico | No | No | Google Sheets ilimitado | ✅ Nuevo |

---

## Lecciones Aprendidas

### Técnicas
1. Verificar siempre los niveles lógicos de voltaje (3.3V vs 5V) antes de conectar módulos — la incompatibilidad puede dañar hardware irreversiblemente
2. Los GPIO con funciones reservadas del chip (UART, SPI, USB) no deben usarse para sensores que cambien constantemente su dirección
3. Las llamadas HTTP síncronas bloquean el loop principal en IoT — FreeRTOS es la solución correcta para concurrencia en ESP32-S3

### De Proceso
1. La prueba de concepto mínima (v1) es indispensable — identifica problemas reales que el diseño en papel no predice
2. Documentar cada falla y su solución en tiempo real facilita la elaboración del FUENTES.md y el protocolo de testing al final del proyecto

### De Trabajo en Equipo
1. Dividir el trabajo por componentes (hardware, software, diseño 3D, documentación) permite avanzar en paralelo sin bloquear al equipo
2. Comunicar los cambios de hardware al equipo de software inmediatamente evita que dos personas trabajen sobre supuestos distintos

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
