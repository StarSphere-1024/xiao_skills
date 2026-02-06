# SPI Library for XIAO

## Overview

SPI (Serial Peripheral Interface) is a synchronous serial communication protocol used by many sensors, displays, and flash memory.

## XIAO Unified Standard

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | Description |
|-----------|-----|-------------|
| **SPI SCK** | D8 | Clock (default) |
| **SPI MISO** | D9 | Master In Slave Out (default) |
| **SPI MOSI** | D10 | Master Out Slave In (default) |
| **CS** | User-defined | Chip Select (varies per device) |

**Important**: D8, D9, D10 are the standard SPI pins across all XIAO boards. Use these pins for compatibility with XIAO expansion boards.

## Multiple SPI Buses

Some XIAO boards support multiple SPI interfaces:

| Board | SPI Interfaces | Default SPI | Secondary SPI Pins |
|-------|---------------|-------------|---------------------|
| ESP32C3/C5/C6 | 2 | D8/D9/D10 (SPI2) | Any GPIO with `SPI.begin(sck, miso, mosi, ss)` |
| ESP32S3 | 2 | D8/D9/D10 (SPI2) | Any GPIO with `SPI.begin(sck, miso, mosi, ss)` |
| nRF52840 | 1 | D8/D9/D10 | Not available |
| RP2040 | 2 | D8/D9/D10 (SPI0) | Alternate pins available |
| RP2350 | 2 | D8/D9/D10 (SPI0) | Alternate pins available |
| SAMD21 | 1 | D8/D9/D10 | Not available |
| MG24 | 2 | D8/D9/D10 (Primary) | D17/D16/D15/D18 (Secondary) |
| RA4M1 | 1 | D8/D9/D10 | Not available |

## Basic Usage

### Using Default SPI Pins (All XIAO Boards)

```cpp
#include <SPI.h>

// CS pin is user-defined, commonly D4 or D3
const int csPin = D4;

void setup() {
    SPI.begin();  // Uses D8 (SCK), D9 (MISO), D10 (MOSI)
    pinMode(csPin, OUTPUT);
    digitalWrite(csPin, HIGH);  // Deselect device
}

void loop() {
    // Select device
    digitalWrite(csPin, LOW);

    // Transfer data
    byte data = SPI.transfer(0x00);

    // Deselect device
    digitalWrite(csPin, HIGH);

    delay(100);
}
```

## Custom SPI Pins (ESP32 Series)

ESP32C3, ESP32C5, ESP32C6, and ESP32S3 can use any GPIO pins for SPI.

```cpp
#include <SPI.h>

// Define custom pins
#define SPI_SCK   D4
#define SPI_MISO  D5
#define SPI_MOSI  D6
#define SPI_CS    D7

SPIClass spi(HSPI);  // Use HSPI for ESP32C3/S3

void setup() {
    // Begin with custom pins
    spi.begin(SPI_SCK, SPI_MISO, SPI_MOSI, SPI_CS);

    pinMode(SPI_CS, OUTPUT);
    digitalWrite(SPI_CS, HIGH);
}

void loop() {
    digitalWrite(SPI_CS, LOW);
    byte data = spi.transfer(0x00);
    digitalWrite(SPI_CS, HIGH);
    delay(100);
}
```

## Multiple SPI (RP2040/RP2350)

RP2040 and RP2350 support multiple SPI buses with flexible pin assignment.

```cpp
#include <SPI.h>

SPIClass SPI1(HSPI);  // Secondary SPI
SPIClass SPI2(VSPI);  // Another SPI instance

void setup() {
    // Default SPI uses D8/D9/D10
    SPI.begin();

    // Secondary SPI with custom pins
    SPI1.begin(D1, D2, D3, D4);  // SCK, MISO, MOSI, CS
}

void loop() {
    // Use default SPI
    digitalWrite(D4, HIGH);
    byte data1 = SPI.transfer(0x00);
    digitalWrite(D4, LOW);

    // Use secondary SPI
    byte data2 = SPI1.transfer(0x00);
}
```

## Multiple SPI Devices

Share SPI bus with multiple devices using separate CS pins.

```cpp
#include <SPI.h>

const int cs1 = D4;
const int cs2 = D3;

void setup() {
    SPI.begin();  // D8, D9, D10
    pinMode(cs1, OUTPUT);
    pinMode(cs2, OUTPUT);
    digitalWrite(cs1, HIGH);
    digitalWrite(cs2, HIGH);
}

void readDevice1() {
    digitalWrite(cs1, LOW);
    byte data = SPI.transfer(0x00);
    digitalWrite(cs1, HIGH);
}

void readDevice2() {
    digitalWrite(cs2, LOW);
    byte data = SPI.transfer(0x00);
    digitalWrite(cs2, HIGH);
}
```

## SPI Settings

### SPISettings

