# XIAO SAMD21 Reference

**Chipset**: SAMD21G18 (ARM Cortex-M0+ 48MHz)
**Wireless**: None
**Flash**: 256KB
**RAM**: 32KB

## Pin Definitions

| Pin | SAMD Pin | Arduino | Functions | Notes |
|-----|----------|---------|-----------|-------|
| D0 | PA02 | A0 | ADC | EXTINT[2] |
| D1 | PA04 | A1 | ADC | EXTINT[4] |
| D2 | PA10 | A2 | ADC | EXTINT[10] |
| D3 | PA11 | A3 | ADC | EXTINT[11] |
| D4 | PA08 | A4 | I2C SDA, ADC | Default I2C SDA (XIAO standard) |
| D5 | PA09 | A5 | I2C SCL, ADC | Default I2C SCL (XIAO standard) |
| D6 | PB08 | A6 | UART TX, ADC | Default UART TX (XIAO standard) |
| D7 | PB09 | A7 | UART RX, ADC | Default UART RX (XIAO standard) |
| D8 | PA07 | A8 | SPI SCK, ADC | Default SPI SCK (XIAO standard) |
| D9 | PA05 | A9 | SPI MISO, ADC | Default SPI MISO (XIAO standard) |
| D10 | PA06 | A10 | SPI MOSI, ADC | Default SPI MOSI (XIAO standard) |

### Other Board Pins

- **TX_LED**: PA19
- **RX_LED**: PA18
- **USER_LED / LED_BUILTIN**: PA17 (note: on some cores this LED is active-low)

## XIAO Standard Peripheral Mapping

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | SAMD Pin |
|-----------|-----|----------|
| **I2C SDA** | D4 | PA08 |
| **I2C SCL** | D5 | PA09 |
| **UART TX** | D6 | PB08 |
| **UART RX** | D7 | PB09 |
| **SPI SCK** | D8 | PA07 |
| **SPI MISO** | D9 | PA05 |
| **SPI MOSI** | D10 | PA06 |

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
