# WiFi MQTT Sensor Example (Arduino)

## Complete IoT Sensor Project

This example shows a complete IoT sensor node with WiFi, MQTT, sensor reading, and data logging.

## Hardware

- XIAO ESP32C3
- BME280 sensor (I2C)
- LED indicator

## Wiring

```cpp
XIAO ESP32C3    BME280
-------------    ------
D0 (SDA)    -->  SDA
D1 (SCL)    -->  SCL
3.3V        -->  VCC
GND         -->  GND

LED: D2 (with 220 ohm resistor)
```

## Code

```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <Adafruit_BME280.h>
#include <ArduinoJson.h>

// WiFi credentials
const char* ssid = "your-SSID";
const char* password = "your-PASSWORD";

// MQTT broker
const char* mqtt_server = "broker.hivemq.com";
const int mqtt_port = 1883;

// Topics
const char* sensor_topic = "xiao/sensor/bme280";
const char* status_topic = "xiao/status";

// LED pin
const int LED_PIN = D2;

// Sensor
Adafruit_BME280 bme;
TwoWire I2CBME = TwoWire(0);

// WiFi client
WiFiClient espClient;
PubSubClient client(espClient);

// Timing
unsigned long lastSensorRead = 0;
const long sensorInterval = 5000;  // 5 seconds

unsigned long lastStatusSend = 0;
const long statusInterval = 60000;  // 60 seconds

// Sensor data
struct SensorData {
    float temperature;
    float humidity;
    float pressure;
    float altitude;
};

// Function prototypes
void setupWiFi();
void reconnectMQTT();
void readSensor(SensorData& data);
void publishSensor(const SensorData& data);
void publishStatus(const char* status);
void setLED(bool state);

void setup() {
    Serial.begin(115200);
    delay(1000);

    // Initialize LED
    pinMode(LED_PIN, OUTPUT);
    setLED(false);

    Serial.println("\n=== XIAO IoT Sensor ===");

    // Initialize sensor
    I2CBME.begin(D0, D1);
    if (!bme.begin(0x76, &I2CBME)) {
        Serial.println("Could not find BME280 sensor!");
        while (1) {
            setLED(true);
            delay(100);
            setLED(false);
            delay(100);
        }
    }
    Serial.println("BME280 sensor initialized");

    // Setup WiFi
    setupWiFi();

    // Setup MQTT
    client.setServer(mqtt_server, mqtt_port);

    // Initial status
    publishStatus("online");

    Serial.println("Setup complete!");
}

void loop() {
    // Reconnect if needed
    if (!client.connected()) {
        reconnectMQTT();
    }
    client.loop();

    unsigned long now = millis();

    // Read and publish sensor data
    if (now - lastSensorRead >= sensorInterval) {
        lastSensorRead = now;

        SensorData data;
        readSensor(data);
        publishSensor(data);

        // Blink LED to indicate activity
        setLED(true);
        delay(50);
        setLED(false);
    }

    // Send periodic status
    if (now - lastStatusSend >= statusInterval) {
        lastStatusSend = now;
        publishStatus("online");
    }
}

void setupWiFi() {
    Serial.print("Connecting to WiFi");

    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
        setLED(!digitalRead(LED_PIN));
    }

    Serial.println("\nWiFi connected!");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
    Serial.print("RSSI: ");
    Serial.println(WiFi.RSSI());

    setLED(false);
}

void reconnectMQTT() {
    while (!client.connected()) {
        Serial.print("Attempting MQTT connection...");

        // Create client ID
        String clientId = "XIAO-";
        clientId += String(random(0xffff), HEX);

        // Attempt connection
        if (client.connect(clientId.c_str())) {
            Serial.println("connected");
            publishStatus("online");
        } else {
            Serial.print("failed, rc=");
            Serial.print(client.state());
            Serial.println(" retrying in 5 seconds");

            // Fast blink error
            for (int i = 0; i < 5; i++) {
                setLED(true);
                delay(100);
                setLED(false);
                delay(100);
            }

            delay(5000);
        }
    }
}

void readSensor(SensorData& data) {
    data.temperature = bme.readTemperature();
    data.humidity = bme.readHumidity();
    data.pressure = bme.readPressure() / 100.0F;
    data.altitude = bme.readAltitude(1013.25);

    Serial.printf("Temp: %.2fC, Hum: %.2f%%, Press: %.2fhPa\n",
                  data.temperature, data.humidity, data.pressure);
}

void publishSensor(const SensorData& data) {
    // Create JSON document
    StaticJsonDocument<200> doc;
    doc["temperature"] = round(data.temperature * 10) / 10.0;
    doc["humidity"] = round(data.humidity * 10) / 10.0;
    doc["pressure"] = round(data.pressure * 10) / 10.0;
    doc["altitude"] = round(data.altitude * 10) / 10.0;
    doc["rssi"] = WiFi.RSSI();
    doc["device"] = "XIAO_ESP32C3";

    // Serialize to JSON string
    String jsonString;
    serializeJson(doc, jsonString);

    // Publish
    if (client.publish(sensor_topic, jsonString.c_str())) {
        Serial.println("Sensor data published");
    } else {
        Serial.println("Failed to publish sensor data");
    }
}

void publishStatus(const char* status) {
    StaticJsonDocument<100> doc;
    doc["status"] = status;
    doc["ip"] = WiFi.localIP().toString();
    doc["uptime"] = millis() / 1000;

    String jsonString;
    serializeJson(doc, jsonString);

    client.publish(status_topic, jsonString.c_str());
}

void setLED(bool state) {
    digitalWrite(LED_PIN, state ? HIGH : LOW);
}
```

