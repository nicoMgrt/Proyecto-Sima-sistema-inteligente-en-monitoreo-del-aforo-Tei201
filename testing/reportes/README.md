# Reportes de Testing — SIMA

## Documento Principal

### `protocolo_pruebas_sima_v1.pdf`

Protocolo de pruebas completo del sistema SIMA con resultados y análisis.

---

## Estructura del Reporte

### 1. Introducción
**Objetivo del testing:**
Validar que el sistema SIMA detecta correctamente la dirección del flujo de personas (entrada vs. salida) en una puerta con una tasa de precisión igual o superior al 90%, y que los eventos se almacenan correctamente en Google Sheets con timestamp para análisis posterior.

**Metodología utilizada:**
Pruebas controladas de precisión direccional + pruebas de casos borde (timeout, persistencia, operación offline).

---

### 2. Descripción del Sistema Bajo Prueba

**Hardware:**
- ESP32-S3 Dual-core Xtensa LX7 @ 240 MHz
- 2× Sensor Ultrasónico Seeed Grove (3 pines, SIG compartido)
- Sensor A en GPIO 4 (pasillo exterior), Sensor B en GPIO 5 (interior biblioteca)
- Umbral de detección: 40 cm
- Frecuencia de muestreo: 10 Hz (pausa de 100 ms entre ciclos)

**Software:**
- Firmware v4 con arquitectura FreeRTOS dual-core
- Google Sheets como repositorio persistente de datos
- Dashboard web local con actualización cada 3 segundos

---

### 3. Protocolo Aplicado

**Prueba 1 — Precisión de Detección Direccional**

| Parámetro | Valor |
|---|---|
| Total de cruces | 30 (15 entradas + 15 salidas) |
| Velocidad de cruce | Normal (caminata) |
| Distancia al sensor | Marco de la puerta (~60 cm del suelo a los sensores) |
| Criterio de éxito | ≥27 de 30 correctos (90%) |

**Pasos:**
1. Encender el sistema y esperar confirmación de WiFi en Serial (`¡Conectado! IP: X.X.X.X`)
2. Registrar el valor inicial del contador
3. Ejecutar 15 cruces completos de entrada (A → B) verificando que el contador sume +1 en cada uno
4. Ejecutar 15 cruces completos de salida (B → A) verificando que el contador reste -1 en cada uno
5. Verificar en Google Sheets que cada evento tiene timestamp correcto y tipo de evento correcto

---

**Prueba 2 — Sistema Anti-bloqueo (Timeout)**

| Parámetro | Valor |
|---|---|
| Timeout configurado | 2.000 ms |
| Condición | Objeto estático frente a sensor A o B por >2 segundos |
| Criterio de éxito | 0 eventos falsos en 5 pruebas |

**Pasos:**
1. Colocar la mano frente al Sensor A durante 3 segundos sin mover hacia B
2. Verificar en Serial que aparece: `Alguien se quedó bloqueando la puerta. Conteo cancelado.`
3. Verificar que el contador no cambió
4. Repetir con Sensor B
5. Verificar que Google Sheets no registró ningún evento durante la prueba

---

**Prueba 3 — Persistencia ante Corte de Energía**

| Parámetro | Valor |
|---|---|
| Valor de prueba | Contador en 10 personas |
| Método | Apagado con switch físico del encapsulado |
| Criterio de éxito | Contador recupera exactamente el valor 10 al encender |

**Pasos:**
1. Ejecutar 10 entradas para llevar el contador a 10
2. Apagar el dispositivo con el switch lateral
3. Esperar 10 segundos
4. Encender y observar el Monitor Serial
5. Verificar que aparece: `Contador recuperado: 10`

---

**Prueba 4 — Operación Offline (sin WiFi)**

| Parámetro | Valor |
|---|---|
| Condición | Red WiFi no disponible al encender |
| Criterio de éxito | Sistema opera y cuenta personas correctamente |

**Pasos:**
1. Apagar el router antes de encender el dispositivo
2. Esperar 15 segundos (timeout de conexión WiFi del firmware)
3. Verificar que aparece: `Sin WiFi — Funcionando en modo offline`
4. Ejecutar 5 cruces de entrada y salida
5. Verificar que el contador responde correctamente aunque los eventos no lleguen a Sheets

---

### 4. Resultados Cuantitativos

**Tabla resumen de resultados:**

| Prueba | Total casos | Correctos | Incorrectos | Tasa de éxito |
|---|---|---|---|---|
| Detección entrada | 15 | [completar] | [completar] | [X]% |
| Detección salida | 15 | [completar] | [completar] | [X]% |
| Anti-bloqueo timeout | 5 | [completar] | [completar] | [X]% |
| Persistencia energía | 3 | [completar] | [completar] | [X]% |
| Operación offline | 1 | [completar] | [completar] | [X]% |
| **TOTAL** | **39** | **[completar]** | **[completar]** | **[X]%** |

