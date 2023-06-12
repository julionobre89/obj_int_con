# obj_int_con
Repositório do código do trabalho de conclusão da disciplina: Objetos Inteligentes Interconectados.

**
 * @file ESP32_DHT22_Send_MQTT.ino
 * @author Saulo Aislan
 * @brief Firmware que ler o sensor de temperatura, envia os dados via MQTT.
 * @version 0.1
 * 
 * @copyright Copyright (c) 2022
 * 
*/

#include "EspMQTTClient.h"
#include "DHTesp.h"
EspMQTTClient client(
  "Wokwi-GUEST",         // SSID WiFi
  "",                    // Password WiFi
  "test.mosquitto.org",  // MQTT Broker
  "mqtt-wokwi",          // Client
  1883                   // MQTT port
);
const int DHT_PIN = 15;
DHTesp dhtSensor;
void setup()
{
  Serial.begin(115200);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  client.enableDebuggingMessages(); // Enable debugging messages sent to serial output
  client.enableHTTPWebUpdater(); // Enable the web updater. User and password default to values of MQTTUsername and MQTTPassword. These can be overridded with enableHTTPWebUpdater("user", "password").
  client.enableOTA(); // Enable OTA (Over The Air) updates. Password defaults to MQTTPassword. Port is the default OTA port. Can be overridden with enableOTA("password", port).
  client.enableLastWillMessage("TestClient/lastwill", "Vou ficar offline");
}
/**
 * @brief Ler os dados do Sensor imprime e envia via MQTT
 */
void lerEnviarDados() {
  TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  Serial.println("Temp: " + String(data.temperature, 2) + "°C");
  Serial.println("Humidade: " + String(data.humidity, 1) + "%");
  Serial.println("---");
  client.publish("topicowokwi/Temp", String(data.temperature, 2) + "°C"); 
  client.publish("topicowokwi/Humidade", String(data.humidity, 1) + "%");
}
/**
 * @brief Esta função é chamada quando tudo estiver conectado (Wifi e MQTT),
 * para utilizá-la deve-se implemtentar o struct EspMQTTClient
 */
void onConnectionEstablished()
{
  // Subscribe no "topicowokwi/msgRecebida/#" e mostra a mensagem recebida na Serial
    client.subscribe("topicowokwi/msgRecebida/#", [](const String & topic, const String & payload) {
    Serial.println("Mensagem recebida no topic: " + topic + ", payload: " + payload);
  });
  lerEnviarDados();
}
void loop()
{
  client.loop(); // Executa em loop
}
