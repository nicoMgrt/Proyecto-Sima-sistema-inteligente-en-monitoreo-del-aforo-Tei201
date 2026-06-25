# Evidencias de Testing — SIMA

## Contenido Requerido

### Fotografías

#### Prototipo Ensamblado
- `prototipo_frontal_01.jpg` — Vista frontal del encapsulado terminado con orificios de sensores visibles
- `prototipo_lateral_01.jpg` — Vista lateral mostrando orificio micro USB y switch de encendido
- `prototipo_interior_01.jpg` — Vista con tapa removida mostrando componentes internos (ESP32-S3, batería, sensores)

**Requisitos:**
- Buena iluminación, sin sombras sobre los componentes
- Fondo neutro (mesa blanca o gris)
- Al menos 1200×900 px de resolución

#### Instalación en Terreno
- `instalacion_puerta_01.jpg` — Dispositivo instalado en el marco de la puerta de la Biblioteca F, UAI
- `instalacion_contexto_01.jpg` — Vista general que muestra la ubicación del dispositivo en el acceso de la biblioteca con referencia de escala

**Requisitos:**
- El dispositivo debe ser claramente visible en la imagen
- Incluir referencia de escala (una persona de pie cerca, o una regla)
- Mostrar condición real de operación (iluminación natural del recinto)

#### Dashboard y Monitoreo
- `dashboard_web_captura.png` — Captura del dashboard web en el navegador mostrando porcentaje de ocupación y estado de color
- `monitor_serial_captura.png` — Captura del Monitor Serial de Arduino IDE mostrando detecciones en tiempo real (líneas `¡Alguien entró!` o `¡Alguien salió!`)
- `sheets_captura.png` — Captura de Google Sheets con filas de eventos registrados, timestamp y total de personas

---

### Videos

#### Video de Funcionamiento del Sistema
`demo_sima_funcionando.mp4`

**Contenido:**
- Duración: 2–4 minutos
- Mostrar el dispositivo encendido y conectado
- Realizar 2–3 cruces de entrada y salida frente a la cámara
- Mostrar en paralelo cómo cambia el contador en el dashboard web
- Mostrar cómo aparece el nuevo evento en Google Sheets con timestamp
- Calidad mínima: 720p

#### Video de Contexto de Instalación
`instalacion_contexto.mp4`

**Contenido:**
- Duración: 1–2 minutos
- Dispositivo instalado en el marco real de la puerta de la Biblioteca F
- Mostrar el ángulo y la posición de ambos sensores
- Mostrar una o dos personas cruzando naturalmente mientras el sistema opera

---

## Privacidad y Consentimiento

### Antes de Fotografiar/Grabar
- Obtener consentimiento verbal de personas que aparezcan en las imágenes
- Explicar que las imágenes son para un proyecto académico de la UAI (TEI201)
- Ofrecer la opción de no aparecer o de difuminar el rostro

### Anonimización
Si no hay consentimiento explícito para mostrar rostros:
- Tomar fotos desde ángulos que no muestren la cara (de espalda, de lado, de lejos)
- El prototipo es el sujeto principal de las imágenes — las personas son contexto

---

## Organización de Archivos

```
evidencias/
├── fotos/
│   ├── prototipo/
│   │   ├── prototipo_frontal_01.jpg
│   │   ├── prototipo_lateral_01.jpg
│   │   └── prototipo_interior_01.jpg
│   ├── instalacion/
│   │   ├── instalacion_puerta_01.jpg
│   │   └── instalacion_contexto_01.jpg
│   └── capturas/
│       ├── dashboard_web_captura.png
│       ├── monitor_serial_captura.png
│       └── sheets_captura.png
└── videos/
    ├── demo_sima_funcionando.mp4
    └── instalacion_contexto.mp4
```

---

## Nomenclatura de Archivos

**Fotos:**
```
[tipo]_[descripcion]_[numero].jpg

Ejemplos reales del proyecto:
prototipo_frontal_01.jpg
instalacion_puerta_01.jpg
dashboard_web_captura.png
```

**Videos:**
```
[tipo]_[descripcion].mp4

Ejemplos reales del proyecto:
demo_sima_funcionando.mp4
instalacion_contexto.mp4
```

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
