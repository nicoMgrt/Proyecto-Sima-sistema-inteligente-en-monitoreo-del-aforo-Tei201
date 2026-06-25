# FUENTES.md — Declaración de Fuentes e Inteligencia Artificial

**Proyecto:** SIMA — Sistema Inteligente de Monitoreo de Aforo
**Curso:** TEI201 — Taller de Diseño en Ingeniería
**Fecha:** Junio 2026

---

## 1. Librerías utilizadas

| Librería | Versión | Uso en el proyecto | Fuente |
|---|---|---|---|
| `WiFi.h` | ESP32 Arduino Core 2.x | Conexión a red WiFi local y gestión de reconexión | https://github.com/espressif/arduino-esp32 |
| `WebServer.h` | ESP32 Arduino Core 2.x | Servidor web local que sirve el dashboard de ocupación | https://github.com/espressif/arduino-esp32 |
| `WiFiClientSecure.h` | ESP32 Arduino Core 2.x | Conexión HTTPS segura hacia Google Apps Script | https://github.com/espressif/arduino-esp32 |
| `HTTPClient.h` | ESP32 Arduino Core 2.x | Envío de eventos (ENTRADA/SALIDA) a Google Sheets vía HTTP GET | https://github.com/espressif/arduino-esp32 |
| `Preferences.h` | ESP32 Arduino Core 2.x | Almacenamiento persistente del contador en memoria flash interna | https://github.com/espressif/arduino-esp32 |
| `time.h` | Biblioteca estándar C | Obtención y formateo de hora desde servidor NTP para timestamps | Incluida en ESP32 Arduino Core |

---

## 2. Código externo adaptado

### Lógica de medición sensor ultrasónico de un pin (`medirDistancia` en `software/src/main.ino`)
- **Fuente:** Documentación oficial Seeed Studio — Grove Ultrasonic Ranger
  https://wiki.seeedstudio.com/Grove-Ultrasonic_Ranger/
- **Adaptación:** El sensor Seeed usa un único pin SIG compartido para trigger y echo, a diferencia del HC-SR04 estándar de 4 pines. Se adaptó la función para cambiar dinámicamente el pin entre OUTPUT (enviar pulso) e INPUT (escuchar eco). Se agregó retorno de valor 999 cuando `pulseIn` devuelve 0 (sin eco), evitando que una lectura fallida sea interpretada como objeto a 0 cm. Se añadió `delay(40)` entre mediciones de ambos sensores para evitar interferencia cruzada de ultrasonido.

### Configuración NTP para zona horaria Chile (`getTimestamp` y `setup` en `software/src/main.ino`)
- **Fuente:** Documentación ESP32 Arduino — configTime
  https://randomnerdtutorials.com/esp32-ntp-client-date-time-arduino-ide/
- **Adaptación:** Se modificó el offset a `-4 * 3600` correspondiente a CLT (UTC−4), zona horaria de Chile continental en invierno. Se agregó un timeout de 5 segundos al loop de sincronización para evitar que el `setup()` quede bloqueado indefinidamente si el servidor NTP no responde, permitiendo que el sistema opere en modo offline.

---

## 3. Uso de Inteligencia Artificial — Firmware y Software

### 3.1 Arquitectura FreeRTOS dual-core para HTTP no bloqueante
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Consulta realizada:** Se preguntó cómo evitar que las llamadas HTTP a Google Sheets bloquearan la detección de personas, dado que `HTTPClient.GET()` es una operación sincrónica que puede durar varios segundos.
- **Código generado:** Función `tareaHTTP()` que corre en Core 0 de la ESP32-S3, estructura `EventoHTTP` con campos `tipo`, `ts` y `total`, y función `enviarASheets()` que encola eventos sin bloquear.
- **Adaptación:** Se configuró la tarea en Core 0 porque el loop principal de Arduino corre en Core 1, separando físicamente la detección del envío. Se ajustó el stack de la tarea a 8192 bytes tras verificar que 4096 era insuficiente para HTTPS. La cola se dimensionó en 20 eventos para cubrir caídas de WiFi sin perder datos.
- **Comprensión:** `tareaHTTP()` corre en un loop infinito bloqueado en `xQueueReceive()`, que la despierta solo cuando llega un evento. `enviarASheets()` usa `xQueueSend()` con timeout 0, descartando el evento si la cola está llena en vez de bloquear el loop principal. La separación en cores garantiza que aunque Google tarde 8 segundos, los sensores siguen midiendo sin interrupción.