## Features

1. **WiFi auto-reconnect** - Handles WiFi drops
2. **MQTT auto-reconnect** - Maintains connection
3. **JSON payload** - Structured sensor data
4. **Status updates** - Periodic health checks
5. **LED feedback** - Visual status indication
6. **RSSI monitoring** - Signal strength tracking
7. **Error handling** - Robust error recovery

## Serial Output

```cpp
=== XIAO IoT Sensor ===
BME280 sensor initialized
Connecting to WiFi.....
WiFi connected!
IP address: 192.168.1.100
RSSI: -45
Attempting MQTT connection...connected
Temp: 23.50C, Hum: 45.20%, Press: 1013.25hPa
Sensor data published
Temp: 23.60C, Hum: 45.30%, Press: 1013.20hPa
Sensor data published
```

## MQTT Payload

```json
{
  "temperature": 23.5,
  "humidity": 45.2,
  "pressure": 1013.2,
  "altitude": 100.5,
  "rssi": -45,
  "device": "XIAO_ESP32C3"
}
```cpp
## Customization

### Change sensor type

Replace BME280 with other sensors:
- DHT11/22: Simple temp/humidity
- BMP280: Temperature and pressure
- SHT30: More accurate temp/humidity

### Add SD card logging

```cpp
#include <SD.h>

File logFile;

void logToSD(const SensorData& data) {
    logFile = SD.open("/data.csv", FILE_WRITE);
    if (logFile) {
        logFile.printf("%lu,%.2f,%.2f,%.2f\n",
                       millis(), data.temperature,
                       data.humidity, data.pressure);
        logFile.close();
    }
}
```cpp
### Add deep sleep

```cpp
#include "esp_sleep.h"

#define SLEEP_DURATION 300e6  // 5 minutes

void loop() {
    // Read and publish
    SensorData data;
    readSensor(data);
    publishSensor(data);

    // Enter deep sleep
    Serial.println("Entering deep sleep...");
    esp_deep_sleep_start();
}
```

## Troubleshooting

### Can't connect to WiFi

1. Check SSID and password
2. Verify router is within range
3. Check for MAC filtering on router

### MQTT connection fails

1. Verify broker address and port
2. Check network connectivity
3. Try public broker: broker.hivemq.com

### Sensor returns NaN

1. Check I2C wiring
2. Verify I2C address (0x76 or 0x77)
3. Check sensor power supply

### Random reboots

1. Check power supply stability
2. Add 100uF capacitor
3. Monitor free heap: `Serial.println(ESP.getFreeHeap())`
