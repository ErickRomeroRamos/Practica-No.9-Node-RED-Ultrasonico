# Practica-No.9-Node-RED-Ultrasonico
# Introducción
## Descripción
### La ESP32 se utiliza en un entorno de adquisición de datos, por lo cual en esta práctica utilizaremos un sensor DHT22 para obtener el registro de temperatura y humedad y el sensor Ultrasonico para el registro de distancia. Dichos registros serán mostrados de manera visual en Node-RED. Para esta practica se hace uso del programa Node-RED y de un simulador llamado WOKWI.

## Material necesario
## Para realizar esta práctica necesitas lo siguiente

WOKWI

Tarjeta ESP32

sensor DHT11

Sensor Ultrasonico

Programa Node-RED

## Instrucciones
## AGREGAR UN NUEVA FUNCION LA DE DISTACIA
![image](https://github.com/ErickRomeroRamos/Practica-No.9-Node-RED-Ultrasonico/assets/153964793/03d4a5be-2451-4ee2-83b3-0e0bfb5f7ec8)

## CONFUGURAMOS LA NUEVA FUNCION
![image](https://github.com/ErickRomeroRamos/Practica-No.9-Node-RED-Ultrasonico/assets/153964793/0499cc29-cd15-43a4-b60c-9465b5e9d022)

## AGREGAMOS AL CODIGO LA FUNCION DE DISTANCIA EN WOKI

#include <ArduinoJson.h>

#include <WiFi.h>

#include <PubSubClient.h>

#define BUILTIN_LED 2

#include "DHTesp.h"

const int DHT_PIN = 15;
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 2;   //Pin digital 3 para el Echo del sensor
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "18.193.219.109";
String username_mqtt="erickke";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
long t; //timepo que demora en llegar el eco
long d; //distancia en centimetros

digitalWrite(Trigger, HIGH);
delayMicroseconds(10);          //Enviamos un pulso de 10us
digitalWrite(Trigger, LOW);
  
t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
d = t/59;             //escalamos el tiempo a una distancia en cm
  
Serial.print("Distancia: ");
Serial.print(d);      //Enviamos serialmente el valor de la distancia
Serial.print("cm");
Serial.println();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    doc["DISTANCIA"] = String(d);
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("ericktemp", output.c_str());
  }
}

## MODIFICAMOS LA SEÑAL PARA QUE SE PUEDA VISUALIZAR EN NOTE RED
![image](https://github.com/ErickRomeroRamos/Practica-No.9-Node-RED-Ultrasonico/assets/153964793/b5b17a83-1b2e-43d5-8814-686914052bde)

![image](https://github.com/ErickRomeroRamos/Practica-No.9-Node-RED-Ultrasonico/assets/153964793/99d05c15-7dd3-430b-a038-80d8263b0a2f)


![image](https://github.com/ErickRomeroRamos/Practica-No.9-Node-RED-Ultrasonico/assets/153964793/b206f7f1-76e8-424d-94b3-086f7ee6d17d)
