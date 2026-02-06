# XIAO ESP32C6 Reference

**Chipset**: ESP32-C6 (RISC-V, single-core 160MHz)
**Wireless**: WiFi 6, Bluetooth 5 (LE), 802.15.4 (Zigbee/Thread)
**Flash**: 8MB
**RAM**: 512KB SRAM

## Pin Definitions

| Pin | GPIO | Arduino | Functions | Notes |
|-----|------|---------|-----------|-------|
| D0 | GPIO0 | D0 | GPIO, ADC | - |
| D1 | GPIO1 | D1 | GPIO, ADC | - |
| D2 | GPIO2 | D2 | GPIO, ADC | - |
| D3 | GPIO21 | D3 | GPIO | - |
| D4 | GPIO22 | D4 | I2C SDA | Default I2C SDA (XIAO standard) |
| D5 | GPIO23 | D5 | I2C SCL | Default I2C SCL (XIAO standard) |
| D6 | GPIO16 | D6 | UART TX | Default UART TX (XIAO standard) |
| D7 | GPIO17 | D7 | UART RX | Default UART RX (XIAO standard) |
| D8 | GPIO19 | D8 | SPI SCK | Default SPI SCK (XIAO standard) |
| D9 | GPIO20 | D9 | SPI MISO | Default SPI MISO (XIAO standard) |
| D10 | GPIO18 | D10 | SPI MOSI | Default SPI MOSI (XIAO standard) |
| USER_LED | GPIO15 | LED_BUILTIN | LED | User light |
| BOOT | GPIO9 | BOOT | GPIO | Enter boot mode |
| WIFI_ANT_CONFIG | GPIO14 | - | GPIO | RF switch port select |
| WIFI_ANT_POWER | GPIO3 | - | GPIO | RF switch power |

## XIAO Standard Peripheral Mapping

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | GPIO |
|-----------|-----|------|
| **I2C SDA** | D4 | GPIO22 |
| **I2C SCL** | D5 | GPIO23 |
| **UART TX** | D6 | GPIO16 |
| **UART RX** | D7 | GPIO17 |
| **SPI SCK** | D8 | GPIO19 |
| **SPI MISO** | D9 | GPIO20 |
| **SPI MOSI** | D10 | GPIO18 |

## I2C

```cpp
#include <Wire.h>

void setup() {
    Wire.begin();  // Uses D4 (SDA) and D5 (SCL) by default
}
```

## SPI

```cpp
#include <SPI.h>

void setup() {
    SPI.begin();  // Uses D8 (SCK), D9 (MISO), D10 (MOSI) by default
}
```

## UART

```cpp
#define TX_PIN D6  // GPIO16
#define RX_PIN D7  // GPIO17

void setup() {
    Serial1.begin(115200, SERIAL_8N1, TX_PIN, RX_PIN);
}
```

## Key Features

- WiFi 6 (802.11ax) support
- BLE 5.3
- Zigbee 3.0 and Thread
- Matter support
- Low power consumption

## Important Pin Warnings

### BOOT pin (GPIO9)
- Connected to BOOT button / boot mode
- Avoid forcing LOW during normal boot unless entering download mode

## Power

- **Input**: 5V via USB
- **Operating Voltage**: 3.3V
- **Deep Sleep**: ~10mA (with WiFi off)
- **Light Sleep**: ~2mA

## Arduino Board Manager

- Board: "Seeed XIAO ESP32C6"
- Package: esp32 by Espressif Systems
