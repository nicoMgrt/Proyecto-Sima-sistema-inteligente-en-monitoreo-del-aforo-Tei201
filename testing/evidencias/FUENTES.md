# FUENTES.md — Declaración de Fuentes e Inteligencia Artificial

**Proyecto:** Sistema IoT Contador de Personas — Biblioteca UAI  
**Curso:** TEI201 — Taller de Diseño en Ingeniería  
**Fecha:** Junio 2026

---

## 1. Librerías utilizadas

| Librería | Versión | Uso en el proyecto | Fuente |
|---|---|---|---|
| WiFi.h | ESP32 Arduino Core 2.x | Conexión a red WiFi local y gestión de reconexión | https://github.com/espressif/arduino-esp32 |
| WebServer.h | ESP32 Arduino Core 2.x | Servidor web local que sirve el dashboard de ocupación | https://github.com/espressif/arduino-esp32 |
| WiFiClientSecure.h | ESP32 Arduino Core 2.x | Conexión HTTPS segura hacia Google Apps Script | https://github.com/espressif/arduino-esp32 |
| HTTPClient.h | ESP32 Arduino Core 2.x | Envío de eventos (ENTRADA/SALIDA) a Google Sheets vía HTTP GET | https://github.com/espressif/arduino-esp32 |
| Preferences.h | ESP32 Arduino Core 2.x | Almacenamiento persistente del contador en memoria flash interna | https://github.com/espressif/arduino-esp32 |
| time.h | Biblioteca estándar C | Obtención y formateo de hora desde servidor NTP para timestamps | Incluida en ESP32 Arduino Core |

---

## 2. Código externo adaptado

### Lógica de medición sensor ultrasónico de un pin (medirDistancia en firmware/main.ino)
- **Fuente:** Documentación oficial Seeed Studio — Grove Ultrasonic Ranger  
  https://wiki.seeedstudio.com/Grove-Ultrasonic_Ranger/
- **Adaptación:** El sensor Seeed usa un único pin SIG compartido para trigger y echo, a diferencia del HC-SR04 estándar de 4 pines. Se adaptó la función para cambiar dinámicamente el pin entre OUTPUT (enviar pulso) e INPUT (escuchar eco). Se agregó retorno de valor 999 cuando `pulseIn` devuelve 0 (sin eco), evitando que una lectura fallida sea interpretada como objeto a 0 cm. Se añadió además un `delay(40)` entre mediciones de ambos sensores para evitar interferencia cruzada de ultrasonido.

### Configuración NTP para zona horaria Chile
- **Fuente:** Documentación ESP32 Arduino — configTime  
  https://randomnerdtutorials.com/esp32-ntp-client-date-time-arduino-ide/
- **Adaptación:** Se modificó el offset a `-4 * 3600` correspondiente a CLT (UTC−4), zona horaria de Chile continental en invierno. Se agregó un timeout de 5 segundos al loop de sincronización para evitar que el `setup()` quede bloqueado indefinidamente si el servidor NTP no responde, permitiendo que el sistema opere en modo offline.

---

## 3. Uso de Inteligencia Artificial

### 3.1 Arquitectura FreeRTOS dual-core para HTTP no bloqueante
- **Herramienta:** Claude Sonnet (Anthropic) — Junio 2026
- **Consulta realizada:** Se preguntó cómo evitar que las llamadas HTTP a Google Sheets bloquearan la detección de personas, dado que `HTTPClient.GET()` es una operación sincrónica que puede durar varios segundos.
- **Código generado:** Función `tareaHTTP()` que corre en Core 0 de la ESP32-S3, estructura `EventoHTTP` con campos `tipo`, `ts` y `total`, y función `enviarASheets()` que encola eventos sin bloquear.
- **Adaptación:** Se configuró la tarea en Core 0 específicamente porque el loop principal de Arduino corre en Core 1, separando físicamente la detección de sensores del envío a internet. Se ajustó el stack de la tarea a 8192 bytes después de verificar que 4096 era insuficiente para manejar la conexión HTTPS. La cola se dimensionó en 20 eventos para cubrir caídas de WiFi de duración media sin perder datos.
- **Comprensión:** La función `tareaHTTP()` corre en un loop infinito bloqueado en `xQueueReceive()`, que la despierta solo cuando llega un evento. `enviarASheets()` usa `xQueueSend()` con timeout 0, lo que significa que descarta el evento si la cola está llena en vez de bloquear el loop principal. La separación en cores garantiza que aunque Google tarde 8 segundos en responder, los sensores siguen midiendo sin interrupción.

