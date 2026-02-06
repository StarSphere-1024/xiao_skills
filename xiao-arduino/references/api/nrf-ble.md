# nRF52 BLE API for XIAO nRF52840

## Overview

Bluetooth Low Energy using ArduinoBLE library on nRF52840.

## Basic BLE Peripheral

```cpp
#include <ArduinoBLE.h>

void setup() {
    Serial.begin(115200);

    if (!BLE.begin()) {
        Serial.println("BLE init failed!");
        while (1);
    }

    // Set local name
    BLE.setLocalName("XIAO-nRF");

    // Start advertising
    BLE.advertise();
    Serial.println("BLE advertising...");
}

void loop() {
    BLEDevice central = BLE.central();
    if (central) {
        Serial.print("Connected to: ");
        Serial.println(central.address());

        while (central.connected()) {
            delay(100);
        }
        Serial.println("Disconnected");
    }
}
```

## BLE Service and Characteristic

```cpp
#include <ArduinoBLE.h>

// UUID definitions
BLEService myService("19B10000-E8F2-537E-4F6C-D104768A1214");
BLEByteCharacteristic myChar("19B10001-E8F2-537E-4F6C-D104768A1214",
                              BLERead | BLEWrite);

void setup() {
    Serial.begin(115200);

    if (!BLE.begin()) {
        while (1);
    }

    BLE.setLocalName("XIAO-Service");

    // Set up service
    BLE.setAdvertisedService(myService);
    myService.addCharacteristic(myChar);
    BLE.addService(myService);

    // Set initial value
    myChar.writeValue(0);

    BLE.advertise();
}

void loop() {
    BLEDevice central = BLE.central();
    if (central) {
        while (central.connected()) {
            // Check if written
            if (myChar.written()) {
                byte value = myChar.value();
                Serial.printf("Received: %d\n", value);
            }
        }
    }
}
```

## Custom Service with Multiple Characteristics

```cpp
#include <ArduinoBLE.h>

BLEService sensorService("180D");  // Heart Rate service

// Read-only characteristic
BLEByteCharacteristic heartRateChar("2A37", BLERead);

// Read/Write characteristic
BLEIntCharacteristic controlChar("2A58", BLERead | BLEWrite);

// Notify characteristic
BLECharacteristic dataChar("2A58", BLERead | BLENotify, 5);

void setup() {
    Serial.begin(115200);

    if (!BLE.begin()) {
        while (1);
    }

    BLE.setLocalName("XIAO-Sensor");
    BLE.setAdvertisedService(sensorService);

    sensorService.addCharacteristic(heartRateChar);
    sensorService.addCharacteristic(controlChar);
    sensorService.addCharacteristic(dataChar);

    BLE.addService(sensorService);

    heartRateChar.writeValue(0);
    controlChar.writeValue(0);
    dataChar.writeValue((byte*)"\x01\x02\x03\x04\x05", 5);

    BLE.advertise();
}

void loop() {
    BLEDevice central = BLE.central();
    if (central) {
        while (central.connected()) {
            // Send notifications
            byte sensorData[5] = {0x01, 0x02, 0x03, 0x04, 0x05};
            dataChar.writeValue(sensorData, 5);
            delay(1000);
        }
    }
}
```

## BLE Descriptor

```cpp
#include <ArduinoBLE.h>

BLEService myService("19B10000-E8F2-537E-4F6C-D104768A1214");
BLECharacteristic myChar("19B10001-E8F2-537E-4F6C-D104768A1214",
                         BLERead | BLEWrite, 20);

// Add descriptor
const char* description = "Sensor Data";
BLEDescriptor desc("2901", description);

void setup() {
    BLE.begin();
    BLE.setLocalName("XIAO-Desc");
    BLE.setAdvertisedService(myService);
    myService.addCharacteristic(myChar);
    myChar.addDescriptor(desc);
    BLE.addService(myService);
    BLE.advertise();
}

void loop() {}
```