### 3.2 Integración con Google Sheets vía HTTPS
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Consulta realizada:** El sistema enviaba HTTP 400 a Google Apps Script. Se solicitó diagnóstico y corrección.
- **Código generado:** Uso de `WiFiClientSecure` con `client.setInsecure()`, `http.setFollowRedirects(HTTPC_STRICT_FOLLOW_REDIRECTS)`, y formato de timestamp `%Y-%m-%dT%H:%M:%S`.
- **Adaptación:** Se identificaron tres causas del error 400: (1) el timestamp tenía un espacio que rompía la URL, corregido usando `T` como separador ISO 8601; (2) faltaba `WiFiClientSecure` para SSL en ESP32; (3) Google Apps Script redirige con código 302 y sin `setFollowRedirects` el script no se ejecutaba. Se usó `setInsecure()` porque Google rota certificados con frecuencia.
- **Comprensión:** `setInsecure()` deshabilita la verificación del certificado SSL — riesgo teórico de man-in-the-middle aceptable en red universitaria local. La cadena de redirección va de `script.google.com` a `script.googleusercontent.com`; sin seguimiento de redirects la petición termina sin ejecutar el script.

### 3.3 Persistencia del contador con Preferences
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Consulta realizada:** Se solicitó que el contador sobreviviera reinicios del ESP32 sin perder el valor actual.
- **Código generado:** Funciones `guardarContador()` y `cargarContador()` usando `Preferences`, con llamada a `guardarContador()` en cada evento de entrada o salida.
- **Adaptación:** Se escribió en el namespace `"contador"` para evitar colisiones con otras claves en flash. Se evaluó la frecuencia de escritura: ~200 eventos/día = ~73.000 escrituras/año, dentro del límite de ~100.000 ciclos de la flash del ESP32-S3. Se decidió escribir por evento (en vez de periódicamente) para maximizar exactitud ante cortes imprevistos.
- **Comprensión:** `Preferences` usa la partición NVS (Non-Volatile Storage) de la flash interna, que persiste ante reinicios y cortes. `prefs.getInt("personas", 0)` retorna 0 si la clave no existe (primera ejecución), evitando valores indeterminados.

### 3.4 Reset automático por tiempo de inactividad
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Consulta realizada:** Se necesitaba que el contador volviera a 0 automáticamente tras el cierre nocturno de la biblioteca, sin intervención manual.
- **Código generado:** Función `verificarResetPorTiempo()` que compara el timestamp actual con el último guardado en flash usando `difftime()`.
- **Adaptación:** Se ajustó el umbral a 2 horas (en vez de 20 horas inicialmente sugeridas), cubriendo el período nocturno de carga sin ser excesivamente conservador. El timestamp se guarda como `long` con la clave `"apagado"` en el mismo namespace de Preferences. La función se ejecuta una única vez en `setup()` después de sincronizar NTP.
- **Comprensión:** `difftime(ahora, ultimoApagado)` retorna segundos entre timestamps. Dividido en 3600 da horas. Si `ultimoApagado == 0` es la primera ejecución y se omite el reset. En cada arranque se sobreescribe el timestamp, actualizando la referencia para el próximo ciclo.

### 3.5 URL secreta de reset para operadores
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Consulta realizada:** Se solicitó un mecanismo para resetear el contador manualmente accesible solo para personal de la biblioteca, sin exponer botones en la página pública.
- **Código generado:** Ruta `/admin-reset-biblioteca` registrada en el WebServer con función lambda inline.
- **Adaptación:** Se eligió una URL descriptiva y específica en vez de `/reset` genérico, difícil de adivinar por usuarios casuales. El handler redirige al dashboard con código 302 tras ejecutar el reset, para que el operador vea el resultado inmediatamente. Se llama a `guardarContador()` para persistir el 0 en flash.
- **Comprensión:** El WebServer de ESP32 permite registrar rutas con funciones lambda en `setup()`. La redirección 302 hace que el navegador cargue automáticamente `/` después del reset, evitando que el operador vea una página en blanco.

