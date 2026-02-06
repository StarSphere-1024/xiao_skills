# Flash Storage for XIAO ESP32

## Overview

XIAO ESP32 boards provide multiple ways to permanently store data in flash memory. Data persists across resets and power failures, useful for storing configuration, WiFi credentials, or device state.

## Storage Options Comparison

| Method | Use Case | Size Limit | Write Cycles |
|--------|----------|------------|--------------|
| **Preferences** | Key-value pairs, settings | ~4000 bytes | ~100,000+ |
| **EEPROM** | Byte-level access, Arduino compatibility | ~512 bytes default | ~100,000 |
| **SPIFFS** | File system | ~1.5MB | ~10,000 |
| **SD Card** | Large files, data logging | GBs | Varies |

:::tip
For simple configuration (WiFi credentials, API keys), use **Preferences**. For file-like storage, use **SPIFFS** or SD card.
:::

---

## Preferences Library (Recommended)

The `Preferences.h` library uses key-value pairs and is the recommended method for storing configuration data.

### Board Support

| Board | Preferences Support |
|-------|-------------------|
| ESP32C3/C5/C6/S3 | ✅ |
| nRF52840 | ✅ (different API) |
| RP2040/RP2350 | ✅ (FlashIAP) |
| SAMD21 | ✅ (Flash emulation) |
| RA4M1 | ✅ (Flash emulation) |

### Basic Counter Example

```cpp
#include <Preferences.h>

Preferences preferences;

void setup() {
    Serial.begin(115200);
    delay(3000);
    Serial.println();

    // Open namespace "my-app" in read/write mode
    // Namespace limited to 15 characters
    preferences.begin("my-app", false);

    // Get counter value, default to 0 if not exists
    unsigned int counter = preferences.getUInt("counter", 0);

    // Increment counter
    counter++;

    // Print current value
    Serial.printf("Current counter value: %u\n", counter);

    // Save to flash
    preferences.putUInt("counter", counter);

    // Close preferences
    preferences.end();

    Serial.println("Restarting in 10 seconds...");
    delay(10000);
    ESP.restart();
}

void loop() {}
```

### Data Types

Preferences supports these data types:

| Type | Put Method | Get Method |
|------|-----------|-----------|
| Char | `putChar(key, int8_t)` | `getChar(key, default)` |
| UChar | `putUChar(key, uint8_t)` | `getUChar(key, default)` |
| Short | `putShort(key, int16_t)` | `getShort(key, default)` |
| Int | `putInt(key, int32_t)` | `getInt(key, default)` |
| UInt | `putUInt(key, uint32_t)` | `getUInt(key, default)` |
| Long | `putLong(key, int32_t)` | `getLong(key, default)` |
| Float | `putFloat(key, float)` | `getFloat(key, default)` |
| Double | `putDouble(key, double)` | `getDouble(key, default)` |
| Bool | `putBool(key, bool)` | `getBool(key, default)` |
| String | `putString(key, String)` | `getString(key, default)` |
| Bytes | `putBytes(key, void*, len)` | `getBytes(key, buf, maxLen)` |

### WiFi Credentials Storage

```cpp
#include <Preferences.h>
#include <WiFi.h>

Preferences preferences;

// Save credentials (run once)
void saveCredentials() {
    const char* ssid = "your-ssid";
    const char* password = "your-password";

    preferences.begin("credentials", false);
    preferences.putString("ssid", ssid);
    preferences.putString("password", password);
    preferences.end();

    Serial.println("Credentials saved!");
}

// Load and connect
void loadAndConnect() {
    String ssid, password;

    preferences.begin("credentials", true);  // Read-only mode
    ssid = preferences.getString("ssid", "");
    password = preferences.getString("password", "");
    preferences.end();

    if (ssid == "" || password == "") {
        Serial.println("No credentials saved");
        return;
    }

    Serial.print("Connecting to ");
    Serial.println(ssid);

    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid.c_str(), password.c_str());

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println(WiFi.localIP());
}

void setup() {
    Serial.begin(115200);
    delay(3000);

    // Uncomment to save new credentials
    // saveCredentials();

    loadAndConnect();
}

void loop() {}
```

