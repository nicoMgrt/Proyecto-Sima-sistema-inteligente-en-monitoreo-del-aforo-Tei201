# Documentación de Software

## Contenido

### Diagramas de Flujo

Se van a exportar en otra parte los diagramas de flujo, los cuales son tres:
- Lectura de sensores y lógica (ciclo principal)
- Envío a Internet (tarea en segundo plano)
- Interfaz visual (servidor web)

### Documentación de Funciones

#### Funciones Principales

**medirDistancia()**
- **Propósito: Lanza el sonido del sensor ultrasónico y cuenta cuánto tarda en rebotar para calcular la distancia. Convierte el tiempo del sonido en centímetros usando una fórmula matemática.**
- **Parámetros: pin (int) - Especifica el pin del sensor a leer (por ejemplo, SIG_A o SIG_B).  Retorna: long - La distancia calculada en centímetros. Si el eco no devuelve señal (0), asume que no hay nadie y retorna 999.**
- **Ejemplo de uso:**
```cpp
// long distA = medirDistancia(SIG_A);
//long distB = medirDistancia(SIG_B);
```
**enviarASheets()**
- **Propósito: Mete un "sobre" de datos virtuales a la sala de espera (cola) para que el procesador secundario lo envíe a Google Sheets. Esto registra si el evento fue una entrada o salida junto con el aforo actual.**
- **Parámetros: tipo (const char) - Texto que indica el evento ocurrido, como "ENTRADA", "SALIDA" o "RESET".**
- **Retorna: void**
- **Ejemplo de uso:**
```cpp
//enviarASheets("ENTRADA");
```
**verificarResetPorTiempo()**
- **Propósito: Se asegura de reiniciar el contador de aforo a cero si el equipo detecta que pasó mucho tiempo apagado (por ejemplo, más de 2 horas) durante el inicio de un nuevo día.**
- **Parámetros: Ninguno.**
- **Retorna: void**
- **Ejemplo de uso:**
```cpp
verificarResetPorTiempo();
```

---

## Manual de Instalación

1. Conexión del Hardware: Conectar los sensores de la puerta en los pines indicados. El sensor externo (lado pasillo) debe conectarse al pin 4 (SIG_A) y el sensor interno (lado biblioteca) al pin 5 (SIG_B).
2. Configuración de Red: Modificar el código fuente para ingresar las credenciales del WiFi. Cambiar ssid a "Hermes my beloved" y la password a "HermesWaton".
3. Conexión a Base de Datos: Confirmar que la variable SHEETS_URL contenga la dirección correcta del Excel (Google Sheets) donde el equipo anotará los datos.
4. Carga del Software: Compilar y subir el código a la placa utilizando un entorno que soporte las librerías base, tales como WiFi.h, WebServer.h y Preferences.h.  

---

## Manual de Operación

### Encendido del Sistema
1. Energizar el dispositivo. El sistema iniciará automáticamente recuperando el último conteo de ocupación guardado en su "pendrive interno".

2. El sistema intentará conectarse a la red WiFi programada por un máximo de 15 segundos. Una vez conectado, solicitará la hora a los servidores de Google y mostrará la dirección IP asignada para acceder al panel visual. 

### Operación Normal
1. Abrir un navegador web e ingresar la dirección IP otorgada por el sistema para monitorear el porcentaje de ocupación, estado y las estadísticas de flujo del día en tiempo real.
   
2. El sistema funcionará de manera autónoma disparando ambos sensores. Detectará la secuencia de entrada o salida de las personas verificando el paso por los sensores A y B, actualizará la ocupación si la distancia detectada es menor a 40 cm y registrará los datos en internet. 

### Apagado del Sistema
1. Cortar el suministro eléctrico del equipo. No hay pérdida de datos, ya que el número actual de personas se guarda de forma segura en la memoria cada vez que alguien entra o sale.

2. Al volver a encenderse, el sistema verificará automáticamente si han pasado más de 2 horas inactivo; de ser así, restablecerá el recuento de aforo a 0 persona

---

## Troubleshooting

### Problemas Comunes

**Problema 1: No conecta a WiFiProblema 1: El WiFi se desconecta y el sistema no vuelve a subir datos**
- Causa probable: El router pierde señal o hay un microcorte. Como el código solo intenta conectarse a internet una única vez al encenderse (en la configuración inicial), si la red se cae durante el día, el equipo queda atrapado en "modo offline" de forma permanente sin intentar reconectarse.
- Solución: Agregar una validación constante dentro del ciclo de la tarea en segundo plano que verifique el estado del WiFi y, si detecta desconexión, vuelva a intentar conectarse sin detener la lectura de la puerta.

**Problema 2: Pérdida del registro de personas en Google Sheets**
- Causa probable: La "sala de espera" virtual de datos tiene un límite estricto de 20 eventos. Si hay una caída temporal de internet o Google tarda en responder, y cruzan más de 20 personas en ese lapso, la sala se llena y el sistema descarta los nuevos datos automáticamente.
- Solución: Aumentar el tamaño límite de la sala de espera en la configuración inicial, o programar el equipo para que guarde los eventos pendientes en su memoria interna hasta que recupere la conexión.

**Problema 3: Personas cruzan la puerta pero no son sumadas al aforo**
- Causa probable: El sistema anti-bloqueo cancela la medición si una persona tarda más de 2 segundos en cruzar ambos sensores. Si alguien camina lento, se detiene a mirar el celular o duda en el umbral, el tiempo expira y el cruce se ignora.
- Solución: Modificar la variable del límite de tiempo para darle un mayor margen a los caminantes lentos (por ejemplo, subirlo a 4 o 5 segundos).
