# SD Card Library for XIAO

## Overview

SD cards using SPI interface for data logging and storage.

## Wiring (SPI Mode)

| SD Pin | XIAO Pin (ESP32C3) | XIAO Pin (RP2040) |
|--------|-------------------|-------------------|
| CS | D4 | D3 |
| SCK | D7 | D2 (GPIO18) |
| MOSI | D5 | D0 (GPIO26) |
| MISO | D6 | D1 (GPIO27) |
| VCC | 3.3V or 5V* | 3.3V or 5V* |
| GND | GND | GND |

*Some SD card modules have voltage regulator

## Basic Usage

```cpp
#include <SPI.h>
#include <SD.h>

const int chipSelect = D4;

void setup() {
    Serial.begin(115200);
    SPI.begin();

    if (!SD.begin(chipSelect)) {
        Serial.println("SD card failed!");
        return;
    }

    Serial.println("SD card initialized");

    // Write file
    File dataFile = SD.open("data.txt", FILE_WRITE);
    if (dataFile) {
        dataFile.println("Hello SD!");
        dataFile.close();
    }

    // Read file
    dataFile = SD.open("data.txt", FILE_READ);
    if (dataFile) {
        while (dataFile.available()) {
            Serial.write(dataFile.read());
        }
        dataFile.close();
    }
}

void loop() {}
```

## File Listing

```cpp
void listFiles() {
    File root = SD.open("/");
    File file = root.openNextFile();

    while (file) {
        Serial.print("  ");
        Serial.print(file.name());
        if (file.isDirectory()) {
            Serial.println("/");
        } else {
            Serial.print("  ");
            Serial.println(file.size());
        }
        file.close();
        file = root.openNextFile();
    }
}
```

## Data Logging

```cpp
#include <SPI.h>
#include <SD.h>

const int chipSelect = D4;

void setup() {
    Serial.begin(115200);
    SPI.begin();
    SD.begin(chipSelect);

    // CSV header
    File dataFile = SD.open("log.csv", FILE_WRITE);
    dataFile.println("timestamp,value1,value2");
    dataFile.close();
}

void loop() {
    File dataFile = SD.open("log.csv", FILE_WRITE);
    if (dataFile) {
        dataFile.print(millis());
        dataFile.print(",");
        dataFile.print(analogRead(A0));
        dataFile.print(",");
        dataFile.println(analogRead(A1));
        dataFile.close();
    }
    delay(5000);
}
```

## Read File to String

```cpp
String readFile(String filename) {
    File file = SD.open(filename, FILE_READ);
    if (!file) {
        return "";
    }

    String content = "";
    while (file.available()) {
        content += (char)file.read();
    }
    file.close();
    return content;
}
```

## Check File Exists

```cpp
bool fileExists(String filename) {
    return SD.exists(filename);
}

void setup() {
    Serial.begin(115200);
    SD.begin(D4);

    if (SD.exists("data.txt")) {
        Serial.println("File exists!");
    }
}

void loop() {}
```

## Append to File

```cpp
void appendToFile(String filename, String message) {
    File file = SD.open(filename, FILE_WRITE);
    if (file) {
        file.seek(file.size());  // Go to end of file
        file.println(message);
        file.close();
    }
}
```

## Delete File

```cpp
bool deleteFile(String filename) {
    if (SD.exists(filename)) {
        return SD.remove(filename);
    }
    return false;
}
```

## Get File Size

```cpp
uint32_t getFileSize(String filename) {
    File file = SD.open(filename, FILE_READ);
    if (!file) {
        return 0;
    }
    uint32_t size = file.size();
    file.close();
    return size;
}
```

## Format SD Card

```cpp
// ESP32: Use SPIFFS/LittleFS to erase, not SD
// SD card format: Use PC and SD formatter tool
```

## SD Card Info

```cpp
void printSDInfo() {
    uint8_t cardType = SD.cardType();

    Serial.print("Card type: ");
    switch (cardType) {
        case CARD_MMC:
            Serial.println("MMC");
            break;
        case CARD_SD:
            Serial.println("SDSC");
            break;
        case CARD_SDHC:
            Serial.println("SDHC");
            break;
        default:
            Serial.println("UNKNOWN");
    }

    uint64_t cardSize = SD.cardSize() / (1024 * 1024);
    Serial.printf("Size: %llu MB\n", cardSize);
}
```

## Large File Writes

```cpp
void writeLargeData() {
    File file = SD.open("large.bin", FILE_WRITE);
    if (file) {
        for (int i = 0; i < 10000; i++) {
            file.write("This is a test line\r\n");
            if (i % 100 == 0) {
                Serial.printf("Written %d lines\n", i);
            }
        }
        file.close();
        Serial.println("Done!");
    }
}
```

## Binary Data

```cpp
void writeBinary() {
    File file = SD.open("data.bin", FILE_WRITE);
    if (file) {
        byte data[256];
        // Fill array
        file.write(data, 256);
        file.close();
    }
}

void readBinary() {
    File file = SD.open("data.bin", FILE_READ);
    if (file) {
        byte data[256];
        file.read(data, 256);
        file.close();
    }
}
```

## Troubleshooting

### Initialization failed

1. Check wiring (CS, SCK, MOSI, MISO)
2. Verify 3.3V or 5V power
3. Try lower SPI speed: `SD.begin(chipSelect, SPI, 25000000)`
4. Format SD card as FAT32

### Can't create files

1. Check if SD is initialized
2. Check write protect tab on SD card
3. Check available space

### File corruption

1. Ensure file is closed before reset
2. Use flush() after writes
3. Check power stability during writes

### Slow writes

1. Use smaller buffer size
2. Reduce SPI frequency
3. Write in larger chunks
