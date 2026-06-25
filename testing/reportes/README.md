# Reportes de Testing — SIMA

## Documento Principal

### `protocolo_pruebas_sima_v1.pdf`
Protocolo de pruebas completo del sistema SIMA con resultados y análisis.

---

## Estructura del Reporte

### 1. Introducción
**Objetivo del testing:** Validar que el sistema SIMA detecta correctamente la dirección del flujo de personas (entrada vs. salida) con una tasa de precisión igual o superior al 90%, y que los eventos se almacenan correctamente en Google Sheets con timestamp para análisis posterior.

**Metodología:** Pruebas controladas de precisión direccional + pruebas de casos borde (timeout, persistencia, operación offline).

---

### 2. Descripción del Sistema Bajo Prueba

**Hardware:**
- ESP32-S3 Dual-core Xtensa LX7 @ 240 MHz
- 2× Sensor Ultrasónico Seeed Grove (3 pines, SIG compartido)
- Sensor A en GPIO 4 (pasillo exterior), Sensor B en GPIO 5 (interior biblioteca)
- Umbral de detección: 40 cm
- Frecuencia de muestreo: 10 Hz (100 ms por ciclo — justificación: personas caminan ~1m/s, puerta ~60cm)

**Software:**
- Firmware v4 con arquitectura FreeRTOS dual-core
- Google Sheets como repositorio persistente de datos
- Dashboard web local con actualización cada 3 segundos

---

### 3. Protocolo Aplicado

**Prueba 1 — Precisión de Detección Direccional (Entrada)**

| Parámetro | Valor |
|---|---|
| Total de intentos | 10 |
| Velocidad de cruce | Normal (caminata) |
| Criterio de éxito | ≥9 de 10 correctos (90%) |

**Pasos:**
1. Encender el sistema y esperar `¡Conectado! IP: X.X.X.X` en Monitor Serial
2. Pasar frente a Sensor A (GPIO4) primero, luego Sensor B (GPIO5)
3. Verificar que el contador sube +1 y aparece evento en Google Sheets
4. Repetir 10 veces registrando cada resultado

---

**Prueba 2 — Precisión de Detección Direccional (Salida)**

| Parámetro | Valor |
|---|---|
| Total de intentos | 10 |
| Velocidad de cruce | Normal (caminata) |
| Criterio de éxito | ≥8 de 10 correctos (80%) |

**Pasos:**
1. Pasar frente a Sensor B (GPIO5) primero, luego Sensor A (GPIO4)
2. Verificar que el contador baja -1 y no baja de 0 (protección activa)
3. Repetir 10 veces registrando cada resultado

---

**Prueba 3 — Sistema Anti-bloqueo (Timeout)**

| Parámetro | Valor |
|---|---|
| Timeout configurado | 2.000 ms |
| Condición | Objeto estático frente a sensor A o B por >2 segundos |
| Criterio de éxito | 0 eventos falsos en 5 pruebas |

**Pasos:**
1. Colocar la mano frente al Sensor A durante 3 segundos sin mover hacia B
2. Verificar en Serial: `Alguien se quedó bloqueando la puerta. Conteo cancelado.`
3. Verificar que el contador no cambió y Google Sheets no registró evento

---

**Prueba 4 — Persistencia ante Corte de Energía**

| Parámetro | Valor |
|---|---|
| Valor de prueba | Contador en N personas |
| Método | Apagado con switch físico del encapsulado |
| Criterio de éxito | Contador recupera exactamente el valor N al encender |

**Pasos:**
1. Anotar el valor actual del contador (N)
2. Apagar el dispositivo con el switch lateral
3. Esperar 10 segundos → encender
4. Verificar en Monitor Serial: `Contador recuperado: N`

---

**Prueba 5 — Envío de Datos a Google Sheets**

| Parámetro | Valor |
|---|---|
| Condición | ESP32-S3 conectada al WiFi, Apps Script desplegado |
| Criterio de éxito | HTTP 200 OK + fila en Sheets con timestamp correcto |

**Pasos:**
1. Simular entrada (A→B)
2. Verificar nueva fila en Sheets con timestamp, evento y personas correctos
3. Confirmar código HTTP 200 en Monitor Serial

---

### 4. Resultados Cuantitativos

| Prueba | Total casos | Correctos | Incorrectos | Tasa de éxito |
|---|---|---|---|---|
| Detección entrada (A→B) | 10 | 9 | 1 | **90%** ✓ |
| Detección salida (B→A) | 10 | 8 | 2 | **80%** ✓ |
| Anti-bloqueo timeout | 5 | 5 | 0 | **100%** ✓ |
| Persistencia energía | 5 | 5 | 0 | **100%** ✓ |
| Envío a Google Sheets | 5 | 5 | 0 | **100%** ✓ |
| **TOTAL** | **35** | **32** | **3** | **91.4%** ✓ |