```cpp
void setup() {
    // Begin with custom settings
    SPI.beginTransaction(SPISettings(
        1000000,      // Clock frequency (Hz)
        MSBFIRST,     // Bit order: MSBFIRST or LSBFIRST
        SPI_MODE0     // SPI mode: 0, 1, 2, or 3
    ));

    // Transfer
    byte data = SPI.transfer(0x00);

    SPI.endTransaction();
}
```

### SPI Modes

| Mode | CPOL | CPHA | Clock Idle | Data Capture |
|------|------|------|------------|--------------|
| 0 | 0 | 0 | LOW | Rising edge |
| 1 | 0 | 1 | LOW | Falling edge |
| 2 | 1 | 0 | HIGH | Falling edge |
| 3 | 1 | 1 | HIGH | Rising edge |

**Common modes by device type:**
- Most sensors: SPI_MODE0
- SD cards: SPI_MODE0
- Some displays: SPI_MODE3

## Common SPI Devices

### SD Card (SPI Mode)

```cpp
#include <SD.h>
#include <SPI.h>

const int chipSelect = D4;

void setup() {
    SPI.begin();  // D8, D9, D10
    if (!SD.begin(chipSelect, SPI, 25000000)) {
        Serial.println("SD card failed!");
        return;
    }

    File dataFile = SD.open("datalog.txt", FILE_WRITE);
    if (dataFile) {
        dataFile.println("Data: 123");
        dataFile.close();
    }
}

void loop() {}
```

### BMP280 Pressure Sensor

```cpp
#include <SPI.h>
#include <Adafruit_BMP280.h>

#define BMP_CS D4

Adafruit_BMP280 bmp;

void setup() {
    SPI.begin();  // D8, D9, D10
    if (!bmp.begin(BMP_CS)) {
        Serial.println("BMP280 not found!");
        while (1);
    }
}

void loop() {
    Serial.print("Temperature: ");
    Serial.print(bmp.readTemperature());
    Serial.println(" *C");

    Serial.print("Pressure: ");
    Serial.print(bmp.readPressure() / 100);
    Serial.println(" hPa");

    delay(2000);
}
```

### W25Qxx Flash Memory

```cpp
#include <SPI.h>

#define FLASH_CS D4

SPIClass spi(HSPI);  // For ESP32 with custom pins

void setup() {
    spi.begin(D8, D9, D10, FLASH_CS);
    pinMode(FLASH_CS, OUTPUT);
    digitalWrite(FLASH_CS, HIGH);
}

void flashReadID() {
    digitalWrite(FLASH_CS, LOW);
    spi.transfer(0x9F);  // JEDEC ID command
    byte id1 = spi.transfer(0x00);
    byte id2 = spi.transfer(0x00);
    byte id3 = spi.transfer(0x00);
    digitalWrite(FLASH_CS, HIGH);

    Serial.printf("Flash ID: %02X %02X %02X\n", id1, id2, id3);
}

void loop() {
    flashReadID();
    delay(5000);
}
```

### ST7735 TFT Display

```cpp
#include <SPI.h>
#include <Adafruit_ST7735.h>
#include <Adafruit_GFX.h>

#define TFT_CS   D4
#define TFT_DC   D3
#define TFT_RST  D2

Adafruit_ST7735 tft = Adafruit_ST7735(TFT_CS, TFT_DC, TFT_RST);

void setup() {
    SPI.begin();  // D8, D9, D10
    tft.initR(INITR_144GREENTAB);
    tft.fillScreen(ST7735_BLACK);
    tft.setTextColor(ST7735_WHITE);
    tft.setTextSize(2);
    tft.println("Hello XIAO!");
}

void loop() {}
```

## High Speed SPI (ESP32)

```cpp
#include <SPI.h>

void setup() {
    SPI.begin();

    // 80MHz clock (ESP32 maximum)
    SPI.beginTransaction(SPISettings(80000000, MSBFIRST, SPI_MODE0));

    // Transfer data
    byte data = SPI.transfer(0x00);

    SPI.endTransaction();
}

void loop() {}
```

## Troubleshooting

### No response from device

1. Check wiring (MOSI to MOSI, MISO to MISO, SCK to SCK)
2. Verify CS pin is correct
3. Check SPI mode (MODE0 is most common)
4. Try lower clock speed
5. Verify device is powered (3.3V)

### Wrong data received

1. Check bit order (MSBFIRST vs LSBFIRST)
2. Verify SPI mode matches device requirements
3. Check clock polarity and phase
4. Ensure correct CS pin timing

### Multiple devices conflict

1. Ensure each device has dedicated CS pin
2. Only one CS LOW at a time
3. Add delay between chip selections if needed

### ESP32 SPI Issues

For ESP32 boards, use HSPI for custom pins:
```cpp
SPIClass spi(HSPI);
spi.begin(sck, miso, mosi, ss);
```

### SD Card Issues

1. Try lower clock speed: `SD.begin(cs, SPI, 16000000)`
2. Verify CS pin is not conflicting
3. Check SD card formatting (FAT32)
4. Ensure 3.3V power supply
