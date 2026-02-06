# Expansion Base RTC and SD Card Examples - Arduino

## Overview

Complete Arduino examples for using the RTC (PCF8563) and SD card on the XIAO Expansion Base board. These examples demonstrate timekeeping, data logging, and file operations.

**Hardware Reference:** `/xiao/references/expansion_boards/expansion-base.md`
**Arduino Implementation:** `/xiao-arduino/references/expansion-boards/expansion-base.md`

## Required Libraries

Install via Arduino Library Manager:
- **PCF8563** - RTC driver
- **SD** - Built-in for ESP32/SAMD21/RP2040
- **SdFat** - For nRF52840 (alternative)

## Example 1: RTC Time Display

```cpp
#include <Arduino.h>
#include <PCF8563.h>
#include <Wire.h>

PCF8563 pcf;

void setup() {
  Serial.begin(115200);
  Wire.begin();
  pcf.init();

  // Set initial time (uncomment to set)
  /*
  pcf.stopClock();
  pcf.setYear(24);
  pcf.setMonth(1);
  pcf.setDay(15);
  pcf.setHour(12);
  pcf.setMinut(30);
  pcf.setSecond(0);
  pcf.startClock();
  */
}

void loop() {
  Time nowTime = pcf.getTime();

  Serial.print("Time: ");
  Serial.print(nowTime.hour);
  Serial.print(":");
  if (nowTime.minute < 10) Serial.print("0");
  Serial.print(nowTime.minute);
  Serial.print(":");
  if (nowTime.second < 10) Serial.print("0");
  Serial.println(nowTime.second);

  Serial.print("Date: ");
  Serial.print(nowTime.day);
  Serial.print("/");
  Serial.print(nowTime.month);
  Serial.print("/20");
  Serial.println(nowTime.year);

  delay(1000);
}
```

## Example 2: SD Card Basic Read/Write

```cpp
#include <Arduino.h>
#include <SD.h>
#include <SPI.h>

const int chipSelect = D2;

void setup() {
  Serial.begin(115200);
  while (!Serial && millis() < 5000);

  Serial.print("Initializing SD card...");

  pinMode(chipSelect, OUTPUT);
  if (!SD.begin(chipSelect)) {
    Serial.println("initialization failed!");
    return;
  }
  Serial.println("initialization done.");

  // Write test file
  File myFile = SD.open("/test.txt", FILE_WRITE);
  if (myFile) {
    Serial.print("Writing to test.txt...");
    myFile.println("Testing 1, 2, 3.");
    myFile.println("XIAO Expansion Base SD Card Test");
    myFile.close();
    Serial.println("done.");
  } else {
    Serial.println("error opening test.txt");
  }

  // Read test file
  myFile = SD.open("/test.txt");
  if (myFile) {
    Serial.println("test.txt:");
    while (myFile.available()) {
      Serial.write(myFile.read());
    }
    myFile.close();
  } else {
    Serial.println("error opening test.txt");
  }
}

void loop() {}
```

## Example 3: Data Logger (RTC + SD)

