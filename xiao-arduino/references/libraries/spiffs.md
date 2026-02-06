# SPIFFS/LittleFS for XIAO

## Overview

SPIFFS (SPI Flash File System) and LittleFS provide file storage in XIAO's flash memory. Useful for logging, configuration files, web assets, and data caching.

## Board Support

| Board | SPIFFS | LittleFS |
|-------|--------|----------|
| ESP32C3 | ✅ | ✅ (Recommended) |
| ESP32C5 | ✅ | ✅ (Recommended) |
| ESP32C6 | ✅ | ✅ (Recommended) |
| ESP32S3 | ✅ | ✅ (Recommended) |
| nRF52840 | ⚠️ Limited | ✅ (Recommended) |
| RP2040 | ❌ | ✅ (Recommended) |
| SAMD21 | ❌ | ✅ (External flash) |

## LittleFS (Recommended for ESP32)

```cpp
#include <LittleFS.h>

void setup() {
    Serial.begin(115200);

    // Mount LittleFS
    if (!LittleFS.begin(true)) {  // true = format on failure
        Serial.println("LittleFS mount failed");
        return;
    }

    Serial.println("LittleFS mounted");

    // Write file
    File file = LittleFS.open("/test.txt", "w");
    if (file) {
        file.println("Hello from XIAO!");
        file.close();
        Serial.println("File written");
    }

    // Read file
    file = LittleFS.open("/test.txt", "r");
    if (file) {
        Serial.println("File content:");
        while (file.available()) {
            Serial.write(file.read());
        }
        file.close();
    }

    // Get file info
    File root = LittleFS.open("/");
    File f = root.openNextFile();
    while (f) {
        Serial.printf("File: %s, Size: %u\n", f.name(), f.size());
        f = root.openNextFile();
    }

    LittleFS.end();
}

void loop() {
}
```

## SPIFFS (Legacy)

```cpp
#include <SPIFFS.h>

void setup() {
    Serial.begin(115200);

    // Mount SPIFFS
    if (!SPIFFS.begin(true)) {
        Serial.println("SPIFFS mount failed");
        return;
    }

    Serial.println("SPIFFS mounted");

    // Write file
    File file = SPIFFS.open("/data.txt", "w");
    if (file) {
        file.printf("Temperature: %.2f\n", 23.5);
        file.printf("Humidity: %.2f\n", 65.2);
        file.close();
    }

    // Read file
    file = SPIFFS.open("/data.txt", "r");
    if (file) {
        Serial.println("Reading file:");
        while (file.available()) {
            Serial.write(file.read());
        }
        file.close();
    }

    // File info
    Serial.printf("Total: %u bytes\n", SPIFFS.totalBytes());
    Serial.printf("Used: %u bytes\n", SPIFFS.usedBytes());

    SPIFFS.end();
}

void loop() {
}
```

## Data Logging

```cpp
#include <LittleFS.h>

const char* LOG_FILE = "/datalog.txt";

void logData(const char* message) {
    File file = LittleFS.open(LOG_FILE, "a");  // Append mode
    if (file) {
        file.printf("[%lu] %s\n", millis(), message);
        file.close();
    }
}

void readLog() {
    File file = LittleFS.open(LOG_FILE, "r");
    if (file) {
        Serial.println("--- Log File ---");
        while (file.available()) {
            Serial.write(file.read());
        }
        file.close();
    }
}

void clearLog() {
    LittleFS.remove(LOG_FILE);
    Serial.println("Log cleared");
}

void setup() {
    Serial.begin(115200);
    LittleFS.begin();

    // Log some data
    logData("System started");
    logData("Sensor initialized");
    logData("First reading taken");

    // Read log
    readLog();

    LittleFS.end();
}

void loop() {
}
```

## Configuration File

