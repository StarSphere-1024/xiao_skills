# BLE iBeacon for XIAO ESP32

## Overview

iBeacon is a Bluetooth Low Energy (BLE) protocol developed by Apple for location-based services and proximity detection. XIAO ESP32 boards can act as iBeacon transmitters to broadcast data to nearby devices.

## Board Support

| Board | BLE Support | iBeacon Compatible |
|-------|-------------|-------------------|
| ESP32C3 | ✅ | ✅ |
| ESP32C5 | ✅ | ✅ |
| ESP32C6 | ✅ | ✅ |
| ESP32S3 | ✅ | ✅ |
| nRF52840 | ✅ | ✅ |
| MG24 | ✅ | ✅ |
| RP2040/RP2350 | ❌ | No BLE |
| SAMD21 | ❌ | No BLE |
| RA4M1 | ❌ | No BLE |

## iBeacon vs Standard BLE

| Feature | iBeacon | Standard BLE |
|---------|---------|--------------|
| **Purpose** | Location/proximity sensing | General data exchange |
| **Direction** | One-way broadcast | Bidirectional communication |
| **Connections** | No connection required | Requires connection |
| **Power** | Ultra low (advertising only) | Higher (active connection) |
| **Data Size** | Limited (~31 bytes) | Larger (MTU up to 512 bytes) |
| **Use Cases** | Retail tracking, museums, indoor navigation | Sensors, control, data transfer |

## Basic iBeacon

### What is an iBeacon Packet?

An iBeacon advertisement packet contains:

```
[Company ID (2 bytes)][UUID (16 bytes)][Major (2 bytes)][Minor (2 bytes)][TX Power (1 byte)]
```

- **Company ID**: Apple's ID (0x004C) or custom
- **UUID**: Unique identifier for your beacon network
- **Major**: Groups related beacons (e.g., store ID)
- **Minor**: Individual beacon ID (e.g., shelf number)
- **TX Power**: Signal strength at 1 meter (calibration)

### Simple iBeacon Example

```cpp
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEServer.h>

// Define your iBeacon data
#define BEACON_UUID "87b99b2c-90fd-11e9-bc42-526af7764f64"
#define MAJOR_VERSION 1
#define MINOR_VERSION 1
#define TX_POWER -59

BLEAdvertising *pAdvertising;
void setup() {
    Serial.begin(115200);

    // Initialize BLE
    BLEDevice::init("XIAO-iBeacon");

    // Create advertising object
    pAdvertising = BLEDevice::getAdvertising();

    // Set iBeacon data
    BLEBeacon myBeacon = BLEBeacon();
    myBeacon.setProximityUUID(BLEUUID(BEACON_UUID));
    myBeacon.setMajor(MAJOR_VERSION);
    myBeacon.setMinor(MINOR_VERSION);
    myBeacon.setTxPower(TX_POWER);

    // Set advertising data
    pAdvertising->setScanResponseData(myBeacon.getData());
    pAdvertising->setMinPreferred(0x06);  // Functions that help with iPhone connections
    pAdvertising->setMinPreferred(0x12);

    // Start advertising
    BLEDevice::startAdvertising();

    Serial.println("iBeacon advertising started!");
    Serial.printf("UUID: %s\n", BEACON_UUID);
    Serial.printf("Major: %d, Minor: %d\n", MAJOR_VERSION, MINOR_VERSION);
}

void loop() {
    delay(1000);
    Serial.println("Beaconing...");
}
```

### Testing iBeacon with Mobile Apps

1. **iOS**: Download "LightBlue" or "nRF Connect"
2. **Android**: Download "nRF Connect" or "BLE Scanner"

Apps will detect your beacon as:
- Name: "XIAO-iBeacon"
- UUID: Your defined UUID
- Major/Minor: Your configured values
- RSSI: Signal strength indicator

## Eddystone Beacon (Google Alternative)

Eddystone is an open beacon protocol from Google that supports multiple packet types:

### Eddystone-UID Example

