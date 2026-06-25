# Iteración 1 - Concepto Inicial

## Información General

* **Fecha:** Abril 2026
* **Versión:** v1.0
* **Estado:** Prototipo alpha - Prueba de concepto en laboratorio
* **Avance asociado:** Laboratorio Inicial / Avance #2

## Descripción

Primera versión del prototipo, enfocada en validar la factibilidad técnica del concepto de conteo de aforo mediante sensores de proximidad.
**Objetivo de esta iteración:**
* Probar conexión de sensores ultrasónicos estándar.
* Validar comunicación con microcontrolador y envío de datos vía Wi-Fi.
* Demostrar concepto de actuación mediante un dashboard web local.
* Circuito básico ensamblado en protoboard.

## Componentes Utilizados

**Hardware**
* Microcontrolador: ESP32-S3 N16R8
* Sensores: 2x Sensores Ultrasónicos HC-SR04 (4 pines)
* Actuadores: Dashboard web HTML incrustado (Pantalla de dispositivo cliente)
* Otros: Protoboard, cables jumper (macho-macho, macho-hembra), cable USB para alimentación.

**Software**
* IDE: Arduino IDE (v2.x)
* Librerías: `WiFi.h`, `WebServer.h`
* Versión código: v1.0 (Básica)

## Fotos del Prototipo

*(Nota: Incluir en esta carpeta)*
* `v1_circuito_protoboard.jpg`
* `v1_vista_general.jpg`
* `v1_detalle_conexiones.jpg`

## Resultados de Testing Inicial

**Funcionalidad Lograda**
* Levantar un servidor web básico incrustado en la placa.
* Detección cruda de movimiento frente al sensor.

**Funciones Fallidas**
* Mantener conexión estable a la red institucional (UAI).
* Lectura confiable de distancias de ambos sensores simultáneamente.
* Lógica de conteo estricto (se registraban múltiples ingresos por una sola persona).

## Problemas Identificados

**Problema 1: Incompatibilidad de niveles lógicos (Voltaje)**
* **Descripción:** Los sensores fallaban al leer los ecos o enviaban señales peligrosas para la placa.
* **Causa probable:** El sensor HC-SR04 opera con lógica de 5V, mientras que los pines GPIO de la ESP32-S3 operan a 3.3V. 
* **Solución propuesta:** Migrar a sensores ultrasónicos Seeed de 3 pines que operan de forma nativa a 3.3V para proteger el hardware y simplificar el cableado.

**Problema 2: Bloqueo de Red IoT**
* **Descripción:** La ESP32-S3 no lograba establecer conexión estable a internet.
* **Causa probable:** La red Wi-Fi de la universidad utiliza un "portal cautivo" (página de login) que los microcontroladores no pueden saltar automáticamente.
* **Solución propuesta:** Modificar la arquitectura para que la ESP32 opere en modo Access Point (`WiFi.softAP`) o utilizar un Hotspot móvil (anclaje de red) para las pruebas de campo.

**Problema 3: Doble conteo (Falsos positivos)**
* **Descripción:** Si un estudiante se detenía a conversar en el marco de la puerta, el sistema sumaba +1 repetidamente.
* **Causa probable:** El código registraba el evento apenas se activaba el segundo sensor, sin esperar a que la persona liberara el espacio.
* **Solución propuesta:** Rediseñar la máquina de estados en el código (filtro anti-rebote lógico) para consolidar el conteo solo cuando ambos sensores vuelvan a estado de reposo.

## Aprendizajes

**Técnicos**
* Es imperativo verificar siempre los niveles lógicos de voltaje (3.3V vs 5V) de las hojas de datos (datasheets) antes de interconectar módulos.
* Las redes institucionales no están preparadas para el despliegue directo de dispositivos IoT autónomos debido a sus protocolos de seguridad.

**De Diseño**
* El comportamiento real del usuario es impredecible. El hardware y el software deben estar preparados para oclusiones prolongadas (personas detenidas en las puertas) sin corromper la base de datos de aforo.

## Plan para Iteración 2

**Mejoras Planificadas**

* **Hardware:**
  * Reemplazar los HC-SR04 por sensores ultrasónicos Seeed (3 pines).
  * Integrar un *Battery Shield V3* y una celda Li-ion 18650 de 3400mAh para independizar energéticamente el prototipo.
* **Software:**
  * Implementar secuencia estricta de validación direccional en el código C++.
  * Migrar a red independiente (Hotspot temporal / modo AP).
* **Diseño:**
  * Iniciar diseño de encapsulado tridimensional (gemelo digital) en Fusion 360 para organizar la placa, el shield y la batería.

## Archivos en esta Carpeta

* `v1_codigo.ino` - Código inicial con errores de rebote.
* `v1_esquema.png` - Esquema del circuito inicial (5V a 3.3V).
* `v1_fotos/` - Fotografías del prototipo sobre la mesa.
* `v1_notas.txt` - Bitácora de laboratorio.