### Multiple Namespaces

```cpp
#include <Preferences.h>

Preferences wifiPrefs;
Preferences configPrefs;
Preferences sensorPrefs;

void setup() {
    Serial.begin(115200);

    // WiFi namespace
    wifiPrefs.begin("wifi", false);
    wifiPrefs.putString("ssid", "MyNetwork");
    wifiPrefs.end();

    // Config namespace
    configPrefs.begin("config", false);
    configPrefs.putInt("interval", 1000);
    configPrefs.putBool("debug", true);
    configPrefs.end();

    // Sensor calibration namespace
    sensorPrefs.begin("sensors", false);
    sensorPrefs.putFloat("temp_offset", -2.5);
    sensorPrefs.putFloat("hum_offset", 5.0);
    sensorPrefs.end();

    Serial.println("All preferences saved");
}

void loop() {}
```

### Clear Preferences

```cpp
#include <Preferences.h>

Preferences preferences;

void clearAllPreferences() {
    preferences.begin("my-app", false);

    // Remove specific key
    preferences.remove("counter");

    // Or clear all keys in namespace
    preferences.clear();

    preferences.end();
    Serial.println("Preferences cleared");
}

// Complete reset (erases NVS partition)
#include <nvs_flash.h>

void factoryReset() {
    Serial.println("Factory reset...");
    nvs_flash_erase();  // Erase NVS partition
    nvs_flash_init();   // Reinitialize
    ESP.restart();
}

void setup() {
    Serial.begin(115200);

    // Uncomment to clear all preferences
    // clearAllPreferences();
    // factoryReset();
}

void loop() {}
```

---

## EEPROM Library

The EEPROM library provides Arduino-compatible byte-level access to flash memory.

:::caution
- **Write cycles limited to ~100,000**
- **Default size is 512 bytes**
- **Always call `commit()` after writing**
- **Minimize write frequency**
:::

### LED State Memory

```cpp
#include <EEPROM.h>

#define EEPROM_SIZE 1

const int ledPin = D10;
int ledState = LOW;
unsigned long previousMillis = 0;
const long interval = 10000;  // 10 seconds

void setup() {
    Serial.begin(115200);

    EEPROM.begin(EEPROM_SIZE);
    pinMode(ledPin, OUTPUT);

    // Read last LED state from flash
    ledState = EEPROM.read(0);
    digitalWrite(ledPin, ledState);

    Serial.printf("Restored LED state: %s\n",
                 ledState ? "ON" : "OFF");
}

void loop() {
    unsigned long currentMillis = millis();

    if (currentMillis - previousMillis >= interval) {
        previousMillis = currentMillis;

        // Toggle LED state
        ledState = (ledState == LOW) ? HIGH : LOW;

        // Save to flash
        EEPROM.write(0, ledState);
        EEPROM.commit();

        digitalWrite(ledPin, ledState);

        Serial.printf("State changed to: %s\n",
                     ledState ? "ON" : "OFF");
    }
}
```

### EEPROM with Multiple Variables

```cpp
#include <EEPROM.h>

#define EEPROM_SIZE 32

struct Settings {
    int counter;
    float temperature;
    bool enabled;
    char name[16];
};

void setup() {
    Serial.begin(115200);
    EEPROM.begin(EEPROM_SIZE);

    // Write settings
    Settings settings = {
        .counter = 100,
        .temperature = 23.5,
        .enabled = true,
        .name = "XIAO-ESP32"
    };

    // Write struct to EEPROM
    EEPROM.put(0, settings);
    EEPROM.commit();
    Serial.println("Settings saved");

    // Read settings
    Settings loaded;
    EEPROM.get(0, loaded);

    Serial.printf("Counter: %d\n", loaded.counter);
    Serial.printf("Temperature: %.1f\n", loaded.temperature);
    Serial.printf("Enabled: %s\n", loaded.enabled ? "Yes" : "No");
    Serial.printf("Name: %s\n", loaded.name);
}

void loop() {}
```