### 3.2 Integración con Google Sheets vía HTTPS
- **Herramienta:** Claude Sonnet (Anthropic) — Junio 2026
- **Consulta realizada:** El sistema enviaba HTTP 400 a Google Apps Script. Se solicitó diagnóstico y corrección.
- **Código generado:** Uso de `WiFiClientSecure` con `client.setInsecure()`, `http.setFollowRedirects(HTTPC_STRICT_FOLLOW_REDIRECTS)`, y formato de timestamp `%Y-%m-%dT%H:%M:%S`.
- **Adaptación:** Se identificaron tres causas del error 400: (1) el timestamp tenía un espacio entre fecha y hora que rompía la URL, corregido usando `T` como separador ISO 8601; (2) faltaba `WiFiClientSecure` para conexiones HTTPS, ya que `HTTPClient` solo no maneja SSL en ESP32; (3) Google Apps Script siempre redirige con código 302 y sin `setFollowRedirects` el script nunca se ejecutaba. Se usó `setInsecure()` en vez de un certificado específico porque Google rota sus certificados con frecuencia.
- **Comprensión:** El equipo comprende que `setInsecure()` deshabilita la verificación del certificado SSL, lo que implica un riesgo teórico de man-in-the-middle aceptable en este contexto de red universitaria local. La cadena de redirección de Google Apps Script va de `script.google.com` a `script.googleusercontent.com`, por lo que sin seguimiento de redirects la petición termina en el primer servidor sin ejecutar el script.

### 3.3 Persistencia del contador con Preferences
- **Herramienta:** Claude Sonnet (Anthropic) — Junio 2026
- **Consulta realizada:** Se solicitó una forma de que el contador sobreviviera reinicios de la ESP sin perder el valor actual.
- **Código generado:** Funciones `guardarContador()` y `cargarContador()` usando la librería `Preferences`, y llamada a `guardarContador()` en cada evento de entrada o salida.
- **Adaptación:** Se escribió en el namespace `"contador"` para evitar colisiones con otras claves en flash. Se evaluó la frecuencia de escritura: una biblioteca con 200 eventos diarios representa ~73.000 escrituras anuales, dentro del límite de ~100.000 ciclos de escritura de la flash del ESP32-S3. Se decidió escribir por evento en vez de periódicamente para maximizar la exactitud ante cortes de energía imprevistos.
- **Comprensión:** `Preferences` utiliza la partición NVS (Non-Volatile Storage) de la flash interna, que persiste ante reinicios y cortes de energía. `prefs.getInt("personas", 0)` retorna 0 si la clave no existe (primera ejecución), evitando valores indeterminados.

### 3.4 Reset automático por tiempo de inactividad
- **Herramienta:** Claude Sonnet (Anthropic) — Junio 2026
- **Consulta realizada:** Se necesitaba que el contador volviera a 0 automáticamente después de que la biblioteca cierra y la ESP pasa varias horas apagada, sin requerir intervención manual.
- **Código generado:** Función `verificarResetPorTiempo()` que compara el timestamp actual con el último guardado en flash usando `difftime()`.
- **Adaptación:** Se ajustó el umbral a 2 horas (parámetro `2.0` fácilmente modificable) en vez de las 20 horas inicialmente sugeridas, para que cubra el período nocturno de carga de batería sin ser tan conservador. Se guardó el timestamp como `long` con la clave `"apagado"` en el mismo namespace de Preferences. La función se ejecuta una única vez en `setup()` después de sincronizar NTP, porque requiere hora válida para comparar.
- **Comprensión:** `difftime(ahora, ultimoApagado)` retorna los segundos transcurridos entre ambos timestamps. Dividido en 3600 da las horas. Si `ultimoApagado == 0` significa que es la primera ejecución y no hay referencia, por lo que se omite el reset. En cada arranque se sobreescribe el timestamp con la hora actual, actualizando la referencia para el próximo ciclo.

### 3.5 URL secreta de reset para operadores
- **Herramienta:** Claude Sonnet (Anthropic) — Junio 2026
- **Consulta realizada:** Se solicitó una forma de resetear el contador manualmente accesible solo para el personal de la biblioteca, sin exponer botones en la página pública.
- **Código generado:** Ruta `/admin-reset-biblioteca` registrada en el WebServer con una función lambda inline.
- **Adaptación:** Se eligió una URL descriptiva y específica en vez de una ruta genérica como `/reset`, de modo que sea difícil de adivinar por usuarios casuales. El handler redirige al dashboard principal con código 302 tras ejecutar el reset, para que el operador vea el resultado inmediatamente. Se llama a `guardarContador()` para persistir el 0 en flash.
- **Comprensión:** El WebServer de ESP32 permite registrar rutas con funciones lambda directamente en `setup()`. La redirección 302 hace que el navegador cargue automáticamente `/` después del reset, evitando que el operador vea una página en blanco.

