#include <Wire.h>
#include <BH1750.h>
#include <WiFiNINA.h>
#include <ArduinoHttpClient.h>

// Wi-Fi credentials
const char ssid[] = "TelstraA6C2A3";
const char password[] = "bj35mguku9";

// IFTTT Webhook settings
const char serverAddress[] = "maker.ifttt.com";
const int port = 80;
const char eventNameStart[] = "sunlight_detected";
const char eventNameStop[] = "sunlight_stopped";
const char IFTTT_Key[] = "iHq8qYoXFuzo9W6CaRPTU-E84Uqb6FM-A0k79x27NP_";

// WiFi and HTTP client objects
WiFiClient wifi;
HttpClient client = HttpClient(wifi, serverAddress, port);

// BH1750 Light Sensor
BH1750 lightMeter;

// Light threshold in lux
const float threshold = 3000.0;
bool sunlightDetected = false;

void setup() {
  Serial.begin(9600);
  Wire.begin();

  // Connect to Wi-Fi
  Serial.print("Connecting to Wi-Fi...");
  while (WiFi.begin(ssid, password) != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("\nConnected to Wi-Fi!");

  // Initialize the BH1750 sensor
  if (lightMeter.begin()) {
    Serial.println("BH1750 initialized.");
  } else {
    Serial.println("Error initializing BH1750.");
    while (1);
  }
}

void sendIFTTTNotification(const char* eventName, float lux) {
  String url = String("/trigger/") + eventName + "/with/key/" + IFTTT_Key + "?value1=" + String(lux);
  client.get(url);

  int statusCode = client.responseStatusCode();
  String response = client.responseBody();

  Serial.print("Status Code: ");
  Serial.println(statusCode);
  Serial.print("Response: ");
  Serial.println(response);
}

void loop() {
  float lux = lightMeter.readLightLevel();
  Serial.print("Light Intensity: ");
  Serial.print(lux);
  Serial.println(" lx");

  if (lux > threshold && !sunlightDetected) {
    Serial.println("Sunlight detected! Sending email...");
    sendIFTTTNotification(eventNameStart, lux);
    sunlightDetected = true;
  } 
  else if (lux <= threshold && sunlightDetected) {
    Serial.println("Sunlight stopped! Sending email...");
    sendIFTTTNotification(eventNameStop, lux);
    sunlightDetected = false;
  }

  delay(10000);  // 10-second interval
}