```cpp
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEServer.h>
#include <BLEAdvertising.h>

// Eddystone UID namespace and instance
const uint8_t uidNamespace[] = {0x00, 0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88, 0x99};
const uint8_t uidInstance[] = {0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF};

BLEAdvertising *pAdvertising;

void setEddystoneUID() {
    // Eddystone UID frame: [Type][Length][UID Frame][TX Power][Namespace][Instance][RFU]
    uint8_t eddystoneData[] = {
        0x02,  // Length
        0x01,  // Flags: BLE limited discoverable
        0x06,  // Length
        0x03,  // 16-bit Service UUID list
        0xAA, 0xFE,  // Eddystone Service UUID
        0x17,  // Length (23 bytes)
        0x16,  // Service Data
        0xAA, 0xFE,  // Eddystone UUID
        0x00,  // Eddystone-UID frame type
        0x00,  // TX Power (calibrated at 0 meters)
        // Namespace (10 bytes)
        0x00, 0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88, 0x99,
        // Instance (6 bytes)
        0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF,
        0x00  // RFU (Reserved for Future Use)
    };

    pAdvertising->setAdvData(BLEAdvertising().createScanResponseData(
        std::string((char*)eddystoneData, sizeof(eddystoneData))
    ));
}

void setup() {
    Serial.begin(115200);

    BLEDevice::init("XIAO-Eddystone");
    pAdvertising = BLEDevice::getAdvertising();

    setEddystoneUID();

    pAdvertising->start();
    Serial.println("Eddystone UID beacon started!");
}

void loop() {
    delay(1000);
}
```

## Custom BLE Advertisement (Sensor Data)

Broadcast sensor readings via BLE without connection:

```cpp
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEServer.h>

// Custom service UUID for sensor data
#define SENSOR_SERVICE_UUID "0000AAAA-0000-1000-8000-00805F9B34FB"

BLEAdvertising *pAdvertising;

// Simulated sensor data
float temperature = 23.5;
float humidity = 65.0;
int battery = 85;

void updateAdvertisementData() {
    // Create manufacturer-specific data
    // Format: [Company ID (2)][Temp (2)][Hum (1)][Battery (1)]
    uint16_t companyId = 0x02E5;  // Espressif
    int16_t tempEncoded = (int16_t)(temperature * 100);  // Scale by 100
    uint8_t humEncoded = (uint8_t)humidity;
    uint8_t batteryEncoded = (uint8_t)battery;

    uint8_t manufacturerData[6] = {
        (uint8_t)(companyId & 0xFF),
        (uint8_t)((companyId >> 8) & 0xFF),
        (uint8_t)(tempEncoded & 0xFF),
        (uint8_t)((tempEncoded >> 8) & 0xFF),
        humEncoded,
        batteryEncoded
    };

    pAdvertising->setManufacturerData(std::string((char*)manufacturerData, 6));
}

void setup() {
    Serial.begin(115200);

    BLEDevice::init("XIAO-Sensor");
    pAdvertising = BLEDevice::getAdvertising();
    pAdvertising->setMinPreferred(0x06);
    pAdvertising->setMinPreferred(0x12);

    updateAdvertisementData();

    BLEDevice::startAdvertising();
    Serial.println("Sensor beacon started!");
}

void loop() {
    // Update sensor readings (simulated)
    temperature = 20.0 + (rand() % 100) / 10.0;
    humidity = 40 + (rand() % 40);
    battery = max(0, battery - 1);

    updateAdvertisementData();

    Serial.printf("Advertising: %.1f°C, %d%%, %d%%\n",
                 temperature, (int)humidity, battery);

    delay(5000);
}
```

## Configurable iBeacon with Web Server

Configure iBeacon parameters via web interface:

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <BLEDevice.h>

// WiFi credentials
const char* ssid = "your-ssid";
const char* password = "your-password";

WebServer server(80);

// iBeacon parameters (default values)
char beaconUUID[37] = "87b99b2c-90fd-11e9-bc42-526af7764f64";
uint16_t major = 1;
uint16_t minor = 1;
int8_t txPower = -59;
bool advertising = false;

BLEAdvertising *pAdvertising;

void startBeacon() {
    if (advertising) return;

    BLEDevice::init("XIAO-iBeacon");
    pAdvertising = BLEDevice::getAdvertising();

    BLEBeacon myBeacon = BLEBeacon();
    myBeacon.setProximityUUID(BLEUUID(beaconUUID));
    myBeacon.setMajor(major);
    myBeacon.setMinor(minor);
    myBeacon.setTxPower(txPower);

    pAdvertising->setScanResponseData(myBeacon.getData());
    pAdvertising->start();

    advertising = true;
    Serial.println("Beacon started");
}