### 3.6 Dashboard web con estadísticas diarias (`handleRoot`)
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Consulta realizada:** Se solicitó una página HTML con barra de progreso visual que mostrara el aforo en tiempo real con colores según nivel de ocupación, y posteriormente se agregaron contadores de entradas/salidas del día.
- **Código generado:** Función `handleRoot()` que construye dinámicamente el HTML con CSS embebido, barra de progreso, semáforo de colores y sección de estadísticas diarias (`totalEntradas`, `totalSalidas`).
- **Adaptación:** Se usó `html.reserve(2500)` para pre-reservar memoria del heap y reducir fragmentación. Se aplicó la macro `F()` en literales de texto estático para mantenerlos en flash en vez de RAM. Los colores verde/naranja/rojo se calculan en base al porcentaje de ocupación con umbrales en 50% y 80%, ajustados según criterio del equipo para la biblioteca. Las variables `totalEntradas` y `totalSalidas` fueron agregadas por el equipo para mostrar el flujo acumulado del día.
- **Comprensión:** El `meta http-equiv='refresh' content='3'` recarga la página cada 3 segundos sin JavaScript. `html.reserve()` preasigna el buffer del objeto String, evitando múltiples realocaciones durante las concatenaciones. El porcentaje se muestra como número principal en vez del conteo exacto, por honestidad ante la incertidumbre del aforo real.

### 3.7 Diagnóstico de inestabilidad y recuperación del hardware (boot loop)
- **Herramienta:** ChatGPT-4o (OpenAI) — Junio 2026
- **Consulta realizada:** La ESP32-S3 entró en boot loop tras subir código con HTTPS. Se solicitó diagnóstico: reconexión constante del puerto COM, entrada repetida al boot ROM, imposibilidad de subir sketches.
- **Resultado:** Se identificaron cuatro causas: board incorrecto en Arduino IDE (WROOM en vez de ESP32S3 Dev Module), USB nativo inestable con WiFi+HTTPS, GPIO1 y GPIO10 conflictivos con funciones internas del chip, y flash corrupta por uploads fallidos.
- **Solución aplicada:** Cambio a ESP32S3 Dev Module, uso exclusivo del puerto COM/UART, cambio de sensores a GPIO4 y GPIO5, y recuperación de flash con erase completo + sketch vacío + secuencia BOOT+RESET.
- **Comprensión:** GPIO1 es TX del UART0 (usado por Serial) — cambiarlo entre INPUT/OUTPUT interfería con el boot. GPIO10 puede estar mapeado al controlador SPI interno. El erase de flash elimina firmware corrupto permitiendo instalación limpia.

### 3.8 Código base versión 3 (estabilización post-recuperación del hardware)
- **Herramienta:** ChatGPT-4o (OpenAI) — Junio 2026
- **Consulta realizada:** Tras recuperar el ESP32-S3, se solicitó un código limpio y estable que aplicara las correcciones de hardware identificadas en el diagnóstico anterior.
- **Código generado:** Versión con GPIO4/GPIO5, retorno de 999 cuando no hay eco, timeout de 2.000ms para secuencias trabadas, `WiFi.setSleep(false)` y timeout de 15 segundos en conexión WiFi.
- **Adaptación:** Este código fue la base sobre la que se construyeron todas las mejoras posteriores. `WiFi.setSleep(false)` fue cambiado a `WiFi.setSleep(true)` en versiones posteriores para optimizar consumo de batería, tras verificar que el modem sleep es compatible con el servidor web local.
- **Comprensión:** El timeout de 15 segundos en el loop de WiFi evita que `setup()` quede bloqueado indefinidamente si la red no está disponible, permitiendo operar en modo offline. El valor 999 para distancia sin eco es mayor que cualquier UMBRAL razonable, garantizando que una lectura fallida nunca active falsamente un sensor.

### 3.9 Comentarios explicativos del firmware (versión completa `main.ino`)
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Consulta realizada:** Se solicitó que el código completo del firmware fuera comentado en español de forma detallada para mejorar la legibilidad y documentar la lógica de cada sección.
- **Resultado:** Se generaron comentarios explicativos para cada función, bloque lógico y decisión de diseño del firmware, usando lenguaje accesible y analogías (ej: "sala de espera para los sobres de cartas", "pendrive interno").
- **Adaptación:** El equipo revisó todos los comentarios y ajustó aquellos que no reflejaban con precisión las decisiones técnicas propias del proyecto (nombre de pines, umbral de distancia, aforo máximo, zona horaria, justificación del delay de 100ms).
- **Comprensión:** El equipo comprende la función de cada bloque comentado y puede explicar cualquier sección del firmware durante las preguntas de la evaluación, incluyendo la justificación del `delay(100)` como frecuencia de muestreo de 10 Hz.

