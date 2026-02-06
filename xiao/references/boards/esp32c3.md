# XIAO ESP32C3 Reference

**Chipset**: ESP32-C3 (RISC-V 32-bit, up to 160 MHz)
**Wireless**: 2.4 GHz Wi‑Fi, Bluetooth 5 (LE)
**Flash**: 4MB
**RAM**: 400KB SRAM

## Pin Definitions

| Pin | GPIO | Arduino | Functions | Notes |
|-----|------|---------|-----------|-------|
| D0 | GPIO2 | D0 | GPIO, ADC | Strapping pin |
| D1 | GPIO3 | D1 | GPIO, ADC | - |
| D2 | GPIO4 | D2 | GPIO, ADC | - |
| D3 | GPIO5 | D3 | GPIO, ADC | ADC2 (less reliable than ADC1 on some use-cases) |
| D4 | GPIO6 | D4 | I2C SDA | Default I2C SDA (XIAO standard) |
| D5 | GPIO7 | D5 | I2C SCL | Default I2C SCL (XIAO standard) |
| D6 | GPIO21 | D6 | UART TX | Default UART TX (XIAO standard) |
| D7 | GPIO20 | D7 | UART RX | Default UART RX (XIAO standard) |
| D8 | GPIO8 | D8 | SPI SCK | Strapping pin |
| D9 | GPIO9 | D9 | SPI MISO | BOOT / strapping pin |
| D10 | GPIO10 | D10 | SPI MOSI | - |
| A0 | GPIO2 | A0 | ADC | Same as D0 |
| A1 | GPIO3 | A1 | ADC | Same as D1 |
| A2 | GPIO4 | A2 | ADC | Same as D2 |
| A3 | GPIO5 | A3 | ADC | Same as D3 (ADC2) |

## XIAO Standard Peripheral Mapping

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | GPIO |
|-----------|-----|------|
| **I2C SDA** | D4 | GPIO6 |
| **I2C SCL** | D5 | GPIO7 |
| **UART TX** | D6 | GPIO21 |
| **UART RX** | D7 | GPIO20 |
| **SPI SCK** | D8 | GPIO8 |
| **SPI MISO** | D9 | GPIO9 |
| **SPI MOSI** | D10 | GPIO10 |

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
#define TX_PIN D6  // GPIO21
#define RX_PIN D7  // GPIO20

void setup() {
    Serial1.begin(115200, SERIAL_8N1, TX_PIN, RX_PIN);
}
```

Note: USB serial (Serial) is enabled by default. Serial1 uses the XIAO standard pins.

## ADC

- 4 channels: A0, A1, A2, A3
- Range: 0-2500mV
- 12-bit resolution (0-4095)

## Important Pin Warnings

### Strapping pins
- GPIO2 (D0/A0), GPIO8 (D8), GPIO9 (D9) are strapping pins
- Avoid forcing these pins HIGH/LOW during reset/boot unless you know the boot-mode implications

### D9 (GPIO9) - BOOT button / strapping pin
- Connected to the BOOT button
- Best used as switch input; avoid holding LOW during boot unless entering download mode

### D6 (GPIO3) - UART TX
- Outputs UART data at startup if Serial1 is used
- Recommend using with proper pull-up/down

## PWM

All GPIO pins support PWM.

## Communication Interfaces

- **2x UART**: USB Serial + Serial1 (D6/D7)
- **1x I2C**: Hardware I2C on D4/D5
- **1x SPI**: Hardware SPI on D8/D9/D10
- **1x I2S**: Available (check ESP-IDF)

## Power

- **Input**: 5V via USB
- **Operating Voltage**: 3.3V
- **Deep Sleep**: ~10mA (with WiFi off)
- **Light Sleep**: ~2mA
- **Modem Sleep**: ~20mA (WiFi connected, idle)

## Board Size

- 20mm x 17.5mm
- Castellated contacts for soldering

## Arduino Board Manager

- Board: "Seeed XIAO ESP32C3"
- Package: esp32 by Espressif Systems
