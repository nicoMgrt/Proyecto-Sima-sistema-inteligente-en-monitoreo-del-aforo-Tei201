// Estas son las herramientas o "diccionarios" que el equipo necesita para funcionar:
// Conectarse a internet, crear una página web, saber la hora y guardar datos sin que se borren.

#include <WiFi.h>
#include <WebServer.h>
#include <WiFiClientSecure.h>
#include <HTTPClient.h>
#include <Preferences.h>
#include <time.h>


// CONFIGURACIÓN

// Aquí se indica en qué pines (cables) conectamos los sensores de la puerta
#define SIG_A      4  // Sensor externo (lado pasillo)
#define SIG_B      5  // Sensor interno (lado biblioteca)

// Distancia en centímetros para considerar que alguien cruzó la puerta
#define UMBRAL     40

// Capacidad máxima de sillas en la biblioteca
#define AFORO_MAX  295

// Datos para que el equipo se conecte a un internet en específico
const char* ssid     = "Hermes my beloved";
const char* password = "HermesWaton";

// La dirección del Excel (Google Sheets) donde se anotarán los datos
const char* SHEETS_URL = "https://script.google.com/macros/s/AKfycbzaTBYKLCBcEypn2CnMuBvIoC5K5lWiQX7KSveCFqLumkzdJO3LunnjWyBDapQ87rtG/exec";


// VARIABLES GLOBALES

WebServer   server(80);   // Creamos el servidor para mostrar la página web
Preferences prefs;        // Herramienta para guardar datos como si fuera un pendrive interno

int  personas         = 0;     // El contador principal de cuánta gente hay adentro
bool activadoA        = false; // ¿Alguien pasó el sensor de afuera?
bool activadoB        = false; // ¿Alguien pasó el sensor de adentro?
bool secuenciaEntrada = false; // ¿Se está completando el paso hacia adentro?
bool secuenciaSalida  = false; // ¿Se está completando el paso hacia afuera?

unsigned long tiempoSecuencia    = 0; // Un cronómetro para saber cuánto tarda la persona en pasar
const unsigned long TIMEOUT_SENSOR = 2000; // Si alguien se queda parado más de 2 segundos, se cancela el conteo


// SISTEMA DE ENVÍO DE DATOS (Para que no se quede pegado)

// Se crea un "sobre de carta" virtual que contendrá la información a enviar a internet
struct EventoHTTP {
char tipo[10]; // Si fue una "ENTRADA" o "SALIDA"
  char ts[25];   // La fecha y hora exacta
  int  total;    // Cuántas personas quedan en total
};

QueueHandle_t colaHTTP;  // Esto es como una sala de espera para los sobres de cartas
TaskHandle_t  tareaHTTPHandle;


// GUARDADO SEGURO (para no perder conteo si se corta la luz)

// Función que anota en el "pendrive interno" el número actual de personas
void guardarContador() {
  prefs.putInt("personas", personas);
}

// Función que, al prender el equipo, revisa si había personas anotadas previamente
void cargarContador() {
  prefs.begin("contador", false); // Se abre el archivo
  personas = prefs.getInt("personas", 0); // Si no hay valor guardado, empieza en 0
  Serial.println("Contador recuperado: " + String(personas));
}


// REINICIO AUTOMÁTICO (para el inicio de un nuevo día)

// Función que se asegura de empezar en 0 si pasó mucho tiempo apagado
void verificarResetPorTiempo() {
  struct tm t;
  if (!getLocalTime(&t)) return; // Si no se tiene la hora de internet, no se puede hacer nada

// Revisamos a qué hora se apagó por última vez
  time_t ultimoApagado = (time_t)prefs.getLong("apagado", 0);
  time_t ahora;
  time(&ahora);

// Guardamos la hora actual como la nueva referencia
  prefs.putLong("apagado", (long)ahora);

  if (ultimoApagado == 0) return; // Si es primera vez que se enciende, no hay referencia

// Se calcula cuántas horas pasaron desde que se apagó
  double horas = difftime(ahora, ultimoApagado) / 3600.0;
  Serial.println("Tiempo apagada: " + String(horas, 1) + "h");

// Si estuvo apagado más de 2 horas (por ejemplo, toda la noche), el aforo vuelve a cero
  if (horas > 2.0) { // cambia 2.0 si necesitas más margen
    personas = 0;
    guardarContador();
    Serial.println("Reset por inactividad (" + String(horas, 1) + "h apagada)");
  }
}