void stopBeacon() {
    if (!advertising) return;

    pAdvertising->stop();
    BLEDevice::deinit(true);

    advertising = false;
    Serial.println("Beacon stopped");
}

void handleRoot() {
    String html = "<html><body>";
    html += "<h1>XIAO iBeacon Config</h1>";

    html += "<form action='/save'>";
    html += "UUID: <input name='uuid' value='" + String(beaconUUID) + "'><br>";
    html += "Major: <input name='major' value='" + String(major) + "'><br>";
    html += "Minor: <input name='minor' value='" + String(minor) + "'><br>";
    html += "TX Power: <input name='txpower' value='" + String(txPower) + "'><br>";
    html += "<input type='submit' value='Save & Restart'>";
    html += "</form>";

    html += "<p>Status: ";
    html += advertising ? "<strong>Advertising</strong>" : "Stopped";
    html += "</p>";

    html += "<a href='/start'>Start Beacon</a> | ";
    html += "<a href='/stop'>Stop Beacon</a>";
    html += "</body></html>";

    server.send(200, "text/html", html);
}

void handleSave() {
    if (server.hasArg("uuid")) {
        server.arg("uuid").toCharArray(beaconUUID, 37);
    }
    if (server.hasArg("major")) {
        major = server.arg("major").toInt();
    }
    if (server.hasArg("minor")) {
        minor = server.arg("minor").toInt();
    }
    if (server.hasArg("txpower")) {
        txPower = server.arg("txpower").toInt();
    }

    Serial.println("Settings saved");

    // Restart beacon
    stopBeacon();
    delay(100);
    startBeacon();

    server.sendHeader("Location", "/");
    server.send(303);
}

void handleStart() {
    startBeacon();
    server.sendHeader("Location", "/");
    server.send(303);
}

void handleStop() {
    stopBeacon();
    server.sendHeader("Location", "/");
    server.send(303);
}

void setup() {
    Serial.begin(115200);

    // Connect to WiFi
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println(WiFi.localIP());

    // Setup web server
    server.on("/", handleRoot);
    server.on("/save", handleSave);
    server.on("/start", handleStart);
    server.on("/stop", handleStop);
    server.begin();

    // Start beacon
    startBeacon();

    Serial.println("Web server ready");
    Serial.printf("Access at http://%s\n", WiFi.localIP().toString().c_str());
}

void loop() {
    server.handleClient();
}
```

## Battery-Powered iBeacon with Deep Sleep

```cpp
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <esp_sleep.h>

#define BEACON_UUID "87b99b2c-90fd-11e9-bc42-526af7764f64"
#define MAJOR_VERSION 1
#define MINOR_VERSION 1
#define TX_POWER -59

#define ADVERTISING_TIME 5    // Seconds
#define SLEEP_INTERVAL 30     // Seconds

BLEAdvertising *pAdvertising;

void setup() {
    // Initialize BLE
    BLEDevice::init("XIAO-iBeacon");
    pAdvertising = BLEDevice::getAdvertising();

    // Configure beacon
    BLEBeacon myBeacon = BLEBeacon();
    myBeacon.setProximityUUID(BLEUUID(BEACON_UUID));
    myBeacon.setMajor(MAJOR_VERSION);
    myBeacon.setMinor(MINOR_VERSION);
    myBeacon.setTxPower(TX_POWER);

    pAdvertising->setScanResponseData(myBeacon.getData());
    pAdvertising->start();

    // Keep LED indicator during advertising
    pinMode(LED_BUILTIN, OUTPUT);
    digitalWrite(LED_BUILTIN, HIGH);

    // Advertise for configured time
    delay(ADVERTISING_TIME * 1000);

    // Stop advertising and go to sleep
    digitalWrite(LED_BUILTIN, LOW);
    pAdvertising->stop();
    BLEDevice::deinit(true);

    // Configure deep sleep
    Serial.println("Going to sleep...");
    esp_sleep_enable_timer_wakeup(SLEEP_INTERVAL * 1000000);
    esp_deep_sleep_start();
}

void loop() {}
```

## Distance Estimation

Calculate approximate distance based on RSSI:

```cpp
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEScan.h>

BLEScan *pBLEScan;

// Known beacon TX power at 1 meter
int8_t beaconTxPower = -59;