---

### 5. Fallas Encontradas y Resueltas

**Falla 1 — Boot loop por GPIO conflictivos**

- **Descripción:** El ESP32-S3 entraba en ciclo de reinicio constante al usar GPIO1 (TX del UART0) y GPIO10 (mapeado al controlador SPI interno) para los sensores ultrasónicos.
- **Diagnóstico:** Cambiar los pines entre OUTPUT e INPUT durante `medirDistancia()` interfería con las funciones internas del chip. El proceso de boot también usa esos pines, generando inestabilidad crítica.
- **Solución:** Migración a GPIO4 y GPIO5 (sin funciones internas conflictivas). Erase completo de flash con herramienta `esptool` y reinstalación limpia del firmware.
- **Resultado:** Sistema completamente estable. Cero boot loops en todas las pruebas posteriores.

---

**Falla 2 — Error HTTP 400 en Google Sheets**

- **Descripción:** Las peticiones al Google Apps Script retornaban código 400 (Bad Request). Los datos no se almacenaban en Sheets.
- **Diagnóstico:** Tres causas simultáneas identificadas:
  1. Timestamp con espacio (`2026-06-24 11:32`) que rompía la URL al pasarlo como parámetro GET
  2. Ausencia de `WiFiClientSecure` para gestionar conexiones HTTPS
  3. Google Apps Script redirige con código 302 — sin `setFollowRedirects` la petición terminaba sin ejecutar el script
- **Solución:** Separador ISO 8601 `T` en el timestamp, incorporación de `WiFiClientSecure` con `client.setInsecure()`, y `http.setFollowRedirects(HTTPC_STRICT_FOLLOW_REDIRECTS)`.
- **Resultado:** 100% de eventos registrados correctamente en Sheets desde la corrección. Código de respuesta HTTP 200 confirmado en Monitor Serial.

---

### 6. Validación de Impacto ODS 11

**Problema original (Avance #1):** Los estudiantes de la UAI no tienen información sobre la disponibilidad de espacio en la Biblioteca de Pregrado antes de desplazarse. El 74.2% de 69 encuestados ha tenido que abandonar la biblioteca por falta de espacio, y el 39.7% pierde entre 5 y más de 10 minutos buscando asiento en hora peak (M3–M5).

**Respuesta de SIMA:** El sistema entrega el porcentaje de ocupación en tiempo real a través de un dashboard web accesible desde cualquier dispositivo en la misma red, actualizado cada 3 segundos. Los datos se almacenan en Google Sheets con timestamp, permitiendo análisis histórico de patrones de ocupación por hora y día.

**Indicadores medidos:**

| Indicador | Baseline | Meta | Alcanzado |
|---|---|---|---|
| Información de aforo en tiempo real | 0% disponibilidad | Sistema operativo | Dashboard web + Sheets ✓ |
| Frecuencia de actualización del dato | Sin sistema | ≤5 segundos | 3 segundos ✓ |
| Persistencia de datos históricos | Sin almacenamiento | Registro continuo | Google Sheets ilimitado ✓ |
| Autonomía del dispositivo | Sin referencia | Jornada completa (8h) | 16–20 horas ✓ |

**Proyección de escalabilidad:**
Con 8.000 estudiantes diarios en el campus y 39.7% afectados por la falta de información de aforo, un despliegue completo de SIMA en los accesos principales de la Biblioteca de Pregrado podría eliminar el costo de oportunidad temporal para aproximadamente 3.176 estudiantes por día. La arquitectura es replicable a casinos, salas de postgrado y otros espacios de alta demanda del campus con un costo marginal de $28.430 CLP por nodo adicional.

---

### 7. Conclusiones

**Fortalezas del prototipo:**
- Detección direccional funcional con arquitectura dual-core que garantiza operación continua
- Persistencia de datos ante cortes de energía mediante NVS flash
- Ciclo completo captura → almacenamiento → visualización implementado y operativo
- Costo total de $28.430 CLP — viable para replicación institucional

**Limitaciones identificadas:**
- El sistema está optimizado para flujo en fila simple — no discrimina cruces simultáneos
- La dependencia de red WiFi estable para sincronización con Sheets puede ser un punto de falla en redes congestionadas
- El aforo de 295 personas no fue validado con conteo manual de asientos reales

**Recomendaciones para mejoras futuras:**
- Agregar sensor infrarrojo de barrera como redundancia para condiciones de baja iluminación
- Implementar integración con Telegram o app móvil para consulta remota sin estar en la red local
- Validar el aforo real de la Biblioteca F con conteo físico de asientos

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