// RELOJ INTERNO

// Función que devuelve la fecha y hora en formato de texto listo para leer
String getTimestamp() {
  struct tm t;
  if (!getLocalTime(&t)) return "SIN_HORA"; // Si no hay internet, avisa que no sabe la hora
  char buf[25];
  strftime(buf, sizeof(buf), "%Y-%m-%dT%H:%M:%S", &t); // Le da el formato Año-Mes-Día Hora:Minuto
  return String(buf);
}


// LECTURA DE LOS SENSORES

// Función que lanza el sonido del sensor y cuenta cuánto tarda en rebotar
long medirDistancia(int pin) {
  pinMode(pin, OUTPUT);      // Preparamos el sensor para activar el sonido
  digitalWrite(pin, LOW);
  delayMicroseconds(2);
  digitalWrite(pin, HIGH);   // Se lanza el sonido
  delayMicroseconds(10);
  digitalWrite(pin, LOW);    // Se apaga el sonido
  
  pinMode(pin, INPUT);       // Se prepara el sensor para escuchar el eco
  long dur = pulseIn(pin, HIGH, 30000); // Se cuenta cuánto demoró el eco
  
  if (dur == 0) return 999;  // Si no escuchó nada, se asume que no hay nadie (distancia súper larga)
  return dur * 0.034 / 2;    // Fórmula matemática para convertir el tiempo del sonido en centímetros
}


// ENVÍO DE DATOS A INTERNET (el "segundo cerebro")

// Esta es una tarea especial que corre en segundo plano para no interrumpir a los sensores
void tareaHTTP(void* param) {
  EventoHTTP ev;
  for (;;) { // Esto se repite por siempre
    // Revisa si hay "sobres" en la sala de espera
    if (xQueueReceive(colaHTTP, &ev, portMAX_DELAY) == pdTRUE) {
      if (WiFi.status() != WL_CONNECTED) {
        Serial.println("Error: Sin WiFi. No se pudo enviar el dato.");
        continue;
      }

      // Se prepara la conexión segura para hablar con Google
      WiFiClientSecure client;
      client.setInsecure();
      HTTPClient http;
      
      // Se arma el link exacto con la hora, qué pasó (entrada/salida) y cuánta gente hay
      String url = String(SHEETS_URL)
        + "?timestamp=" + String(ev.ts)
        + "&evento="    + String(ev.tipo)
        + "&personas="  + String(ev.total);
        
      http.begin(client, url); // Se llama a Google
      http.setFollowRedirects(HTTPC_STRICT_FOLLOW_REDIRECTS);
      http.setTimeout(8000);   // Se espera máximo 8 segundos para que responda
      int code = http.GET();   // Se recibe la confirmación
      http.end();
      
      Serial.print("Dato enviado a Sheets. Código de respuesta: ");
      Serial.println(code);
    }
  }
}

// Función auxiliar: Mete el sobre a la sala de espera para que el segundo cerebro lo suba a internet
void enviarASheets(const char* tipo) {
  EventoHTTP ev;
  strncpy(ev.tipo, tipo, sizeof(ev.tipo) - 1);
  strncpy(ev.ts, getTimestamp().c_str(), sizeof(ev.ts) - 1);
  ev.tipo[sizeof(ev.tipo) - 1] = '\0';
  ev.ts[sizeof(ev.ts) - 1] = '\0';
  ev.total = personas;
  
  // Si la sala de espera está llena, se descarta el mensaje
  if (xQueueSend(colaHTTP, &ev, 0) != pdTRUE) {
    Serial.println("Sala de espera llena, no se pudo procesar el evento.");
  }
}


// PÁGINA WEB — contador principal

