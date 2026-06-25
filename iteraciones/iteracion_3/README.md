# Iteración 3 — Versión Final SIMA

## Información General

**Fecha:** Junio 2026
**Versión:** v4.0 (versión final)
**Estado:** Prototipo funcional completo — listo para presentación
**Avance asociado:** Avance #3

---

## Descripción

Versión final del sistema SIMA, que incorpora todas las mejoras identificadas en las iteraciones anteriores. El sistema implementa el ciclo completo de datos: captura direccional con sensores ultrasónicos → procesamiento dual-core en ESP32-S3 → almacenamiento persistente en Google Sheets → visualización en dashboard web local y Google Looker Studio.

**Objetivos alcanzados:**
- Sistema IoT completamente funcional con precisión global del 91.4%
- Encapsulado diseñado en Fusion 360 con todos los componentes internos representados
- Ciclo completo captura → almacenamiento → visualización operativo
- Persistencia de datos ante cortes de energía (100% en 5 reinicios)
- Envío a Google Sheets con HTTP 200 OK confirmado y latencia de 1.5–3 segundos

---

## Cambios Respecto a v2 (Avance #2)

### Hardware

**Mejoras implementadas:**

1. **Cambio de GPIO1/GPIO10 a GPIO4/GPIO5 para los sensores**
   - Razón: GPIO1 es el TX del UART0 y GPIO10 está mapeado al controlador SPI interno. Usarlos para los sensores causaba un boot loop crítico que impedía el funcionamiento del sistema
   - Impacto: Sistema completamente estable — cero boot loops en todas las pruebas posteriores

2. **Uso exclusivo del puerto COM/UART para programación y operación**
   - Razón: El puerto USB nativo del ESP32-S3 resultó inestable al operar simultáneamente WiFi + HTTPS
   - Impacto: Eliminación de desconexiones espontáneas durante operación

3. **Delay(40ms) entre mediciones de ambos sensores**
   - Razón: Sin este delay, el eco de un sensor interfería con la lectura del otro (cross-talk ultrasónico)
   - Impacto: Reducción de lecturas erróneas por interferencia cruzada

### Software

**Nuevas funcionalidades:**

- **Arquitectura FreeRTOS dual-core:** Core 1 detecta personas de forma continua, Core 0 maneja exclusivamente el envío HTTP a Google Sheets. Elimina la latencia bloqueante de versiones anteriores
- **Persistencia NVS con Preferences:** El contador sobrevive cortes de energía y reinicios — recuperación en <2 segundos desde encendido
- **Reset automático por tiempo de inactividad:** Si el ESP32 estuvo apagado más de 2 horas, el contador vuelve a 0 automáticamente al encender (cubre cierre nocturno de la biblioteca)
- **Integración Google Sheets vía HTTPS:** Cada evento (ENTRADA/SALIDA/RESET) se registra con timestamp ISO 8601, tipo de evento y total de personas
- **Contadores de flujo diario:** El dashboard muestra entradas y salidas acumuladas del día además del aforo actual
- **URL secreta de reset para operadores:** `/admin-reset-biblioteca` permite reiniciar el contador sin exposición pública

**Optimizaciones finales:**

- Uso de macro `F()` para literales de texto estático en flash (reduce uso de RAM)
- `html.reserve(2500)` para pre-reservar memoria del heap y evitar fragmentación
- `WiFi.setSleep(true)` para modo modem sleep — autonomía de batería de ~16–20 horas
- Cola de 20 eventos para absorber picos de envío sin perder datos

**Manejo de errores:**
- Timeout de 2.000ms para sensores bloqueados — previene conteos fantasma
- Timeout de 8.000ms para respuesta de Google Sheets — evita bloqueos indefinidos
- Modo offline automático si no hay WiFi disponible — el sistema sigue contando localmente
- Retorno de valor 999 cuando `pulseIn` no detecta eco — evita distancia 0 como falso positivo

### Diseño Mecánico

**Encapsulado finalizado:**
- Modelado completo en Fusion 360 con gemelo digital de todos los componentes internos (ESP32-S3, sensores HC-SR04, batería NCR18650B, shield, interruptor)
- Orificios frontales calibrados para transductores de ⌀15.8mm de los sensores Seeed
- Orificio lateral para micro USB (carga de batería) e interruptor de encendido
- Tapa desmontable sin destruir el ensamble — diseño para reparación en campo
- Postes espaciadores M3 internos para separar la PCB del piso del encapsulado
- Material: PLA (impresión 3D FDM) — costo de filamento incluido en BOM