### 3.6 Página web del dashboard de ocupación
- **Herramienta:** Claude Sonnet (Anthropic) — Junio 2026
- **Consulta realizada:** Se solicitó una página HTML con barra de progreso visual que mostrara el aforo en tiempo real, con colores según nivel de ocupación.
- **Código generado:** Función `handleRoot()` que construye dinámicamente el HTML con CSS embebido, barra de progreso y actualización automática cada 3 segundos.
- **Adaptación:** Se usó `html.reserve(2200)` para pre-reservar memoria del heap y reducir fragmentación en operaciones de concatenación de String. Se aplicó la macro `F()` en literales de texto estático para mantenerlos en flash en vez de copiarlos a RAM. Los colores verde/naranja/rojo se calculan en base al porcentaje de ocupación con umbrales en 50% y 80%, ajustados según criterio del equipo para la biblioteca.
- **Comprensión:** El `meta http-equiv='refresh' content='3'` hace que el navegador recargue la página cada 3 segundos sin JavaScript. `html.reserve()` preasigna el buffer interno del objeto String, evitando múltiples realocaciones de memoria durante las concatenaciones sucesivas.

### 3.7 Diagnóstico de inestabilidad y recuperación del hardware
- **Herramienta:** ChatGPT-4o (OpenAI) — Junio 2026
- **Consulta realizada:** La ESP32-S3 entró en boot loop y dejó de funcionar tras subir código con HTTPS. Se solicitó diagnóstico de los síntomas: reconexión constante del puerto COM, entrada repetida al boot ROM, imposibilidad de subir sketches.
- **Resultado:** Se identificaron cuatro causas: board incorrecto en Arduino IDE (variante WROOM en vez de ESP32S3 Dev Module), uso del puerto USB nativo inestable con WiFi/HTTPS, GPIO1 y GPIO10 conflictivos con funciones internas del chip, y flash en estado inestable por uploads fallidos.
- **Solución aplicada:** Cambio a ESP32S3 Dev Module en Arduino IDE, uso exclusivo del puerto COM/UART, cambio de sensores a GPIO4 y GPIO5, y recuperación de flash con erase completo + sketch vacío + secuencia BOOT+RESET.
- **Comprensión:** GPIO1 en ESP32-S3 es el pin TX del UART0 (usado por Serial), por lo que cambiarlo constantemente entre INPUT y OUTPUT interfería con la comunicación serial y el proceso de boot. GPIO10 puede estar mapeado a funciones del controlador SPI interno según la configuración del chip. El erase de flash elimina cualquier firmware corrupto y permite una instalación limpia desde cero.

### 3.8 Código base versión 3 (estabilización post-recuperación)
- **Herramienta:** ChatGPT-4o (OpenAI) — Junio 2026
- **Consulta realizada:** Tras recuperar la ESP, se solicitó un código limpio y estable que aplicara las correcciones de hardware identificadas.
- **Código generado:** Versión con GPIO4/GPIO5, retorno de 999 en vez de 0 cuando no hay eco, timeout de 2000ms para secuencias trabadas, `WiFi.setSleep(false)` y timeout de 15 segundos en conexión WiFi.
- **Adaptación:** Este código fue la base sobre la que se construyeron todas las mejoras posteriores: se integró la arquitectura FreeRTOS, la persistencia con Preferences, el reset por tiempo, la integración HTTPS con Google Sheets y la URL secreta. `WiFi.setSleep(false)` fue cambiado a `WiFi.setSleep(true)` en versiones posteriores para optimizar el consumo de batería, tras verificar que el modem sleep es compatible con el servidor web local.
- **Comprensión:** El timeout de 15 segundos en el loop de WiFi evita que `setup()` quede bloqueado indefinidamente si la red no está disponible, permitiendo que el sistema opere en modo offline contando personas aunque sin enviar a Sheets. El valor 999 para distancia sin eco es mayor que cualquier UMBRAL razonable, garantizando que una lectura fallida nunca active falsamente un sensor.

---

## Resumen de contribuciones

| Componente | Origen | Adaptación del equipo |
|---|---|---|
| Detección entrada/salida con 2 sensores | Desarrollo propio + Claude | Lógica de secuencia A→B y B→A, timeout de reset |
| Arquitectura FreeRTOS dual-core | Claude | Dimensionamiento de stack, separación Core 0/1 |
| Integración Google Sheets HTTPS | Claude | Diagnóstico error 400, certificado, redirect |
| Persistencia Preferences | Claude | Evaluación ciclos de escritura, namespace |
| Reset por tiempo apagada | Claude | Umbral 2h, manejo primera ejecución |
| Dashboard HTML | Claude | Umbrales de color, reserve(), macro F() |
| URL secreta de reset | Claude | Nombre de ruta, redirect post-reset |
| Diagnóstico hardware ESP32-S3 | ChatGPT | Aplicación de correcciones al proyecto real |
| Código base estabilizado | ChatGPT | Integración con todas las mejoras posteriores |
| Sensor ultrasónico modo un pin | Seeed Wiki | Adaptación a lógica de detección direccional |
