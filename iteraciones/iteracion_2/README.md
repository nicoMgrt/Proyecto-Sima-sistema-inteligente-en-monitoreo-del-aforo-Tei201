# Iteración 2 - Primera Mejora Significativa

## Información General

* **Fecha:** Mayo 2026
* **Versión:** v2.0
* **Estado:** Prototipo funcional mejorado
* **Avance asociado:** Avance #2

## Descripción

Segunda versión del prototipo, incorporando feedback de la iteración 1 y del Avance #1. Incluye mejoras significativas en funcionalidad, estabilización de niveles lógicos de hardware y primeros pasos en el diseño del encapsulado físico.
**Objetivos de esta iteración:**
* Corregir problemas identificados en v1 (niveles de voltaje y bloqueos de red).
* Agregar funcionalidades adicionales (autonomía energética y dashboard embebido).
* Iniciar diseño 3D del encapsulado en Fusion 360.
* Realizar primeras pruebas de lógica secuencial con usuarios.

## Cambios Respecto a v1

**Hardware**
* **Agregado:** * Battery Shield V3
  * Celda Li-ion 18650 (3400mAh Panasonic/NCR)
* **Modificado:** * Sensores HC-SR04 reemplazados por sensores ultrasónicos Seeed (3 pines) - **Razón:** Incompatibilidad de voltaje. Los Seeed operan a 3.3V nativos, protegiendo los pines GPIO de la ESP32-S3 que sufrían riesgo eléctrico con los 5V del HC-SR04.
* **Removido:** * Conexión cableada por USB al computador portátil - **Razón:** El sistema ahora es energéticamente autónomo gracias al Battery Shield.

**Software**
* **Nuevas funcionalidades:**
  * Despliegue de Dashboard Web HTML/CSS embebido con actualización automática cada 2 segundos.
  * Migración de red institucional a formato *Hotspot* / Modo Access Point (`WiFi.softAP`).
* **Optimizaciones:**
  * Algoritmo de detección secuencial estricta (A -> B para entrada, B -> A para salida).
* **Bugs corregidos:**
  * [Falso positivo de v1]: Se corrigió el doble conteo de usuarios. Ahora el algoritmo exige estrictamente que ambos transductores ultrasónicos confirmen que el obstáculo ya no está presente (`!detectaA && !detectaB`) antes de consolidar el conteo final.

## Diseño Mecánico

* Primer diseño de encapsulado en Fusion 360.
* Distribución interna tipo "sándwich": Battery Shield y celda en la base, placa ESP32-S3 en la parte superior.
* Integración de aberturas calculadas en el panel frontal para garantizar separación acústica de los sensores Seeed (prevención de *cross-talk*).
* Diseño de cara posterior plana para montaje no invasivo con adhesivo 3M.

## Fotos del Prototipo v2

*(Nota: Incluir en esta carpeta)*
* `v2_prototipo_completo.jpg`
* `v2_circuito_integrado.jpg`
* `v2_encapsulado_inicial.jpg`
* `v2_testing_usuario.jpg`

## Resultados de Testing

**Funcionalidad Lograda**

* ✅ Autonomía energética (funcionamiento >6 horas continuas).
* ✅ Conteo bidireccional sin falsos positivos por rebote.
* ✅ Despliegue de Interfaz Gráfica (Semáforo dinámico de aforo).
* ⚠️ Almacenamiento persistente de datos (Parcial: El contador se reinicia si la placa pierde energía).

## Feedback del Avance #2

* **Comentario del profesor/ayudante 1:** El sistema demuestra el flujo, pero los datos generados se pierden y no se aprovechan para tomar decisiones a largo plazo.
  * **Acción:** [Plan de mejora para v3] Integrar peticiones HTTP POST hacia Google Sheets para registrar el evento y el *timestamp*, generando una base de datos histórica.
* **Comentario del profesor/ayudante 2:** ¿Qué ocurre si un estudiante bloquea el marco de la puerta intencionalmente por varios segundos?
  * **Acción:** [Plan de mejora para v3] Programar un sistema *anti-bloqueo (Timeout)* en el código que cancele la secuencia si la detección supera los 2000 milisegundos.

## Testing con Usuarios (Entorno Controlado)

* **Resultados cuantitativos:**
  * Precisión de conteo (paso normal): 4.8/5
  * Estabilidad de la red local: 5/5
* **Feedback cualitativo:**
  * **Positivo:** El dashboard es claro e intuitivo; el cambio de colores ayuda a entender la saturación rápidamente.
  * **A mejorar:** El sistema no detectaba a personas muy delgadas o que cruzaban estrictamente por el centro del marco. Se determinó que el umbral de detección de 30cm era muy corto. Se elevó a 40cm.

## Problemas Pendientes

**Problema 1: Desempeño del servidor web local al procesar envío a la nube**
* **Descripción:** Al intentar enviar datos a la nube, la placa podría congelarse durante 1 a 2 segundos, interrumpiendo la lectura de los sensores.
* **Prioridad:** Alta
* **Plan para v3:** Implementar *Dual-Core Processing* con FreeRTOS. Asignar el loop de lectura de sensores al Núcleo 1 y las peticiones HTTP al Núcleo 0.

**Problema 2: Pérdida de variables por corte de energía**
* **Descripción:** Si el Battery Shield se descarga, el aforo actual vuelve a cero al reiniciar.
* **Prioridad:** Alta
* **Plan para v3:** Integrar la librería `Preferences.h` para almacenar el contador en la memoria Flash/EEPROM de la placa en cada cambio de estado.

## Comparación v1 vs v2

| Aspecto | v1 | v2 | Mejora |
| :--- | :--- | :--- | :--- |
| **Estabilidad Eléctrica** | Inestable (Lógica 5V vs 3.3V) | Estable (Lógica 3.3V nativa) | ✅ Hardware seguro |
| **Conectividad** | Bloqueo por portal cautivo | Red autónoma (AP/Hotspot) | ✅ 100% |
| **Precisión de Conteo** | Rebotes / Falsos positivos | Secuencia estricta validada | ✅ |
| **Autonomía** | Dependiente de USB | Independiente (Li-ion 18650) | ✅ |
| **Encapsulado** | ❌ Inexistente | Prototipo CAD (Fusion 360) | ✅ |

## Plan para Iteración 3 (Versión Final)

**Mejoras Críticas**

* **Hardware:**
  * Finalizar ensamble físico dentro de la carcasa impresa en 3D.
* **Software:**
  * Implementar conexión HTTPS y API con Google Apps Script (Google Sheets).
  * Habilitar persistencia de datos (Memoria Flash).
  * Desarrollar Timeout de 2 segundos (Sistema anti-bloqueo).
* **Diseño:**
  * Refinar tolerancias del encapsulado para el puerto de carga micro-USB.
  * Finalizar sistema de montaje plano posterior.
* **Testing:**
  * Testear con flujos continuos simulando bloques M3-M5.
  * Validar métricas de estadía cruzando los datos en Sheets.

## Archivos en esta Carpeta

* `v2_codigo.ino` - Código de esta versión (Con servidor web y anti-rebote).
* `v2_esquema.png` - Esquema del circuito actualizado a 3.3V.
* `v2_modelo3d.f3d` - Primer diseño en Fusion 360 ("sándwich").
* `v2_fotos/` - Fotografías del prototipo sobre protoboard y shield.
