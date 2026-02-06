# WebSockets Library for XIAO ESP32

## Overview

WebSockets enable real-time bidirectional communication between XIAO ESP32 and WebSocket servers. Ideal for IoT dashboards, live monitoring, and interactive applications.

## Board Support

| Board | WebSockets Support |
|-------|-------------------|
| ESP32C3 | ✅ Full support |
| ESP32C5 | ✅ Full support |
| ESP32C6 | ✅ Full support |
| ESP32S3 | ✅ Full support |
| nRF52840 | ❌ Requires external WiFi module |
| RP2040 | ❌ Requires external WiFi module |

## Installation

```bash
# Install via Arduino Library Manager
# Library: "WebSockets" by Markus Sattler
```cpp
## Basic WebSocket Client

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>

const char* ssid = "your-ssid";
const char* password = "your-password";

WebSocketsClient webSocket;

void webSocketEvent(WStype_t type, uint8_t* payload, size_t length) {
    switch (type) {
        case WStype_DISCONNECTED:
            Serial.println("Disconnected");
            break;
        case WStype_CONNECTED:
            Serial.println("Connected");
            // Send message to server
            webSocket.sendTXT("Hello Server!");
            break;
        case WStype_TEXT:
            Serial.printf("Text: %s\n", payload);
            break;
        case WStype_BIN:
            Serial.printf("Binary: %u bytes\n", length);
            break;
    }
}

void setup() {
    Serial.begin(115200);

    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println(" connected");

    // Connect to WebSocket server
    webSocket.begin("192.168.1.100", 81, "/");  // Server IP, port, path
    webSocket.onEvent(webSocketEvent);
}

void loop() {
    webSocket.loop();
}
```cpp
## Secure WebSocket (WSS)

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>

WebSocketsClient webSocket;

void setup() {
    Serial.begin(115200);

    WiFi.begin("ssid", "password");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    // Secure WebSocket connection
    webSocket.beginSSL("echo.websocket.org", 443, "/");
    webSocket.onEvent([](WStype_t type, uint8_t* payload, size_t length) {
        if (type == WStype_TEXT) {
            Serial.printf("Received: %s\n", payload);
        }
    });
}

void loop() {
    webSocket.loop();
}
```cpp
## Sensor Data Streaming

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>

const char* ssid = "your-ssid";
const char* password = "your-password";

WebSocketsClient webSocket;
unsigned long lastSend = 0;

void setup() {
    Serial.begin(115200);

    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    webSocket.begin("192.168.1.100", 81, "/sensors");
    webSocket.onEvent([](WStype_t type, uint8_t* payload, size_t length) {
        if (type == WStype_CONNECTED) {
            Serial.println("WebSocket connected");
        }
    });
}

void loop() {
    webSocket.loop();

    // Send sensor data every second
    if (millis() - lastSend > 1000) {
        lastSend = millis();

        // Read sensors
        float temperature = random(200, 300) / 10.0;
        float humidity = random(400, 800) / 10.0;

        // Create JSON
        char json[100];
        snprintf(json, sizeof(json), "{\"temp\":%.1f,\"humidity\":%.1f}", temperature, humidity);

        // Send to server
        webSocket.sendTXT(json);
    }
}
```cpp
## Bidirectional Communication

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>

WebSocketsClient webSocket;

void webSocketEvent(WStype_t type, uint8_t* payload, size_t length) {
    switch (type) {
        case WStype_CONNECTED:
            Serial.println("Connected to server");
            webSocket.sendTXT("Hello from XIAO!");
            break;

        case WStype_TEXT:
            Serial.printf("Received: %s\n", payload);

            // Parse and respond
            if (strcmp((char*)payload, "status") == 0) {
                webSocket.sendTXT("Status: OK");
            } else if (strcmp((char*)payload, "ping") == 0) {
                webSocket.sendTXT("pong");
            }
            break;

        case WStype_DISCONNECTED:
            Serial.println("Disconnected");
            break;
    }
}

void setup() {
    Serial.begin(115200);

    WiFi.begin("ssid", "password");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    webSocket.begin("192.168.1.100", 81, "/");
    webSocket.onEvent(webSocketEvent);
}

void loop() {
    webSocket.loop();
}
```cpp
## Binary Data Transfer

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>

WebSocketsClient webSocket;

void setup() {
    Serial.begin(115200);

    WiFi.begin("ssid", "password");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    webSocket.begin("192.168.1.100", 81, "/binary");
    webSocket.onEvent([](WStype_t type, uint8_t* payload, size_t length) {
        if (type == WStype_BIN) {
            Serial.printf("Received %u bytes\n", length);

            // Process binary data
            for (size_t i = 0; i < length; i++) {
                Serial.printf("%02X ", payload[i]);
            }
            Serial.println();
        }
    });
}