// Función que dibuja la página que veremos en el celular o computador
void handleRoot() {
  // Se calcula el porcentaje de ocupación
  float  pct    = min((personas * 100.0f) / AFORO_MAX, 100.0f);
  
  // Se elige un color dependiendo de qué tan lleno esté
  // Verde si hay menos de la mitad, Naranja si va acercándose al límite, Rojo si está lleno
  String color  = pct < 50 ? "#4caf50" : pct < 80 ? "#ff9800" : "#f44336";
  String estado = pct < 50 ? "Espacio disponible" : pct < 80 ? "Casi lleno" : "Lleno";

  // Aquí se empieza a armar la estructura visual de la página web (usando lenguaje HTML)
  String html;
  html.reserve(2200);
  html += F("<!DOCTYPE html><html><head>"
            "<meta charset='UTF-8'>"
            "<meta name='viewport' content='width=device-width,initial-scale=1'>"
            "<meta http-equiv='refresh' content='3'>" // La página se actualiza sola cada 3 segundos
            "<title>Biblioteca</title>"
            "<style>"
            "body{margin:0;font-family:sans-serif;background:#f0f4f8;"
            "display:flex;align-items:center;justify-content:center;min-height:100vh;}"
            ".card{background:#fff;border-radius:20px;padding:40px 32px;"
            "max-width:360px;width:90%;box-shadow:0 8px 24px rgba(0,0,0,.1);text-align:center;}"
            "h1{margin:0 0 4px;font-size:20px;color:#555;font-weight:500;}"
            ".num{font-size:80px;font-weight:700;color:#222;line-height:1.1;}"
            ".den{font-size:36px;color:#bbb;font-weight:400;}"
            ".barra{background:#eee;border-radius:12px;height:22px;margin:20px 0;overflow:hidden;}"
            ".fill{height:100%;border-radius:12px;transition:width .4s;}"
            ".estado{font-size:20px;font-weight:600;margin:0;}"
            ".hora{margin-top:14px;font-size:12px;color:#ccc;}"
            "</style></head><body><div class='card'>"
            "<h1>Biblioteca</h1>");

  // Se agrega el número gigante de personas que hay actualmente vs el límite
  html += "<div class='num'>" + String(personas);
  html += "<span class='den'>/" + String(AFORO_MAX) + "</span></div>";

  // Se dibuja la barra de progreso pintada del color que calculamos antes
  html += "<div class='barra'><div class='fill' style='width:" + String(pct, 0) + "%;background:" + color + "'></div></div>";

  // Se escribe el mensaje final (Ej: "Espacio disponible" en verde) y la hora
  html += "<p class='estado' style='color:" + color + "'>" + estado + "</p>";
  html += "<p class='hora'>" + getTimestamp() + "</p>";

  html += "</div></body></html>";

  // Se le manda esta página lista a quien sea que haya entrado desde su navegador
  server.send(200, "text/html", html);
}



// CONFIGURACIÓN INICIAL (Ocurre solo 1 vez al encender)

void setup() {
  Serial.begin(115200); // Se prende la pantalla interna para leer mensajes de prueba
  delay(500);
  Serial.println("\n--- Iniciando ---");
  
  // Lo primero que se hace: recuperar el último conteo guardado
  cargarContador();
  
  // Se prepara la sala de espera para 20 sobres máximo
  colaHTTP = xQueueCreate(20, sizeof(EventoHTTP));
  
  // Se enciende el "segundo cerebro" (Core 0) para que se encargue exclusivamente del internet
  xTaskCreatePinnedToCore(tareaHTTP, "HTTP", 8192, NULL, 1, &tareaHTTPHandle, 0);

  // Nos tratamos de conectar al WiFi
  WiFi.begin(ssid, password);
  WiFi.setSleep(true); // Modo ahorro de energía
  Serial.print("Conectando a WiFi");
  unsigned long t0 = millis();
  
  // Esperamos hasta 15 segundos intentando conectar
  while (WiFi.status() != WL_CONNECTED && millis() - t0 < 15000) {
    delay(500);
    Serial.print(".");
  }

  // Si se logra conectar exitosamente:
  if (WiFi.status() == WL_CONNECTED) {
    Serial.println(" ¡Conectado! IP: " + WiFi.localIP().toString());
    
    // se le pregunta la hora a los servidores de Google (zona horaria de Chile GMT-4)
    configTime(-4 * 3600, 0, "pool.ntp.org", "time.google.com");
    struct tm ti;
    t0 = millis();
    while (!getLocalTime(&ti) && millis() - t0 < 5000) delay(300); // Se esperan 5 segundos para obtenerla
    
    Serial.println("Hora actual: " + getTimestamp());
    verificarResetPorTiempo(); // Se ve si hay que reiniciar la cuenta a cero
    
  } else {
    // Si no hay internet, el sistema sigue funcionando pero solo mostrará luces/mensajes locales
    Serial.println(" Sin WiFi — Funcionando en modo offline");
  }

  // Se le dice al equipo: "Cuando alguien entre a tu dirección web, muéstrale la página visual"
  server.on("/", handleRoot);
  server.begin(); // Se enciende el servidor web

  Serial.println("Sistema completamente listo\n");
}


