# Presentación Final — SIMA · Avance #3

Guía completa para la presentación del sistema **SIMA (Sistema Inteligente de Monitoreo de Aforo)** en el Avance #3 de TEI201.

---

## Archivo de presentación

| Archivo | Formato | Ubicación |
|---|---|---|
| `presentacion_avance3.pptx` | PowerPoint 16:9 | Esta carpeta |
| `presentacion_avance3.pdf` | PDF de respaldo | Esta carpeta |
| `video_backup_demo.mp4` | Video de respaldo por si falla el hardware | Esta carpeta |

**Especificaciones técnicas:** 10 minutos · 14 slides · 16:9 widescreen · Fuente mínima 24pt cuerpo / 32pt títulos · Fuente recomendada: Inter o Calibri

---

## Estructura slide a slide — contenido real del proyecto

### Slide 1 — Portada (30 seg)
- **Título:** SIMA — Sistema Inteligente de Monitoreo de Aforo
- **Subtítulo:** Contador de Personas Direccional para Biblioteca UAI
- **Integrantes:** Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez
- **ODS:** ODS 11 — Ciudades y Comunidades Sostenibles
- **Curso:** TEI201 Taller de Diseño en Ingeniería · UAI · Junio 2026
- **Visual sugerido:** Logo UAI + ícono de ODS 11 + foto del prototipo terminado

---

### Slide 2 — Problemática ODS (45 seg)
**ODS 11: Ciudades y Comunidades Sostenibles**

Datos de contexto para el slide:
- El ODS 11 busca optimizar el uso de infraestructura urbana existente antes de expandirla físicamente
- La UAI Campus Peñalolén concentra más de **8.000 estudiantes diarios** compitiendo por espacios de estudio
- La Biblioteca de Pregrado tiene capacidad para **295 personas** — sin sistema de monitoreo en tiempo real
- Gestionar el flujo de personas con tecnología IoT reduce la presión sobre infraestructura sin construir nuevo espacio

**Visual sugerido:** Ícono ODS 11 grande + dato clave "8.000 estudiantes / 295 asientos" en tipografía grande

---

### Slide 3 — Problema Específico (45 seg)
**El problema local que resuelve SIMA:**

Datos reales de la encuesta del Avance #1 (69 respuestas):
- **74.2%** de estudiantes se ha visto forzado a abandonar la biblioteca por falta de espacio
- **39.7%** pierde entre 5 y más de 10 minutos buscando un asiento disponible
- El **50.7%** asiste en bloque M3 (11:30) — el horario de mayor saturación
- **59.4%** reporta contaminación acústica en horas peak

**Causa raíz:** La biblioteca carece de un sistema que informe la ocupación real antes de que el estudiante se desplace.

**¿A quiénes afecta?** A los ~8.000 estudiantes de pregrado del campus Peñalolén, especialmente en franjas M3–M5.

**Visual sugerido:** Gráfico de barras del formulario (bloque horario más difícil) + dato 74.2% en grande

---

### Slide 4 — Visión General del Sistema (45 seg)
**¿Qué hace SIMA?**

Detecta entradas y salidas de la biblioteca de forma automática, calcula el aforo en tiempo real y lo comunica a los estudiantes vía web antes de que se desplacen.

**Diagrama de bloques para el slide:**
```
Sensor A (pasillo) ──┐
                     ├──► ESP32-S3 ──► Google Sheets ──► Looker Studio
Sensor B (interior) ─┘        │
                               └──► Dashboard web local (IP)
```

**Ciclo de datos:**
Sensores capturan → ESP32 procesa dirección → Google Sheets almacena → Looker Studio visualiza → Estudiante decide

**Visual sugerido:** Foto del prototipo funcionando + diagrama de bloques simple

---

### Slide 5 — Hardware (45 seg)
**Componentes principales:**

| Componente | Función | Costo |
|---|---|---|
| ESP32-S3 (Espressif) | Cerebro: procesamiento dual-core + WiFi | $6.990 |
| 2× HC-SR04 Seeed (3 pines) | Detección direccional entrada/salida | $9.980 |
| Batería NCR18650B Panasonic | Autonomía 16–20 horas continuas | $2.000 |
| Shield cargador 18650 | Gestión de batería + boost 5V | $3.990 |
| Encapsulado PLA (impresión 3D) | Protección y montaje | $5.000 |
| **TOTAL** | | **$27.960 CLP** |

