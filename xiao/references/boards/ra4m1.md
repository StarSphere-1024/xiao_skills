# XIAO RA4M1 Reference

**Chipset**: RA4M1 (ARM Cortex-M33 120MHz)
**Wireless**: None
**Flash**: 512KB
**RAM**: 128KB

## Pin Definitions

| Pin | Chip Pin | Arduino | Functions | Notes |
|-----|----------|---------|-----------|-------|
| D0 | P014 | D0 | ADC | Analog input |
| D1 | P000 | D1 | ADC | Analog input |
| D2 | P001 | D2 | ADC | Analog input |
| D3 | P002 | D3 | ADC | Analog input |
| D4 | P206 | D4 | I2C SDA | Default I2C SDA (XIAO standard) |
| D5 | P100 | D5 | I2C SCL | Default I2C SCL (XIAO standard) |
| D6 | P302 | D6 | UART TX, I2C2 SDA | Default UART TX, Secondary I2C SDA (XIAO standard) |
| D7 | P301 | D7 | UART RX, I2C2 SCL | Default UART RX, Secondary I2C SCL (XIAO standard) |
| D8 | P111 | D8 | SPI SCK | Default SPI SCK (XIAO standard) |
| D9 | P110 | D9 | SPI MISO | Default SPI MISO (XIAO standard) |
| D10 | P109 | D10 | SPI MOSI | Default SPI MOSI (XIAO standard) |
| D11 | P408 | D11 | UART RX | Secondary UART |
| D12 | P409 | D12 | UART TX | Secondary UART |
| D13 | P013 | D13 | GPIO | - |
| D14 | P012 | D14 | GPIO | - |
| D15 | P101 | D15 | UART TX, I2C0 SDA | Secondary UART, Tertiary I2C SDA |
| D16 | P104 | D16 | UART RX, I2C0 SCL | Secondary UART, Tertiary I2C SCL |
| D17 | P102 | D17 | UART CRX | Secondary UART |
| D18 | P103 | D18 | UART CTX | Secondary UART |

## XIAO Standard Peripheral Mapping

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | Chip Pin |
|-----------|-----|----------|
| **I2C SDA** | D4 | P206 |
| **I2C SCL** | D5 | P100 |
| **UART TX** | D6 | P302 |
| **UART RX** | D7 | P301 |
| **SPI SCK** | D8 | P111 |
| **SPI MISO** | D9 | P110 |
| **SPI MOSI** | D10 | P109 |

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
#define TX_PIN D6  // P302
#define RX_PIN D7  // P301

void setup() {
    Serial1.begin(115200);
}
```

## ADC

- Multiple channels
- 12-bit resolution (0-4095)
- Range: 0-3.3V

## Power

- **Input**: 5V via USB
- **Operating Voltage**: 3.3V
- **Active**: ~15mA
- **Sleep**: ~10µA

## Multiple I2C Buses

RA4M1 supports 3 I2C interfaces:

| I2C Bus | SDA Pin | SCL Pin | Notes |
|---------|---------|---------|-------|
| Wire (I2C1) | D4 | D5 | Default XIAO I2C pins |
| I2C2 | D6 | D7 | Alternate function (shared with UART2) |
| I2C0 | D15 | D16 | Alternate function (shared with UART0) |

```cpp
#include <Wire.h>

// Using default I2C1 on D4/D5
void setup() {
    Wire.begin();
}

// Using secondary I2C2 on D6/D7 (requires alternate pin configuration)
// Note: Check RA4M1 Arduino library for specific API
```

## Key Features

- ARM Cortex-M33 core (same as Arduino UNO R4)
- More flash and RAM than SAMD21
- Multiple UART interfaces (UART0, UART2, UART9)
- Three I2C interfaces (D4/D5, D6/D7, D15/D16)
