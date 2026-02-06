# XIAO ESP32C5 Reference

**Chipset**: ESP32-C5 (RISC-V, dual-core 240MHz)
**Wireless**: WiFi 6, BLE 5.4, Zigbee
**Flash**: 8MB
**RAM**: 512KB SRAM

## Pin Definitions

| Pin | GPIO | Arduino | Functions | Notes |
|-----|------|---------|-----------|-------|
| D0 | GPIO8 | D0 | - | Strapping pin - must be HIGH at boot |
| D1 | GPIO9 | D1 | - | Connected to BOOT button |
| D2 | GPIO7 | D2 | - | - |
| D3 | GPIO6 | D3 | - | - |
| D4 | GPIO5 | D4 | I2C SDA | Default I2C SDA (XIAO standard) |
| D5 | GPIO4 | D5 | I2C SCL | Default I2C SCL (XIAO standard) |
| D6 | GPIO3 | D6 | UART TX | Default UART TX (XIAO standard) |
| D7 | GPIO10 | D7 | UART RX | Default UART RX (XIAO standard) |
| D8 | GPIO1 | D8 | SPI SCK | Default SPI SCK (XIAO standard) |
| D9 | GPIO2 | D9 | SPI MISO | Default SPI MISO (XIAO standard) |
| D10 | GPIO18 | D10 | SPI MOSI | Default SPI MOSI (XIAO standard) |
| A0 | GPIO0 | A0 | ADC0 | Analog input |
| A1 | GPIO1 | A1 | ADC1 | Analog input |
| A2 | GPIO2 | A2 | ADC2 | Analog input |
| A3 | GPIO3 | A3 | ADC3 | Analog input |

## XIAO Standard Peripheral Mapping

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | GPIO |
|-----------|-----|------|
| **I2C SDA** | D4 | GPIO5 |
| **I2C SCL** | D5 | GPIO4 |
| **UART TX** | D6 | GPIO3 |
| **UART RX** | D7 | GPIO10 |
| **SPI SCK** | D8 | GPIO1 |
| **SPI MISO** | D9 | GPIO2 |
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
#define TX_PIN D6  // GPIO3
#define RX_PIN D7  // GPIO10

void setup() {
    Serial1.begin(115200, SERIAL_8N1, TX_PIN, RX_PIN);
}
```

## Key Features

- WiFi 6 (802.11ax) support
- BLE 5.4 with long range
- Zigbee and Thread support
- More RAM than ESP32C3
- Similar pinout to ESP32C3

## Important Pin Warnings

### D0 (GPIO8) - Strapping Pin
- Must be HIGH at boot for download mode
- Add pull-up resistor if using for I/O

### D1 (GPIO9) - BOOT Button
- Connected to BOOT button
- Best used as switch input

## Power

- **Input**: 5V via USB
- **Operating Voltage**: 3.3V
- **Deep Sleep**: ~10mA (with WiFi off)
- **Light Sleep**: ~2mA

## Arduino Board Manager

- Board: "Seeed XIAO ESP32C5"
- Package: esp32 by Espressif Systems
