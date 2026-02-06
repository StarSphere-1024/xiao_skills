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
| D0 | GPIO44 | D0 | - | - |
| D1 | GPIO43 | D1 | - | - |
| D2 | GPIO5 | D2 | - | - |
| D3 | GPIO6 | D3 | - | - |
| D4 | GPIO4 | D4 | - | - |
| D5 | GPIO7 | D5 | - | - |
| D6 | GPIO8 | D6 | I2C SDA | - |
| D7 | GPIO9 | D7 | I2C SCL | - |
| D8 | GPIO38 | D8 | - | - |
| D9 | GPIO39 | D9 | BOOT button | - |
| D10 | GPIO37 | D10 | - | - |
| A0 | GPIO1 | A0 | ADC0 | - |
| A1 | GPIO2 | A1 | ADC1 | - |
| A2 | GPIO3 | A2 | ADC2 | - |
| A3 | GPIO4 | A3 | ADC3 | Shared with D4 |

## I2C

```cpp
#include <Wire.h>

void setup() {
    Wire.begin();  // D6 (SDA), D7 (SCL)
}

void loop() {}
```

## Camera (Sense Model Only)

```cpp
#include "esp_camera.h"

// Camera pins for XIAO ESP32S3 Sense
#define PWDN_GPIO_NUM     -1
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM      15
#define SIOD_GPIO_NUM      6   // SDA
#define SIOC_GPIO_NUM      7   // SCL
#define Y8_GPIO_NUM        9
#define Y7_GPIO_NUM        8
#define Y6_GPIO_NUM        10
#define Y5_GPIO_NUM        12
#define Y4_GPIO_NUM        11
#define Y3_GPIO_NUM        5
#define Y2_GPIO_NUM        3
#define VSYNC_GPIO_NUM     1
#define HREF_GPIO_NUM      0
#define PCLK_GPIO_NUM      2

void setup() {
    camera_config_t config;
    config.pin_pwdn = PWDN_GPIO_NUM;
    config.pin_reset = RESET_GPIO_NUM;
    config.pin_xclk = XCLK_GPIO_NUM;
    config.pin_sscb = SIOD_GPIO_NUM;
    config.pin_sscc = SIOC_GPIO_NUM;
    // ... (see Sense guide)
    esp_camera_init(&config);
}
```

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

D9 is connected to BOOT button. Use as input with caution.

### Memory issues

ESP32S3 has more RAM. Use `ps_malloc()` for large buffers.
