# ESP32 BLE API (Arduino)

## Overview

ESP32 BLE (Bluetooth Low Energy) using ESP-IDF BLE library wrapped for Arduino. Examples below follow the BLE usage patterns in the XIAO ESP32C3/C6 reference docs.

## Basic BLE Peripheral

```cpp
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

#define SERVICE_UUID        "4fafc201-1fb5-459e-8fcc-c5c9c331914b"
#define CHARACTERISTIC_UUID "beb5483e-36e1-4688-b7f5-ea07361b26a8"


BLEServer *pServer;
BLECharacteristic *pCharacteristic;

void setup() {
    Serial.begin(115200);
    BLEDevice::init("XIAO_ESP32");

    pServer = BLEDevice::createServer();
    BLEService *pService = pServer->createService(SERVICE_UUID);

    pCharacteristic = pService->createCharacteristic(
        CHARACTERISTIC_UUID,
        BLECharacteristic::PROPERTY_READ |
        BLECharacteristic::PROPERTY_WRITE
    );

    pCharacteristic->setValue("Hello XIAO!");
    pService->start();

    BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
    pAdvertising->addServiceUUID(SERVICE_UUID);
    pAdvertising->setScanResponse(true);
    pAdvertising->setMinPreferred(0x06);  // helps with iPhone connections
    pAdvertising->setMinPreferred(0x12);
    BLEDevice::startAdvertising();
    Serial.println("Characteristic defined! Now you can read it in your phone!");
}

void loop() {
    if (pServer->getConnectedCount() > 0) {
        // Connected
    }
    delay(1000);
}
```

## BLE Server Callbacks

```cpp
class MyServerCallbacks : public BLEServerCallbacks {
    void onConnect(BLEServer* pServer) {
        Serial.println("Client connected");
    };

    void onDisconnect(BLEServer* pServer) {
        Serial.println("Client disconnected");
    }
};

void setup() {

    BLEDevice::init("XIAO_ESP32");
    BLEServer *pServer = BLEDevice::createServer();
    pServer->setCallbacks(new MyServerCallbacks());
}
```

## BLE Characteristic Callbacks

```cpp
class MyCallbacks : public BLECharacteristicCallbacks {
    void onWrite(BLECharacteristic *pCharacteristic) {
        String value = pCharacteristic->getValue();
        Serial.println("********");
        Serial.print("New value: ");
        for (int i = 0; i < value.length(); i++)
            Serial.print(value[i]);
        Serial.println();
        Serial.println("********");
    }
};

void setup() {
    // ... setup characteristic
    pCharacteristic->setCallbacks(new MyCallbacks());

}
```

:::tip
If you use ESP32 Arduino core 3.0.0 or above, prefer `String value = pCharacteristic->getValue();` as shown here.
:::

## BLE Advertising

```cpp
void setup() {
    BLEDevice::init("XIAO_ESP32");
    BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();

    // Set advertising parameters
    pAdvertising->setScanResponse(true);
    pAdvertising->setMinPreferred(0x06);  // 30ms
    pAdvertising->setMinPreferred(0x12);  // 120ms

    // Start advertising
    BLEDevice::startAdvertising();
}
```

## BLE Scan

```cpp
#include <BLEDevice.h>
#include <BLEScan.h>
#include <BLEAdvertisedDevice.h>

BLEScan* pBLEScan;


void setup() {
    Serial.begin(115200);
    BLEDevice::init();
    pBLEScan = BLEDevice::getScan();
    pBLEScan->setActiveScan(true);
}

void loop() {
    BLEScanResults foundDevices = pBLEScan->start(5, false);  // 5 seconds

    Serial.println("Devices found:");
    for (int i = 0; i < foundDevices.getCount(); i++) {
        BLEAdvertisedDevice advertisedDevice = foundDevices.getDevice(i);
        Serial.printf("Device: %s, RSSI: %d\n",
            advertisedDevice.getName().c_str(),
            advertisedDevice.getRSSI());
    }


    pBLEScan->clearResults();
    delay(10000);
}
```

:::tip
If you have already upgraded your ESP32 development board to version 3.0.0 or above, use the pointer return type:

1. `BLEScanResults foundDevices = pBLEScan->start(5, false);` → `BLEScanResults* foundDevices = pBLEScan->start(5, false);`
2. `foundDevices.getCount()` → `foundDevices->getCount()`
:::

## BLE Client (Connecting to peripheral)

```cpp
#include <BLEDevice.h>

static BLEAddress targetAddress("xx:xx:xx:xx:xx:xx");
static boolean connected = false;
static BLEClient *pClient = nullptr;

class MyClientCallbacks : public BLEClientCallbacks {
    void onConnect(BLEClient *pClient) {
        connected = true;
    }

    void onDisconnect(BLEClient *pClient) {
        connected = false;
    }
};

void setup() {
    BLEDevice::init();
    pClient = BLEDevice::createClient();
    pClient->setClientCallbacks(new MyClientCallbacks());
    pClient->connect(targetAddress);
}

void loop() {
    // add your client logic here
}
```

## BLE UUID Reference

| UUID Type | Description |
|-----------|-------------|
| 0x1800 | Generic Access |
| 0x1801 | Generic Attribute |
| 0x180A | Device Information |
| 0x2800 | Primary Service |
| 0x2803 | Characteristic Declaration |

## Custom Service with Notify

```cpp
#include <BLE2902.h>

void setup() {
    BLEDevice::init("XIAO_ESP32");
    BLEServer *pServer = BLEDevice::createServer();

    BLEService *pService = pServer->createService(SERVICE_UUID);

    BLECharacteristic *pCharacteristic = pService->createCharacteristic(
        CHARACTERISTIC_UUID,
        BLECharacteristic::PROPERTY_READ |
        BLECharacteristic::PROPERTY_WRITE |
        BLECharacteristic::PROPERTY_NOTIFY
    );

    BLE2902 *p2902 = new BLE2902();
    p2902->setNotifications(true);
    pCharacteristic->addDescriptor(p2902);

    pService->start();
    BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
    pAdvertising->addServiceUUID(SERVICE_UUID);
    pAdvertising->setScanResponse(true);
    pAdvertising->setMinPreferred(0x06);
    pAdvertising->setMinPreferred(0x12);
    BLEDevice::startAdvertising();
}
```

## Send Notification

```cpp
void loop() {
    if (pServer->getConnectedCount() > 0) {
        String value = "Sensor data: 25.5";
        pCharacteristic->setValue(value);
        pCharacteristic->notify();
        delay(5000);
    }
}
```


## Best Practices

1. **Short names**: Keep BLE device name short
2. **Low advertising interval**: Saves power
3. **Use notify instead of read**: For periodic data
4. **Handle disconnects**: Always check connection state
5. **Clear connections**: Reset between tests