**Datos adicionales obtenidos:**
- Latencia de detección: ~100–200ms (1–2 ciclos del loop)
- Latencia ESP32 → Google Sheets: 1.5–3 segundos
- HTTP response code: 200 OK en todos los envíos tras corrección v4
- Tiempo hasta timeout sensor bloqueado: 2.000ms exactos
- Tiempo de recuperación contador tras corte: <2 segundos desde encendido

---

### 5. Fallas Encontradas y Resueltas

**Falla 1 — Boot Loop por GPIO Conflictivos**
- **Síntomas:** Reconexión constante del puerto COM, entrada repetida al boot ROM (`ESP-ROM:esp32s3-20210327`), imposibilidad de subir código
- **Causas:** GPIO1 (TX del UART0) y GPIO10 (controlador SPI interno) cambiados entre INPUT/OUTPUT cada 100ms, interfiriendo con el proceso de boot
- **Solución:** Migración a GPIO4 y GPIO5 + erase completo de flash + secuencia BOOT+RESET para recuperar la placa
- **Resultado:** Cero boot loops en todas las pruebas posteriores ✓

**Falla 2 — Error HTTP 400 en Google Sheets**
- **Síntomas:** `Sheets HTTP: 400` en Monitor Serial — datos no registrados en Sheets
- **Causas identificadas:**
  1. Timestamp con espacio (`2026-06-24 11:32`) rompía la URL del GET request
  2. Faltaba `WiFiClientSecure` para conexiones HTTPS
  3. Google Apps Script redirige con código 302 — sin `setFollowRedirects` el script no se ejecutaba
- **Solución:** Separador ISO 8601 `T` + `WiFiClientSecure` con `setInsecure()` + `HTTPC_STRICT_FOLLOW_REDIRECTS`
- **Resultado:** HTTP 200 OK confirmado en todos los envíos posteriores ✓

---

### 6. Validación de Impacto ODS 11

**Problema original (Avance #1):** El 74.2% de los estudiantes de la UAI ha tenido que abandonar la Biblioteca de Pregrado por falta de espacio, y el 39.7% pierde entre 5 y más de 10 minutos buscando asiento en hora peak (M3–M5).

| Necesidad identificada | Solución implementada | Evidencia |
|---|---|---|
| Saber si hay espacio antes de ir | Dashboard web con % de ocupación en tiempo real | Demo en vivo durante presentación |
| Datos históricos para planificar | Google Sheets registra cada evento con timestamp | Dashboard Looker Studio |
| Indicador visual intuitivo | Semáforo: verde (<50%), naranja (<80%), rojo (lleno) | Capturas del dashboard |
| Funcionamiento continuo | Persistencia NVS + reset automático + modo offline | Prueba 4: 5/5 reinicios ✓ |

| Indicador | Baseline | Meta | Alcanzado |
|---|---|---|---|
| Información de aforo en tiempo real | 0% | Sistema operativo | Dashboard web + Sheets ✓ |
| Frecuencia de actualización | Sin sistema | ≤5 segundos | 3 segundos ✓ |
| Persistencia de datos históricos | Sin almacenamiento | Registro continuo | Google Sheets ilimitado ✓ |
| Autonomía del dispositivo | Sin referencia | Jornada completa (8h) | 16–20 horas ✓ |

**Proyección de escalabilidad:** Con 8.000 estudiantes diarios y 39.7% afectados, el despliegue completo de SIMA podría eliminar el costo de oportunidad temporal para ~3.176 estudiantes por día. Costo por nodo: $28.430 CLP. Costo marginal por consulta: $0.

---

### 7. Conclusiones

**Fortalezas del prototipo:**
- Precisión global del 91.4% en todas las pruebas realizadas
- Arquitectura dual-core garantiza detección continua sin interrupciones por envío HTTP
- Persistencia de datos ante cortes de energía: 100% en 5 reinicios forzados
- Ciclo completo captura → almacenamiento → visualización implementado y validado
- Costo total de $28.430 CLP — viable para replicación institucional

**Limitaciones identificadas:**
- Sistema optimizado para flujo en fila simple — no discrimina cruces simultáneos
- Dependencia de red WiFi estable para sincronización con Sheets
- Aforo de 295 personas no validado con conteo manual de asientos reales

**Recomendaciones para mejoras futuras:**
- Agregar sensor infrarrojo de barrera como redundancia para condiciones de baja iluminación
- Implementar integración con Telegram para consulta remota sin estar en la red local
- Validar el aforo real de la Biblioteca F con conteo físico de asientos

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
