# Datos de Testing — SIMA

## Archivos de Datos

### `datos_google_sheets_export.csv`
Exportación del registro completo de eventos desde Google Sheets.

**Estructura real del archivo:**

| timestamp | evento | personas |
|---|---|---|
| 2026-06-25T14:30:00 | ENTRADA | 1 |
| 2026-06-25T14:31:45 | ENTRADA | 2 |
| 2026-06-25T14:35:10 | SALIDA | 1 |
| 2026-06-25T14:37:00 | RESET | 0 |

**Descripción de columnas:**
- `timestamp` — Fecha y hora exacta en formato ISO 8601 (UTC−4, hora Chile)
- `evento` — Tipo: `ENTRADA` (sensor A→B), `SALIDA` (sensor B→A), `RESET` (apagado >2h)
- `personas` — Total de personas dentro del recinto tras el evento

**Acceso en vivo:** https://docs.google.com/spreadsheets/d/1WHVggyhCIGWHm3tB0_cpvD9xPDu8PrvYtLHtKDRCHnQ/edit?usp=drivesdk

---

### `resultados_pruebas_precision.xlsx`
Resultados de las pruebas de detección direccional controladas.

**Sheet 1: Pruebas de Entrada (Sensor A → B)**

| ID Prueba | Dirección real | Resultado | Tasa de éxito |
|---|---|---|---|
| E01–E10 | Entrada | 9/10 correctos | **90%** |

**Sheet 2: Pruebas de Salida (Sensor B → A)**

| ID Prueba | Dirección real | Resultado | Tasa de éxito |
|---|---|---|---|
| S01–S10 | Salida | 8/10 correctos | **80%** |

**Sheet 3: Pruebas de Casos Borde**

| ID Prueba | Tipo | Condición | Resultado obtenido |
|---|---|---|---|
| CB01 | Timeout | Objeto estático >2s frente a sensor A | Sin conteo ✓ — reset a 2000ms exactos |
| CB02 | Timeout | Objeto estático >2s frente a sensor B | Sin conteo ✓ — sin conteos fantasma |
| CB03 | Persistencia | Reset energía con contador = N | Recupera N ✓ — 5/5 reinicios sin pérdida |
| CB04 | Offline | Sin WiFi al encender | Cuenta sin enviar ✓ |
| CB05 | Sheets | Envío HTTP tras evento | HTTP 200 OK ✓ — latencia 1.5–3 seg |

---

### `comparativa_versiones.xlsx`
Comparación de métricas técnicas entre versiones del firmware.

| Métrica | v1 Alpha | v2 Beta | v3 | v4 Final | Mejora total |
|---|---|---|---|---|---|
| Tasa de precisión entrada | Sin medir | Sin medir | Sin medir | **90%** | Nueva métrica |
| Tasa de precisión salida | Sin medir | Sin medir | Sin medir | **80%** | Nueva métrica |
| Falsos positivos | Alto | Medio | Bajo | Mínimo (<5%) | −95% estimado |
| Persistencia ante corte energía | No | No | No | Sí (NVS) — 5/5 ✓ | Nueva función |
| Almacenamiento en nube | No | No | Parcial | Sí — HTTP 200 OK | Nueva función |
| Latencia envío datos | Bloqueante | Bloqueante | Bloqueante | 1.5–3 seg (FreeRTOS) | Elimina pérdidas |
| Autonomía batería | ~4h | ~4h | ~8h | ~16–20h | +400% |
| Reset automático nocturno | No | No | No | Sí (>2h inactivo) | Nueva función |
| Conectividad | AP propio | AP propio | Router WiFi | Router + Sheets | Escalable |

---

## Análisis Estadístico

### Estadísticos de Precisión Direccional
```
Métrica: Detección de Entrada (Sensor A → B)
N = 10 intentos controlados
Correctos: 9
Incorrectos: 1
Tasa de precisión: 90%

Métrica: Detección de Salida (Sensor B → A)
N = 10 intentos controlados
Correctos: 8
Incorrectos: 2
Tasa de precisión: 80%

Precisión global: 85–90%
Frecuencia de muestreo: 10 Hz (100ms por ciclo)
Justificación: personas caminan ~1m/s, puerta ~60cm de ancho
Latencia de detección: ~100–200ms (1–2 ciclos del loop)
```

### Estadísticos del Envío HTTP a Google Sheets
```
Métrica: Código de respuesta HTTP
Resultado: 200 OK en todos los envíos tras corrección de v4
Errores previos resueltos: 400 (URL rota) y 302 sin redirect

Métrica: Latencia ESP32 → Google Sheets
Mínimo: 1.5 segundos
Máximo: 3.0 segundos
Promedio estimado: ~2.0 segundos
Timeout configurado: 8.0 segundos
```

### Estadísticos de Persistencia
```
Métrica: Recuperación de contador tras corte de energía
N = 5 reinicios forzados
Recuperaciones correctas: 5/5 (100%)
Tiempo de recuperación: <2 segundos desde encendido
Confirmación: Serial Monitor imprime 'Contador recuperado de memoria: N'
```

---

## Datos de Impacto ODS 11

### Baseline vs. Resultado

**Antes del sistema SIMA (baseline — Avance #1, 69 encuestados):**
- Información de aforo disponible en tiempo real: **0%**
- Tiempo buscando asiento en hora peak: **5–10 min** (39.7% de usuarios)
- Estudiantes que abandonaron por falta de espacio: **74.2%**

**Con el sistema SIMA operativo (validado en pruebas):**
- Información de aforo disponible en tiempo real: **100%** (usuarios en la red)
- Actualización del dato de ocupación: **cada 3 segundos**
- Precisión de detección: **85–90%** en condiciones de prueba controladas
- Latencia de registro en Sheets: **<3 segundos** por evento
- Histórico de datos almacenado: **ilimitado** (Google Sheets)

### Tabla de Validación contra el Problema Original

| Necesidad identificada (Avance #1) | Solución implementada | Evidencia |
|---|---|---|
| Saber si hay espacio antes de ir | Dashboard web con % de ocupación en tiempo real | Demo en vivo durante presentación |
| Datos históricos para planificar | Google Sheets registra cada evento con timestamp | Dashboard Looker Studio |
| Indicador visual intuitivo | Semáforo: verde (<50%), naranja (<80%), rojo (lleno) | Capturas del dashboard |
| Funcionamiento continuo | Persistencia NVS + reset automático + modo offline | Prueba 4: 5/5 reinicios ✓ |

**Conclusión de validación:**
El sistema detectó correctamente el 85–90% de los pasos frente a los sensores, registró todos los eventos en Google Sheets con latencia menor a 3 segundos, y mostró el estado de ocupación en tiempo real. Los datos históricos permiten identificar horarios de mayor ocupación, entregando información concreta para que los administradores de la biblioteca tomen decisiones sobre horarios y distribución de espacios.

**Impacto proyectado:**
- Estudiantes que podrían evitar desplazamientos infructuosos: ~3.176 por día (39.7% de 8.000)
- Costo del sistema por nodo: $28.430 CLP
- Costo marginal por consulta de aforo: $0

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
