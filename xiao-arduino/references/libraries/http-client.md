# HTTPClient Library for XIAO ESP32

## Overview

HTTPClient simplifies HTTP requests for ESP32-based XIAO boards. Built on top of WiFiClient, it provides a cleaner API for REST API calls, web scraping, and cloud services.

## Board Support

| Board | HTTPClient Support |
|-------|-------------------|
| ESP32C3 | ✅ Full support |
| ESP32C5 | ✅ Full support |
| ESP32C6 | ✅ Full support |
| ESP32S3 | ✅ Full support |
| nRF52840 | ❌ No WiFi (use HTTP with BLE or external module) |
| RP2040 | ❌ Requires external WiFi module |
| SAMD21 | ❌ Requires external WiFi module |

## Basic HTTP GET

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "your-ssid";
const char* password = "your-password";
const char* serverUrl = "http://example.com/api/data";

HTTPClient http;

void setup() {
    Serial.begin(115200);

    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println(" connected");

    // Configure HTTP client
    http.begin(serverUrl);
    http.setTimeout(10000);  // 10 seconds

    // Send GET request
    int httpCode = http.GET();

    if (httpCode > 0) {
        Serial.printf("HTTP Code: %d\n", httpCode);

        if (httpCode == HTTP_CODE_OK) {
            String payload = http.getString();
            Serial.println(payload);
        }
    } else {
        Serial.printf("Error: %s\n", http.errorToString(httpCode).c_str());
    }

    http.end();
}

void loop() {
}
```

## HTTP POST with JSON

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

const char* ssid = "your-ssid";
const char* password = "your-password";
const char* serverUrl = "http://api.example.com/data";

HTTPClient http;

void postData(float temperature, float humidity) {
    http.begin(serverUrl);
    http.addHeader("Content-Type", "application/json");

    // Create JSON payload
    StaticJsonDocument<200> doc;
    doc["temperature"] = temperature;
    doc["humidity"] = humidity;
    doc["timestamp"] = millis();

    String jsonString;
    serializeJson(doc, jsonString);

    // Send POST request
    int httpCode = http.POST(jsonString);

    if (httpCode > 0) {
        Serial.printf("HTTP Code: %d\n", httpCode);

        if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_CREATED) {
            String response = http.getString();
            Serial.println("Response: " + response);
        }
    } else {
        Serial.printf("POST failed: %s\n", http.errorToString(httpCode).c_str());
    }

    http.end();
}

void setup() {
    Serial.begin(115200);

    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    postData(23.5, 65.2);
}

void loop() {
}
```

## HTTP Authentication

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

const char* serverUrl = "http://example.com/protected";

HTTPClient http;

void basicAuth() {
    http.begin(serverUrl);

    // Basic authentication
    http.setAuthorization("username", "password");

    int httpCode = http.GET();

    if (httpCode == HTTP_CODE_OK) {
        String response = http.getString();
        Serial.println(response);
    }

    http.end();
}

void bearerTokenAuth() {
    http.begin(serverUrl);

    // Bearer token authentication
    http.addHeader("Authorization", "Bearer YOUR_TOKEN_HERE");

    int httpCode = http.GET();

    if (httpCode == HTTP_CODE_OK) {
        String response = http.getString();
        Serial.println(response);
    }

    http.end();
}

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    basicAuth();
    bearerTokenAuth();
}

void loop() {
}
```

## HTTPS with Certificate Validation

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <WiFiClientSecure.h>

const char* ssid = "your-ssid";
const char* password = "your-password";
const char* serverUrl = "https://api.github.com";

HTTPClient http;
WiFiClientSecure client;

void setup() {
    Serial.begin(115200);

    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    // For production, set CA certificate
    // const char* root_ca = \
    // "-----BEGIN CERTIFICATE-----\n" \
    // "...certificate content...\n" \
    // "-----END CERTIFICATE-----\n";
    // client.setCACert(root_ca);

    // For development, can skip verification
    client.setInsecure();

    http.begin(client, serverUrl);
    http.addHeader("User-Agent", "XIAO-ESP32");

    int httpCode = http.GET();

    if (httpCode == HTTP_CODE_OK) {
        String response = http.getString();
        Serial.println(response);
    }

    http.end();
}

void loop() {
}
```

## HTTP Headers

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

HTTPClient http;

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    http.begin("http://example.com/api");

    // Add custom headers
    http.addHeader("Content-Type", "application/json");
    http.addHeader("Accept", "application/json");
    http.addHeader("User-Agent", "XIAO-ESP32/1.0");
    http.addHeader("X-Custom-Header", "CustomValue");

    int httpCode = http.GET();

    if (httpCode == HTTP_CODE_OK) {
        // Read response headers
        String headers = http.headers();
        Serial.printf("Headers: %s\n", headers.c_str());

        // Access specific header
        String contentType = http.header("Content-Type");
        Serial.printf("Content-Type: %s\n", contentType.c_str());

        String response = http.getString();
        Serial.println(response);
    }

    http.end();
}

