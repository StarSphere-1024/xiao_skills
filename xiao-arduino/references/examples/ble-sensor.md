# BLE Sensor Example (Arduino)

## Complete BLE Sensor with Battery Monitoring

This example shows a complete BLE sensor node with battery monitoring and notifications.

## Hardware

- XIAO nRF52840
- BME280 sensor (I2C)
- LED indicator
- Battery (optional)

## Code

```cpp
#include <ArduinoBLE.h>
#include <Adafruit_BME280.h>
#include <ArduinoJson.h>

// BLE Service
#define ENV_SENSING_SERVICE "181A"
#define TEMPERATURE_CHAR "2A6E"
#define HUMIDITY_CHAR "2A6F"
#define PRESSURE_CHAR "2A6D"
#define BATTERY_SERVICE "180F"
#define BATTERY_CHAR "2A19"

// Sensor
Adafruit_BME280 bme;
TwoWire I2CBME = TwoWire(0);

// BLE Service
BLEService envService(ENV_SENSING_SERVICE);
BLEFloatCharacteristic tempChar(TEMPERATURE_CHAR,
                                  BLERead | BLENotify);
BLEFloatCharacteristic humidChar(HUMIDITY_CHAR,
                                  BLERead | BLENotify);
BLEFloatCharacteristic pressChar(PRESSURE_CHAR,
                                  BLERead | BLENotify);

// Battery Service
BLEService batteryService(BATTERY_SERVICE);
BLEUnsignedCharCharacteristic batteryChar(BATTERY_CHAR,
                                          BLERead | BLENotify);

// Timing
unsigned long lastRead = 0;
const long readInterval = 2000;  // 2 seconds

// Battery
const int BATTERY_PIN = A0;
const float BATTERY_DIVIDER = 2.0;

// Device name
const char* DEVICE_NAME = "XIAO-Sensor";

void setup() {
    Serial.begin(115200);
    delay(1000);

    Serial.println("=== XIAO BLE Sensor ===");

    // Initialize sensor
    I2CBME.begin(D0, D1);
    if (!bme.begin(0x76, &I2CBME)) {
        Serial.println("Could not find BME280!");
        while (1) {
            delay(100);
        }
    }
    Serial.println("BME280 initialized");

    // Initialize BLE
    if (!BLE.begin()) {
        Serial.println("BLE initialization failed!");
        while (1);
    }

    // Set device name and appearance
    BLE.setLocalName(DEVICE_NAME);
    BLE.setDeviceName(DEVICE_NAME);
    BLE.setAppearance(0x0540);  // Environmental sensor

    // Configure battery service
    batteryService.addCharacteristic(batteryChar);
    BLE.addService(batteryService);
    batteryChar.writeValue(100);  // Initial value

    // Configure environment service
    envService.addCharacteristic(tempChar);
    envService.addCharacteristic(humidChar);
    envService.addCharacteristic(pressChar);
    BLE.addService(envService);

    // Set initial values
    tempChar.writeValue(0.0);
    humidChar.writeValue(0.0);
    pressChar.writeValue(0.0);

    // Start advertising
    BLE.setAdvertisingInterval(160);  // 100ms
    BLE.setConnectable(true);
    BLE.discoverable();

    BLEAdvertisingData advData;
    advData.setServiceUUID(envService);
    advData.setLocalName(DEVICE_NAME);
    BLE.setAdvertisingData(advData);

    BLE.advertise();

    Serial.println("BLE advertising...");
    Serial.println("Connect to: " + String(DEVICE_NAME));
}

void loop() {
    BLEDevice central = BLE.central();

    if (central) {
        Serial.print("Connected to: ");
        Serial.println(central.address());
        Serial.print("RSSI: ");
        Serial.println(central.rssi());

        // LED on when connected
        pinMode(LED_BUILTIN, OUTPUT);
        digitalWrite(LED_BUILTIN, HIGH);

        while (central.connected()) {
            unsigned long now = millis();

            if (now - lastRead >= readInterval) {
                lastRead = now;
                readAndPublish();
            }
        }

        // LED off when disconnected
        digitalWrite(LED_BUILTIN, LOW);
        Serial.println("Disconnected");
    }
}

void readAndPublish() {
    // Read sensor
    float temp = bme.readTemperature();
    float humid = bme.readHumidity();
    float press = bme.readPressure() / 100.0F;

    // Read battery
    int batt = readBattery();

    // Update characteristics
    tempChar.writeValue(temp);
    humidChar.writeValue(humid);
    pressChar.writeValue(press);
    batteryChar.writeValue(batt);

    // Print to serial
    Serial.printf("Temp: %.2f°C, Hum: %.2f%%, Press: %.2fhPa, Batt: %d%%\n",
                  temp, humid, press, batt);
}

int readBattery() {
    // Read battery voltage (simple divider)
    int adc = analogRead(BATTERY_PIN);
    float voltage = (adc / 1024.0) * 3.6 * BATTERY_DIVIDER;

    // Convert to percentage (approximate)
    int percent = map(voltage * 100, 300, 420, 0, 100);
    percent = constrain(percent, 0, 100);

    return percent;
}
```