```cpp
#include <LittleFS.h>
#include <ArduinoJson.h>

struct Config {
    char ssid[32];
    char password[64];
    int interval;
    bool enabled;
};

bool saveConfig(const Config& config) {
    File file = LittleFS.open("/config.json", "w");
    if (!file) {
        return false;
    }

    StaticJsonDocument<256> doc;
    doc["ssid"] = config.ssid;
    doc["password"] = config.password;
    doc["interval"] = config.interval;
    doc["enabled"] = config.enabled;

    if (serializeJson(doc, file) == 0) {
        file.close();
        return false;
    }

    file.close();
    return true;
}

bool loadConfig(Config& config) {
    File file = LittleFS.open("/config.json", "r");
    if (!file) {
        return false;
    }

    StaticJsonDocument<256> doc;
    DeserializationError error = deserializeJson(doc, file);
    file.close();

    if (error) {
        return false;
    }

    strlcpy(config.ssid, doc["ssid"] | "", sizeof(config.ssid));
    strlcpy(config.password, doc["password"] | "", sizeof(config.password));
    config.interval = doc["interval"] | 1000;
    config.enabled = doc["enabled"] | true;

    return true;
}

void setup() {
    Serial.begin(115200);
    LittleFS.begin();

    // Save default config
    Config config = {
        .ssid = "MyNetwork",
        .password = "MyPassword",
        .interval = 5000,
        .enabled = true
    };
    saveConfig(config);

    // Load config
    Config loaded;
    if (loadConfig(loaded)) {
        Serial.printf("SSID: %s\n", loaded.ssid);
        Serial.printf("Interval: %d\n", loaded.interval);
    }

    LittleFS.end();
}

void loop() {
}
```

## File Management

```cpp
#include <LittleFS.h>

void listFiles() {
    File root = LittleFS.open("/");
    File file = root.openNextFile();

    Serial.println("Files in filesystem:");
    while (file) {
        Serial.printf("  %s (%u bytes)\n", file.name(), file.size());
        file = root.openNextFile();
    }
}

void deleteFile(const char* path) {
    if (LittleFS.remove(path)) {
        Serial.printf("Deleted: %s\n", path);
    } else {
        Serial.printf("Failed to delete: %s\n", path);
    }
}

void renameFile(const char* oldPath, const char* newPath) {
    if (LittleFS.rename(oldPath, newPath)) {
        Serial.printf("Renamed %s to %s\n", oldPath, newPath);
    } else {
        Serial.println("Rename failed");
    }
}

void formatFS() {
    Serial.println("Formatting filesystem...");
    LittleFS.format();
    Serial.println("Format complete");
}

void setup() {
    Serial.begin(115200);
    LittleFS.begin();

    // Create test file
    File file = LittleFS.open("/test.txt", "w");
    file.println("Test data");
    file.close();

    // List files
    listFiles();

    // Rename file
    renameFile("/test.txt", "/renamed.txt");

    // Delete file
    deleteFile("/renamed.txt");

    LittleFS.end();
}

void loop() {
}
```

## Binary File

```cpp
#include <LittleFS.h>

void writeBinary(const char* path, const uint8_t* data, size_t len) {
    File file = LittleFS.open(path, "w");
    if (file) {
        file.write(data, len);
        file.close();
    }
}

void readBinary(const char* path, uint8_t* buffer, size_t len) {
    File file = LittleFS.open(path, "r");
    if (file) {
        file.read(buffer, len);
        file.close();
    }
}

void setup() {
    Serial.begin(115200);
    LittleFS.begin();

    // Write binary data
    uint8_t data[] = {0x01, 0x02, 0x03, 0x04, 0x05};
    writeBinary("/binary.dat", data, sizeof(data));

    // Read binary data
    uint8_t buffer[10];
    size_t bytesRead = LittleFS.open("/binary.dat", "r").read(buffer, sizeof(buffer));

    Serial.printf("Read %u bytes\n", bytesRead);
    for (size_t i = 0; i < bytesRead; i++) {
        Serial.printf("%02X ", buffer[i]);
    }
    Serial.println();

    LittleFS.end();
}

void loop() {
}
```

## Web Server Files

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <LittleFS.h>

