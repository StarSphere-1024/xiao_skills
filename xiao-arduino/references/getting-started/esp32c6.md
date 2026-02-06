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
| Architecture | RISC-V single-core up to 160MHz | RISC-V single-core 160MHz |
| Flash | 4MB | 8MB |
| RAM | 400KB | 512KB |
| WiFi | WiFi 4 (802.11b/g/n) | WiFi 6 (802.11ax) |
| BLE | BLE 5.0 | BLE 5.3 |
| Zigbee/Thread | No | Yes (802.15.4) |
| Native USB | Yes | Yes |

## Pin Mapping

| Pin | GPIO | Arduino | Function | Notes |
|-----|------|---------|----------|-------|
| D0 | GPIO0 | D0 | ADC | - |
| D1 | GPIO1 | D1 | ADC | - |
| D2 | GPIO2 | D2 | ADC | - |
| D3 | GPIO21 | D3 | GPIO | - |
| D4 | GPIO22 | D4 | I2C SDA | Default I2C SDA |
| D5 | GPIO23 | D5 | I2C SCL | Default I2C SCL |
| D6 | GPIO16 | D6 | UART TX | Default UART TX |
| D7 | GPIO17 | D7 | UART RX | Default UART RX |
| D8 | GPIO19 | D8 | SPI SCK | Default SPI SCK |
| D9 | GPIO20 | D9 | SPI MISO | Default SPI MISO |
| D10 | GPIO18 | D10 | SPI MOSI | Default SPI MOSI |
| USER_LED | GPIO15 | LED_BUILTIN | LED | User light |

## First Sketch

```cpp
void setup() {
    Serial.begin(115200);
    pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
    digitalWrite(LED_BUILTIN, HIGH);
    Serial.println("LED ON");
    delay(1000);
    digitalWrite(LED_BUILTIN, LOW);
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