**Dato clave para defender:** Se eligió el sensor Seeed de 3 pines sobre el HC-SR04 de 4 pines porque opera a 3.3V de forma nativa, eliminando el riesgo de quemar el ESP32-S3 y simplificando el cableado.

**Visual sugerido:** Foto del hardware ensamblado desde 2–3 ángulos + tabla de componentes simplificada

---

### Slide 6 — Software y Diseño 3D (45 seg)
**Arquitectura del software (3 capas):**

1. **Firmware ESP32-S3** — Detección, lógica direccional, servidor web local, envío FreeRTOS no bloqueante
2. **Google Sheets + Apps Script** — Repositorio persistente con timestamp y tipo de evento
3. **Google Looker Studio** — Dashboard histórico con tendencias y métricas

**Características clave del firmware:**
- Detección a 10 Hz (100ms) — suficiente para personas caminando, sin saturar el procesador
- Arquitectura dual-core: Core 1 detecta personas, Core 0 maneja el WiFi/HTTP
- Persistencia en flash NVS — el contador sobrevive cortes de energía
- Reset automático por inactividad >2 horas (cierre nocturno)

**Diseño 3D:**
- Encapsulado modelado en Fusion 360 con todos los componentes internos
- Apertura sin destruir el ensamble (diseño para la reparación)
- Orificios frontales calibrados para los transductores de 15.8mm

**Visual sugerido:** Render del encapsulado (exterior + interior) + diagrama de flujo principal simplificado

---

### Slide 7 — Evolución del Diseño — Iteraciones (1 min)
**Tres iteraciones documentadas:**

| Versión | Problema | Solución |
|---|---|---|
| **v1** | Sensores HC-SR04 de 4 pines incompatibles con 3.3V del ESP32-S3 | Migración a Seeed de 3 pines — compatibilidad nativa sin resistencias |
| **v2** | Portal cautivo de la red UAI bloqueaba dispositivos IoT | Modo Access Point autónomo → luego integración con router propio y Google Sheets |
| **v3** | Falsos positivos: persona parada en la puerta se contaba múltiples veces | Anti-rebote lógico: requiere que ambos sensores vuelvan a estado inactivo para confirmar |

**Dato adicional para preguntas:** También hubo una crisis de hardware (boot loop) causada por uso de GPIO1 y GPIO10 (conflictivos con funciones internas del chip). Solución: cambio a GPIO4 y GPIO5, y erase completo de flash.

**Visual sugerido:** 3 columnas lado a lado con ícono de versión, problema y solución en texto mínimo

---

### Slide 8 — Mejoras Cuantitativas (45 seg)
**Tabla comparativa de métricas entre versiones:**

| Métrica | v1 (Alpha) | v3 (Beta) | v4 (Final) |
|---|---|---|---|
| Tasa de falsos positivos | Alta (sin timeout) | Baja | Mínima (<5%) |
| Persistencia ante corte de luz | No | No | Sí (NVS flash) |
| Almacenamiento de datos | No | No | Sí (Google Sheets) |
| Visualización histórica | No | No | Sí (Looker Studio) |
| Autonomía batería | ~4h | ~8h | ~16–20h (modem sleep) |
| Latencia envío datos | Bloqueante | Bloqueante | No bloqueante (FreeRTOS) |
| Reset automático nocturno | No | No | Sí (>2h inactivo) |

**Visual sugerido:** Tabla con íconos de check/cruz por versión — fácil de leer desde lejos

---

### Slide 9 — Testing y Validación (45 seg)
**Protocolo de pruebas aplicado:**

- **Prueba de precisión direccional:** 30 cruces controlados (15 entradas + 15 salidas) — resultado esperado: ≥90% detección correcta
- **Prueba de falsos positivos:** Persona estacionada 10 segundos frente a sensores — debe activarse el timeout sin contar
- **Prueba de persistencia:** Reset de energía con contador en valor conocido — debe recuperarse al encender
- **Prueba de conectividad offline:** Desconectar WiFi durante operación — debe seguir contando localmente

**Fallas encontradas y resueltas:**
1. Boot loop por GPIO conflictivos → cambio a GPIO4/GPIO5
2. Error HTTP 400 en Google Sheets → diagnóstico: timestamp con espacio + falta de redirect + ausencia de SSL