## Read Request Callback

```cpp
#include <ArduinoBLE.h>

BLEService myService("19B10000-E8F2-537E-4F6C-D104768A1214");
BLEByteCharacteristic myChar("19B10001-E8F2-537E-4F6C-D104768A1214",
                              BLERead);

void onRead(BLEDevice central, BLECharacteristic characteristic) {
    Serial.println("Read request");
    byte value = analogRead(A0) / 4;
    characteristic.writeValue(value);
}

void setup() {
    Serial.begin(115200);
    BLE.begin();
    BLE.setLocalName("XIAO-Read");

    BLE.setAdvertisedService(myService);
    myService.addCharacteristic(myChar);
    BLE.addService(myService);

    myChar.setReadPropertyCallback(onRead);
    myChar.writeValue(0);

    BLE.advertise();
}

void loop() {}
```

## Write Callback

```cpp
#include <ArduinoBLE.h>

BLEService myService("19B10000-E8F2-537E-4F6C-D104768A1214");
BLEByteCharacteristic ledChar("19B10001-E8F2-537E-4F6C-D104768A1214",
                              BLERead | BLEWrite);

void onWrite(BLEDevice central, BLECharacteristic characteristic) {
    byte value = ledChar.value();
    Serial.printf("LED: %d\n", value);
    digitalWrite(LED_BUILTIN, value);
}

void setup() {
    pinMode(LED_BUILTIN, OUTPUT);
    Serial.begin(115200);

    BLE.begin();
    BLE.setLocalName("XIAO-LED");

    BLE.setAdvertisedService(myService);
    myService.addCharacteristic(ledChar);
    BLE.addService(myService);

    ledChar.setWritePropertyCallback(onWrite);
    ledChar.writeValue(0);

    BLE.advertise();
}

void loop() {}
```

## BLE Battery Service

```cpp
#include <ArduinoBLE.h>

BLEService batteryService("180F");
BLEUnsignedCharCharacteristic batteryLevelChar("2A19", BLERead | BLENotify);

void updateBatteryLevel() {
    // Read battery voltage (example)
    int battery = map(analogRead(A0), 0, 1023, 0, 100);
    batteryLevelChar.writeValue(battery);
}

void setup() {
    Serial.begin(115200);

    if (!BLE.begin()) {
        while (1);
    }

    BLE.setLocalName("XIAO-Battery");
    BLE.setAdvertisedService(batteryService);
    batteryService.addCharacteristic(batteryLevelChar);
    BLE.addService(batteryService);

    updateBatteryLevel();
    BLE.advertise();
}

void loop() {
    BLEDevice central = BLE.central();
    if (central) {
        while (central.connected()) {
            updateBatteryLevel();
            delay(5000);
        }
    }
}
```

## Advertising with Data

```cpp
#include <ArduinoBLE.h>

void setup() {
    Serial.begin(115200);

    if (!BLE.begin()) {
        while (1);
    }

    // Set advertising data
    uint8_t advData[] = {
        0x02, 0x01, 0x06,  // Flags
        0x03, 0x03, 0x0A, 0x18,  // UUID
        0x09, 0x09, 'X', 'I', 'A', 'O', ' ', 'n', 'R', 'F'  // Name
    };

    BLE.setAdvertisingData(advData, sizeof(advData));
    BLE.advertise();
    Serial.println("Advertising with custom data");
}

void loop() {
    delay(1000);
}
```

## Bonding and Encryption

