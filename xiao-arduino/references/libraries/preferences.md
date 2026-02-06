# Preferences Library for XIAO ESP32

## Overview

The Preferences library provides NVS (Non-Volatile Storage) for ESP32-based XIAO boards. Like EEPROM but with better organization, wear leveling, and larger capacity.

## Board Support

| Board | Preferences Support |
|-------|-------------------|
| ESP32C3 | ✅ NVS in Flash |
| ESP32C5 | ✅ NVS in Flash |
| ESP32C6 | ✅ NVS in Flash |
| ESP32S3 | ✅ NVS in Flash |
| nRF52840 | ❌ Use EEPROM library |
| RP2040 | ❌ Use EEPROM library |
| SAMD21 | ❌ Use EEPROM library |

## Basic Usage

```cpp
#include <Preferences.h>

Preferences prefs;

void setup() {
    Serial.begin(115200);

    // Open namespace (like a folder)
    prefs.begin("my-app", false);  // false = read-write mode

    // Store integer
    prefs.putInt("counter", 42);

    // Read integer
    int value = prefs.getInt("counter", 0);  // 0 is default if not found
    Serial.printf("Counter: %d\n", value);

    // Close when done
    prefs.end();
}

void loop() {
}
```

## Data Types

```cpp
#include <Preferences.h>

Preferences prefs;

void setup() {
    Serial.begin(115200);
    prefs.begin("data-types", false);

    // Integer types
    prefs.putChar("byte", 127);        // 8-bit signed
    prefs.putUChar("ubyte", 255);      // 8-bit unsigned
    prefs.putShort("short", 32767);    // 16-bit signed
    prefs.putUShort("ushort", 65535);  // 16-bit unsigned
    prefs.putInt("int", 2147483647);   // 32-bit signed
    prefs.putUInt("uint", 4294967295); // 32-bit unsigned

    // Floating point
    prefs.putFloat("float", 3.14159);
    prefs.putDouble("double", 2.71828);

    // String (max 32 characters by default)
    prefs.putString("name", "XIAO ESP32");

    // Bytes (binary data)
    uint8_t binary[] = {0x01, 0x02, 0x03};
    prefs.putBytes("data", binary, sizeof(binary));

    // Read back
    Serial.printf("int: %d\n", prefs.getInt("int"));
    Serial.printf("float: %.2f\n", prefs.getFloat("float"));
    Serial.printf("string: %s\n", prefs.getString("name").c_str());

    prefs.end();
}

void loop() {
}
```

## Counter Example

```cpp
#include <Preferences.h>

Preferences prefs;

void setup() {
    Serial.begin(115200);
    prefs.begin("counter", false);

    // Get current count
    int count = prefs.getInt("bootCount", 0);
    count++;

    // Save new count
    prefs.putInt("bootCount", count);

    Serial.printf("Boot count: %d\n", count);
    prefs.end();
}

void loop() {
}
```

## WiFi Credentials Storage

```cpp
#include <Preferences.h>
#include <WiFi.h>

Preferences prefs;

void saveWiFiCredentials(const char* ssid, const char* password) {
    prefs.begin("wifi", false);
    prefs.putString("ssid", ssid);
    prefs.putString("password", password);
    prefs.end();
}

void loadWiFiCredentials(char* ssid, char* password) {
    prefs.begin("wifi", true);  // Read-only mode
    String s = prefs.getString("ssid", "");
    String p = prefs.getString("password", "");
    prefs.end();

    strcpy(ssid, s.c_str());
    strcpy(password, p.c_str());
}

void setup() {
    Serial.begin(115200);

    // Save credentials
    saveWiFiCredentials("MyNetwork", "MyPassword");

    // Load credentials
    char ssid[32];
    char password[64];
    loadWiFiCredentials(ssid, password);

    // Connect to WiFi
    WiFi.begin(ssid, password);
}

void loop() {
}
```

## Configuration Storage

```cpp
#include <Preferences.h>

Preferences prefs;

struct Config {
    int baudRate;
    int sensorInterval;
    bool enableLED;
    float threshold;
};

void saveConfig(const Config& config) {
    prefs.begin("config", false);
    prefs.putInt("baudRate", config.baudRate);
    prefs.putInt("sensorInterval", config.sensorInterval);
    prefs.putBool("enableLED", config.enableLED);
    prefs.putFloat("threshold", config.threshold);
    prefs.end();
}

void loadConfig(Config& config) {
    prefs.begin("config", true);
    config.baudRate = prefs.getInt("baudRate", 115200);
    config.sensorInterval = prefs.getInt("sensorInterval", 1000);
    config.enableLED = prefs.getBool("enableLED", true);
    config.threshold = prefs.getFloat("threshold", 25.0);
    prefs.end();
}

void setup() {
    Serial.begin(115200);

    // Save default config
    Config defaultConfig = {
        .baudRate = 115200,
        .sensorInterval = 1000,
        .enableLED = true,
        .threshold = 25.0
    };
    saveConfig(defaultConfig);

    // Load config
    Config config;
    loadConfig(config);

    Serial.printf("Baud rate: %d\n", config.baudRate);
    Serial.printf("Sensor interval: %d\n", config.sensorInterval);
}

void loop() {
}
```