---

## Fotos del Prototipo Final

| Archivo | Descripción |
|---|---|
| `v3_prototipo_frontal.jpg` | Vista frontal del encapsulado con orificios de sensores visibles |
| `v3_prototipo_lateral.jpg` | Vista lateral mostrando orificio USB y switch de encendido |
| `v3_interior_componentes.jpg` | Vista con tapa removida mostrando componentes internos |
| `v3_instalado_puerta.jpg` | Dispositivo instalado en el marco de la puerta de la Biblioteca F |
| `v3_dashboard_captura.png` | Dashboard web mostrando porcentaje de ocupación en tiempo real |
| `v3_sheets_captura.png` | Google Sheets con eventos registrados y timestamps reales |

---

## Resultados Finales

### Funcionalidad Completa

✅ Detección direccional entrada/salida operativa (precisión 85–90%)
✅ Sistema opera sin intervención manual — ciclo automático a 10 Hz
✅ Conectividad WiFi estable con envío a Google Sheets (HTTP 200 OK)
✅ Dashboard web con actualización automática cada 3 segundos
✅ Persistencia de contador ante cortes de energía (5/5 reinicios ✓)
✅ Reset automático nocturno por tiempo de inactividad >2 horas
✅ Modo offline funcional — cuenta sin necesidad de red

### Resultados de Pruebas Técnicas

| Prueba | Casos | Correctos | Tasa de éxito |
|---|---|---|---|
| Detección entrada (A→B) | 10 | 9 | **90%** |
| Detección salida (B→A) | 10 | 8 | **80%** |
| Anti-bloqueo timeout | 5 | 5 | **100%** |
| Persistencia energía | 5 | 5 | **100%** |
| Envío a Google Sheets | 5 | 5 | **100%** |
| **TOTAL** | **35** | **32** | **91.4%** |

### Validación de Impacto ODS 11

**ODS Seleccionado:** ODS 11 — Ciudades y Comunidades Sostenibles

**Métricas de impacto:**

1. **Disponibilidad de información de aforo en tiempo real**
   - Baseline (sin SIMA): 0% — ningún sistema existente
   - Con SIMA: 100% para usuarios en la red local
   - Mejora: De 0 a información actualizada cada 3 segundos

2. **Persistencia y análisis histórico de datos**
   - Baseline: Sin almacenamiento de ningún tipo
   - Con SIMA: Registro ilimitado en Google Sheets con timestamp por evento
   - Mejora: Permite identificar patrones de ocupación por hora y día

3. **Autonomía del sistema**
   - Baseline: Sin referencia
   - Con SIMA: 16–20 horas continuas con batería NCR18650B en modo modem sleep
   - Mejora: Cubre una jornada completa de biblioteca sin recarga