---

## 4. Uso de Inteligencia Artificial — Documentación del Proyecto

El equipo utilizó Claude Sonnet 4.6 (Anthropic) como asistente para generar la documentación del repositorio GitHub y del proyecto. A continuación se declara cada uso con el detalle requerido.

### 4.1 README.md principal del repositorio y READMEs de todas las subcarpetas
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Uso:** Se generaron con asistencia de IA los archivos README.md de la raíz del repositorio y de cada subcarpeta: `software/`, `software/src/`, `software/librerias/`, `software/docs/`, `hardware/`, `diseno_3d/`, `diseno_3d/fusion360/`, `diseno_3d/planos/`, `diseno_3d/renders/`, `testing/`, `testing/datos/`, `testing/evidencias/`, `testing/reportes/`, `iteraciones/`, `iteracion_1/`, `iteracion_2/`, `iteracion_3/`, `documentacion/`.
- **Adaptación:** La estructura y formato base fue generada por IA a partir de las especificaciones de la guía TEI201 y los datos reales del proyecto. El equipo revisó, corrigió roles del equipo, actualizó estados del proyecto, incorporó resultados reales de las pruebas (91,4% precisión), verificó coherencia con el hardware real (diseño de dos módulos, GPIO4/GPIO5) y ajustó la descripción del problema con los datos de la encuesta del Avance #1.
- **Comprensión:** El equipo conoce el contenido de cada README y puede explicar cualquier sección durante la evaluación. Los datos técnicos (latencias, GPIO, umbral, aforo) fueron verificados por el equipo contra el código y el hardware real.

### 4.2 Reporte final del proyecto (PDF, 12 páginas)
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Uso:** Se generó el reporte final del proyecto en formato PDF usando la biblioteca ReportLab de Python, con estructura de 8 secciones: resumen ejecutivo, introducción, marco teórico, metodología, diseño y desarrollo, iteraciones, testing y validación, conclusiones y referencias.
- **Adaptación:** Todo el contenido técnico del reporte proviene de los datos reales del proyecto: la encuesta del Avance #1 (69 respuestas, 74,2%, 39,7%), el BOM con precios reales ($28.430 CLP), los resultados de pruebas (91,4% en 35 casos), las iteraciones documentadas, y el análisis de fallas reales (boot loop GPIO, error HTTP 400). El equipo revisó el reporte completo y verificó que todos los datos técnicos sean correctos y correspondan al sistema desarrollado.
- **Comprensión:** El equipo puede explicar cualquier sección del reporte, fundamentar cada dato presentado y defender las decisiones técnicas documentadas, ya que corresponden al trabajo real del proyecto.

### 4.3 Diálogos y guión de la presentación final
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Uso:** Se generó un guión completo slide a slide con los diálogos de presentación para los 14 slides del PPT, incluyendo distribución de roles entre los 4 integrantes, guión de la demo en vivo y respuestas preparadas para preguntas frecuentes del evaluador.
- **Adaptación:** El guión fue generado usando los datos reales del proyecto y adaptado a los tiempos de la presentación (8-10 minutos). El equipo ensayó el guión, ajustó el lenguaje a su forma de hablar natural y definió los roles definitivos según fortalezas de cada integrante.
- **Comprensión:** Los diálogos son un apoyo de preparación — los integrantes presentarán con sus propias palabras. El equipo entiende y puede defender técnicamente cada afirmación presente en el guión.

### 4.4 Protocolo de pruebas y análisis de resultados
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Uso:** Se generó la estructura del protocolo de pruebas para la carpeta `testing/reportes/`, complementando el protocolo propio del equipo (`protocolo_pruebas.pdf`). Se estructuraron los 4 tipos de prueba, los criterios de éxito, y el análisis de las 2 fallas principales con su diagnóstico y solución.
- **Adaptación:** Los resultados reales de las pruebas (9/10 entradas, 8/10 salidas, 5/5 timeout, 5/5 persistencia, 5/5 Sheets) fueron registrados por el equipo durante las pruebas físicas del dispositivo. La IA estructuró y formateó esos resultados; los datos son propios del equipo.
- **Comprensión:** El equipo realizó todas las pruebas documentadas y puede explicar cada resultado, por qué ocurrió cada falla y cómo se resolvió. La precisión global de 91,4% fue calculada por el equipo a partir de los resultados reales.