### EEPROM Update (Saves Cycles)

```cpp
#include <EEPROM.h>

#define EEPROM_SIZE 10

// EEPROM.update() only writes if value differs
// This extends flash life by avoiding unnecessary writes

void setup() {
    Serial.begin(115200);
    EEPROM.begin(EEPROM_SIZE);

    int value = 50;

    // This will write the first time
    EEPROM.update(0, value);
    EEPROM.commit();
    Serial.println("First write");

    // This won't write (same value)
    EEPROM.update(0, value);
    EEPROM.commit();
    Serial.println("No write needed (same value)");

    // This will write (different value)
    EEPROM.update(0, value + 1);
    EEPROM.commit();
    Serial.println("Write needed (value changed)");
}

void loop() {}
```

---

## Advanced Examples

### Sensor Calibration Storage

```cpp
#include <Preferences.h>

Preferences calibration;

struct SensorCalibration {
    float tempOffset;
    float humOffset;
    float pressureOffset;
    float lightGain;
};

void saveCalibration(SensorCalibration cal) {
    calibration.begin("calibration", false);
    calibration.putBytes("sensors", &cal, sizeof(cal));
    calibration.end();
    Serial.println("Calibration saved");
}

SensorCalibration loadCalibration() {
    SensorCalibration cal = {0, 0, 0, 1.0};  // Default values

    calibration.begin("calibration", true);
    calibration.getBytes("sensors", &cal, sizeof(cal));
    calibration.end();

    return cal;
}

void factoryCalibrate() {
    SensorCalibration cal = {0, 0, 0, 1.0};
    saveCalibration(cal);
    Serial.println("Factory calibration applied");
}

void setup() {
    Serial.begin(115200);

    // Load saved calibration
    SensorCalibration cal = loadCalibration();

    Serial.printf("Temp Offset: %.2f\n", cal.tempOffset);
    Serial.printf("Hum Offset: %.2f\n", cal.humOffset);
    Serial.printf("Pressure Offset: %.2f\n", cal.pressureOffset);
    Serial.printf("Light Gain: %.2f\n", cal.lightGain);

    // Uncomment to apply factory calibration
    // factoryCalibrate();
}

void loop() {}
```

### Multi-Language Settings

```cpp
#include <Preferences.h>

Preferences settings;

enum Language { ENGLISH, SPANISH, FRENCH, GERMAN };
enum UnitSystem { METRIC, IMPERIAL };

struct DeviceConfig {
    Language language;
    UnitSystem units;
    int brightness;
    bool notifications;
};

void saveConfig(DeviceConfig config) {
    settings.begin("config", false);
    settings.putInt("language", config.language);
    settings.putInt("units", config.units);
    settings.putInt("brightness", config.brightness);
    settings.putBool("notifications", config.notifications);
    settings.end();
}

DeviceConfig loadConfig() {
    DeviceConfig config = {
        ENGLISH,      // Default language
        METRIC,       // Default units
        128,          // Default brightness
        true          // Notifications on
    };

    settings.begin("config", true);
    config.language = (Language)settings.getInt("language", ENGLISH);
    config.units = (UnitSystem)settings.getInt("units", METRIC);
    config.brightness = settings.getInt("brightness", 128);
    config.notifications = settings.getBool("notifications", true);
    settings.end();

    return config;
}

void setup() {
    Serial.begin(115200);

    DeviceConfig config = loadConfig();

    Serial.printf("Language: %d\n", config.language);
    Serial.printf("Units: %d\n", config.units);
    Serial.printf("Brightness: %d\n", config.brightness);
    Serial.printf("Notifications: %s\n",
                 config.notifications ? "On" : "Off");
}

void loop() {}
```

