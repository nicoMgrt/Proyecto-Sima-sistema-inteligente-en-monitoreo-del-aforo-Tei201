# Hardware - Documentación

## Contenido
Esta carpeta contiene toda la documentación técnica del hardware del proyecto.

### 📁 Carpetas

#### `esquemas/`
Diagramas de circuitos y esquemáticos del sistema

#### `bom/`
Bill of Materials (Lista de materiales)

#### `fotos/`
Fotografías de alta resolución del prototipo ensamblado

---
Acá está completo con los datos reales de tu proyecto:
markdown# Hardware - Documentación

## Contenido
Esta carpeta contiene toda la documentación técnica del hardware del proyecto.

## 📁 Carpetas

### esquemas/
Diagramas de circuitos y esquemáticos del sistema
- `wokwi_diagram.json` — Archivo fuente del circuito en Wokwi
- `wokwi_screenshot.png` — Captura del circuito completo
- `wokwi_link.txt` — Link al proyecto online en Wokwi

### bom/
Bill of Materials (Lista de materiales)
- `BOM_contador_biblioteca.xlsx` — BOM editable con fórmulas
- `BOM_contador_biblioteca.pdf` — BOM en PDF para presentación
- `datasheets/` — Datasheets de componentes críticos

### fotos/
Fotografías de alta resolución del prototipo ensamblado
- `01_vista_general.jpg`
- `02_circuito_interno.jpg`
- `03_detalle_conexiones.jpg`
- `04_sensores.jpg`
- `05_shield_bateria.jpg`
- `06_encapsulado_cerrado.jpg`
- `07_contexto_uso.jpg`

---

## Componentes Principales

| Componente | Función |
|---|---|
| ESP32-S3 Dev Module | Microcontrolador principal — procesa sensores, sirve la página web y envía datos a Google Sheets |
| Sensor Ultrasónico Seeed Grove x2 | Detección de personas — sensor A (pasillo) y sensor B (biblioteca) |
| Batería Li-ion NCR18650B | Fuente de energía portátil — 3400mAh reales, autonomía ~16-20 horas |
| Shield Cargador 18650 con USB-A | Gestión de batería — boost converter 3.7V→5V, protección y carga integrada |
| Protoboard 400 puntos | Conexión de sensores a la ESP32-S3 sin soldadura |

---

## Pasos de Ensamblaje

1. Conectar el sensor A al GPIO4 de la ESP32-S3: VCC→3.3V, GND→GND, SIG→GPIO4
2. Conectar el sensor B al GPIO5 de la ESP32-S3: VCC→3.3V, GND→GND, SIG→GPIO5
3. Insertar la batería NCR18650B en el shield cargador
4. Conectar el shield a la ESP32-S3 mediante cable USB-A a USB-C en el puerto COM/UART
5. Configurar el firmware con las credenciales WiFi y URL del Google Apps Script
6. Instalar los sensores en la puerta: sensor A en el lado del pasillo, sensor B en el lado de la biblioteca, separados por el ancho de la puerta

---

## Precauciones

⚠️ Usar siempre el puerto COM/UART para programar y alimentar la ESP32-S3, NUNCA el puerto USB nativo — causa inestabilidad con WiFi+HTTPS activos simultáneamente

⚠️ Los sensores Seeed Grove operan a 3.3V — conectar siempre al pin 3.3V de la ESP32-S3, NUNCA al pin 5V para evitar dañar la placa

⚠️ No conectar ni desconectar los sensores con el sistema encendido (hot-plug) — puede dañar el sensor permanentemente

⚠️ La batería NCR18650B no debe descargarse bajo 2.5V — el shield incluye protección, pero evitar dejar el sistema encendido sin supervisión cuando la batería está baja

---

## Especificaciones Técnicas

| Parámetro | Valor |
|---|---|
| Alimentación | 3.7V batería Li-ion → 5V regulado por shield |
| Consumo promedio | ~80-100 mA (WiFi modem sleep activo) |
| Consumo peak | ~240 mA (durante transmisión WiFi) |
| Autonomía estimada | 16-20 horas con NCR18650B (3400 mAh) |
| Dimensiones encapsulado | 125 x 75 x 50 mm |
| Rango sensores | 3 cm – 350 cm (umbral configurado: 40 cm) |
| Frecuencia de muestreo | Cada 100 ms |

---

## Troubleshooting

| Problema | Posible Causa | Solución |
|---|---|---|
| ESP32 no aparece en puertos | Usando el puerto USB nativo | Cambiar al puerto COM/UART (el otro conector) |
| Sensores no detectan | GPIO incorrecto o cables sueltos | Verificar GPIO4 y GPIO5, revisar conexiones en protoboard |
| Contador no sube al pasar la mano | Objeto estático frente a un sensor | Alejar obstáculos, verificar que "Sensor bloqueado" no aparezca en Serial Monitor |
| No envía datos a Sheets | HTTP 400 o sin WiFi | Verificar credenciales WiFi y URL del script en el código |
| ESP32 en boot loop | Board incorrecto en Arduino IDE | Seleccionar ESP32S3 Dev Module, hacer erase flash |
## Instrucciones de Ensamble