// CICLO PRINCIPAL (el equipo repite esto siempre)

void loop() {
  server.handleClient(); // Mantiene activa la página web para quien la esté mirando

  // Se disparan ambos sensores para ver a qué distancia hay un objeto
  long distA = medirDistancia(SIG_A);
  long distB = medirDistancia(SIG_B);

  // Se ve si hay alguien bloqueando algún sensor (la distancia es menor a nuestro límite)
  bool detectaA = (distA > 0 && distA < UMBRAL);
  bool detectaB = (distB > 0 && distB < UMBRAL);

  // LÓGICA DE ENTRADA: La persona primero pasa el sensor del pasillo (A), luego el de adentro (B)
  if (detectaA && !activadoB) {
    activadoA = true;
    tiempoSecuencia = millis(); // Empeiza el cronómetro
  }
  if (activadoA && detectaB) secuenciaEntrada = true; // Efectivamente está entrando

  // LÓGICA DE SALIDA: La persona primero pasa el sensor de adentro (B), luego el de la pasillo (A)
  if (detectaB && !activadoA) {
    activadoB = true;
    tiempoSecuencia = millis(); // Inicia el cronómetro
  }
  if (activadoB && detectaA) secuenciaSalida = true; // Efectivamente está saliendo

  // CONFIRMAR ENTRADA: La persona ya pasó ambos sensores y completó el cruce
  if (secuenciaEntrada && !detectaA && !detectaB) {
    if (personas < AFORO_MAX) { // Se suman siempre y cuando la biblioteca no esté colapsada
      personas++;
      guardarContador(); // Se anota el cambio en el pendrive interno
      Serial.println("¡Alguien entró! Total: " + String(personas));
      enviarASheets("ENTRADA"); // Se avisa a internet
    }
    // Se reinician los sensores para la siguiente persona
    activadoA = activadoB = secuenciaEntrada = secuenciaSalida = false;
  }

  // CONFIRMAR SALIDA: La persona ya pasó ambos sensores saliendo del lugar
  if (secuenciaSalida && !detectaA && !detectaB) {
    if (personas > 0) { // Se resta solo si hay gente adentro (no puede haber personas negativas)
      personas--;
      guardarContador(); // Se anota el cambio
      Serial.println("¡Alguien salió! Total: " + String(personas));
      enviarASheets("SALIDA"); // Se avisa a internet
    }
    // Se reinician los sensores para la siguiente persona
    activadoA = activadoB = secuenciaEntrada = secuenciaSalida = false;
  }

  // SISTEMA ANTI-BLOQUEO: Si alguien se queda parado frente a un sensor por mucho rato, se cancela todo
  if ((activadoA || activadoB) && millis() - tiempoSecuencia > TIMEOUT_SENSOR) {
    activadoA = activadoB = secuenciaEntrada = secuenciaSalida = false;
    Serial.println("Alguien se quedó bloqueando la puerta. Conteo cancelado.");
  }

  // PAUSA DE JUSTIFICACIÓN DE MUESTREO:
  // Se hace que el equipo parpadee 10 veces por segundo (100 milisegundos).
  // Es la velocidad perfecta: detecta gente caminando normal, pero evita que los 
  // sonidos ultrasónicos choquen entre sí y formen ruido o se sature el procesador.
  delay(100);
}
