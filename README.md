# SIMA — Sistema Inteligente de Monitoreo de Aforo

Sistema IoT que monitorea el aforo de la Biblioteca de Pregrado de la UAI en tiempo real, detectando entradas y salidas mediante dos sensores ultrasónicos y determinando la dirección del flujo según el orden de activación. Precisión global validada: **91,4% en 35 pruebas controladas.**

---

## Equipo

| Integrante | Rol principal | GitHub |
|---|---|---|
| Nicolás Marinkovic Grant | Iteraciones · Documentación · Testing | [@nicoMgrt](https://github.com/nicoMgrt) |
| Bárbara Chaparro Torres | Software · Firmware · Documentación | [@barbarachaparro](https://github.com/barbarachaparro) |
| Valentina Ramírez Gómez | Diseño 3D · Iteraciones | [@valenramirez-Hub](https://github.com/valenramirez-Hub) |
| Cristóbal Pérez | Hardware · Software · Diseño 3D | [@CPerex](https://github.com/CPerex) |

---

## Descripción del Problema

Los estudiantes de la Universidad Adolfo Ibáñez no tienen forma de saber si hay espacio disponible en la Biblioteca de Pregrado antes de desplazarse. Una encuesta a 69 estudiantes (Avance #1, marzo 2026) reveló que el **74,2% ha tenido que abandonar la biblioteca por falta de espacio** y el **39,7% pierde entre 5 y más de 10 minutos buscando asiento en hora peak** (bloques M3–M5).

SIMA resuelve esta brecha entregando el porcentaje de ocupación en tiempo real, accesible desde cualquier dispositivo en la misma red, antes de que el estudiante se desplace.

*Proyecto enmarcado en el ODS 11 — Ciudades y Comunidades Sostenibles · Avance #1 TEI201, 2026.*

---

## Solución — Arquitectura del Sistema IoT

```
Sensor A (pasillo exterior, GPIO4)
Sensor B (interior biblioteca, GPIO5)
         ↓
     ESP32-S3
(detección direccional + FreeRTOS dual-core)
         ↓
  Google Apps Script (HTTPS)
         ↓
  Google Sheets (histórico persistente)
         ↓
  Google Looker Studio (dashboard con gráficas)
         +
  Dashboard web local (IP del dispositivo, cada 3 seg)
```

El sistema detecta si una persona **entra o sale** según qué sensor se activa primero. La arquitectura FreeRTOS separa la detección (Core 1) del envío HTTP (Core 0), garantizando que el sistema siga midiendo aunque el envío a Sheets tome hasta 3 segundos.

**El prototipo se compone de dos módulos físicos** conectados por cable: el módulo principal (ESP32-S3 + batería + sensor B) y el módulo secundario (sensor A), instalados en lados opuestos del marco de la puerta para maximizar la separación entre sensores y mejorar la detección direccional.

---

## Componentes de Hardware

| Componente | Cantidad | Especificación | Costo |
|---|---|---|---|
| ESP32-S3 Dev Module | 1 | Dual-core Xtensa LX7 240MHz, WiFi 2.4GHz, 512KB SRAM | $6.990 |
| HC-SR04 Seeed Grove (3 pines) | 2 | 3.3V nativo, rango 3–350cm, GPIO4 y GPIO5 | $9.980 |
| Batería NCR18650B Panasonic | 1 | Li-ion 3.7V, 3.400mAh, autonomía ~16–20h | $2.000 |
| Shield cargador 18650 | 1 | Micro-USB entrada, USB-A salida 5V/1A, boost converter | $3.990 |
| Cable USB-A a USB-C | 1 | 30cm, conexión shield → ESP32-S3 (COM/UART) | $1.990 |
| Jumper wires M-M | 1 set | 20cm, 40 unidades | $1.490 |
| Protoboard 400 puntos | 1 | Conexión sin soldadura | $1.990 |
| Filamento PLA | ~50g | Encapsulado impreso en 3D | $5.000 |
| **TOTAL** | | | **$27.960 CLP** |

---

## Instrucciones de Uso

### 1. Cargar el firmware

1. Abre `software/src/main.ino` en **Arduino IDE 2.x**
2. Instala el soporte ESP32: Board Manager → buscar **ESP32 by Espressif** → instalar
3. Las librerías (`WiFi.h`, `WebServer.h`, `Preferences.h`, `HTTPClient.h`, `time.h`) vienen incluidas con el core de ESP32 — no requieren instalación adicional
4. Edita las credenciales WiFi:
   ```cpp
   const char* ssid     = "TU_RED_WIFI";
   const char* password = "TU_CONTRASEÑA";
   ```
5. Ajusta el aforo máximo si es necesario:
   ```cpp
   #define AFORO_MAX  295
   ```
6. Selecciona la placa **ESP32S3 Dev Module** (no ESP32 genérico) y el puerto **COM/UART**
7. Sube el código con **Upload**

### 2. Conectar los sensores

| Sensor | GPIO | Ubicación |
|---|---|---|
| Sensor A | GPIO 4 | Lado pasillo exterior |
| Sensor B | GPIO 5 | Lado interior de la biblioteca |

Alimentación de ambos sensores: **3.3V y GND** del ESP32-S3.

### 3. Ver el dashboard local

1. Abre el **Monitor Serial** (115200 baudios) tras cargar el firmware
2. Espera: `¡Conectado! IP: 192.168.X.X`
3. Ingresa esa IP desde cualquier navegador en la misma red
4. El dashboard se actualiza automáticamente cada 3 segundos — muestra % de ocupación, estado (Disponible / Casi lleno / Lleno) y contadores de entradas/salidas del día

### 4. Reset manual del contador (operadores)

Accede desde un navegador en la misma red:
```
http://[IP_DEL_DISPOSITIVO]/admin-reset-biblioteca
```

### 5. Ver los datos históricos

- 📊 **Google Sheets:** [Ver registro de eventos](https://docs.google.com/spreadsheets/d/TU_ID_DE_SCRIPT)
- 📈 **Google Looker Studio:** *Próximamente*

---

## Resultados de Validación

| Prueba | Casos | Tasa de éxito |
|---|---|---|
| Detección de entrada (A→B) | 10 | 90% |
| Detección de salida (B→A) | 10 | 80% |
| Anti-bloqueo timeout (2.000ms) | 5 | 100% |
| Persistencia ante corte de energía | 5 | 100% |
| Envío a Google Sheets (HTTP 200 OK) | 5 | 100% |
| **Precisión global** | **35** | **91,4%** |

---

## Estructura del Repositorio

```
SIMA-contador-personas-TEI201/
├── README.md                        ← Este archivo
├── FUENTES.md                       ← Declaración de librerías, código externo e IA
│
├── software/
│   ├── src/                         ← Firmware principal (main.ino)
│   ├── librerias/                   ← Documentación de librerías utilizadas
│   └── docs/                        ← Diagramas de flujo y documentación técnica
│
├── hardware/
│   ├── esquematico.pdf              ← Diagrama de conexiones del circuito
│   ├── BOM.xlsx                     ← Lista de componentes con costos
│   ├── datasheets.pdf               ← Especificaciones técnicas de componentes
│   └── fotos/                       ← Fotografías del prototipo (3+ ángulos)
│
├── diseno_3d/
│   ├── fusion360/                   ← Archivo .f3d del ensamble completo
│   ├── planos/                      ← Planos técnicos con cotas en PDF
│   └── renders/                     ← Renders exterior, interior y explosionado
│
├── testing/
│   ├── datos/                       ← Exportación Google Sheets + resultados de pruebas
│   ├── evidencias/                  ← Fotos y capturas del prototipo en operación
│   └── reportes/                    ← Protocolo de pruebas completo (PDF)
│
├── documentacion/
│   ├── presentacion/                ← Slides de la presentación final (PPTX/PDF)
│   └── reporte_final/               ← Reporte final del proyecto (PDF, 12 páginas)
│
└── iteraciones/
    ├── iteracion_1/                 ← v1 Alpha — Abril 2026
    ├── iteracion_2/                 ← v2 Beta — Mayo 2026
    └── iteracion_3/                 ← v4 Final — Junio 2026
```

---

## Estado del Proyecto

- [x] Firmware con detección direccional entrada/salida (FreeRTOS dual-core)
- [x] Dashboard web local en tiempo real (semáforo + estadísticas diarias)
- [x] Envío de datos a Google Sheets (HTTP 200 OK confirmado)
- [x] Persistencia ante cortes de energía (NVS flash — 5/5 reinicios ✓)
- [x] Encapsulado modelado en Fusion 360 con gemelo digital completo
- [x] Protocolo de pruebas documentado (91,4% precisión global)
- [x] Reporte final del proyecto (12 páginas)
- [x] Fotos del prototipo (3 ángulos)
- [ ] Google Looker Studio — en desarrollo
- [ ] Planos técnicos PDF exportados desde Fusion 360
- [ ] Renders finales del encapsulado

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Marinkovic · Chaparro · Ramírez · Pérez*