void sendBinaryData() {
    // Create binary packet
    uint8_t data[10] = {0xAA, 0xBB, 0xCC, 0xDD, 0xEE};
    webSocket.sendBIN(data, 5);
}

void loop() {
    webSocket.loop();

    static unsigned long lastSend = 0;
    if (millis() - lastSend > 5000) {
        lastSend = millis();
        sendBinaryData();
    }
}
```cpp
## Auto-Reconnect

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>

WebSocketsClient webSocket;
bool connected = false;

void webSocketEvent(WStype_t type, uint8_t* payload, size_t length) {
    switch (type) {
        case WStype_CONNECTED:
            Serial.println("Connected");
            connected = true;
            break;
        case WStype_DISCONNECTED:
            Serial.println("Disconnected - reconnecting...");
            connected = false;
            break;
        case WStype_TEXT:
            Serial.printf("Message: %s\n", payload);
            break;
    }
}

void setup() {
    Serial.begin(115200);

    WiFi.begin("ssid", "password");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    webSocket.begin("192.168.1.100", 81, "/");
    webSocket.onEvent(webSocketEvent);

    // Enable automatic reconnect
    webSocket.setReconnectInterval(5000);  // 5 seconds
}

void loop() {
    webSocket.loop();

    // Send data only when connected
    static unsigned long lastSend = 0;
    if (connected && millis() - lastSend > 1000) {
        lastSend = millis();
        webSocket.sendTXT("Heartbeat");
    }
}
```cpp
## HTTP Authentication

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>

WebSocketsClient webSocket;

void setup() {
    Serial.begin(115200);

    WiFi.begin("ssid", "password");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    webSocket.begin("192.168.1.100", 81, "/");

    // Set authentication
    webSocket.setAuthorization("username", "password");

    webSocket.onEvent([](WStype_t type, uint8_t* payload, size_t length) {
        if (type == WStype_CONNECTED) {
            Serial.println("Authenticated and connected");
        }
    });
}

void loop() {
    webSocket.loop();
}
```cpp
## Header Customization

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>

WebSocketsClient webSocket;

void setup() {
    Serial.begin(115200);

    WiFi.begin("ssid", "password");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    webSocket.begin("192.168.1.100", 81, "/");

    // Set custom headers
    webSocket.setExtraHeaders("X-Custom-Header: MyValue\r\nUser-Agent: XIAO-ESP32");

    webSocket.onEvent([](WStype_t type, uint8_t* payload, size_t length) {
        if (type == WStype_CONNECTED) {
            Serial.println("Connected with custom headers");
        }
    });
}

void loop() {
    webSocket.loop();
}
```cpp
## Best Practices

```cpp
#include <WiFi.h>
#include <WebSocketsClient.h>

WebSocketsClient webSocket;

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    // Best practice 1: Use reasonable heartbeat interval
    webSocket.begin("192.168.1.100", 81, "/");
    webSocket.setReconnectInterval(5000);

    // Best practice 2: Enable ping/pong for connection health
    webSocket.enableHeartbeat(15000, 3000, 2);  // 15s ping, 3s timeout, 2 retries

    webSocket.onEvent([](WStype_t type, uint8_t* payload, size_t length) {
        if (type == WStype_TEXT) {
            // Best practice 3: Validate incoming messages
            if (length > 0 && length < 1024) {  // Reasonable size
                Serial.printf("Valid message: %s\n", payload);
            }
        }
    });
}

// Best practice 4: Check connection before sending
void safeSend(const char* message) {
    if (webSocket.isConnected()) {
        webSocket.sendTXT(message);
    } else {
        Serial.println("Not connected, message not sent");
    }
}

// Best practice 5: Handle large messages in chunks
void sendLargeData(const uint8_t* data, size_t length) {
    const size_t CHUNK_SIZE = 128;
    for (size_t i = 0; i < length; i += CHUNK_SIZE) {
        size_t chunkSize = min(CHUNK_SIZE, length - i);
        webSocket.sendBIN((uint8_t*)&data[i], chunkSize);
        delay(10);  // Small delay between chunks
    }
}

void loop() {
    webSocket.loop();
}
```

## References

- [WebSockets Library](https://github.com/Links2004/arduinoWebSockets)
- [WebSocket Protocol RFC](https://tools.ietf.org/html/rfc6455)