**Proyección de impacto a escala:**
Con 8.000 estudiantes diarios en el campus y el 39.7% perdiendo 5–10 minutos buscando asiento en hora peak (dato Avance #1, 69 encuestados), un despliegue completo de SIMA podría eliminar ese costo de oportunidad temporal para ~3.176 estudiantes por día. Costo por nodo: $28.430 CLP. Costo marginal por consulta: $0.

---

## Comparación de las 3 Iteraciones

| Aspecto | v1 Alpha | v2 Beta | v4 Final | Mejora total |
|---|---|---|---|---|
| **GPIO sensores** | GPIO1/GPIO10 | GPIO1/GPIO10 | GPIO4/GPIO5 | ✅ Estable |
| **Conectividad** | AP propio | AP propio | Router + Sheets | ✅ Escalable |
| **Almacenamiento** | Sin persistencia | Sin persistencia | NVS + Sheets | ✅ Nuevo |
| **Envío a nube** | No | No | Google Sheets | ✅ Nuevo |
| **Latencia envío** | Bloqueante | Bloqueante | 1.5–3s (FreeRTOS) | ✅ No bloqueante |
| **Autonomía batería** | ~4h | ~4h | ~16–20h | ↑ +400% |
| **Tasa de error sensores** | Alta (GPIO conflictivos) | Alta | <10% | ↓ ~90% |
| **Reset automático nocturno** | No | No | Sí (>2h inactivo) | ✅ Nuevo |
| **Encapsulado** | Sin encapsulado | Boceto inicial | Fusion 360 completo | ✅ Completo |
| **Precisión global** | Sin medir | Sin medir | 91.4% (35 pruebas) | ✅ Validado |

---

## Fortalezas del Prototipo Final

1. **Arquitectura dual-core no bloqueante**
   - Evidencia: En pruebas simultáneas de detección y envío HTTP, el sistema mantuvo la detección a 10 Hz sin interrupciones durante envíos que tardaron hasta 3 segundos

2. **Persistencia total ante fallos de energía**
   - Evidencia: 5/5 reinicios forzados con recuperación correcta del contador en <2 segundos

3. **Costo de implementación bajo**
   - Evidencia: $28.430 CLP por nodo completo con autonomía de 16–20 horas — replicable a cualquier espacio de acceso controlado del campus

---

## Limitaciones Identificadas

1. **Flujo en fila simple**
   - Descripción: El sistema no discrimina correctamente cuando dos personas cruzan simultáneamente
   - Impacto: Posible subestimación del aforo real en entradas de alta demanda
   - Posible solución futura: Sensor infrarrojo de barrera completa o cámara de profundidad

2. **Dependencia de red WiFi**
   - Descripción: Sin WiFi, los datos no llegan a Google Sheets ni a Looker Studio
   - Impacto: En modo offline el dashboard local funciona, pero no hay histórico ni visualización remota
   - Posible solución futura: Módulo SIM (datos celulares) como respaldo de conectividad

3. **Aforo de 295 personas sin validación empírica**
   - Descripción: El valor fue estimado sin conteo físico de asientos reales
   - Impacto: El porcentaje mostrado puede no reflejar la ocupación real exacta
   - Posible solución futura: Conteo manual de asientos de la Biblioteca F para calibración

---

## Aprendizajes del Proceso Completo

### Técnicos
1. Los GPIO con funciones reservadas del chip (UART, SPI, USB) no deben usarse para sensores que cambien constantemente su dirección — verificar el pinout antes de conectar
2. Las llamadas HTTP síncronas bloquean el loop principal en sistemas IoT — FreeRTOS es la solución correcta para concurrencia en ESP32

### De Diseño
1. Iterar sobre fallas reales produce sistemas más robustos que diseñar sin probar — las tres iteraciones documentadas resolvieron problemas que no eran predecibles en papel
2. El diseño para la reparación (tapa desmontable, postes M3) debe considerarse desde el primer boceto, no añadirse al final

### De Trabajo en Equipo
1. Dividir el trabajo por componentes (hardware, software, diseño 3D, documentación) permite avanzar en paralelo sin bloquear al equipo
2. Documentar cada decisión técnica en tiempo real facilita la elaboración del FUENTES.md y evita reconstruir el proceso al final

### De Gestión de Proyecto
1. Las pruebas de hardware (GPIO, voltajes, compatibilidad) deben hacerse antes de desarrollar el software para no tener que reescribir firmware por problemas físicos
2. Reservar tiempo específico para documentación y GitHub — es tan importante como el desarrollo técnico y tiene peso en la evaluación

---

## Trabajo Futuro Recomendado

### Mejoras a Corto Plazo
- [ ] Validar aforo real de la Biblioteca F con conteo físico de asientos
- [ ] Integrar notificación por Telegram o WhatsApp para consulta remota sin estar en la red

### Mejoras a Mediano Plazo
- [ ] Agregar sensor infrarrojo de barrera como redundancia para condiciones de baja iluminación
- [ ] Implementar módulo SIM como respaldo de conectividad sin depender de WiFi

### Escalabilidad
- [ ] Despliegue en múltiples accesos del campus (casinos, salas de postgrado, gimnasio)
- [ ] Servidor central (Raspberry Pi o servidor UAI) que agregue datos de todos los nodos SIMA en un dashboard unificado
- [ ] Modelo de replicación a otras instituciones educativas con infraestructura similar

---

## Archivos en esta Carpeta

| Archivo | Descripción |
|---|---|
| `v3_codigo_final/` | Carpeta con firmware final comentado (main.ino) |
| `v3_esquema_final.pdf` | Esquemático definitivo del circuito |
| `v3_modelo3d_final.f3d` | Diseño 3D completo del encapsulado en Fusion 360 |
| `v3_fotos/` | Fotografías del prototipo final (mín. 3 ángulos) |
| `v3_protocolo_pruebas.pdf` | Reporte de testing con resultados reales |
| `v3_comparativa_versiones.xlsx` | Datos comparativos v1–v2–v4 Final |

---

*TEI201 — Taller de Diseño en Ingeniería · Universidad Adolfo Ibáñez · 2026*
*Proyecto SIMA — Nicolás Marinkovic · Bárbara Chaparro · Valentina Ramírez · Cristóbal Pérez*
