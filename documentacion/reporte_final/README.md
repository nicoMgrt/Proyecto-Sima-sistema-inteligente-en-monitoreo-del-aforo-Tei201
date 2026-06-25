# SIMA — Sistema Inteligente de Monitoreo de Aforo
## Reporte Final del Proyecto

**ODS 11 — Ciudades y Comunidades Sostenibles**

| Integrante | Rol |
|---|---|
| Nicolás Marinkovic | Documentación / Iteración / Testing |
| Bárbara Chaparro | Software / Firmware / Documentación |
| Valentina Ramírez | Iteración / Diseño 3D |
| Cristóbal Pérez | Software / Hardwear / Diseño 3D |

**Taller de Diseño en Ingeniería — TEI201**
Design Engineering Center · Universidad Adolfo Ibáñez · Junio 2026

---

## Resumen Ejecutivo

El sistema SIMA (Sistema Inteligente de Monitoreo de Aforo) es un dispositivo IoT de bajo costo diseñado para proveer información de ocupación en tiempo real de la Biblioteca de Pregrado de la Universidad Adolfo Ibáñez, Campus Peñalolén. El sistema resuelve una brecha de información identificada mediante encuesta a 69 estudiantes: el 74,2% ha tenido que abandonar la biblioteca por falta de espacio y el 39,7% pierde entre 5 y más de 10 minutos buscando un asiento disponible en hora peak.

La solución integra dos sensores ultrasónicos Seeed Grove conectados a un microcontrolador ESP32-S3, que determina la dirección del flujo de personas según el orden de activación de los sensores. Los datos se almacenan en Google Sheets y se visualizan en un dashboard web local con indicador semafórico de ocupación. El costo total del prototipo es de $27.960 CLP, con una precisión global validada de 91,4% en 35 pruebas controladas y autonomía de 16 a 20 horas continuas.

**Conclusión clave:** SIMA demuestra que es posible optimizar el uso de infraestructura educativa existente mediante tecnología IoT de bajo costo, sin expansión física y respetando la privacidad de los usuarios, en alineación con el ODS 11.

---

## 1. Introducción

### 1.1 Contexto General

La Universidad Adolfo Ibáñez en el Campus Peñalolén concentra más de 8.000 estudiantes diarios que compiten por espacios de estudio. La Biblioteca de Pregrado, con una capacidad estimada de 295 personas, opera sin ningún sistema de monitoreo de aforo, lo que genera una asimetría de información que afecta directamente la experiencia académica del estudiantado.

### 1.2 Problemática ODS Específica