class MyAdvertisedDeviceCallbacks : public BLEAdvertisedDeviceCallbacks {
    void onResult(BLEAdvertisedDevice advertisedDevice) {
        // Check if this is our beacon
        if (advertisedDevice.haveName() &&
            advertisedDevice.getName() == "XIAO-iBeacon") {

            int rssi = advertisedDevice.getRSSI();

            // Calculate distance using path loss model
            // distance = 10^((TxPower - RSSI) / (10 * n))
            // where n is the path loss exponent (2-4, typically 2)
            float distance = pow(10, ((beaconTxPower - rssi) / (10 * 2.0)));

            Serial.printf("Device: %s, RSSI: %d dBm, Distance: %.2f m\n",
                         advertisedDevice.getName().c_str(),
                         rssi,
                         distance);
        }
    }
};

void setup() {
    Serial.begin(115200);

    BLEDevice::init("");
    pBLEScan = BLEDevice::getScan();
    pBLEScan->setAdvertisedDeviceCallbacks(new MyAdvertisedDeviceCallbacks());
    pBLEScan->setActiveScan(true);  // Active scan uses more power but gets more data
    pBLEScan->setInterval(100);
    pBLEScan->setWindow(99);

    Serial.println("Scanning for beacons...");
}

void loop() {
    BLEScanResults foundDevices = pBLEScan->start(5, false);
    Serial.printf("Found %d devices\n", foundDevices.getCount());
    pBLEScan->clearResults();
    delay(2000);
}
```

## Proximity Detection

Detect when a beacon is near/far/immediate:

```cpp
#include <BLEDevice.h>
#include <BLEScan.h>

BLEScan *pBLEScan;
int8_t beaconTxPower = -59;

enum ProximityZone {
    ZONE_UNKNOWN,
    ZONE_IMMEDIATE,  // < 0.5m
    ZONE_NEAR,       // 0.5m - 3m
    ZONE_FAR         // > 3m
};

ProximityZone getZone(int rssi, int8_t txPower) {
    float distance = pow(10, ((txPower - rssi) / (10 * 2.0)));

    if (distance < 0.5) return ZONE_IMMEDIATE;
    if (distance < 3.0) return ZONE_NEAR;
    return ZONE_FAR;
}

const char* zoneToString(ProximityZone zone) {
    switch (zone) {
        case ZONE_IMMEDIATE: return "Immediate";
        case ZONE_NEAR: return "Near";
        case ZONE_FAR: return "Far";
        default: return "Unknown";
    }
}

class MyAdvertisedDeviceCallbacks : public BLEAdvertisedDeviceCallbacks {
    void onResult(BLEAdvertisedDevice advertisedDevice) {
        if (advertisedDevice.haveName() &&
            advertisedDevice.getName() == "XIAO-iBeacon") {

            int rssi = advertisedDevice.getRSSI();
            ProximityZone zone = getZone(rssi, beaconTxPower);

            Serial.printf("Zone: %s (%.2f m)\n",
                         zoneToString(zone),
                         pow(10, ((beaconTxPower - rssi) / (10 * 2.0))));
        }
    }
};

void setup() {
    Serial.begin(115200);
    BLEDevice::init("");
    pBLEScan = BLEDevice::getScan();
    pBLEScan->setAdvertisedDeviceCallbacks(new MyAdvertisedDeviceCallbacks());
}

void loop() {
    pBLEScan->start(5);
    pBLEScan->clearResults();
    delay(1000);
}
```

## Multiple Beacons with Different IDs

Create a network of beacons with different IDs:

```cpp
#include <BLEDevice.h>
#include <BLEUtils.h>

#define BASE_UUID "87b99b2c-90fd-11e9-bc42-526af7764f64"

BLEAdvertising *pAdvertising;

// Beacon configurations
struct BeaconConfig {
    const char* uuid;
    uint16_t major;
    uint16_t minor;
    int8_t txPower;
};

BeaconConfig beacons[] = {
    {BASE_UUID, 1, 1, -59},  // Beacon 1
    {BASE_UUID, 1, 2, -59},  // Beacon 2
    {BASE_UUID, 2, 1, -59},  // Beacon 3 (different group)
};

int currentBeacon = 0;
unsigned long lastSwitch = 0;
const long SWITCH_INTERVAL = 10000;  // Switch every 10 seconds

