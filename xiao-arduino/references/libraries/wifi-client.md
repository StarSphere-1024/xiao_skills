# WiFiClient Library for XIAO ESP32

## Overview

WiFiClient provides TCP client connectivity for ESP32-based XIAO boards (ESP32C3, ESP32C5, ESP32C6, ESP32S3). Used for connecting to TCP servers, REST APIs, and custom protocols.

## Board Support

| Board | WiFi Support |
|-------|-------------|
| ESP32C3 | ✅ 2.4GHz 802.11 b/g/n |
| ESP32C5 | ✅ 2.4GHz 802.11 b/g/n |
| ESP32C6 | ✅ 2.4GHz 802.11 b/g/n |
| ESP32S3 | ✅ 2.4GHz 802.11 b/g/n |
| nRF52840 | ❌ No WiFi (use BLE) |
| RP2040 | ❌ No WiFi (requires external module) |
| SAMD21 | ❌ No WiFi (requires external module) |

## Basic TCP Connection

```cpp
#include <WiFi.h>

const char* ssid = "your-ssid";
const char* password = "your-password";
const char* host = "example.com";
const int port = 80;

WiFiClient client;

void setup() {
    Serial.begin(115200);

    // Connect to WiFi
    Serial.print("Connecting to WiFi");
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println(" connected");

    // Connect to server
    if (client.connect(host, port)) {
        Serial.println("Connected to server");

        // Send request
        client.println("GET / HTTP/1.1");
        client.print("Host: ");
        client.println(host);
        client.println("Connection: close");
        client.println();
    } else {
        Serial.println("Connection failed");
    }
}

void loop() {
    // Read response
    while (client.available()) {
        char c = client.read();
        Serial.write(c);
    }

    // Check if disconnected
    if (!client.connected()) {
        Serial.println();
        Serial.println("Server disconnected");
        client.stop();
        while(1);  // Stop
    }
}
```

## HTTP GET with WiFiClient

```cpp
#include <WiFi.h>

const char* ssid = "your-ssid";
const char* password = "your-password";
const char* server = "api.example.com";

WiFiClient client;

void httpGet(const char* path) {
    if (client.connect(server, 80)) {
        Serial.println("Connected to server");

        // Send HTTP GET request
        client.print("GET ");
        client.print(path);
        client.println(" HTTP/1.1");
        client.print("Host: ");
        client.println(server);
        client.println("Connection: close");
        client.println();

        // Read response
        while (client.connected()) {
            while (client.available()) {
                char c = client.read();
                Serial.write(c);
            }
        }

        client.stop();
    } else {
        Serial.println("Connection failed");
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

    httpGet("/data");
}

void loop() {
}
```

## HTTP POST with WiFiClient

```cpp
#include <WiFi.h>

const char* ssid = "your-ssid";
const char* password = "your-password";
const char* server = "api.example.com";

WiFiClient client;

void httpPost(const char* path, const char* data) {
    if (client.connect(server, 80)) {
        // Send HTTP POST request
        client.print("POST ");
        client.print(path);
        client.println(" HTTP/1.1");
        client.print("Host: ");
        client.println(server);
        client.println("Content-Type: application/json");
        client.print("Content-Length: ");
        client.println(strlen(data));
        client.println("Connection: close");
        client.println();
        client.println(data);

        // Read response
        while (client.connected() || client.available()) {
            if (client.available()) {
                char c = client.read();
                Serial.write(c);
            }
        }

        client.stop();
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

    const char* jsonData = "{\"sensor\":\"temperature\",\"value\":23.5}";
    httpPost("/api/data", jsonData);
}

void loop() {
}
```

## WiFiClient with Timeout

```cpp
#include <WiFi.h>

WiFiClient client;

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    // Set connection timeout
    client.setTimeout(5000);  // 5 seconds

    if (client.connect("example.com", 80)) {
        // Read with timeout
        String response = client.readStringUntil('\n');
        Serial.println(response);
        client.stop();
    }
}

void loop() {
}
```

## Persistent Connection

```cpp
#include <WiFi.h>

WiFiClient client;

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    if (client.connect("example.com", 80)) {
        client.println("GET / HTTP/1.1");
        client.println("Host: example.com");
        client.println("Connection: keep-alive");
        client.println();
    }
}

void loop() {
    // Keep connection alive
    if (client.connected()) {
        if (client.available()) {
            char c = client.read();
            Serial.write(c);
        }
    } else {
        // Reconnect if disconnected
        if (client.connect("example.com", 80)) {
            client.println("GET /stream HTTP/1.1");
            client.println("Host: example.com");
            client.println();
        }
    }
    delay(10);
}
```

## Secure Connection (WiFiClientSecure)

```cpp
#include <WiFi.h>
#include <WiFiClientSecure.h>

const char* ssid = "your-ssid";
const char* password = "your-password";
const char* server = "api.github.com";

WiFiClientSecure client;

void setup() {
    Serial.begin(115200);

    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println(" connected");

    // For self-signed certs, set fingerprint or CA cert
    // client.setCACert(root_ca);

    // Or skip verification (not recommended for production)
    // client.setInsecure();

    if (client.connect(server, 443)) {
        Serial.println("Connected to secure server");

        client.println("GET /users/octocat HTTP/1.1");
        client.println("Host: api.github.com");
        client.println("User-Agent: XIAO-ESP32");
        client.println("Connection: close");
        client.println();

        while (client.connected() || client.available()) {
            if (client.available()) {
                String line = client.readStringUntil('\n');
                Serial.println(line);
            }
        }

        client.stop();
    }
}

void loop() {
}
```

## Multiple Connections

```cpp
#include <WiFi.h>

WiFiClient client1;
WiFiClient client2;

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    // Connect to server 1
    if (client1.connect("server1.com", 80)) {
        client1.println("GET / HTTP/1.1");
        client1.println("Host: server1.com");
        client1.println();
    }

    // Connect to server 2
    if (client2.connect("server2.com", 80)) {
        client2.println("GET / HTTP/1.1");
        client2.println("Host: server2.com");
        client2.println();
    }
}

void loop() {
    // Read from both clients
    if (client1.available()) {
        Serial.write(client1.read());
    }
    if (client2.available()) {
        Serial.write(client2.read());
    }
}
```

## Best Practices

```cpp
#include <WiFi.h>

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }
}

// Best practice 1: Always check connection before sending
void safeSend(WiFiClient& client, const char* data) {
    if (client.connected()) {
        client.print(data);
    } else {
        Serial.println("Client not connected");
    }
}

// Best practice 2: Handle connection errors gracefully
void robustConnect(const char* host, int port) {
    WiFiClient client;

    for (int i = 0; i < 3; i++) {  // Retry 3 times
        if (client.connect(host, port)) {
            Serial.println("Connected");
            return;
        }
        Serial.println("Connection failed, retrying...");
        delay(1000);
    }
    Serial.println("Failed to connect after 3 attempts");
}

// Best practice 3: Clean up properly
void cleanup(WiFiClient& client) {
    if (client.connected()) {
        client.flush();
        client.stop();
    }
}

// Best practice 4: Use appropriate timeouts
void setTimeoutExample() {
    WiFiClient client;
    client.connect("example.com", 80);
    client.setTimeout(10000);  // 10 seconds for reads
}

void loop() {
}
```

## References

- [ESP32 WiFi Documentation](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/wifi.html)
- [Arduino Client Reference](https://www.arduino.cc/en/Reference/ClientConstructor)