### Data Logging with Rotation

```cpp
#include <Preferences.h>

Preferences logger;

#define MAX_READINGS 50

struct Reading {
    float value;
    unsigned long timestamp;
};

void logReading(float value) {
    logger.begin("datalog", false);

    // Get current index
    int index = logger.getInt("index", 0);

    // Store reading
    char key[16];
    snprintf(key, sizeof(key), "reading_%d", index);
    logger.putBytes(key, &value, sizeof(float));

    // Store timestamp
    snprintf(key, sizeof(key), "time_%d", index);
    unsigned long time = millis();
    logger.putULong(key, time);

    // Increment and wrap index
    index = (index + 1) % MAX_READINGS;
    logger.putInt("index", index);

    logger.end();
}

void printReadings() {
    logger.begin("datalog", true);
    int count = logger.getInt("index", 0);

    Serial.printf("--- Last %d Readings ---\n", MAX_READINGS);

    for (int i = 0; i < MAX_READINGS; i++) {
        char key[16];

        // Read value
        snprintf(key, sizeof(key), "reading_%d", i);
        float value;
        logger.getBytes(key, &value, sizeof(float));

        // Read timestamp
        snprintf(key, sizeof(key), "time_%d", i);
        unsigned long time = logger.getULong(key, 0);

        Serial.printf("[%d] %.2f @ %lu ms\n", i, value, time);
    }

    logger.end();
}

void clearLog() {
    logger.begin("datalog", false);
    logger.clear();
    logger.putInt("index", 0);
    logger.end();
    Serial.println("Log cleared");
}

void setup() {
    Serial.begin(115200);

    // Uncomment to clear log
    // clearLog();

    // Log some readings
    for (int i = 0; i < 10; i++) {
        logReading(20.0 + i * 0.5);
        delay(100);
    }

    printReadings();
}

void loop() {}
```

---

## Troubleshooting

### Preferences Not Saving

1. **Check namespace**: Ensure begin() is called with `false` (read/write mode)
2. **Check key length**: Keys limited to 15 characters
3. **Check value type**: Use matching put/get methods
4. **Check free space**: NVS partition may be full, try `nvs_flash_erase()`

### EEPROM Data Lost

1. **Missing commit()**: Always call `EEPROM.commit()` after writes
2. **Size mismatch**: Ensure `EEPROM.begin()` size matches data size
3. **Power loss during write**: Add capacitor or ensure stable power

### Best Practices

1. **Minimize writes**: Use `EEPROM.update()` or `preferences` instead of frequent writes
2. **Use defaults**: Always provide default values when reading
3. **Organize namespaces**: Group related settings in same namespace
4. **Handle errors**: Check return values from critical operations
5. **Version your config**: Store version number to handle migrations

```cpp
// Good example with error handling
#include <Preferences.h>

Preferences prefs;

bool saveWiFi(const char* ssid, const char* password) {
    prefs.begin("wifi", false);

    if (!prefs.putString("ssid", ssid)) {
        prefs.end();
        return false;
    }

    if (!prefs.putString("password", password)) {
        prefs.end();
        return false;
    }

    prefs.end();
    return true;
}

bool loadWiFi(String& ssid, String& password) {
    prefs.begin("wifi", true);

    ssid = prefs.getString("ssid", "");
    if (ssid == "") {
        prefs.end();
        return false;
    }

    password = prefs.getString("password", "");
    prefs.end();

    return password != "";
}
```

## References

- [ESP32 Preferences Library](https://github.com/espressif/arduino-esp32/tree/master/libraries/Preferences)
- [ESP32 EEPROM Library](https://github.com/espressif/arduino-esp32/tree/master/libraries/EEPROM)
- [Random Nerd Tutorials - ESP32 Flash Memory](https://randomnerdtutorials.com/esp32-flash-memory/)
