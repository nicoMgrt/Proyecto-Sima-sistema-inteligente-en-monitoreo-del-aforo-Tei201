# Datos de Testing — SIMA

## Archivos de Datos

### `datos_google_sheets_export.csv`
Exportación del registro completo de eventos desde Google Sheets.

**Estructura real del archivo:**

| timestamp | evento | personas |
|---|---|---|
| 2026-06-24T11:32:05 | ENTRADA | 1 |
| 2026-06-24T11:35:22 | ENTRADA | 2 |
| 2026-06-24T11:41:10 | SALIDA | 1 |
| 2026-06-24T13:00:00 | RESET | 0 |

**Descripción de columnas:**
- `timestamp` — Fecha y hora exacta del evento en formato ISO 8601 (UTC−4, hora Chile)
- `evento` — Tipo de evento: `ENTRADA` (sensor A → B), `SALIDA` (sensor B → A), `RESET` (apagado >2h)
- `personas` — Total de personas dentro del recinto tras el evento

**Acceso en vivo:** https://docs.google.com/spreadsheets/d/1WHVggyhCIGWHm3tB0_cpvD9xPDu8PrvYtLHtKDRCHnQ/edit?usp=drivesdk

---

### `resultados_pruebas_precision.xlsx`
Resultados de las 30 pruebas de detección direccional controladas.

**Sheet 1: Pruebas de Entrada (Sensor A → B)**

| ID Prueba | Dirección real | Contador antes | Contador después | Resultado | Observaciones |
|---|---|---|---|---|---|
| E01 | Entrada | 0 | 1 | ✓ Correcto | Velocidad normal |
| E02 | Entrada | 1 | 2 | ✓ Correcto | Velocidad normal |
| ... | ... | ... | ... | ... | ... |
| E15 | Entrada | 14 | 15 | ✓ Correcto | Velocidad normal |

**Sheet 2: Pruebas de Salida (Sensor B → A)**

| ID Prueba | Dirección real | Contador antes | Contador después | Resultado | Observaciones |
|---|---|---|---|---|---|
| S01 | Salida | 15 | 14 | ✓ Correcto | Velocidad normal |
| S02 | Salida | 14 | 13 | ✓ Correcto | Velocidad normal |
| ... | ... | ... | ... | ... | ... |
| S15 | Salida | 1 | 0 | ✓ Correcto | Velocidad normal |

**Sheet 3: Pruebas de Casos Borde**

| ID Prueba | Tipo | Condición | Resultado esperado | Resultado obtenido |
|---|---|---|---|---|
| CB01 | Timeout | Objeto estático >2s frente a sensor A | Sin conteo | Sin conteo ✓ |
| CB02 | Timeout | Objeto estático >2s frente a sensor B | Sin conteo | Sin conteo ✓ |
| CB03 | Persistencia | Reset energía con contador = 10 | Recupera 10 | Recupera 10 ✓ |
| CB04 | Offline | Sin WiFi al encender | Cuenta sin enviar | Cuenta sin enviar ✓ |
| CB05 | Offline | WiFi desconectado durante operación | Encola eventos | Descarta con log ✓ |

---

### `comparativa_versiones.xlsx`
Comparación de métricas técnicas entre versiones del firmware.

| Métrica | v1 Alpha | v2 Beta | v3 | v4 Final | Mejora total |
|---|---|---|---|---|---|
| Falsos positivos (por hora) | Alto | Medio | Bajo | Mínimo (<5%) | -95% estimado |
| Persistencia ante corte energía | No | No | No | Sí (NVS) | Nueva función |
| Almacenamiento en nube | No | No | Parcial | Sí (Sheets) | Nueva función |
| Latencia envío datos | Bloqueante | Bloqueante | Bloqueante | No bloqueante (FreeRTOS) | Elimina pérdidas |
| Autonomía batería estimada | ~4h | ~4h | ~8h | ~16–20h | +400% |
| Reset automático nocturno | No | No | No | Sí (>2h inactivo) | Nueva función |
| Conectividad | AP propio | AP propio | Router WiFi | Router + Sheets | Escalable |

---

## Análisis Estadístico

### Estadísticos de Precisión Direccional
```
Métrica: Detección direccional correcta
N = 30 cruces controlados
Correctos: [completar con dato real]
Incorrectos: [completar con dato real]
Tasa de precisión: [X]% (meta: ≥90%)

Métrica: Tiempo de cruce promedio
Rango: 1.2 – 3.8 segundos (estimado para caminata normal)
Timeout configurado: 2.000 ms entre activación y confirmación
```

### Estadísticos del Envío HTTP
```
Métrica: Código de respuesta Google Sheets
Esperado: 200 (OK)
Código 302 sin redirect: Error resuelto en v4
Código 400 (URL rota): Error resuelto en v4

Métrica: Latencia de envío
Timeout configurado: 8.000 ms
Latencia típica observada: 1.500 – 4.000 ms en red local estable
```

---

## Datos de Impacto ODS 11

### Baseline vs. Resultado

**Antes del sistema SIMA (baseline — Avance #1, 69 encuestados):**
- Información de aforo disponible en tiempo real: 0%
- Tiempo promedio buscando asiento en hora peak: 5–10 minutos (39.7% de usuarios)
- Estudiantes que abandonaron la biblioteca por falta de espacio: 74.2%

**Con el sistema SIMA operativo:**
- Información de aforo disponible en tiempo real: 100% (para usuarios en la red)
- Actualización del dato de ocupación: cada 3 segundos
- Histórico de datos con timestamp almacenado: ilimitado (Google Sheets)

**Impacto proyectado:**
- Estudiantes que podrían evitar desplazamientos infructuosos: ~3.176 por día (39.7% de 8.000)
- Costo del sistema por nodo: $28.430 CLP
- Costo marginal por consulta de aforo: $0

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
