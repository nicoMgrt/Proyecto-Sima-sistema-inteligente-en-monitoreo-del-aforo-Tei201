# Librerías — Software

Esta carpeta documenta todas las librerías utilizadas en el firmware del sistema contador de personas. Todas forman parte del **ESP32 Arduino Core** y se instalan automáticamente al agregar el soporte de ESP32 en Arduino IDE — no requieren instalación adicional por separado.

---

## Librerías utilizadas

### `WiFi.h`
- **Origen:** ESP32 Arduino Core by Espressif — https://github.com/espressif/arduino-esp32
- **Uso en el proyecto:** Conecta el ESP32-S3 a la red WiFi 2.4 GHz local. Gestiona el estado de conexión y permite verificar si hay red disponible antes de intentar enviar datos a Google Sheets.
- **Funciones usadas:** `WiFi.begin()`, `WiFi.status()`, `WiFi.localIP()`, `WiFi.setSleep()`

---

### `WebServer.h`
- **Origen:** ESP32 Arduino Core by Espressif — https://github.com/espressif/arduino-esp32
- **Uso en el proyecto:** Levanta un servidor HTTP en el puerto 80 dentro del ESP32-S3. Sirve el dashboard de ocupación en tiempo real a cualquier navegador que acceda a la IP del dispositivo en la misma red.
- **Funciones usadas:** `server.on()`, `server.begin()`, `server.handleClient()`, `server.send()`

---

### `WiFiClientSecure.h`
- **Origen:** ESP32 Arduino Core by Espressif — https://github.com/espressif/arduino-esp32
- **Uso en el proyecto:** Permite conexiones HTTPS (SSL/TLS) necesarias para comunicarse con Google Apps Script, que solo acepta conexiones seguras. Se usa con `setInsecure()` para evitar verificación de certificado, ya que Google rota sus certificados frecuentemente.
- **Funciones usadas:** `client.setInsecure()`

---

### `HTTPClient.h`
- **Origen:** ESP32 Arduino Core by Espressif — https://github.com/espressif/arduino-esp32
- **Uso en el proyecto:** Construye y envía solicitudes HTTP GET hacia la URL del Google Apps Script, incluyendo los parámetros de timestamp, tipo de evento (ENTRADA/SALIDA) y total de personas.
- **Funciones usadas:** `http.begin()`, `http.GET()`, `http.end()`, `http.setFollowRedirects()`, `http.setTimeout()`

---

### `Preferences.h`
- **Origen:** ESP32 Arduino Core by Espressif — https://github.com/espressif/arduino-esp32
- **Uso en el proyecto:** Guarda el contador de personas en la memoria flash no volátil (partición NVS) del ESP32-S3. El valor persiste ante cortes de energía y reinicios, evitando que el aforo se pierda si el dispositivo se apaga inesperadamente.
- **Funciones usadas:** `prefs.begin()`, `prefs.getInt()`, `prefs.putInt()`, `prefs.getLong()`, `prefs.putLong()`

---

### `time.h`
- **Origen:** Biblioteca estándar de C — incluida en el ESP32 Arduino Core
- **Uso en el proyecto:** Sincroniza el reloj interno del ESP32 con servidores NTP (Network Time Protocol) de Google para obtener la hora exacta de Chile (UTC−4). Genera timestamps en formato ISO 8601 que se adjuntan a cada evento enviado a Google Sheets.
- **Funciones usadas:** `configTime()`, `getLocalTime()`, `time()`, `difftime()`, `strftime()`

---

## Cómo instalar las librerías

Al ser todas parte del ESP32 Arduino Core, solo necesitas instalar el soporte de ESP32 una vez:

1. Abre Arduino IDE → **File → Preferences**
2. En *Additional boards manager URLs* agrega:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. Ve a **Tools → Board → Boards Manager**
4. Busca `esp32` e instala **ESP32 by Espressif Systems**
5. Listo — todas las librerías quedan disponibles automáticamente

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