```cpp
#include <Arduino.h>
#include <PCF8563.h>
#include <SD.h>
#include <SPI.h>

PCF8563 pcf;
const int chipSelect = D2;
const int sensorPin = A0;
File logFile;

void setup() {
  Serial.begin(115200);
  Wire.begin();
  pcf.init();

  // Initialize SD card
  pinMode(chipSelect, OUTPUT);
  if (!SD.begin(chipSelect)) {
    Serial.println("SD init failed!");
    return;
  }

  // Create new log file with timestamp
  Time nowTime = pcf.getTime();
  char filename[32];
  sprintf(filename, "/log_%02d%02d%02d_%02d%02d.txt",
          nowTime.year, nowTime.month, nowTime.day,
          nowTime.hour, nowTime.minute);

  logFile = SD.open(filename, FILE_WRITE);
  if (logFile) {
    logFile.println("=== Data Logger Started ===");
    logFile.println("Timestamp,SensorValue,Voltage");
    Serial.print("Logging to: ");
    Serial.println(filename);
  } else {
    Serial.println("Error creating log file");
  }
}

void loop() {
  if (logFile && logFile) {
    // Read sensor
    int sensorValue = analogRead(sensorPin);
    float voltage = sensorValue * (3.3 / 4095.0);

    // Get current time
    Time nowTime = pcf.getTime();

    // Log data to SD card
    logFile.print(nowTime.hour);
    logFile.print(":");
    logFile.print(nowTime.minute);
    logFile.print(":");
    logFile.print(nowTime.second);
    logFile.print(",");
    logFile.print(sensorValue);
    logFile.print(",");
    logFile.println(voltage, 3);

    logFile.flush();  // Ensure data is written

    // Also print to serial
    Serial.print(nowTime.hour);
    Serial.print(":");
    Serial.print(nowTime.minute);
    Serial.print(":");
    Serial.print(nowTime.second);
    Serial.print(" - Sensor: ");
    Serial.print(sensorValue);
    Serial.print(", Voltage: ");
    Serial.print(voltage, 3);
    Serial.println("V");
  }

  delay(5000);  // Log every 5 seconds
}
```

## Example 4: Alarm System

```cpp
#include <Arduino.h>
#include <PCF8563.h>
#include <U8x8lib.h>

PCF8563 pcf;
U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);
const int buzzerPin = D3;

// Alarm time: 07:30:00
const int alarmHour = 7;
const int alarmMinute = 30;
bool alarmTriggered = false;

void setup() {
  Serial.begin(115200);
  Wire.begin();
  u8x8.begin();
  u8x8.setFlipMode(1);

  pcf.init();

  // Set current time (uncomment to set)
  /*
  pcf.stopClock();
  pcf.setYear(24);
  pcf.setMonth(1);
  pcf.setDay(15);
  pcf.setHour(7);
  pcf.setMinut(29);
  pcf.setSecond(50);
  pcf.startClock();
  */

  pinMode(buzzerPin, OUTPUT);
}

void loop() {
  Time nowTime = pcf.getTime();

  // Display current time
  u8x8.clearDisplay();
  u8x8.setFont(u8x8_font_profont29_2x3_f);

  char timeStr[9];
  sprintf(timeStr, "%02d:%02d:%02d", nowTime.hour, nowTime.minute, nowTime.second);
  u8x8.setCursor(0, 2);
  u8x8.print(timeStr);

  // Check alarm
  if (nowTime.hour == alarmHour && nowTime.minute == alarmMinute && !alarmTriggered) {
    alarmTriggered = true;

    // Sound alarm
    for (int i = 0; i < 100; i++) {
      digitalWrite(buzzerPin, HIGH);
      delay(1);
      digitalWrite(buzzerPin, LOW);
      delay(1);
    }
  }

  // Reset alarm trigger after minute passes
  if (nowTime.minute != alarmMinute) {
    alarmTriggered = false;
  }

  delay(1000);
}
```

## Example 5: CSV Data Export

```cpp
#include <Arduino.h>
#include <PCF8563.h>
#include <SD.h>
#include <SPI.h>

PCF8563 pcf;
const int chipSelect = D2;

// Sample data structure
struct DataPoint {
  int id;
  float temperature;
  float humidity;
  int status;
};

DataPoint sensorData[] = {
  {1, 23.5, 45.2, 1},
  {2, 24.1, 46.8, 1},
  {3, 22.9, 44.5, 1},
  {4, 25.3, 48.1, 0},
  {5, 24.8, 47.2, 1}
};

void setup() {
  Serial.begin(115200);
  Wire.begin();
  pcf.init();

  pinMode(chipSelect, OUTPUT);
  if (!SD.begin(chipSelect)) {
    Serial.println("SD init failed!");
    return;
  }

  // Create CSV file
  File csvFile = SD.open("/sensor_data.csv", FILE_WRITE);
  if (csvFile) {
    // Write CSV header
    csvFile.println("Timestamp,ID,Temperature,Humidity,Status");

    // Write data with timestamps
    Time nowTime = pcf.getTime();
    for (int i = 0; i < 5; i++) {
      csvFile.print(nowTime.hour);
      csvFile.print(":");
      csvFile.print(nowTime.minute);
      csvFile.print(":");
      csvFile.print(nowTime.second + i * 10);
      csvFile.print(",");

      csvFile.print(sensorData[i].id);
      csvFile.print(",");
      csvFile.print(sensorData[i].temperature);
      csvFile.print(",");
      csvFile.print(sensorData[i].humidity);
      csvFile.print(",");
      csvFile.println(sensorData[i].status);
    }

    csvFile.close();
    Serial.println("CSV file created: sensor_data.csv");
  }
}

void loop() {}
```