**Conexión con el problema original (Avance #1):**
- Identificamos que 74.2% de estudiantes abandona la biblioteca por falta de información
- SIMA entrega ocupación en tiempo real accesible desde cualquier dispositivo en red
- El dashboard muestra % de ocupación + estado (Disponible / Casi lleno / Lleno) antes de desplazarse

**Visual sugerido:** Tabla de pruebas con resultados + foto del dispositivo en la puerta de la biblioteca

---

### Slide 10 — Resultados e Impacto ODS (45 seg)
**Métricas del sistema en operación:**

- El sistema opera a **10 Hz** — detecta personas caminando a velocidad normal sin errores por interferencia sónica
- La arquitectura FreeRTOS garantiza **0ms de latencia** en la detección aunque el envío a Sheets tarde hasta 8 segundos
- El almacenamiento NVS ha demostrado recuperación del contador en **100% de los reinicios forzados** en pruebas
- La batería NCR18650B proporciona autonomía estimada de **16–20 horas** — suficiente para una jornada completa

**Impacto ODS 11:**
- Optimiza el uso de infraestructura existente sin expansión física
- Transfiere la toma de decisión al estudiante *antes* de desplazarse — elimina el 39.7% de tiempo perdido
- Escalable: la misma arquitectura replica a casinos, salas de postgrado y espacios de la ciudad

**Visual sugerido:** Datos clave en tipografía grande + captura del dashboard de Looker Studio en tiempo real

---

### Slide 11 — Demo en Vivo (3–4 min)
**Slide de transición — texto mínimo:**

> *"SIMA en acción — demo en vivo"*

**Guión de la demo (ensayar esto exactamente):**

1. Mostrar el dispositivo encendido — apuntar al Monitor Serial o dashboard web con la IP
2. Pasar una persona por los sensores de entrada → mostrar que el contador sube en el dashboard
3. Pasar una persona por los sensores de salida → mostrar que el contador baja
4. Mostrar Google Sheets con el registro de los eventos con timestamp
5. Mostrar Looker Studio con el histórico (si está disponible)
6. Mostrar el cambio de color del dashboard: verde → naranja → rojo según el umbral

**Plan B si falla el hardware:**
- Reproducir `video_backup_demo.mp4` que muestra el sistema funcionando
- Tener capturas de pantalla de Sheets y Looker Studio listas en el siguiente slide

---

### Slide 12 — Conclusiones (30 seg)
**Logros alcanzados:**
- Sistema IoT funcional que detecta dirección de flujo con precisión
- Ciclo completo de datos: captura → almacenamiento → visualización → decisión
- Encapsulado diseñado para fabricación real y reparación en campo
- Costo total de $28.430 CLP — viable para replicación masiva

**Limitaciones identificadas:**
- Umbral de distancia (40cm) sensible a movimientos laterales cerca de la puerta
- Dependencia de red WiFi estable para sincronización con Google Sheets
- Aforo de 295 personas sin validación empírica de asientos reales en la biblioteca

**Aprendizajes del equipo:**
- La arquitectura dual-core de FreeRTOS resuelve problemas de concurrencia en IoT
- Los fallos de hardware (GPIO conflictivos) son tan críticos como los de software
- Iterar sobre problemas reales (portal cautivo, falsos positivos) produce sistemas más robustos

---

### Slide 13 — Trabajo Futuro (30 seg)
**Mejoras potenciales:**
- Integrar sensor infrarrojo de barrera como redundancia para condiciones de baja luz
- Implementar app móvil o bot de Telegram para consultar ocupación sin acceder a la IP
- Validar aforo real de la Biblioteca F con conteo manual para calibrar el sistema

**Escalabilidad:**
- La arquitectura replica directamente a casinos, salas de postgrado y espacios comunes de otros campus
- Un nodo central (Raspberry Pi o servidor UAI) podría agregar datos de múltiples dispositivos SIMA en un dashboard unificado por campus

**Proyección ODS 11:**
- Desplegado en todos los espacios comunes del campus: reducción estimada del 40% en desplazamientos infructuosos basada en los datos de la encuesta inicial

---

### Slide 14 — Preguntas (30 seg)
- **"¿Preguntas?"**
- Repositorio GitHub: [URL del repo]
- Dashboard en vivo: [URL de Looker Studio]

---

## Diseño visual — guía de estilo

**Paleta de colores sugerida (consistente con el dashboard del sistema):**
- Verde: `#4CAF50` — estado "Disponible"
- Naranja: `#FF9800` — estado "Casi lleno"
- Rojo: `#F44336` — estado "Lleno"
- Fondo: `#F0F4F8` (gris claro) o blanco
- Texto: `#222222`

**Tipografía:**
- Títulos: Calibri Bold 36–40pt o Inter Bold
- Cuerpo: Calibri 24–28pt
- Datos destacados: 48–64pt en color

**Reglas de diseño:**
- Máximo 6 bullets por slide, máximo 6 palabras por bullet
- Una idea principal por slide — el resto va en notas del orador
- Fotos de alta resolución (>1920×1080) sin pixelado
- Siempre más imagen que texto

---

## Distribución de roles en la presentación

| Segmento | Slides | Tiempo | Presentador sugerido |
|---|---|---|---|
| Intro + Problema | 1–3 | 2 min | [Integrante 1] |
| Solución técnica | 4–6 | 2 min | [Integrante 2] |
| Iteraciones + Testing | 7–9 | 2 min | [Integrante 3] |
| Resultados + Demo | 10–11 | 3 min | [Integrante 4] |
| Cierre + Preguntas | 12–14 | 1 min | Todos |

---

## Checklist pre-presentación

### 1 día antes
- [ ] Presentación finalizada, revisada y exportada en PDF de respaldo
- [ ] Ensayo completo del equipo con cronómetro — debe quedar entre 9:30 y 10:00 minutos
- [ ] Roles definidos y ensayados por cada integrante
- [ ] Prototipo funcionando: sensores, WiFi, Google Sheets y dashboard
- [ ] Batería NCR18650B cargada al 100% (LED verde en el shield)
- [ ] Video de backup grabado y guardado en USB y en el repo
- [ ] Capturas de pantalla de Sheets y Looker Studio como respaldo estático
- [ ] URL del repo y del dashboard anotada en el último slide

### Día de la presentación
- [ ] Llegar 15 minutos antes al lugar
- [ ] Probar el prototipo en el lugar físico — verificar que detecta correctamente
- [ ] Probar conexión de laptop al proyector — slides en 16:9
- [ ] Presentación disponible en USB como respaldo adicional
- [ ] Abrir el dashboard web en el navegador antes de empezar
- [ ] Abrir Looker Studio en otra pestaña como respaldo visual
- [ ] Todos los integrantes presentes y con su sección ensayada

### Durante la presentación
- [ ] Hablar mirando a la audiencia, no leer los slides
- [ ] En la demo: explicar en voz alta qué está pasando mientras se muestra
- [ ] Ante una pregunta técnica difícil: "Eso está documentado en el FUENTES.md del repositorio" es una respuesta válida
- [ ] Si el hardware falla durante la demo: cambiar al video de backup sin pánico — es parte del plan

---

## Preguntas frecuentes que puede hacer el evaluador — y cómo responderlas

**"¿Por qué usaron setInsecure() en el SSL?"**
→ Google rota sus certificados frecuentemente. Usar un certificado fijo haría que el sistema dejara de funcionar cada vez que Google lo actualice. En el contexto de una red universitaria local, el riesgo de man-in-the-middle es aceptable.

**"¿Cómo justifican el umbral de 40cm?"**
→ Los sensores Seeed tienen un ángulo de detección de 15°. A la distancia de instalación en el marco de la puerta, 40cm cubre el paso de una persona sin detectar movimientos laterales externos.

**"¿Qué pasa si dos personas pasan al mismo tiempo?"**
→ El sistema detecta una a la vez. Es una limitación conocida del diseño de sensores secuenciales. Para flujos de alta densidad se requeriría un sensor infrarrojo de barrera completa — está identificado como trabajo futuro.

**"¿Por qué FreeRTOS y no simplemente delay()?"**
→ Con delay(), cada llamada HTTP bloquea el loop principal durante hasta 8 segundos. En ese tiempo el sistema no detecta personas. FreeRTOS permite que Core 0 maneje HTTP mientras Core 1 sigue detectando sin interrupción.

**"¿El aforo de 295 es real?"**
→ Es la mejor estimación disponible. Reconocemos que no fue validado con conteo manual de asientos — está identificado como limitación y trabajo futuro.

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · Junio 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
