# Round Display for XIAO - Arduino

## Overview

Arduino implementation for the Seeed Studio Round Display for XIAO. Features a 1.28" round touchscreen display (240x240), RTC, and SD card slot.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/round-display.md`.

## Pin Summary

| Function | XIAO Pin | Notes |
|----------|----------|-------|
| Display SDA | D4 | I2C SDA |
| Display SCL | D5 | I2C SCL |
| Touch SDA | D4 | I2C SDA (shared) |
| Touch SCL | D5 | I2C SCL (shared) |
| RTC SDA | D4 | I2C SDA (shared) |
| RTC SCL | D5 | I2C SCL (shared) |
| SD Card CS | D2 | SPI CS |
| User Button | D1 | GPIO input |

**Important:** ESP32S3 or nRF52840 recommended for adequate RAM. SAMD21 may have memory limitations.

## Required Libraries

Install these libraries via Arduino Library Manager:

| Library | Purpose |
|---------|---------|
| **Adafruit GFX** | Graphics library |
| **Adafruit GC9A01A** | Round display driver |
| **PCF8563** | RTC (Real-Time Clock) |
| **SD** | SD card (built-in) |

## Display Setup

The round display uses the GC9A01A controller via SPI. Basic dimensions are 240x240 pixels.

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_GC9A01A.h>
#include <SPI.h>

#define TFT_CS   D2
#define TFT_DC   D3
#define TFT_RST  -1  // Connected to XIAO reset

Adafruit_GC9A01A tft(TFT_CS, TFT_DC, TFT_RST);

void setup() {
  Serial.begin(115200);
  tft.begin();
  tft.setRotation(1);  // Adjust rotation as needed
  tft.fillScreen(GC9A01A_BLACK);

  // Draw a circle
  tft.drawCircle(120, 120, 100, GC9A01A_RED);
  tft.fillCircle(120, 120, 50, GC9A01A_BLUE);

  // Draw text
  tft.setTextColor(GC9A01A_WHITE);
  tft.setTextSize(2);
  tft.setCursor(50, 110);
  tft.print("Hello XIAO!");
}

void loop() {}
```

## Touch Support

The touchscreen is typically controlled via I2C (CST816S or similar controller).

```cpp
#include <Wire.h>

#define TOUCH_ADDR 0x15  // Typical I2C address

void setup() {
  Wire.begin();
  Serial.begin(115200);
}

void loop() {
  Wire.beginTransmission(TOUCH_ADDR);
  // Send touch request bytes
  Wire.write(0x02);
  Wire.endTransmission();

  Wire.requestFrom(TOUCH_ADDR, 6);
  if (Wire.available() >= 6) {
    uint8_t data[6];
    for (int i = 0; i < 6; i++) {
      data[i] = Wire.read();
    }

    // Parse touch data (format varies by controller)
    if (data[0] & 0x01) {  // Touch detected
      uint16_t x = (data[2] << 8) | data[3];
      uint16_t y = (data[4] << 8) | data[5];
      Serial.print("Touch: ");
      Serial.print(x);
      Serial.print(", ");
      Serial.println(y);
    }
  }
  delay(50);
}
```

## RTC on Round Display

The RTC shares the I2C bus (D4/D5).

```cpp
#include <PCF8563.h>
#include <Wire.h>

PCF8563 pcf;

void setup() {
  Serial.begin(115200);
  Wire.begin();
  pcf.init();
  pcf.stopClock();
  pcf.setYear(24);
  pcf.setMonth(1);
  pcf.setDay(15);
  pcf.setHour(12);
  pcf.setMinut(0);
  pcf.setSecond(0);
  pcf.startClock();
}

void loop() {
  Time nowTime = pcf.getTime();
  Serial.print("Time: ");
  Serial.print(nowTime.hour);
  Serial.print(":");
  Serial.println(nowTime.minute);
  delay(1000);
}
```

## SD Card

The SD card CS pin is on D2.

```cpp
#include <SD.h>
#include <SPI.h>

void setup() {
  Serial.begin(115200);
  if (!SD.begin(D2)) {
    Serial.println("SD init failed!");
    return;
  }
  Serial.println("SD initialized.");

  File file = SD.open("/test.txt", FILE_WRITE);
  if (file) {
    file.println("Round Display Test");
    file.close();
  }
}

void loop() {}
```

## Drawing Round UI Elements

```cpp
void drawWatchFace() {
  tft.fillScreen(GC9A01A_BLACK);

  // Draw outer circle
  tft.drawCircle(120, 120, 115, GC9A01A_WHITE);

  // Draw hour markers
  for (int i = 0; i < 12; i++) {
    float angle = i * PI / 6;
    int x1 = 120 + cos(angle) * 100;
    int y1 = 120 + sin(angle) * 100;
    int x2 = 120 + cos(angle) * 90;
    int y2 = 120 + sin(angle) * 90;
    tft.drawLine(x1, y1, x2, y2, GC9A01A_WHITE);
  }

  // Draw center circle
  tft.fillCircle(120, 120, 10, GC9A01A_RED);
}
```

## Troubleshooting

### Display Shows White Screen

**Symptom:** Display is all white or black

**Possible Causes:**
1. Wrong pin connections
2. SPI not initialized
3. Power issue

**Solution:** Verify SPI pins (CS=D2, DC=D3), check I2C connections for touch

### Touch Not Responding

**Symptom:** Touch not detected

**Possible Causes:**
1. Wrong I2C address
2. Touch controller not powered

**Solution:** Scan I2C bus to find touch controller address

### Memory Issues on SAMD21

**Symptom:** Sketch too large, out of memory

**Solution:** Use ESP32S3 or nRF52840 for more RAM
