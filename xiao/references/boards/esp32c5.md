# XIAO ESP32C5 Reference

**Chipset**: ESP32-C5 (RISC-V 32-bit, up to 240 MHz)
**Wireless**: 2.4 GHz & 5 GHz dual-band Wi‑Fi 6, Bluetooth 5 (LE)
**Flash**: 8MB
**PSRAM**: 8MB
**SRAM**: 384KB (on-chip)

## Pin Definitions

| Pin | GPIO | Arduino | Functions | Notes |
|-----|------|---------|-----------|-------|
| D0 | GPIO1 | D0 | GPIO, ADC | - |
| D1 | GPIO0 | D1 | GPIO | Connected to BOOT button (strapping) |
| D2 | GPIO25 | D2 | GPIO | - |
| D3 | GPIO7 | D3 | GPIO | SDIO_DATA1 (alt) |
| D4 | GPIO23 | D4 | I2C SDA | Default I2C SDA (XIAO standard) |
| D5 | GPIO24 | D5 | I2C SCL | Default I2C SCL (XIAO standard) |
| D6 | GPIO11 | D6 | UART TX | Default UART TX (XIAO standard) |
| D7 | GPIO12 | D7 | UART RX | Default UART RX (XIAO standard) |
| D8 | GPIO8 | D8 | SPI SCK | Default SPI SCK (XIAO standard) |
| D9 | GPIO9 | D9 | SPI MISO | Default SPI MISO (XIAO standard) |
| D10 | GPIO10 | D10 | SPI MOSI | Default SPI MOSI (XIAO standard) |
| MTDO | GPIO5 | MTDO | JTAG | Reserve for debugging when possible |
| MTDI | GPIO3 | MTDI | JTAG, ADC | Reserve for debugging when possible |
| MTCK | GPIO4 | MTCK | JTAG, ADC | Reserve for debugging when possible |
| MTMS | GPIO2 | MTMS | JTAG | Reserve for debugging when possible |
| ADC_BAT | GPIO6 | ADC_BAT | ADC | Battery voltage sense (via divider) |
| ADC_CTRL | GPIO26 | ADC_CTRL | GPIO | Enables/disables battery measurement circuit |
| BOOT | GPIO28 | BOOT | GPIO | Enter Boot Mode |
| USER_LED | GPIO27 | LED_BUILTIN | LED | User LED |

## XIAO Standard Peripheral Mapping

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | GPIO |
|-----------|-----|------|
| **I2C SDA** | D4 | GPIO23 |
| **I2C SCL** | D5 | GPIO24 |
| **UART TX** | D6 | GPIO11 |
| **UART RX** | D7 | GPIO12 |
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
#define TX_PIN D6  // GPIO11
#define RX_PIN D7  // GPIO12

void setup() {
    Serial1.begin(115200, SERIAL_8N1, TX_PIN, RX_PIN);
}
```

## Key Features

- 2.4 GHz & 5 GHz dual-band Wi‑Fi 6 (802.11ax) support
- Bluetooth 5 (LE)
- 8MB Flash + 8MB PSRAM
- Classic XIAO form-factor and expansion-board-friendly pin mapping

## Important Pin Warnings

### D1 (GPIO0) - BOOT / Strapping Pin
- Connected to the BOOT button
- Pulled LOW at reset enters download/boot mode; avoid forcing LOW during boot

### JTAG pins (MTMS/MTDI/MTCK/MTDO)
- Strongly recommended to reserve these pins for debugging (avoid using them as deep-sleep wake sources)

## Power

- **Input**: 5V via USB
- **Operating Voltage**: 3.3V
- **Battery**: supports 3.7V Li‑ion/Li‑po (via onboard charging/power path)

## Arduino Board Manager

- Board: "Seeed XIAO ESP32C5"
- Package: esp32 by Espressif Systems
