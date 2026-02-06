# XIAO ESP32C6 Arduino Getting Started

## Board Manager Setup

1. Open Arduino IDE 2.x
2. Add to "Additional Boards Manager URLs":
   ```cpp
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
   ```
3. Install "esp32 by Espressif Systems"
4. Select: **Tools > Board > esp32 > Seeed XIAO_ESP32C6**

## Key Differences from ESP32C3

| Feature | ESP32C3 | ESP32C6 |
|---------|----------|----------|
| Architecture | RISC-V dual-core 240MHz | RISC-V single-core 160MHz |
| Flash | 4MB | 8MB |
| RAM | 400KB | 512KB |
| WiFi | WiFi 4 (802.11b/g/n) | WiFi 6 (802.11ax) |
| BLE | BLE 5.0 | BLE 5.3 |
| Zigbee/Thread | No | Yes (802.15.4) |
| Native USB | Yes | Yes |

## Pin Mapping

| Pin | GPIO | Arduino | Function | Notes |
|-----|------|---------|----------|-------|
| D0 | GPIO8 | - | I2C SDA | Strapping pin |
| D1 | GPIO9 | - | I2C SCL | BOOT button |
| D2 | GPIO7 | - | UART RX | - |
| D3 | GPIO6 | - | UART TX | Recommend output only |
| D4 | GPIO5 | - | SPI CS | - |
| D5 | GPIO4 | - | SPI MOSI | - |
| D6 | GPIO3 | - | SPI MISO | - |
| D7 | GPIO10 | - | SPI SCK | - |
| A0 | GPIO1 | - | ADC0 | - |
| A1 | GPIO0 | - | ADC1 | Strapping pin |
| A2 | GPIO2 | - | ADC2 | - |
| A3 | GPIO4 | - | ADC3 | Shared with D5 |

## First Sketch

```cpp
void setup() {
    Serial.begin(115200);
    pinMode(D10, OUTPUT);
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

## WiFi 6 Example

```cpp
#include <WiFi.h>

const char* ssid = "your-SSID";
const char* password = "your-PASSWORD";

void setup() {
    Serial.begin(115200);
    WiFi.mode(WIFI_STA);
    WiFi.setTxPower(10);  // Lower power for WiFi 6
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println(WiFi.localIP());
}

void loop() {}
```

## Zigbee Example

```cpp
// ESP32-C6 Zigbee requires ESP-Zigbee-SDK
// Not directly supported in Arduino
// Use ESP-IDF for Zigbee development
```

## Low Power

```cpp
#include "esp_sleep.h"
#include "WiFi.h"

void setup() {
    // Enable automatic light sleep
    esp_wifi_set_ps(WIFI_PS_MIN_MODEM);

    // Or deep sleep
    esp_sleep_enable_timer_wakeup(60 * 1000000);
    esp_deep_sleep_start();
}

void loop() {}
```

## Troubleshooting

Same as ESP32C3:
- Hold BOOT button while uploading
- Check USB cable (data + power)
- Install CH340/CP2102 drivers
