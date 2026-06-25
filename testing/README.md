# Testing y Validación — SIMA

## Objetivo
Validar el funcionamiento del sistema SIMA con condiciones reales de operación y medir el impacto cuantitativo del dispositivo en la problemática de gestión de aforo identificada en el Avance #1.

---

## Carpetas

#### `reportes/`
Protocolo de pruebas documentado y análisis de resultados técnicos

#### `evidencias/`
Registro fotográfico del prototipo instalado y operando en condiciones reales

#### `datos/`
Registros cuantitativos exportados de Google Sheets y resultados de pruebas de precisión

---

## Metodología de Testing

### Grupo Objetivo
- **Perfil:** Estudiantes de pregrado que utilizan la Biblioteca de Pregrado de la UAI en horario peak
- **Ubicación:** Biblioteca de Pregrado, Campus Peñalolén, Universidad Adolfo Ibáñez
- **Contexto de instalación:** Dispositivo instalado en el marco interior de la puerta principal, con Sensor A apuntando al pasillo exterior y Sensor B al interior del recinto

### Protocolo de Testing

1. **Verificación inicial del sistema** (5 min)
   - Confirmar conexión WiFi y sincronización NTP en Monitor Serial
   - Verificar que el dashboard web responde en la IP asignada
   - Confirmar que Google Sheets recibe eventos de prueba

2. **Pruebas de precisión direccional** (20 min)
   - Ejecutar 30 cruces controlados (15 entradas + 15 salidas) a velocidad normal de caminata
   - Registrar cada resultado como correcto o incorrecto
   - Verificar en Sheets que cada evento tiene timestamp correcto y tipo de evento correcto

3. **Pruebas de casos borde** (10 min)
   - Persona estacionada frente al sensor >2 segundos (debe activar timeout)
   - Corte de energía con contador en valor conocido (debe recuperarse)
   - Operación sin WiFi disponible (debe seguir contando localmente)

4. **Registro de evidencias** (5 min)
   - Captura del Monitor Serial mostrando detecciones en tiempo real
   - Captura del dashboard web durante operación
   - Captura de Google Sheets con datos acumulados

---

## Métricas de Evaluación

### Desempeño Técnico
- Tasa de precisión direccional: detecciones correctas / total de cruces (meta: ≥90%)
- Tasa de falsos positivos: eventos incorrectos por timeout (meta: 0 en 5 pruebas)
- Tasa de recuperación: contador recuperado correctamente tras corte de energía (meta: 100%)
- Latencia de envío a Sheets: tiempo entre evento y registro en la nube (referencia: ≤8 seg)

### Confiabilidad del Sistema
- Tiempo de operación continua sin reinicios espontáneos
- Comportamiento en modo offline (sin WiFi)
- Consistencia del dashboard web (actualización cada 3 segundos)

### Impacto ODS 11 — Ciudades y Comunidades Sostenibles

**Meta específica del proyecto:** Proveer información de ocupación en tiempo real que permita a los estudiantes de la UAI tomar decisiones de desplazamiento informadas antes de ir a la biblioteca, reduciendo el tiempo improductivo causado por la falta de información sobre aforo disponible.

**Indicadores de impacto:**

1. **Disponibilidad de información de aforo**
   - Baseline (Avance #1): 0% — ningún sistema de monitoreo existente
   - Meta: Sistema operativo con datos en tiempo real
   - Alcanzado: Dashboard web con % de ocupación + Google Sheets con histórico

2. **Tiempo de respuesta del sistema**
   - Baseline: Sin referencia (sistema inexistente)
   - Meta: Dashboard actualizado en ≤5 segundos tras un evento
   - Alcanzado: Actualización cada 3 segundos por meta-refresh del HTML

3. **Persistencia de datos**
   - Baseline: Sin almacenamiento
   - Meta: 100% de eventos almacenados en Google Sheets con timestamp
   - Alcanzado: Arquitectura FreeRTOS con cola de 20 eventos y reintento automático

**Proyección de impacto a escala:**
El 39.7% de los estudiantes de la UAI pierde entre 5 y más de 10 minutos buscando asiento en hora peak (dato del Avance #1, 69 encuestados). Con 8.000 estudiantes diarios en el campus, un despliegue completo de SIMA en los accesos principales permitiría eliminar ese costo de oportunidad temporal para aproximadamente 3.176 estudiantes por día. A $0 costo marginal por consulta, el sistema opera indefinidamente con una inversión inicial de $28.430 CLP por nodo.

---

## Análisis de Resultados

### Cuantitativo
- Tasa de precisión por tipo de cruce (entrada vs. salida)
- Comparativa de errores entre versión Alpha (v1) y versión Final (v4)
- Estadísticos del tiempo de respuesta HTTP (mín, máx, promedio)

### Cualitativo
- Comportamiento del sistema ante condiciones no previstas (objetos estáticos, cruces rápidos)
- Legibilidad del dashboard para usuarios sin contexto técnico
- Facilidad de instalación y configuración inicial

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
