# EEPROM/Library for XIAO

## Overview

EEPROM emulation using flash memory for persistent data storage.

## ESP32 EEPROM

### Basic Read/Write

```cpp
#include <EEPROM.h>

#define EEPROM_SIZE 64

void setup() {
    Serial.begin(115200);
    EEPROM.begin(EEPROM_SIZE);

    // Write
    EEPROM.writeByte(0, 42);
    EEPROM.writeULong(4, 123456789);
    EEPROM.commit();

    // Read
    byte value = EEPROM.readByte(0);
    unsigned long timestamp = EEPROM.readULong(4);

    Serial.printf("Value: %d, Time: %lu\n", value, timestamp);
    EEPROM.end();
}

void loop() {}
```

### String Storage

```cpp
void saveString(String data, int addr) {
    int len = data.length() + 1;
    EEPROM.writeByte(addr, len);
    for (int i = 0; i < len; i++) {
        EEPROM.writeByte(addr + 1 + i, data.charAt(i));
    }
    EEPROM.commit();
}

String readString(int addr) {
    int len = EEPROM.readByte(addr);
    char buffer[len];
    for (int i = 0; i < len; i++) {
        buffer[i] = EEPROM.readByte(addr + 1 + i);
    }
    return String(buffer);
}
```

### WiFi Credentials Storage

```cpp
struct WiFiConfig {
    char ssid[32];
    char password[64];
    bool saved;
};

void saveWiFiConfig(String ssid, String password) {
    WiFiConfig config;
    ssid.toCharArray(config.ssid, 32);
    password.toCharArray(config.password, 64);
    config.saved = true;
    EEPROM.put(0, config);
    EEPROM.commit();
}

bool loadWiFiConfig(String &ssid, String &password) {
    WiFiConfig config;
    EEPROM.get(0, config);
    if (config.saved) {
        ssid = String(config.ssid);
        password = String(config.password);
        return true;
    }
    return false;
}
```

### Counter with EEPROM

```cpp
RTC_DATA_ATTR int bootCount = 0;

void setup() {
    Serial.begin(115200);
    EEPROM.begin(10);

    bootCount = EEPROM.readInt(0);
    bootCount++;
    EEPROM.writeInt(0, bootCount);
    EEPROM.commit();

    Serial.printf("Boot count: %d\n", bootCount);
}
```

### Configuration Struct

```cpp
struct Config {
    int sensorInterval;
    float threshold;
    bool enableWiFi;
    char mqttServer[32];
};

void saveConfig(Config config) {
    EEPROM.put(0, config);
    EEPROM.commit();
}

Config loadConfig() {
    Config config;
    EEPROM.get(0, config);
    return config;
}

void setup() {
    EEPROM.begin(256);

    // Default config
    Config config = {5000, 25.5, true, "broker.hivemq.com"};
    saveConfig(config);

    // Load and modify
    Config loaded = loadConfig();
    loaded.sensorInterval = 10000;
    saveConfig(loaded);
}
```

## nRF52 EEPROM

### FlashStorage (nRF52840)

```cpp
#include <FlashStorage.h>

FlashStorage(my_flash_store, int);

void setup() {
    Serial.begin(115200);

    // Write
    int value = 12345;
    my_flash_store.write(value);

    // Read
    int retrieved = my_flash_store.read();
    Serial.println(retrieved);
}
```

### Struct Storage

```cpp
struct UserData {
    int id;
    float value;
    char name[16];
};

FlashStorage(user_storage, UserData);

void setup() {
    UserData data = {1, 99.9, "XIAO"};
    user_storage.write(data);

    UserData loaded = user_storage.read();
    Serial.println(loaded.name);
}
```

## RP2040 EEPROM

### RP2040 Flash

```cpp
#include <EEPROM.h>

void setup() {
    Serial.begin(115200);
    EEPROM.begin(512);  // Size in bytes

    // Write
    EEPROM.write(0, 42);
    EEPROM.commit();

    // Read
    byte value = EEPROM.read(0);
    Serial.println(value);
}
```

### Using Flash as Storage

```cpp
#include <FlashIAP.h>
#include <FlashIAPBlockDevice.h>
#include <TDBStore.h>

FlashIAPBlockDevice bd(XIP_BASE + 0x100000, 0x10000);
TDBStore store(&bd);

void setup() {
    bd.init();
    store.init();

    // Write
    uint8_t data[] = {1, 2, 3, 4};
    store.set("key", data, sizeof(data));

    // Read
    size_t size;
    store.get("key", data, sizeof(data), &size);
}

void loop() {}
```

## SAMD21 EEPROM

### Emulated EEPROM

```cpp
#include <FlashAsEEPROM.h>

void setup() {
    Serial.begin(115200);

    // Write
    EEPROM.write(0, 42);
    EEPROM.commit();

    // Read
    byte value = EEPROM.read(0);
    Serial.println(value);
}
```

## Wear Leveling

### Simple Wear Leveling

```cpp
#define EEPROM_SIZE 256
#define NUM_SLOTS 4
#define SLOT_SIZE (EEPROM_SIZE / NUM_SLOTS)

int getCurrentSlot() {
    for (int i = 0; i < NUM_SLOTS; i++) {
        if (EEPROM.readByte(i * SLOT_SIZE) == 0xFF) {
            return i;
        }
    }
    return 0;
}

void writeWithLeveling(int addr, byte value) {
    static int currentSlot = 0;
    int writeAddr = currentSlot * SLOT_SIZE + addr + 1;
    EEPROM.writeByte(writeAddr, value);

    // Check slot full
    EEPROM.writeByte(currentSlot * SLOT_SIZE, 0xAA);
    currentSlot = (currentSlot + 1) % NUM_SLOTS;
    EEPROM.writeByte(currentSlot * SLOT_SIZE, 0xFF);
    EEPROM.commit();
}
```

## Checksum Validation

```cpp
uint16_t calculateChecksum(byte *data, size_t len) {
    uint16_t checksum = 0;
    for (size_t i = 0; i < len; i++) {
        checksum += data[i];
    }
    return checksum;
}

void saveWithChecksum(byte *data, size_t len, int addr) {
    uint16_t checksum = calculateChecksum(data, len);
    EEPROM.writeBytes(addr, data, len);
    EEPROM.writeShort(addr + len, checksum);
    EEPROM.commit();
}

bool loadWithChecksum(byte *data, size_t len, int addr) {
    EEPROM.readBytes(addr, data, len);
    uint16_t storedChecksum = EEPROM.readShort(addr + len);
    uint16_t calculatedChecksum = calculateChecksum(data, len);
    return storedChecksum == calculatedChecksum;
}
```

## Troubleshooting

### Data not persisting

1. Always call `EEPROM.commit()` after writing
2. Check EEPROM size is sufficient
3. Verify power during write

### Corruption on boot

1. Check for EEPROM.begin() before access
2. Use checksums for validation
3. Implement wear leveling

### Size limits

- ESP32: Virtual, limited by flash partition
- nRF52: Limited by available flash
- RP2040: Limited by flash endurance
- SAMD21: Limited by flash size

### Wear concerns

- ESP32: 100,000 cycles
- nRF52: 10,000 cycles
- RP2040: Flash endurance limited
- Use wear leveling for frequent writes
