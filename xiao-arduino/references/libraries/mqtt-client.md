# PubSubClient (MQTT) Library for XIAO ESP32

## Overview

MQTT (Message Queuing Telemetry Transport) for IoT communication.

## Installation

Arduino IDE: Sketch > Include Library > Manage Libraries > Search "PubSubClient"

## Basic MQTT Publisher

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "your-SSID";
const char* password = "your-PASSWORD";
const char* mqtt_server = "broker.hivemq.com";

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
    Serial.begin(115200);

    // Connect WiFi
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    // Configure MQTT
    client.setServer(mqtt_server, 1883);
}

void reconnect() {
    while (!client.connected()) {
        Serial.print("Attempting MQTT connection...");
        if (client.connect("XIAO_Client")) {
            Serial.println("connected");
        } else {
            Serial.print("failed, rc=");
            Serial.print(client.state());
            delay(5000);
        }
    }
}

void loop() {
    if (!client.connected()) {
        reconnect();
    }
    client.loop();

    // Publish
    client.publish("xiao/sensor", "Hello from XIAO!");
    delay(5000);
}
```

## MQTT Subscriber

```cpp
void callback(char* topic, byte* payload, unsigned int length) {
    Serial.print("Message arrived [");
    Serial.print(topic);
    Serial.print("]: ");
    for (int i = 0; i < length; i++) {
        Serial.print((char)payload[i]);
    }
    Serial.println();
}

void setup() {
    // ... WiFi and MQTT setup

    client.setCallback(callback);
    client.connect();
    client.subscribe("xiao/command");
}

void loop() {
    client.loop();
}
```

## JSON Payload

```cpp
#include <ArduinoJson.h>

void publishSensorData(float temp, float hum) {
    StaticJsonDocument<200> doc;
    doc["temperature"] = temp;
    doc["humidity"] = hum;
    doc["device"] = "XIAO_ESP32C3";

    String jsonStr;
    serializeJson(doc, jsonStr);

    client.publish("xiao/sensor", jsonStr.c_str());
}
```

## QoS Levels

```cpp
// QoS 0: At most once
client.publish("xiao/data", "message", false);

// QoS 1: At least once
client.publish("xiao/data", "message", true);

// QoS 2: Exactly once
// Not supported by PubSubClient library
```

## Last Will Testament

```cpp
#define LWT_TOPIC "xiao/status"
#define LWT_MESSAGE "offline"
#define LWT_QOS 1
#define LWT_RETAIN true

void setup() {
    // ... connect WiFi
    client.setServer(mqtt_server, 1883);
    client.setCallback(callback);

    // Connect with LWT
    if (client.connect("XIAO_Client",
                       "username",      // Optional username
                       "password",      // Optional password
                       LWT_TOPIC,
                       LWT_MESSAGE,
                       LWT_QOS,
                       LWT_RETAIN)) {
        Serial.println("MQTT connected");
    }
}
```

## Secure MQTT (TLS)

```cpp
#include <WiFiClientSecure.h>
#include <PubSubClient.h>

const char* mqtt_server = "broker.hivemq.com";
const int mqtt_port = 8883;

WiFiClientSecure espClient;
PubSubClient client(espClient);

void setup() {
    // ... connect WiFi

    // Skip certificate validation (testing only!)
    espClient.setInsecure();

    client.setServer(mqtt_server, mqtt_port);
    client.connect("XIAO_Client");
}
```

## MQTT over WebSocket

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>
#include <PubSubClient.h>

WebSocketsClient webSocket;
WiFiClient client;

PubSubClient mqtt(client);

void webSocketEvent(WStype_t type, uint8_t * payload, size_t length) {
    // Handle WebSocket events
}

void setup() {
    // ... connect WiFi

    webSocket.begin("mqttbroker.com", 9000, "/", "mqtt");
    webSocket.onEvent(webSocketEvent);
    webSocket.setReconnectInterval(5000);

    // Configure MQTT for WebSocket
    mqtt.setCallback(callback);
}
```

## Multiple Topics

```cpp
const char* topics[] = {
    "xiao/sensor/temperature",
    "xiao/sensor/humidity",
    "xiao/command"
};

void setup() {
    // ... connect

    // Subscribe to multiple topics
    for (int i = 0; i < 3; i++) {
        client.subscribe(topics[i]);
    }
}
```

## Keep Alive

```cpp
#define MQTT_KEEP_ALIVE 60

void setup() {
    // ... connect
    client.connect("XIAO_Client", 0, 0, NULL, NULL, 0, MQTT_KEEP_ALIVE);
}
```

## Debug Connection State

```cpp
void printState() {
    int state = client.state();
    switch (state) {
        case MQTT_CONNECTION_TIMEOUT:
            Serial.println("MQTT_CONNECTION_TIMEOUT");
            break;
        case MQTT_CONNECTION_LOST:
            Serial.println("MQTT_CONNECTION_LOST");
            break;
        case MQTT_CONNECT_FAILED:
            Serial.println("MQTT_CONNECT_FAILED");
            break;
        case MQTT_CONNECTED:
            Serial.println("MQTT_CONNECTED");
            break;
        case MQTT_DISCONNECTED:
            Serial.println("MQTT_DISCONNECTED");
            break;
    }
}
```

## Buffer Size (Large Payloads)

```cpp
#define MQTT_MAX_PACKET_SIZE 256

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
    // ... setup
    client.setBufferSize(256);
    client.setServer(mqtt_server, 1883);
}
```

## Troubleshooting

### Can't connect

1. Check broker address and port
2. Verify WiFi is connected
3. Check firewall settings
4. Try different broker (public brokers sometimes down)

### Connection drops frequently

1. Increase keep-alive interval
2. Check WiFi signal strength
3. Implement reconnect logic

### Large payload fails

1. Increase buffer size: `client.setBufferSize(512)`
2. Check broker max message size
3. Split large messages

### Memory issues

1. Reduce buffer size
2. Free JSON documents after use
3. Check available heap: `Serial.println(ESP.getFreeHeap())`
