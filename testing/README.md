# Testing y Validación — SIMA

## Objetivo
Validar que el sistema SIMA detecta correctamente el flujo de personas (entrada/salida) en condiciones reales de operación, y demostrar que los datos capturados responden directamente al problema de gestión de aforo identificado en el Avance #1.

---

## Carpetas

#### `reportes/`
Protocolo de pruebas completo con 5 pruebas documentadas, análisis de fallas y validación contra el problema original

#### `evidencias/`
Registro fotográfico del prototipo instalado y operando en condiciones reales

#### `datos/`
Registros cuantitativos exportados de Google Sheets y resultados de pruebas de precisión con datos reales

---

## Resumen de Resultados

| Prueba | Resultado | Tasa de éxito |
|---|---|---|
| Detección de entrada (A→B) | ✓ Exitosa | 90% (9/10) |
| Detección de salida (B→A) | ✓ Exitosa | 80% (8/10) |
| Timeout sensor bloqueado | ✓ Exitosa | 100% (5/5) |
| Persistencia tras corte de energía | ✓ Exitosa | 100% (5/5) |
| Envío a Google Sheets | ✓ Exitosa | HTTP 200 OK |
| **Precisión global** | | **85–90%** |

---

## Metodología de Testing

### Contexto de instalación
- **Ubicación:** Biblioteca de Pregrado, Campus Peñalolén, Universidad Adolfo Ibáñez
- **Sensor A** (GPIO4) — lado pasillo exterior
- **Sensor B** (GPIO5) — lado interior de la biblioteca
- **Umbral de detección:** 40 cm
- **Timeout sensor:** 2.000 ms
- **Frecuencia de muestreo:** 100 ms (justificación: personas caminan ~1m/s, puerta ~60cm de ancho)

### Protocolo de Testing

1. **Verificación inicial del sistema**
   - Confirmar conexión WiFi y sincronización NTP en Monitor Serial
   - Verificar que el dashboard web responde en la IP asignada
   - Confirmar que Google Sheets recibe eventos de prueba con HTTP 200 OK

2. **Pruebas de precisión direccional**
   - 10 cruces simulando entrada (A→B) — resultado: 9/10 correctos (90%)
   - 10 cruces simulando salida (B→A) — resultado: 8/10 correctos (80%)
   - Latencia de detección: ~100–200ms por evento

3. **Pruebas de casos borde**
   - Objeto estático frente al sensor >2s → timeout a 2.000ms exactos, sin conteos fantasma
   - Corte de energía con contador en N → recupera N en 100% de los casos (5/5 reinicios)
   - Operación sin WiFi → sistema cuenta localmente sin crashear

4. **Verificación de almacenamiento**
   - Evento registrado en Sheets con timestamp ISO 8601 correcto
   - Latencia promedio ESP32 → Sheets: 1.5–3 segundos
   - HTTP response code: 200 OK confirmado en Monitor Serial

---

## Métricas de Evaluación

### Desempeño Técnico

| Métrica | Valor obtenido | Meta |
|---|---|---|
| Precisión detección entrada | 90% (9/10) | ≥90% ✓ |
| Precisión detección salida | 80% (8/10) | ≥80% ✓ |
| Tiempo hasta timeout | 2.000ms exactos | ≤2.000ms ✓ |
| Latencia envío a Sheets | 1.5–3 segundos | ≤8 segundos ✓ |
| Recuperación tras corte energía | 100% (5/5) | 100% ✓ |
| Frecuencia actualización dashboard | 3 segundos | ≤5 segundos ✓ |

---

## Fallas Encontradas y Resueltas

### Falla 1 — Boot Loop por GPIO Conflictivos
- **Síntomas:** Reconexión constante del puerto COM, entrada repetida al boot ROM
- **Causa:** GPIO1 (TX del UART0) y GPIO10 usados para los sensores interferían con funciones internas del chip
- **Solución:** Migración a GPIO4 y GPIO5 + erase completo de flash + secuencia BOOT+RESET
- **Resultado:** Sistema completamente estable en todas las pruebas posteriores

### Falla 2 — Error HTTP 400 en Google Sheets
- **Síntomas:** `Sheets HTTP: 400` en Monitor Serial, datos no registrados en Sheets
- **Causas:** Timestamp con espacio rompía la URL + falta de `WiFiClientSecure` + ausencia de `setFollowRedirects`
- **Solución:** Separador ISO 8601 `T` + `WiFiClientSecure` con `setInsecure()` + `HTTPC_STRICT_FOLLOW_REDIRECTS`
- **Resultado:** HTTP 200 OK confirmado en todos los envíos posteriores

---

## Validación contra el Problema Original (Avance #1)

**Problema identificado:** Los estudiantes de la UAI no tienen información sobre disponibilidad de espacio en la Biblioteca de Pregrado antes de desplazarse. El 74.2% ha tenido que abandonar la biblioteca por falta de espacio y el 39.7% pierde entre 5 y más de 10 minutos buscando asiento en hora peak.

| Necesidad identificada | Solución implementada | Evidencia |
|---|---|---|
| Saber si hay espacio antes de ir | Dashboard web con % de ocupación en tiempo real | Demo en vivo durante presentación |
| Datos históricos para planificar | Google Sheets registra cada evento con timestamp | Dashboard Looker Studio |
| Indicador visual intuitivo | Semáforo: verde (<50%), naranja (<80%), rojo (lleno) | Capturas del dashboard |
| Funcionamiento continuo | Persistencia NVS + reset automático + modo offline | Prueba 4: 5/5 reinicios ✓ |

**Conclusión:** El sistema detectó correctamente el 85–90% de los pasos, registró todos los eventos en Sheets con latencia menor a 3 segundos, y entregó el estado de ocupación en tiempo real. Los datos históricos permiten identificar horarios de mayor demanda para que los administradores de la biblioteca tomen decisiones informadas.

**Impacto ODS 11 proyectado:**
- Estudiantes que podrían evitar desplazamientos infructuosos: ~3.176 por día (39.7% de 8.000)
- Costo del sistema por nodo: $28.430 CLP
- Costo marginal por consulta de aforo: $0

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