```cpp
#include <ArduinoBLE.h>

BLEService myService("19B10000-E8F2-537E-4F6C-D104768A1214");
BLECharacteristic myChar("19B10001-E8F2-537E-4F6C-D104768A1214",
                         BLERead | BLEWrite | BLEEncrypt);

void setup() {
    Serial.begin(115200);

    if (!BLE.begin()) {
        while (1);
    }

    // Enable pairing
    BLE.setConnectable(true);
    BLE.setPairable(true);

    BLE.setLocalName("XIAO-Secure");
    BLE.setAdvertisedService(myService);
    myService.addCharacteristic(myChar);
    BLE.addService(myService);

    myChar.writeValue(0);
    BLE.advertise();
}

void loop() {
    BLEDevice central = BLE.central();
    if (central) {
        if (central.hasSecureConnection()) {
            Serial.println("Secure connection!");
        }
        while (central.connected()) {
            delay(100);
        }
    }
}
```

## BLE Central (Scanner)

```cpp
#include <ArduinoBLE.h>

void setup() {
    Serial.begin(115200);

    if (!BLE.begin()) {
        while (1);
    }

    Serial.println("Scanning...");
    BLE.scan();
}

void loop() {
    BLEDevice peripheral = BLE.available();
    if (peripheral) {
        Serial.print("Found: ");
        Serial.print(peripheral.address());
        Serial.print(" '");
        Serial.print(peripheral.localName());
        Serial.print("' ");
        Serial.println(peripheral.rssi());
    }
}
```

## Connect to Peripheral

```cpp
#include <ArduinoBLE.h>

BLEDevice peripheral;

void setup() {
    Serial.begin(115200);

    if (!BLE.begin()) {
        while (1);
    }

    BLE.scan();
}

void loop() {
    if (!peripheral || !peripheral.connected()) {
        BLEDevice p = BLE.available();
        if (p && p.localName() == "TargetDevice") {
            BLE.stopScan();
            peripheral = p;
            peripheral.connect();
            Serial.println("Connected!");

            // Discover service
            if (peripheral.discoverAttributes()) {
                BLECharacteristic chars = peripheral.characteristic(0);
                if (chars) {
                    chars.subscribe();
                }
            }
        }
    }

    if (peripheral && peripheral.connected()) {
        // Read notifications
        BLECharacteristic chars = peripheral.characteristic(0);
        if (chars && chars.valueUpdated()) {
            byte value;
            chars.readValue(value);
            Serial.printf("Value: %d\n", value);
        }
    }
}
```

## OTA Service

```cpp
#include <ArduinoBLE.h>

BLEService otaService("1d14-6efd-4ba5-8040-5c0304a4d64c");

void setup() {
    Serial.begin(115200);
    BLE.begin();
    BLE.setLocalName("XIAO-OTA");
    BLE.setAdvertisedService(otaService);
    BLE.addService(otaService);
    BLE.advertise();
}

void loop() {
    // Handle OTA via nRF Connect app or custom
    delay(1000);
}
```

## Connection Parameters

```cpp
void setup() {
    BLE.begin();

    // Set connection parameters
    BLE.setConnectionInterval(0x0006, 0x0C80);  // 7.5ms - 4s
    BLE.setSupervisionTimeout(0x0C80);  // 4s
    BLE.setSlaveLatency(0);

    BLE.setLocalName("XIAO-Params");
    BLE.advertise();
}

void loop() {}
```

## Low Power Advertising

```cpp
void setup() {
    BLE.begin();

    // Low power advertising
    BLE.setAdvertisingInterval(160);  // 100ms
    BLE.setConnectable(false);  // Non-connectable

    BLE.advertise();
}

void loop() {
    // Enter system off periodically
    delay(5000);
}
```

## Troubleshooting

### Can't advertise

1. Check SoftDevice version
2. Verify service UUIDs are valid 128-bit
3. Check other BLE devices interference

### Connection drops

1. Check connection parameters
2. Verify power supply stability
3. Check for buffer overflows

### Write failures

1. Verify characteristic properties
2. Check data size limits
3. Ensure notifications enabled

### Memory issues

1. SoftDevice uses RAM, reduce application buffers
2. Check available heap with `freeMemory()`
3. Limit concurrent connections

### Bonding failures

1. Clear existing bonds: `BLE.deleteBond()`
2. Verify pairing mode is enabled
3. Check security requirements
