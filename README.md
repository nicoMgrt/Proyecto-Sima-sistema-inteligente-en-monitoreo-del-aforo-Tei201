# Contador de Personas Direccional — Biblioteca UAI

Sistema IoT que monitorea el aforo en tiempo real de un espacio cerrado, detectando entradas y salidas mediante dos sensores ultrasónicos y determinando la dirección del flujo según el orden de activación.

---

## Equipo

| Integrante | Rol principal | GitHub |
|---|---|---|
| Nicolás Ignacio Marinkovic Grant | Software / Firmware | [@nicoMgrt](https://github.com/nicoMgrt) |
| Bárbara Carolina Chaparro Torres | Hardware | [@barbarachaparro](https://github.com/barbarachaparro) |
| Valentina Paz Ramírez Gómez | Diseño 3D | [@valenramirez-Hub](https://github.com/valenramirez-Hub) |
| Cristóbal Pérez | Testing / Documentación | [@CPerex](https://github.com/CPerex) |

---

## Descripción del Problema

Las bibliotecas y espacios cerrados de uso público no tienen una forma automática de conocer su nivel de ocupación en tiempo real. El control manual de aforo es inexacto, discontinuo y requiere personal dedicado. Esto impide que los usuarios sepan si el espacio está disponible antes de desplazarse, y que los administradores tomen decisiones informadas sobre ventilación, limpieza o apertura.

*Problema identificado en Avance #1 — TEI201, 2026.*

---

## Solución: Arquitectura del Sistema IoT

```
Sensores HC-SR04 (x2)
        ↓
   ESP32-S3
   (detección direccional + servidor web local)
        ↓
Google Apps Script
        ↓
Google Sheets (repositorio de datos históricos)
        ↓
Google Looker Studio (dashboard con gráficas en tiempo real)
```

El dispositivo detecta si una persona **entra o sale** según qué sensor ultrasónico se activa primero (sensor A = pasillo exterior, sensor B = interior del recinto). El conteo neto se muestra en un dashboard web local accesible desde cualquier dispositivo en la misma red, y se envía a Google Sheets para persistencia y visualización histórica en Looker Studio.

---

## Componentes de Hardware

| Componente | Cantidad | Especificación |
|---|---|---|
| ESP32-S3 | 1 | MCU principal — WiFi + procesamiento dual core |
| HC-SR04 Seeed (3 pines) | 2 | Sensores ultrasónicos de distancia — pin único SIG |
| Batería NCR18650B | 1 | Li-ion 3.7V — autonomía del sistema |
| Shield de batería | 1 | Con cargador micro USB integrado |
| Interruptor | 1 | Encendido/apagado del sistema |

---

## Instrucciones de Uso

### 1. Cargar el firmware

1. Abre `software/main.ino` en **Arduino IDE 2.x**
2. Instala el soporte para ESP32 en Arduino IDE (Board Manager → ESP32 by Espressif)
3. Las librerías usadas (`WiFi.h`, `WebServer.h`, `Preferences.h`, etc.) vienen incluidas con el core de ESP32 — no requieren instalación adicional
4. Edita las credenciales WiFi en el archivo:
   ```cpp
   const char* ssid     = "TU_RED_WIFI";
   const char* password = "TU_CONTRASEÑA";
   ```
5. Ajusta el aforo máximo si es necesario:
   ```cpp
   #define AFORO_MAX  295
   ```
6. Conecta el ESP32-S3 por USB, selecciona la placa **ESP32S3 Dev Module** y el puerto correcto
7. Sube el código con el botón **Upload**

### 2. Conectar los sensores

- **Sensor A** (exterior/pasillo) → Pin **GPIO 4** del ESP32
- **Sensor B** (interior) → Pin **GPIO 5** del ESP32
- Alimentación de sensores: **5V y GND** del ESP32

### 3. Ver el dashboard local

1. Una vez cargado el firmware, abre el **Monitor Serial** (115200 baudios)
2. Espera a que aparezca: `¡Conectado! IP: 192.168.X.X`
3. Ingresa esa IP desde cualquier navegador en la misma red WiFi
4. El dashboard se actualiza automáticamente cada 3 segundos

### 4. Ver los datos históricos

- 📊 **Google Sheets:** [Ver datos en tiempo real](https://docs.google.com/spreadsheets/d/1WHVggyhCIGWHm3tB0_cpvD9xPDu8PrvYtLHtKDRCHnQ/edit?usp=drivesdk)
- 📈 **Google Looker Studio:** *Link disponible próximamente*

---

## Estructura del Repositorio

```
contador-personas-TEI201/
├── README.md                  ← Este archivo
├── FUENTES.md                 ← Declaración de librerías, código externo e IA
├── software/
│   └── main.ino               ← Firmware completo del ESP32-S3
├── hardware/
│   ├── esquematico.pdf        ← Diagrama de conexiones
│   ├── BOM.xlsx               ← Lista de componentes con costos
│   └── fotos/                 ← Fotografías del prototipo ensamblado
├── diseno_3d/
│   ├── encapsulado.f3d        ← Archivo Fusion 360 del encapsulado
│   ├── planos.pdf             ← Planos técnicos con cotas
│   └── renders/               ← Renders del encapsulado (exterior e interior)
├── testing/
│   ├── protocolo_pruebas.pdf
│   └── datos/                 ← Logs y gráficas de validación
├── documentacion/
│   └── reporte_final.pdf
└── iteraciones/               ← Versiones anteriores y registro del proceso
```

---

## Estado del Proyecto

- [x] Firmware con detección direccional entrada/salida
- [x] Dashboard web local en tiempo real
- [x] Envío de datos a Google Sheets
- [x] Encapsulado modelado en Fusion 360
- [ ] Google Looker Studio — en desarrollo
- [ ] Planos técnicos exportados
- [ ] Protocolo de pruebas documentado

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
