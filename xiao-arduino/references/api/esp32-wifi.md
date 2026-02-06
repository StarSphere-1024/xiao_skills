# ESP32 WiFi API (Arduino)

## Overview

ESP32 WiFi library provides WiFi connectivity for ESP32C3/C5/C6/S3 boards.

## Basic Connection

```cpp
#include <WiFi.h>

const char* ssid = "your-SSID";
const char* password = "your-PASSWORD";

void setup() {
    Serial.begin(115200);

    // Start WiFi
    WiFi.begin(ssid, password);

    // Wait for connection
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println("Connected!");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
}
```

## WiFi Mode

```cpp
// Set mode before WiFi.begin()
WiFi.mode(WIFI_STA);      // Station mode (client)
WiFi.mode(WIFI_AP);       // Access Point mode
WiFi.mode(WIFI_AP_STA);   // Both AP and STA
```

## Station Mode (Client)

### Connection with Timeout

```cpp
unsigned long start = millis();
while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    if (millis() - start > 10000) {  // 10 second timeout
        Serial.println("Connection timeout!");
        break;
    }
}
```

### Connection Status

```cpp
WiFi.status();  // Returns:
// WL_CONNECTED: Connected to network
// WL_NO_SSID_AVAIL: Configured SSID not available
// WL_IDLE_STATUS: Temporary status
// WL_DISCONNECTED: Not connected
```

### Auto-Reconnect

```cpp
WiFi.setAutoReconnect(true);  // Default: true
WiFi.setSleep(false);         // Disable WiFi sleep for stability
```

## Access Point Mode

```cpp
const char* apSSID = "XIAO_AP";
const char* apPassword = "12345678";

void setup() {
    Serial.begin(115200);

    // Start AP
    WiFi.softAP(apSSID, apPassword);
    Serial.print("AP IP address: ");
    Serial.println(WiFi.softAPIP());  // Usually 192.168.4.1
}

void loop() {
    // Check connected clients
    Serial.printf("Connected clients: %d\n", WiFi.softAPgetStationNum());
    delay(5000);
}
```

## WiFi Scanning

```cpp
void scanNetworks() {
    Serial.println("Scanning...");
    int n = WiFi.scanNetworks();

    Serial.println("Scan done!");
    if (n == 0) {
        Serial.println("No networks found");
    } else {
        Serial.print(n);
        Serial.println(" networks found:");
        for (int i = 0; i < n; ++i) {
            Serial.printf("%d: %s (%d dBm) %s\n",
                i,
                WiFi.SSID(i).c_str(),
                WiFi.RSSI(i),
                (WiFi.encryptionType(i) == WIFI_AUTH_OPEN) ? " " : "*");
        }
    }
    WiFi.scanDelete();
}
```

## WiFi Events

```cpp
#include <WiFi.h>

void WiFiEvent(WiFiEvent_t event) {
    switch(event) {
        case ARDUINO_EVENT_WIFI_STA_START:
            Serial.println("WiFi Started");
            break;
        case ARDUINO_EVENT_WIFI_STA_CONNECTED:
            Serial.println("WiFi Connected");
            break;
        case ARDUINO_EVENT_WIFI_STA_GOT_IP:
            Serial.print("Got IP: ");
            Serial.println(WiFi.localIP());
            break;
        case ARDUINO_EVENT_WIFI_STA_DISCONNECTED:
            Serial.println("WiFi Disconnected");
            break;
        default:
            break;
    }
}

void setup() {
    WiFi.onEvent(WiFiEvent);
    WiFi.begin(ssid, password);
}
```

## WiFi Configuration

### Static IP

```cpp
IPAddress local_IP(192, 168, 1, 100);
IPAddress gateway(192, 168, 1, 1);
IPAddress subnet(255, 255, 255, 0);
IPAddress dns(8, 8, 8, 8);

if (!WiFi.config(local_IP, gateway, subnet, dns)) {
    Serial.println("STA Failed to configure");
}
```

### Hostname

```cpp
WiFi.setHostname("xiao-esp32c3");  // Set mDNS hostname
```

### Power Settings

```cpp
// Set transmit power (10.5 - 20.5 dBm)
WiFi.setTxPower(10.5);  // Lower power = less range

// Enable low power mode
// WARNING: Can affect connection stability
WiFi.setSleep(true);  // Default: true
```

## WiFi Client (HTTP)

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

HTTPClient http;
WiFiClient client;

void setup() {
    // Connect to WiFi first...

    // Make HTTP request
    http.begin(client, "http://httpbin.org/get");
    int httpCode = http.GET();

    if (httpCode > 0) {
        String payload = http.getString();
        Serial.println(payload);
    }

    http.end();
}
```

## mDNS (Local DNS)

```cpp
#include <ESPmDNS.h>

void setup() {
    // Connect to WiFi...

    // Start mDNS
    if (MDNS.begin("xiao")) {
        Serial.println("mDNS responder started");
        // Access via: http://xiao.local
    }
}
```

## Network Info

```cpp
// IP address
Serial.println(WiFi.localIP());

// Subnet mask
Serial.println(WiFi.subnetMask());

// Gateway
Serial.println(WiFi.gatewayIP());

// DNS
Serial.println(WiFi.dnsIP());

// MAC address
Serial.println(WiFi.macAddress());

// BSSID (router MAC)
Serial.println(WiFi.BSSIDstr());

// RSSI (signal strength)
Serial.println(WiFi.RSSI());

// Network ID
Serial.println(WiFi.networkID());
```

## WiFi Multi (Multiple APs)

```cpp
#include <WiFiMulti.h>

WiFiMulti wifiMulti;

void setup() {
    // Add multiple networks
    wifiMulti.addAP("ssid1", "pass1");
    wifiMulti.addAP("ssid2", "pass2");
    wifiMulti.addAP("ssid3", "pass3");

    Serial.println("Connecting...");
    // Connect to strongest available
    while (wifiMulti.run() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println("Connected!");
}
```

## Best Practices

1. **Wait for connection**: Always check `WiFi.status() == WL_CONNECTED` before using network
2. **Handle disconnects**: Implement reconnection logic
3. **Disable WiFi when not needed**: `WiFi.mode(WIFI_OFF)` for battery
4. **Use WiFiMulti**: For multiple networks
5. **Set proper hostname**: For network identification
6. **Monitor RSSI**: Check signal strength for diagnostics

## Complete Example

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "your-SSID";
const char* password = "your-PASSWORD";

unsigned long lastCheck = 0;
const unsigned long checkInterval = 60000;  // 1 minute

void setup() {
    Serial.begin(115200);

    WiFi.setHostname("xiao-sensor");
    WiFi.setAutoReconnect(true);
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println(WiFi.localIP());
}

void loop() {
    // Periodic connection check
    if (millis() - lastCheck >= checkInterval) {
        lastCheck = millis();
        if (WiFi.status() != WL_CONNECTED) {
            Serial.println("Reconnecting...");
            WiFi.reconnect();
        }
        Serial.printf("RSSI: %d dBm\n", WiFi.RSSI());
    }

    // Your code here...
}
```
