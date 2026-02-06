# XIAO ESP32C3 Arduino Getting Started

## Board Manager Setup

1. Open Arduino IDE 2.x
2. Go to **File > Preferences**
3. Add to "Additional Boards Manager URLs":
   ```cpp
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
   ```
4. Go to **Tools > Board > Boards Manager**
5. Search "esp32" and install "esp32 by Espressif Systems"
6. Select **Tools > Board > esp32 > Seeed XIAO_ESP32C3**

## First Sketch

```cpp
void setup() {
    Serial.begin(115200);
    pinMode(D10, OUTPUT);  // LED on D10 (connect an external LED)
}

void loop() {
    digitalWrite(D10, HIGH);
    Serial.println("LED ON");
    delay(1000);
    digitalWrite(D10, LOW);
    Serial.println("LED OFF");
    delay(1000);
}
```

## Important Board Settings

| Setting | Location | Value |
|---------|----------|-------|
| Upload Speed | Tools | 921600 |
| CPU Frequency | Tools | 80MHz (default) or 160MHz |
| USB CDC On Boot | Tools | Enabled (for Serial) |
| Flash Mode | Tools | QIO |

## Pin Quick Reference

| Pin | GPIO | Function | Notes |
|-----|------|----------|-------|
| D0 | GPIO2 | ADC (A0) | Strapping pin |
| D1 | GPIO3 | ADC (A1) | - |
| D2 | GPIO4 | ADC (A2) | - |
| D3 | GPIO5 | ADC (A3) | ADC2 (less reliable than ADC1 on some use-cases) |
| D4 | GPIO6 | I2C SDA | Default I2C SDA |
| D5 | GPIO7 | I2C SCL | Default I2C SCL |
| D6 | GPIO21 | UART TX | Default UART TX |
| D7 | GPIO20 | UART RX | Default UART RX |
| D8 | GPIO8 | SPI SCK | Strapping pin |
| D9 | GPIO9 | SPI MISO | BOOT / strapping pin |
| D10 | GPIO10 | SPI MOSI | - |

## Known Issues

### D6 Pin Behavior
- Outputs UART data at startup
- HIGH in standby mode
- **Recommend**: Use as output only

### D8 Pin (Strapping)
- Must be HIGH at boot for download mode
- Add pull-up if using download boot

### D9 Pin (BOOT Button)
- Connected to BOOT button
- Pressing connects to GND
- Best for switch input

## WiFi First Test

```cpp
#include <WiFi.h>

const char* ssid = "your-SSID";
const char* password = "your-PASSWORD";

void setup() {
    Serial.begin(115200);
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println(WiFi.localIP());
}

void loop() {}
```

## BLE First Test

```cpp
#include <BLEDevice.h>
#include <BLEServer.h>

void setup() {
    BLEDevice::init("XIAO_ESP32C3");
    BLEServer *pServer = BLEDevice::createServer();
    BLEService *pService = pServer->createService("4fafc201-1fb5-459e-8fcc-c5c9c331914b");
    pService->start();
    BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
    pAdvertising->start();
    Serial.println("BLE advertising started");
}

void loop() {}
```

## Upload Troubleshooting

### "Failed to connect"
1. Hold BOOT button
2. Click Upload
3. Release BOOT button when "Connecting..." appears

### "A fatal error occurred"
1. Check USB cable (data + power required)
2. Try different USB port
3. Reinstall drivers (CH340/CP2102)

### "Permission denied"
1. Add user to dialout group (Linux)
2. Install CH340 driver (Windows)
3. Check port lock (another IDE using it)
