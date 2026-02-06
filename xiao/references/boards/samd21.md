# XIAO SAMD21 Reference

**Chipset**: SAMD21G18 (ARM Cortex-M0+ 48MHz)
**Wireless**: None
**Flash**: 256KB
**RAM**: 32KB

## Pin Definitions

| Pin | SAMD Pin | Arduino | Functions | Notes |
|-----|----------|---------|-----------|-------|
| D0 | PA02 | A0 | ADC | EXTINT[2] |
| D1 | PA03 | - | DAC/VREF | ADC output |
| D2 | PA04 | A1 | ADC | EXTINT[4] |
| D3 | PA05 | A9 | ADC | EXTINT[5] |
| D4 | PA06 | A10 | ADC | EXTINT[6] |
| D5 | PA07 | A8 | ADC | EXTINT[7] |
| D6 | PA08 | A4 | UART TX | Default UART TX (XIAO standard) |
| D7 | PA09 | A5 | UART RX | Default UART RX (XIAO standard) |
| D8 | PA10 | A2 | SPI SCK | Default SPI SCK (XIAO standard) |
| D9 | PA11 | A3 | SPI MISO | Default SPI MISO (XIAO standard) |
| D10 | PB10 | A6 | SPI MOSI | Default SPI MOSI (XIAO standard) |
| D11 | PB11 | A7 | SPI CS | SPI CS |

## XIAO Standard Peripheral Mapping

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | SAMD Pin |
|-----------|-----|----------|
| **I2C SDA** | D4 | PA06 |
| **I2C SCL** | D5 | PA07 |
| **UART TX** | D6 | PA08 |
| **UART RX** | D7 | PA09 |
| **SPI SCK** | D8 | PA10 |
| **SPI MISO** | D9 | PA11 |
| **SPI MOSI** | D10 | PB10 |

## I2C

```cpp
#include <Wire.h>

void setup() {
    Wire.begin();  // Uses D4 (SDA) and D5 (SCL) - XIAO standard
}
```

## SPI

```cpp
#include <SPI.h>

void setup() {
    SPI.begin();  // Uses D8 (SCK), D9 (MISO), D10 (MOSI) - XIAO standard
}
```

## UART

```cpp
#define TX_PIN D6  // PA08
#define RX_PIN D7  // PA09

void setup() {
    Serial1.begin(115200);
}
```

## ADC

- 6 external ADC channels: A0, A1, A2, A3, A4, A5
- 10-bit resolution (0-1023)
- Range: 0-3.3V
- 1 DAC output on PA03 (D1)

## SWD Debug

- **SWCLK**: PA30
- **SWDIO**: PA31

## Power

- **Input**: 5V via USB
- **Operating Voltage**: 3.3V
- **Active**: ~8mA
- **Sleep**: ~5µA

## Important Notes

- Native USB support
- No WiFi/BLE hardware