WebServer server(80);

void handleRoot() {
    File file = LittleFS.open("/index.html", "r");
    if (file) {
        server.streamFile(file, "text/html");
        file.close();
    } else {
        server.send(404, "text/plain", "File not found");
    }
}

void handleFileUpload() {
    // Handle file upload
    HTTPUpload& upload = server.upload();
    static File fsUploadFile;

    if (upload.status == UPLOAD_FILE_START) {
        fsUploadFile = LittleFS.open("/" + upload.filename, "w");
    } else if (upload.status == UPLOAD_FILE_WRITE) {
        fsUploadFile.write(upload.buf, upload.currentSize);
    } else if (upload.status == UPLOAD_FILE_END) {
        fsUploadFile.close();
    }
}

void setup() {
    Serial.begin(115200);
    LittleFS.begin();

    WiFi.begin("ssid", "password");
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
    }

    server.on("/", HTTP_GET, handleRoot);
    server.on("/upload", HTTP_POST, []() {
        server.send(200);
    }, handleFileUpload);

    server.begin();
}

void loop() {
    server.handleClient();
}
```

## Directory Operations

```cpp
#include <LittleFS.h>

void createDirectory(const char* path) {
    if (LittleFS.mkdir(path)) {
        Serial.printf("Created directory: %s\n", path);
    }
}

void removeDirectory(const char* path) {
    if (LittleFS.rmdir(path)) {
        Serial.printf("Removed directory: %s\n", path);
    }
}

void listDirectory(const char* path) {
    File dir = LittleFS.open(path);
    if (!dir || !dir.isDirectory()) {
        Serial.println("Not a directory");
        return;
    }

    File file = dir.openNextFile();
    while (file) {
        if (file.isDirectory()) {
            Serial.printf("DIR: %s\n", file.name());
        } else {
            Serial.printf("FILE: %s (%u bytes)\n", file.name(), file.size());
        }
        file = dir.openNextFile();
    }
}

void setup() {
    Serial.begin(115200);
    LittleFS.begin();

    createDirectory("/data");
    createDirectory("/logs");

    File f = LittleFS.open("/data/sensor.txt", "w");
    f.println("Sensor data");
    f.close();

    listDirectory("/");

    LittleFS.end();
}

void loop() {
}
```

## Best Practices

```cpp
#include <LittleFS.h>

void setup() {
    Serial.begin(115200);

    // Best practice 1: Always check mount result
    if (!LittleFS.begin(true)) {  // Format on mount failure
        Serial.println("Mount failed");
        return;
    }

    // Best practice 2: Check file operations
    File file = LittleFS.open("/test.txt", "w");
    if (!file) {
        Serial.println("Failed to open file");
        return;
    }
    file.close();

    // Best practice 3: Always close files
    file = LittleFS.open("/test.txt", "r");
    if (file) {
        // Read data
        file.close();  // Always close!
    }

    // Best practice 4: Handle file size limits
    size_t freeSpace = LittleFS.totalBytes() - LittleFS.usedBytes();
    Serial.printf("Free space: %u bytes\n", freeSpace);

    // Best practice 5: Use append mode for logging
    File logFile = LittleFS.open("/log.txt", "a");  // Not "w"
    logFile.println("New entry");
    logFile.close();

    // Best practice 6: Unmount when not in use (saves power)
    LittleFS.end();
}

void loop() {
}
```

## SPIFFS vs LittleFS

| Feature | SPIFFS | LittleFS |
|---------|--------|----------|
| Speed | Slower | Faster |
| Power Loss Safety | Poor | Good |
| Wear Leveling | Basic | Advanced |
| File Size Limit | 2GB | 4GB |
| Mount Time | Slower | Faster |
| Memory Usage | Lower | Higher |

**Recommendation**: Use LittleFS for new projects.

## References

- [LittleFS Documentation](https://github.com/earlephilhower/arduino-esp8266littlefs-plugin)
- [SPIFFS Documentation](https://github.com/pellepl/spiffs)