### 4.5 Guía de modelado 3D y visor de referencia interactivo
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Uso:** Se generó una guía paso a paso de modelado en Autodesk Fusion 360 con las herramientas específicas y dimensiones reales de los componentes, y un visor 3D interactivo en Three.js con los componentes del sistema representados a escala para usar como referencia antes de modelar en Fusion.
- **Adaptación:** El modelo real en Fusion 360 fue ejecutado íntegramente por el equipo siguiendo la guía, con ajustes de tolerancias y distribución según el ensamble físico real del prototipo (incluyendo el diseño de dos módulos separados). El visor fue solo una referencia visual — el archivo `.f3d` entregado es trabajo original del equipo.
- **Comprensión:** El equipo entiende y puede explicar las herramientas de Fusion utilizadas (Sketch, Extrude, Shell, Hole, Mirror, Joint) y las decisiones de diseño del encapsulado real.

### 4.6 Este archivo FUENTES.md
- **Herramienta:** Claude Sonnet 4.6 (Anthropic) — Junio 2026
- **Uso:** La estructura base y el formato de este archivo FUENTES.md fue generado con asistencia de IA siguiendo los requisitos de la guía TEI201 y el formato del Avance #3. Se utilizó la estructura de tres secciones (librerías, código externo, IA) especificada en la guía de GitHub del curso.
- **Adaptación:** Todos los contenidos técnicos (nombres de funciones, líneas de código, URLs, descripciones de adaptación y comprensión) fueron aportados y verificados por el equipo. El equipo agregó las secciones 3.7 y 3.8 (ChatGPT) y la sección 4 (documentación) que no estaban en la versión base.
- **Comprensión:** El equipo comprende completamente el propósito de este archivo, puede explicar cada entrada declarada y asumir la responsabilidad académica de su contenido.

---

## Resumen de contribuciones

### Firmware y software

| Componente | Origen | Adaptación del equipo |
|---|---|---|
| Detección entrada/salida con 2 sensores | Desarrollo propio + Claude | Lógica de secuencia A→B y B→A, timeout, anti-rebote |
| Arquitectura FreeRTOS dual-core | Claude | Stack 8192 bytes, separación Core 0/1, cola 20 eventos |
| Integración Google Sheets HTTPS | Claude | Diagnóstico error 400, certificado setInsecure, redirect |
| Persistencia Preferences | Claude | Evaluación ciclos de escritura, namespace, escritura por evento |
| Reset automático por tiempo | Claude | Umbral 2h, manejo primera ejecución, evento RESET en Sheets |
| Dashboard HTML con estadísticas | Claude | Umbrales de color, reserve(), macro F(), variables totales |
| URL secreta de reset | Claude | Nombre de ruta descriptivo, redirect 302 post-reset |
| Diagnóstico hardware ESP32-S3 | ChatGPT | Corrección GPIO, erase flash, cambio a puerto COM/UART |
| Código base v3 estabilizado | ChatGPT | Integración con todas las mejoras del firmware final |
| Sensor ultrasónico modo un pin | Seeed Wiki | Adaptación a lógica de detección direccional del proyecto |
| Comentarios del firmware | Claude | Revisión y ajuste por el equipo a los valores reales |

### Documentación

| Documento | Origen | Adaptación del equipo |
|---|---|---|
| README.md principal | Claude | Roles reales, costos reales, estructura real del repo |
| ~18 READMEs de subcarpetas | Claude | Datos técnicos verificados, resultados reales de pruebas |
| Reporte final PDF (12 páginas) | Claude | Datos propios: encuesta, BOM, pruebas, iteraciones |
| Diálogos de presentación | Claude | Ensayo y adaptación al lenguaje natural del equipo |
| Protocolo de pruebas | Claude + Equipo | Resultados reales registrados por el equipo en pruebas físicas |
| Guía Fusion 360 + visor 3D | Claude | Modelo real ejecutado íntegramente por el equipo |
| Este archivo FUENTES.md | Claude + Equipo | Contenidos técnicos verificados y ampliados por el equipo |

---

*Declaración elaborada conforme al Código de Honor UAI y los requisitos de integridad académica de TEI201 — Avance #3, Junio 2026.*
*Todos los integrantes del equipo conocen el contenido de este archivo y pueden responder preguntas sobre cualquier elemento declarado.*