## Clear All Data

```cpp
#include <Preferences.h>

Preferences prefs;

void clearNamespace(const char* namespaceName) {
    prefs.begin(namespaceName, false);
    prefs.clear();  // Remove all keys in namespace
    prefs.end();
}

void setup() {
    Serial.begin(115200);

    // Clear specific namespace
    clearNamespace("temp-data");

    Serial.println("Namespace cleared");
}

void loop() {
}
```

## Check Key Exists

```cpp
#include <Preferences.h>

Preferences prefs;

void setup() {
    Serial.begin(115200);
    prefs.begin("my-app", false);

    // Check if key exists
    if (prefs.isKey("myKey")) {
        Serial.println("Key exists");
        int value = prefs.getInt("myKey");
    } else {
        Serial.println("Key does not exist");
        prefs.putInt("myKey", 123);
    }

    prefs.end();
}

void loop() {
}
```

## Read-Only Mode

```cpp
#include <Preferences.h>

Preferences prefs;

void readConfig() {
    // Open in read-only mode
    prefs.begin("config", true);

    // Cannot write in read-only mode
    // prefs.putInt("value", 456);  // This will fail silently

    int value = prefs.getInt("value", -1);
    Serial.printf("Value: %d\n", value);

    prefs.end();
}

void setup() {
    Serial.begin(115200);

    // Write mode (default)
    prefs.begin("config", false);
    prefs.putInt("value", 123);
    prefs.end();

    // Read mode
    readConfig();
}

void loop() {
}
```

## Long String Storage

```cpp
#include <Preferences.h>

Preferences prefs;

void saveLongString(const char* key, const String& str) {
    prefs.begin("strings", false);

    // Split into chunks if needed
    const int chunkSize = 32;  // Max 32 bytes per chunk
    int chunks = (str.length() + chunkSize - 1) / chunkSize;

    prefs.putInt(key, chunks);  // Store number of chunks

    for (int i = 0; i < chunks; i++) {
        String chunkKey = String(key) + "_chunk" + i;
        int start = i * chunkSize;
        int len = min(chunkSize, (int)str.length() - start);
        prefs.putString(chunkKey.c_str(), str.substring(start, start + len));
    }

    prefs.end();
}

String readLongString(const char* key) {
    prefs.begin("strings", true);

    int chunks = prefs.getInt(key, 0);
    String result = "";

    for (int i = 0; i < chunks; i++) {
        String chunkKey = String(key) + "_chunk" + i;
        result += prefs.getString(chunkKey.c_str());
    }

    prefs.end();
    return result;
}

void setup() {
    Serial.begin(115200);

    String longText = "This is a very long string that exceeds the normal 32 character limit "
                      "of the Preferences library. We'll split it into chunks.";

    saveLongString("longText", longText);
    String retrieved = readLongString("longText");

    Serial.println(retrieved);
}

void loop() {
}
```

## Free Space Check

```cpp
#include <Preferences.h>

Preferences prefs;

void checkFreeSpace() {
    prefs.begin("my-app", false);

    // ESP32 NVS typically has ~50KB available
    // This varies by board configuration

    size_t used = prefs.putChar("test", 0);
    prefs.remove("test");

    Serial.printf("Used space: %u bytes\n", used);

    prefs.end();
}

void setup() {
    Serial.begin(115200);
    checkFreeSpace();
}

void loop() {
}
```

## Best Practices

```cpp
#include <Preferences.h>

void setup() {
    Serial.begin(115200);

    // Best practice 1: Use descriptive namespace names
    Preferences prefs;
    prefs.begin("xiao-config-v1", false);

    // Best practice 2: Use read-only mode when not writing
    prefs.end();  // Close write mode
    prefs.begin("xiao-config-v1", true);  // Reopen read-only

    // Best practice 3: Provide default values
    int value = prefs.getInt("setting", 100);  // 100 if not found

    // Best practice 4: Always end() when done
    prefs.end();

    // Best practice 5: Check before writing
    prefs.begin("settings", false);
    if (!prefs.isKey("initialized")) {
        // First boot, set defaults
        prefs.putBool("initialized", true);
        prefs.putInt("counter", 0);
    }
    prefs.end();

    // Best practice 6: Use appropriate data types
    prefs.begin("data", false);
    prefs.putBool("flag", true);        // Not putInt("flag", 1)
    prefs.putFloat("value", 1.23);      // Not putString("value", "1.23")
    prefs.end();
}

void loop() {
}
```

## References

- [ESP32 Preferences Documentation](https://docs.espressif.com/projects/arduino-esp32/en/latest/api/preferences.html)
- [ESP-IDF NVS Flash Storage](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/api-reference/storage/nvs_flash.html)