El proyecto se enmarca en el ODS 11 (Ciudades y Comunidades Sostenibles), específicamente en la meta de optimizar el uso de infraestructura existente antes de recurrir a la expansión física. A través de una encuesta a 69 estudiantes del campus (Avance #1, marzo 2026) se cuantificó la magnitud del problema:

- El **74,2%** de los encuestados se ha visto forzado a abandonar la biblioteca por falta de espacio
- El **39,7%** pierde entre 5 y más de 10 minutos buscando un asiento disponible
- El **50,7%** asiste en bloque M3 (11:30), el horario de mayor saturación
- El **59,4%** reporta contaminación acústica en horas peak

### 1.3 Objetivos del Proyecto

- Desarrollar un sistema IoT de detección direccional de personas para monitoreo de aforo en tiempo real
- Almacenar los datos de ocupación en una plataforma accesible con histórico persistente
- Visualizar la información mediante un dashboard que permita tomar decisiones informadas
- Diseñar un encapsulado funcional para instalación en condiciones reales de operación

### 1.4 Alcance

El prototipo SIMA está diseñado para la entrada principal de la Biblioteca de Pregrado, Campus Peñalolén, UAI. Opera en entorno de red WiFi local, es autónomo energéticamente con batería recargable y no requiere intervención manual para su operación cotidiana. El sistema es replicable a otros espacios de acceso controlado del campus.

---

## 2. Marco Teórico y Estado del Arte

### 2.1 ODS 11 — Ciudades y Comunidades Sostenibles

El Objetivo de Desarrollo Sostenible 11 propone hacer las ciudades y los asentamientos humanos inclusivos, seguros, resilientes y sostenibles. En el contexto universitario, esto se traduce en la gestión inteligente de espacios educativos: maximizar la utilidad de la infraestructura existente antes de recurrir a costosas expansiones físicas que generan impacto económico y ambiental adicional.

### 2.2 Problemática Local

La saturación de espacios de estudio universitarios es un fenómeno documentado que deteriora el rendimiento académico e incrementa el estrés cognitivo (Vásquez Meza et al., 2024). En la UAI, la concentración de demanda en bloques M3-M5 genera cuellos de botella que el modelo de gestión manual actual no puede resolver sin información en tiempo real.

### 2.3 Soluciones Existentes

Las soluciones comerciales de conteo de personas incluyen sistemas de cámaras con visión computacional, sensores infrarrojos de barrera y contadores de presión en el suelo. Estos sistemas tienen costos de implementación que oscilan entre USD 500 y USD 5.000 por punto de conteo y, en el caso de las cámaras, implican consideraciones de privacidad. SIMA propone una alternativa de código abierto, bajo costo y sin cámaras.

### 2.4 Justificación de la Propuesta

La combinación ESP32-S3 + sensores ultrasónicos Seeed permite implementar un contador direccional funcional por menos de $30.000 CLP, sin procesamiento de imagen y con privacidad garantizada. La arquitectura FreeRTOS dual-core del ESP32-S3 permite separar la lógica de detección del envío de datos, resolviendo el principal desafío técnico de los sistemas IoT de bajo costo: la latencia bloqueante.

---

## 3. Metodología

El desarrollo de SIMA siguió un proceso de diseño iterativo en tres fases, cada una correspondiente a un avance del curso. El enfoque metodológico combina el Design Thinking para la identificación del problema con el desarrollo ágil para la implementación técnica.

- **Fase 1 (Avance #1):** Investigación del problema con encuesta a 69 estudiantes y definición de requerimientos mediante Framework 6W
- **Fase 2 (Avance #2):** Prototipo alpha con validación de factibilidad técnica e identificación de problemas de hardware y conectividad
- **Fase 3 (Avance #3):** Prototipo funcional completo con integración de nube, persistencia de datos y validación con protocolo de pruebas

**Tabla 1: Herramientas utilizadas en el proyecto**

| Área | Herramienta | Uso |
|---|---|---|
| Firmware | Arduino IDE 2.x | Desarrollo y carga del firmware ESP32-S3 |
| Control de versiones | GitHub | Repositorio del proyecto y documentación |
| Almacenamiento | Google Sheets + Apps Script | Base de datos histórica de eventos |
| Visualización | Google Looker Studio | Dashboard con gráficas de tendencias |
| Diseño 3D | Autodesk Fusion 360 | Gemelo digital y encapsulado |
| Esquemático | Fritzing / Wokwi | Diagrama de conexiones del circuito |

---

## 4. Diseño y Desarrollo

### 4.1 Arquitectura del Sistema

El sistema SIMA implementa el ciclo completo de datos IoT: los sensores capturan el flujo de personas, el ESP32-S3 procesa la dirección y gestiona la comunicación, Google Sheets persiste los datos históricos y Google Looker Studio los visualiza. Adicionalmente, un servidor web embebido en el ESP32-S3 provee acceso local en tiempo real.

**Tabla 2: Arquitectura de capas del sistema SIMA**

| Capa | Componente | Función |
|---|---|---|
| Captura | 2× Seeed Grove HC-SR04 (GPIO4, GPIO5) | Detección de presencia y dirección de cruce |
| Procesamiento | ESP32-S3 Dual-core Xtensa LX7 | Lógica de conteo, servidor web, envío HTTP |
| Almacenamiento | Google Sheets + NVS Flash | Histórico en nube + persistencia local |
| Visualización | Dashboard web local + Looker Studio | Aforo en tiempo real + análisis histórico |

### 4.2 Hardware

El prototipo está construido en dos módulos físicos conectados por cable: el módulo principal contiene el ESP32-S3, la batería, el shield de carga y el sensor B (interior); el módulo secundario contiene el sensor A (pasillo exterior). Esta configuración permite instalar los sensores en los lados opuestos del marco de la puerta sin necesitar una carcasa de gran tamaño, optimizando el tiempo de impresión 3D.

**Tabla 3: Bill of Materials (BOM) del sistema SIMA**

| Componente | Especificación | Cantidad | Costo (CLP) |
|---|---|---|---|
| ESP32-S3 Dev Module | Dual-core 240MHz, WiFi, 512KB SRAM | 1 | $6.990 |
| Seeed Grove HC-SR04 | 3 pines, 3.3V, rango 3-350cm, GPIO4/GPIO5 | 2 | $9.980 |
| Batería NCR18650B | Panasonic, 3.400mAh, 3.7V, >500 ciclos | 1 | $2.000 |
| Shield cargador 18650 | Micro-USB, salida USB-A 5V/1A, boost converter | 1 | $3.990 |
| Cable USB-A a USB-C | 30cm, conexión shield → ESP32-S3 (COM/UART) | 1 | $1.990 |
| Jumper wires M-M | 20cm, 40 unidades | 1 set | $1.490 |
| Protoboard 400 puntos | Conexión sin soldadura de sensores | 1 | $1.990 |
| Filamento PLA | ~50g para encapsulado (impresora UAI) | 1 | $5.000 |
| **TOTAL** | | | **$27.960** |

### 4.3 Software

El firmware del ESP32-S3 implementa una arquitectura FreeRTOS dual-core que resuelve el principal problema de los sistemas IoT síncronos: el bloqueo del loop de detección durante las llamadas HTTP. El Core 1 ejecuta la detección de personas de forma continua a 10 Hz, mientras el Core 0 gestiona exclusivamente el envío de datos a Google Sheets sin interferir con la detección.

**Lógica de Detección Direccional**

El sistema determina la dirección del cruce por el orden de activación de los sensores. Si el Sensor A (pasillo exterior, GPIO4) se activa antes que el Sensor B (interior, GPIO5), se registra una ENTRADA. La secuencia inversa registra una SALIDA. El conteo se consolida únicamente cuando ambos sensores vuelven a su estado inactivo, previniendo falsos positivos. Un timeout de 2.000ms cancela la secuencia si no se completa.

**Gestión de Datos**

- **NVS Flash (Preferences.h):** El contador se escribe en la memoria no volátil del ESP32-S3 en cada evento. Recuperación confirmada en 5/5 reinicios forzados.
- **Google Sheets vía HTTPS:** Cada evento (ENTRADA/SALIDA/RESET) se envía con timestamp ISO 8601 (UTC-4), tipo de evento y total de personas. Latencia: 1,5-3 segundos.
- **Reset automático nocturno:** Si el sistema estuvo apagado más de 2 horas, el contador vuelve a 0 automáticamente al encender.
- **Dashboard web local:** Servidor HTTP en puerto 80, accesible desde cualquier dispositivo en la misma red, con actualización automática cada 3 segundos.

### 4.4 Diseño Mecánico y 3D

El encapsulado fue modelado en Autodesk Fusion 360 como gemelo digital del hardware real, con todos los componentes internos representados a sus dimensiones reales.

- Material: PLA (impresión 3D FDM) — color blanco con tapa rosa
- Orificios frontales calibrados para transductores de 15,8mm de diámetro
- Tapa desmontable sin adhesivos — diseño para reparación en campo
- Orificio lateral USB-A para carga de batería y switch de encendido

---

## 5. Proceso de Iteraciones

### 5.1 Versión 1 Alpha — Concepto Inicial (Abril 2026)

Primera prueba de concepto en laboratorio con sensores HC-SR04 de 4 pines y conexión USB al computador. Validó la factibilidad básica del servidor web embebido pero expuso tres problemas críticos: incompatibilidad de voltaje (5V vs 3,3V en GPIO), bloqueo de la red WiFi institucional por portal cautivo, y falsos positivos cuando una persona se detenía en el marco de la puerta.

### 5.2 Versión 2 Beta — Primera Mejora (Mayo 2026)

Resolvió los tres problemas críticos de v1: migración a sensores Seeed Grove de 3 pines (compatibles con 3,3V nativamente), modo Access Point autónomo para independizarse de la red UAI, y algoritmo anti-rebote con secuencia estricta A→B / B→A. Incorporó autonomía energética con Battery Shield + celda NCR18650B (~6h) y el primer dashboard web con semáforo visual.

### 5.3 Versión 4 Final — Sistema Completo (Junio 2026)

Implementó el ciclo completo de datos con corrección crítica de GPIO (cambio de GPIO1/GPIO10 a GPIO4/GPIO5 para resolver boot loop), arquitectura FreeRTOS dual-core, persistencia NVS, integración HTTPS con Google Sheets (HTTP 200 OK confirmado) y extensión de autonomía a 16-20 horas mediante modo modem sleep.

**Tabla 4: Comparativa de las tres iteraciones del sistema SIMA**

| Aspecto | v1 Alpha | v2 Beta | v4 Final |
|---|---|---|---|
| Sensores | HC-SR04 5V X | Seeed 3.3V ✓ | Seeed 3.3V ✓ |
| GPIO sensores | Sin definir | GPIO1/GPIO10 ⚠ | GPIO4/GPIO5 ✓ |
| Conectividad | Red UAI bloqueada X | AP autónomo ✓ | Router + Sheets ✓ |
| Almacenamiento | Sin persistencia X | Sin persistencia X | NVS + Sheets ✓ |
| Autonomía | USB (sin batería) | ~6 horas | ~16-20 horas |
| Falsos positivos | Sí X | No ✓ | No (<5%) ✓ |
| Encapsulado | Sin encapsulado X | Boceto Fusion 360 | Gemelo 3D completo ✓ |
| Precisión global | Sin medir | 4,8/5 (subjetivo) | 91,4% (35 pruebas) ✓ |

---

## 6. Testing y Validación

### 6.1 Metodología de Testing

El protocolo de pruebas se diseñó para validar cuatro aspectos críticos del sistema: precisión de detección direccional, robustez ante casos borde, persistencia de datos ante fallos de energía y confiabilidad del envío a Google Sheets. Las pruebas se realizaron en condiciones controladas simulando el uso real del dispositivo.

### 6.2 Resultados Cuantitativos

**Tabla 5: Resultados del protocolo de pruebas — 35 casos totales**

| Prueba | Total casos | Correctos | Incorrectos | Tasa de éxito |
|---|---|---|---|---|
| Detección entrada (A→B) | 10 | 9 | 1 | 90% ✓ |
| Detección salida (B→A) | 10 | 8 | 2 | 80% ✓ |
| Anti-bloqueo timeout | 5 | 5 | 0 | 100% ✓ |
| Persistencia energía | 5 | 5 | 0 | 100% ✓ |
| Envío a Google Sheets | 5 | 5 | 0 | 100% ✓ |
| **TOTAL** | **35** | **32** | **3** | **91,4% ✓** |

La latencia de detección fue de 100-200ms (1-2 ciclos del loop a 10Hz), mientras que la latencia de envío a Google Sheets se mantuvo entre 1,5 y 3 segundos, dentro del timeout de 8 segundos configurado.

### 6.3 Análisis de Fallas

**Falla 1 — Boot Loop por GPIO Conflictivos**

El ESP32-S3 entraba en ciclo de reinicio constante al usar GPIO1 (TX del UART0) y GPIO10 (mapeado al controlador SPI interno) para los sensores. La solución fue migrar a GPIO4 y GPIO5, más un erase completo de flash mediante secuencia BOOT+RESET. Resultado: cero boot loops en todas las pruebas posteriores.

**Falla 2 — Error HTTP 400 en Google Sheets**

Las peticiones al Apps Script retornaban código 400 por tres causas simultáneas: timestamp con espacio en la URL, ausencia de WiFiClientSecure para HTTPS, y falta de setFollowRedirects para manejar la redirección 302 de Google. Las tres correcciones aplicadas resultaron en HTTP 200 OK confirmado en todos los envíos posteriores.

### 6.4 Validación de Impacto ODS 11

**Tabla 6: Comparativa baseline vs. resultado — Indicadores de impacto ODS 11**

| Indicador | Baseline (Avance #1) | Con SIMA |
|---|---|---|
| Información aforo en tiempo real | 0% disponibilidad | Dashboard cada 3 seg ✓ |
| Persistencia histórica de datos | Sin almacenamiento | Google Sheets ilimitado ✓ |
| Autonomía del dispositivo | Sin referencia | 16-20 horas ✓ |
| Precisión de detección | Sin referencia | 91,4% (35 pruebas) ✓ |

Con 8.000 estudiantes diarios en el campus y el 39,7% afectado por la falta de información de aforo, un despliegue completo de SIMA en los accesos de la Biblioteca de Pregrado podría eliminar el costo de oportunidad temporal para aproximadamente 3.176 estudiantes por día, a un costo marginal de $0 por consulta.

---

## 7. Conclusiones

### 7.1 Logros Alcanzados

- Sistema IoT funcional con detección direccional de personas y precisión global del 91,4%
- Ciclo completo de datos implementado: captura → procesamiento → almacenamiento → visualización
- Costo total de $28.430 CLP por nodo — viable para replicación institucional
- Persistencia de datos ante cortes de energía: 100% en 5 reinicios forzados
- Autonomía de 16-20 horas continuas — suficiente para una jornada completa de biblioteca
- Encapsulado con diseño para reparación en campo, sin adhesivos permanentes

### 7.2 Limitaciones Identificadas

- El sistema está optimizado para flujo en fila simple — no discrimina correctamente cruces simultáneos
- Dependencia de red WiFi estable para sincronización con Google Sheets
- El aforo de 295 personas no fue validado con conteo físico de asientos reales

### 7.3 Aprendizajes del Equipo

- **Técnico:** Los GPIO con funciones reservadas del chip no deben usarse para sensores que cambien constantemente su dirección — verificar pinout antes de conectar
- **Técnico:** Las llamadas HTTP síncronas bloquean el loop principal en IoT — FreeRTOS es la solución correcta para concurrencia en ESP32
- **Proceso:** La prueba de concepto mínima es indispensable para identificar problemas reales no predecibles en el diseño en papel
- **Equipo:** Documentar cada decisión técnica en tiempo real facilita la elaboración de la documentación final

### 7.4 Trabajo Futuro

- Agregar sensor infrarrojo de barrera como redundancia para condiciones de baja iluminación
- Implementar integración con Telegram o app móvil para consulta remota sin estar en la red local
- Validar el aforo real de la Biblioteca F con conteo físico de asientos para calibrar el sistema
- Despliegue en múltiples accesos del campus con servidor central que agregue datos de todos los nodos SIMA

---

## 8. Referencias

[1] Espressif Systems. (2023). *ESP32-S3 Datasheet*. https://www.espressif.com/sites/default/files/documentation/esp32-s3_datasheet_en.pdf

[2] Seeed Studio. (2023). *Grove — Ultrasonic Ranger (SKU 101020010)*. https://wiki.seeedstudio.com/Grove-Ultrasonic_Ranger/

[3] Panasonic. (2012). *Lithium Ion Battery NCR18650B Datasheet*. https://www.tme.eu/Document/3e0170a1e089819f286f7066e69035b4/NCR18650B.pdf

[4] Vásquez Meza, C. et al. (2024). Impacto del hacinamiento acústico en espacios educativos universitarios sobre el rendimiento cognitivo. *Revista de Psicología Educacional*.

[5] Gómez García, M. et al. (2018). Saturación visual y ruido en recintos de estudio universitario. *Journal of Library Administration*.

[6] Ministerio de Educación Chile. (2026). Matrícula UAI 2026. Portal MiFuturo. https://mifuturo.cl

[7] Espressif Systems. (2023). *ESP-IDF FreeRTOS Documentation*. https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/system/freertos.html

[8] Google. (2024). *Apps Script — UrlFetchApp Reference*. https://developers.google.com/apps-script/reference/url-fetch