void switchBeacon(int index) {
    pAdvertising->stop();

    BLEBeacon myBeacon = BLEBeacon();
    myBeacon.setProximityUUID(BLEUUID(beacons[index].uuid));
    myBeacon.setMajor(beacons[index].major);
    myBeacon.setMinor(beacons[index].minor);
    myBeacon.setTxPower(beacons[index].txPower);

    pAdvertising->setScanResponseData(myBeacon.getData());
    pAdvertising->start();

    Serial.printf("Switched to beacon: Major=%d, Minor=%d\n",
                 beacons[index].major, beacons[index].minor);
}

void setup() {
    Serial.begin(115200);

    BLEDevice::init("XIAO-MultiBeacon");
    pAdvertising = BLEDevice::getAdvertising();
    pAdvertising->setMinPreferred(0x06);
    pAdvertising->setMinPreferred(0x12);

    switchBeacon(0);
}

void loop() {
    unsigned long now = millis();

    if (now - lastSwitch >= SWITCH_INTERVAL) {
        lastSwitch = now;
        currentBeacon = (currentBeacon + 1) % 3;
        switchBeacon(currentBeacon);
    }

    delay(1000);
}
```

## nRF52 iBeacon (XIAO nRF52840/MG24)

```cpp
#include <bluefruit.h>

#define BEACON_UUID "87b99b2c-90fd-11e9-bc42-526af7764f64"
#define MAJOR_VERSION 1
#define MINOR_VERSION 1
#define TX_POWER -59

void setup() {
    Serial.begin(115200);

    // Initialize Bluefruit
    Bluefruit.begin();
    Bluefruit.setName("XIAO-nRFBeacon");

    // Set TX power (valid values: -40, -20, -16, -12, -8, -4, 0, 4)
    Bluefruit.setTxPower(TX_POWER);

    // Set up beacon
    uint8_t beaconUUID[16];
    strUUID2Bytes(BEACON_UUID, beaconUUID);

    // Start advertising
    Bluefruit.Advertising.setBeacon(
        beaconUUID,
        MAJOR_VERSION,
        MINOR_VERSION,
        TX_POWER
    );

    Bluefruit.Advertising.start();
    Serial.println("nRF52 iBeacon started!");
}

void loop() {
    delay(1000);
}

// Helper function to convert UUID string to bytes
void strUUID2Bytes(const char* uuid, uint8_t* bytes) {
    // Remove hyphens and convert to bytes
    // Simple implementation for example
    // In production, use proper UUID parsing
}
```

## Troubleshooting

### Beacon Not Detected

1. **Check BLE is enabled** on your phone
2. **Try different scanner app** (nRF Connect, LightBlue)
3. **Verify advertising interval** (try longer intervals)
4. **Check TX power** (too low may limit range)

### iPhone Detection Issues

1. **Ensure iBeacon format is correct**
2. **Use Apple's Company ID** (0x004C) for best compatibility
3. **Set proper TX power calibration**
4. **Advertising interval should be 100ms-1000ms**

### Battery Drains Quickly

1. **Increase advertising interval** (reduce frequency)
2. **Use deep sleep between advertising**
3. **Lower TX power**
4. **Minimize active advertising time**

### Advertising Stops After Some Time

1. **Check watchdog timer** settings
2. **Verify BLE stack is properly initialized**
3. **Add error handling** for BLE operations

## Best Practices

1. **TX Power Calibration**: Measure actual RSSI at 1 meter and set accordingly
2. **Unique UUIDs**: Use proper UUID generators for production
3. **Major/Minor Planning**: Design your ID hierarchy carefully
4. **Battery Optimization**: Use sleep modes for battery-powered beacons
5. **Privacy Considerations**: Be aware of tracking regulations

## Power Optimization

```cpp
// Lower power advertising setup
pAdvertising->setMinPreferred(0x06);  // Help with connections
pAdvertising->setMinPreferred(0x12);
pAdvertising->setScanFilter(false);   // Disable scan filtering

// Set advertising intervals (in 0.625ms units)
pAdvertising->setMinInterval(160);  // 100ms
pAdvertising->setMaxInterval(160);  // 100ms
```

## References

- [Apple iBeacon Specification](https://developer.apple.com/ibeacon/)
- [Google Eddystone Specification](https://github.com/google/eddystone)
- [ESP32 BLE Documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32c3/api-reference/bluetooth/esp_gap.html)
- [nRF52 Beacon Guide](https://learn.adafruit.com/introducing-adafruit-ble-feather/become-a-beacon)