## Example 6: SD Card File Manager

```cpp
#include <Arduino.h>
#include <SD.h>
#include <SPI.h>

const int chipSelect = D2;

void listFiles(File dir, int numTabs = 0) {
  while (true) {
    File entry = dir.openNextFile();

    if (!entry) {
      // No more files
      break;
    }

    for (uint8_t i = 0; i < numTabs; i++) {
      Serial.print('\t');
    }

    Serial.print(entry.name());
    if (entry.isDirectory()) {
      Serial.println("/");
      listFiles(entry, numTabs + 1);
    } else {
      // Print file size
      Serial.print("\t\t");
      Serial.println(entry.size(), DEC);
    }

    entry.close();
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(chipSelect, OUTPUT);

  if (!SD.begin(chipSelect)) {
    Serial.println("SD init failed!");
    return;
  }

  Serial.println("SD Card Contents:");
  File root = SD.open("/");
  listFiles(root);

  Serial.println("\n--- File Operations ---");

  // Create a test directory
  if (!SD.exists("/testdir")) {
    SD.mkdir("/testdir");
    Serial.println("Created directory: /testdir");
  }

  // Create a file in the directory
  File myFile = SD.open("/testdir/test.txt", FILE_WRITE);
  if (myFile) {
    myFile.println("Hello from XIAO!");
    myFile.close();
    Serial.println("Created file: /testdir/test.txt");
  }

  // Read the file back
  myFile = SD.open("/testdir/test.txt");
  if (myFile) {
    Serial.println("Contents of /testdir/test.txt:");
    while (myFile.available()) {
      Serial.write(myFile.read());
    }
    myFile.close();
  }

  // Get file info
  myFile = SD.open("/testdir/test.txt");
  if (myFile) {
    Serial.print("File size: ");
    Serial.print(myFile.size());
    Serial.println(" bytes");
    myFile.close();
  }
}

void loop() {}
```

## nRF52840 SD Card Example (SdFat)

```cpp
#include <Arduino.h>
#include <SPI.h>
#include "SdFat.h"

SdFat SD;
const int chipSelect = D2;

void setup() {
  Serial.begin(9600);
  while (!Serial);

  Serial.print("Initializing SD card...");

  if (!SD.begin(chipSelect)) {
    Serial.println("initialization failed!");
    return;
  }
  Serial.println("initialization done.");

  File myFile = SD.open("/test.txt", FILE_WRITE);
  if (myFile) {
    Serial.print("Writing to test.txt...");
    myFile.println("Testing SdFat on nRF52840");
    myFile.close();
    Serial.println("done.");
  }
}

void loop() {}
```

## Troubleshooting

### RTC Time Resetting

**Symptom:** Time resets when power removed

**Solution:** Check CR1220 battery voltage (should be ~3V). Replace if low.

### SD Card Not Detected

**Symptom:** "initialization failed!" message

**Solution:**
1. Check SD card is formatted as FAT32
2. Reseat SD card
3. Verify D2 is CS pin
4. Try different SD card

### File Corruption

**Symptom:** Data not written correctly

**Solution:**
- Always call `file.flush()` after writing
- Close files before removing power
- Use shorter file names (8.3 format)

## See Also

- **OLED Display:** See `expansion-base-oled.md` for display integration
- **Hardware Docs:** `/xiao/references/expansion_boards/expansion-base.md`
