# XIAO ESP32S3 Arduino Getting Started

## Board Manager Setup

1. Open Arduino IDE 2.x
2. Add to "Additional Boards Manager URLs":
   ```
   https://espressif.github.io/arduino-esp32/package_esp32_index.json
   ```
3. Install "esp32 by Espressif Systems"
4. Select: **Tools > Board > esp32 > Seeed XIAO_ESP32S3**

## Key Features

| Feature | Description |
|---------|-------------|
| CPU | Xtensa dual-core 240MHz |
| Flash | 8MB (standard) / 16MB (Sense) |
| RAM | 512KB SRAM |
| WiFi | WiFi 4 (802.11b/g/n) |
| BLE | BLE 5.0 + |
| AI | Built-in AI accelerator |
| USB | USB-OTG support |

## Pin Mapping

| Pin | GPIO | Arduino | Function | Notes |
|-----|------|---------|----------|-------|
| D0 | GPIO1 | D0 | ADC | - |
| D1 | GPIO2 | D1 | ADC | - |
| D2 | GPIO3 | D2 | ADC | SD CS (Sense) |
| D3 | GPIO4 | D3 | ADC | - |
| D4 | GPIO5 | D4 | I2C SDA | Default I2C SDA |
| D5 | GPIO6 | D5 | I2C SCL | Default I2C SCL |
| D6 | GPIO43 | D6 | UART TX | Default UART TX |
| D7 | GPIO44 | D7 | UART RX | Default UART RX |
| D8 | GPIO7 | D8 | SPI SCK | Default SPI SCK |
| D9 | GPIO8 | D9 | SPI MISO | Default SPI MISO |
| D10 | GPIO10 | D10 | SPI MOSI | Default SPI MOSI |
| USER_LED | GPIO21 | LED_BUILTIN | LED | User LED |
| BOOT | GPIO0 | BOOT | GPIO | BOOT button |

## I2C

```cpp
#include <Wire.h>

void setup() {
    Wire.begin();  // D4 (SDA), D5 (SCL)
}

void loop() {}
```

## Camera (Sense Model Only)
The Sense model has a camera connector. For a complete, correct pin mapping and working examples, use the official Seeed Studio XIAO ESP32S3 Sense camera examples/documentation.

## Dual Core

```cpp
void setup() {
    // Core 0 runs setup() and loop()
    // Use second core for tasks:
    xTaskCreatePinnedToCore(
        loopTask,   // Function
        "loopTask", // Name
        4096,       // Stack size
        NULL,       // Parameter
        1,          // Priority
        NULL,       // Handle
        1           // Core 1
    );
}

void loopTask(void *pvParameters) {
    while(1) {
        // Code running on Core 1
        delay(1000);
    }
}
```

## AI Acceleration

```cpp
#include <esp_timer.h>
#include <dl_lib.h>

// Load TensorFlow Lite Micro model
// See ESP-DL library for AI inference
```

## USB Serial + Hardware Serial

```cpp
#include <HardwareSerial.h>

HardwareSerial MySerial0(0);  // D6/D7
HardwareSerial MySerial1(1);  // D9/D10

void setup() {
    Serial.begin(115200);      // USB
    MySerial0.begin(115200, SERIAL_8N1, -1, -1);
    MySerial1.begin(115200, SERIAL_8N1, 9, 10);
}

void loop() {
    Serial.println("USB");
    MySerial0.println("UART0");
    MySerial1.println("UART1");
    delay(1000);
}
```

## Deep Sleep

```cpp
#include "esp_sleep.h"

void setup() {
    esp_sleep_enable_timer_wakeup(60 * 1000000);
    esp_deep_sleep_start();
}
```

## Troubleshooting

### Camera not detected (Sense)

1. Check camera module connection
2. Verify correct pin mapping
3. Try different I2C address for camera init

### Boot button issue

BOOT button is GPIO0. Use as input with caution.

### Memory issues

ESP32S3 has more RAM. Use `ps_malloc()` for large buffers.