## nRF Connect App

### Testing with nRF Connect

1. Install nRF Connect on mobile
2. Scan for "XIAO-Sensor"
3. Connect to device
4. Subscribe to characteristics:
   - Temperature (2A6E)
   - Humidity (2A7D)
   - Pressure (2A6D)
   - Battery (2A19)

## Custom Characteristics

### Adding Custom Data

```cpp
// Custom service UUID
BLEService customService("19B10000-E8F2-537E-4F6C-D104768A1214");

// Custom characteristic with 128-bit UUID
BLECharacteristic customData("19B10001-E8F2-537E-4F6C-D104768A1214",
                             BLERead | BLENotify,
                             32);  // Max 32 bytes

void setup() {
    // Add custom service
    BLE.addService(customService);
    customService.addCharacteristic(customData);

    // Set initial value
    uint8_t data[32] = "Custom data";
    customData.writeValue(data, 32);
}
```

## BLE Bonding (Pairing)

```cpp
void setup() {
    BLE.begin();

    // Enable bonding
    BLE.setAdvertisingInterval(160);
    BLE.setConnectable(true);

    // Security settings
    BLE.setBonding(true);
    BLE.setPairing(true);

    BLE.advertise();
}
```

## Connection Parameters

### Optimize for Battery

```cpp
void setup() {
    BLE.begin();

    // Set connection parameters for battery optimization
    // Min interval: 500ms (0x0004 * 1.25ms)
    // Max interval: 1000ms (0x0008 * 1.25ms)
    // Slave latency: 4
    // Timeout: 20 seconds (0x0BB0 * 10ms)
    BLE.setConnectionParameters(0x0004, 0x0008, 4, 0x0BB0);

    BLE.advertise();
}
```

## OTA Service

### Enable OTA Updates

```cpp
#include <ArduinoOTA.h>

void setup() {
    Serial.begin(115200);

    // Initialize BLE OTA
    ArduinoOTA.begin(BLE);

    Serial.println("OTA enabled");
}

void loop() {
    // Handle OTA
    ArduinoOTA.poll();
}
```

## Features

1. **Environmental sensing**: Temperature, humidity, pressure
2. **Battery monitoring**: Percentage based on voltage
3. **BLE notifications**: Real-time updates
4. **LED indication**: Connection status
5. **RSSI monitoring**: Signal strength
6. **Standard services**: Environment and Battery services

## Serial Output

```cpp
=== XIAO BLE Sensor ===
BME280 initialized
BLE advertising...
Connect to: XIAO-Sensor
Connected to: AA:BB:CC:DD:EE:FF
RSSI: -45
Temp: 23.50°C, Hum: 45.20%, Press: 1013.25hPa, Batt: 85%
Temp: 23.60°C, Hum: 45.30%, Press: 1013.20hPa, Batt: 85%
Disconnected
```

## Data Logging

### Log to SD Card

```cpp
#include <SD.h>

#define SD_CS D4

File logFile;

void logData(float temp, float humid, float press, int batt) {
    logFile = SD.open("/sensor.log", FILE_WRITE);
    if (logFile) {
        logFile.printf("%lu,%.2f,%.2f,%.2f,%d\n",
                       millis(), temp, humid, press, batt);
        logFile.close();
    }
}
```

## Custom Advertising Data

```cpp
void setup() {
    BLE.begin();

    // Custom advertising data
    uint8_t advData[] = {
        0x02, 0x01, 0x06,  // Flags
        0x03, 0x03, 0x0A, 0x18,  // Env service UUID
        0x09, 0x09, 'X', 'I', 'A', 'O', '-', 'S', 'E', 'N'  // Name
    };

    BLE.setAdvertisingData(advData, sizeof(advData));
    BLE.advertise();
}
```

## Troubleshooting

### Can't discover device

1. Check advertising interval (not too short)
2. Verify device name is set
3. Check for other BLE interference
4. Restart nRF Connect app

### Connection drops

1. Adjust connection parameters
2. Check power supply stability
3. Verify data isn't too large
4. Monitor connection events

### Battery percentage wrong

1. Calibrate voltage divider
2. Check ADC reference voltage
3. Adjust mapping values
4. Test with known voltage

### Characteristics not updating

1. Verify characteristic properties (Notify)
2. Check client is subscribed
3. Monitor for BLE errors
4. Ensure notifications enabled

## Best Practices

1. **Advertising**: Use minimum interval for battery
2. **Data**: Update only on change
3. **Notifications**: Use sparingly for battery
4. **Bonding**: Enable for secure connections
5. **OTA**: Keep firmware updates small