void loop() {
}
```

## REST API Wrapper

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

class RESTClient {
private:
    HTTPClient http;
    String baseUrl;

public:
    RESTClient(const String& url) : baseUrl(url) {}

    String get(const String& path) {
        http.begin(baseUrl + path);
        int httpCode = http.GET();

        String response;
        if (httpCode == HTTP_CODE_OK) {
            response = http.getString();
        }
        http.end();
        return response;
    }

    String post(const String& path, const String& data) {
        http.begin(baseUrl + path);
        http.addHeader("Content-Type", "application/json");
        int httpCode = http.POST(data);

        String response;
        if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_CREATED) {
            response = http.getString();
        }
        http.end();
        return response;
    }

    String put(const String& path, const String& data) {
        http.begin(baseUrl + path);
        http.addHeader("Content-Type", "application/json");
        int httpCode = http.PUT(data);

        String response;
        if (httpCode == HTTP_CODE_OK) {
            response = http.getString();
        }
        http.end();
        return response;
    }

    String del(const String& path) {
        http.begin(baseUrl + path);
        int httpCode = http.sendRequest("DELETE");

        String response;
        if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_NO_CONTENT) {
            response = http.getString();
        }
        http.end();
        return response;
    }
};

RESTClient api("http://api.example.com");

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    // GET request
    String data = api.get("/users");
    Serial.println(data);

    // POST request
    String json = "{\"name\":\"XIAO\"}";
    String response = api.post("/users", json);
    Serial.println(response);
}

void loop() {
}
```

## Streaming Response

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

HTTPClient http;

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    http.begin("http://example.com/large-data.json");

    int httpCode = http.GET();

    if (httpCode == HTTP_CODE_OK) {
        // Get content length
        int len = http.getSize();
        WiFiClient* stream = http.getStreamPtr();

        // Read in chunks
        uint8_t buffer[128];
        int bytesRead = 0;

        while (http.connected() && (len > 0 || len == -1)) {
            int size = stream->available();

            if (size) {
                int c = stream->readBytes(buffer, ((size > sizeof(buffer)) ? sizeof(buffer) : size));
                bytesRead += c;

                // Process buffer here
                Serial.write(buffer, c);

                if (len > 0) {
                    len -= c;
                }
            }
            delay(1);
        }

        Serial.printf("\nTotal bytes read: %d\n", bytesRead);
    }

    http.end();
}

void loop() {
}
```

## Error Handling

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

HTTPClient http;

void makeRequest() {
    http.begin("http://example.com/api");

    int httpCode = http.GET();

    switch (httpCode) {
        case HTTP_CODE_OK:
            Serial.println("Success (200)");
            Serial.println(http.getString());
            break;
        case HTTP_CODE_MOVED_PERMANENTLY:
        case HTTP_CODE_FOUND:
            Serial.println("Redirect");
            break;
        case HTTP_CODE_UNAUTHORIZED:
            Serial.println("Unauthorized (401)");
            break;
        case HTTP_CODE_NOT_FOUND:
            Serial.println("Not Found (404)");
            break;
        case HTTP_CODE_SERVER_ERROR:
            Serial.println("Server Error (500)");
            break;
        case HTTP_CODE_SERVICE_UNAVAILABLE:
            Serial.println("Service Unavailable (503)");
            break;
        case -1:
            Serial.println("Connection failed");
            break;
        default:
            Serial.printf("Unknown error: %d\n", httpCode);
    }

    http.end();
}

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    makeRequest();
}

void loop() {
}
```

## Retry Logic

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

HTTPClient http;

bool makeRequestWithRetry(const String& url, int maxRetries = 3) {
    for (int i = 0; i < maxRetries; i++) {
        http.begin(url);
        int httpCode = http.GET();

        if (httpCode == HTTP_CODE_OK) {
            String response = http.getString();
            Serial.println(response);
            http.end();
            return true;
        } else {
            Serial.printf("Attempt %d failed: %d\n", i + 1, httpCode);
            http.end();
            delay(1000 * (i + 1));  // Exponential backoff
        }
    }
    Serial.println("All retries failed");
    return false;
}

void setup() {
    Serial.begin(115200);
    WiFi.begin("ssid", "password");

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    makeRequestWithRetry("http://example.com/api");
}

void loop() {
}
```

## References

- [ESP32 HTTPClient Documentation](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/httpclient.html)
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
