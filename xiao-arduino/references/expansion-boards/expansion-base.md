# Expansion Base for XIAO - Arduino

## Overview

Arduino implementation for the Seeed Studio Expansion Base for XIAO. This board provides rich peripherals including OLED display, RTC, SD card slot, passive buzzer, user button, and Grove connectors.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/expansion-base.md`.

## Pin Summary

| Function | XIAO Pin | Notes |
|----------|----------|-------|
| OLED SDA | D4 | I2C SDA |
| OLED SCL | D5 | I2C SCL |
| RTC SDA | D4 | I2C SDA (shared with OLED) |
| RTC SCL | D5 | I2C SCL (shared with OLED) |
| SD Card CS | D2 | SPI CS |
| Buzzer | D3 (A3) | PWM output |
| User Button | D1 | GPIO input |
| Grove I2C | D4/D5 | Default Grove I2C |
| Grove UART | D6/D7 | Serial1 |
| Grove A0/D0 | D0 | Analog/Digital |

**Important:** The Expansion Base does NOT support nRF54L15 or MG24 due to different SWD pin layouts.

## Required Libraries

Install these libraries via Arduino Library Manager:

| Library | Purpose |
|---------|---------|
| **U8g2** | OLED display control |
| **PCF8563** | RTC (Real-Time Clock) |
| **SD** | SD card (built-in for ESP32/SAMD21/RP2040) |
| **SdFat** | SD card for nRF52840 (alternative to SD) |
| **Servo** | Servo motor control |
| **ESP32Servo** | Servo for ESP32 series only |

## OLED Display

The 0.96" OLED display uses the SSD1306 controller via I2C (D4/D5).

### Basic Hello World

```cpp
#include <Arduino.h>
#include <U8x8lib.h>
#include <Wire.h>

U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);

void setup(void) {
  u8x8.begin();
  u8x8.setFlipMode(1);  // Enable 180° rotation
}

void loop(void) {
  u8x8.setFont(u8x8_font_chroma48medium8_r);
  u8x8.setCursor(0, 0);
  u8x8.print("Hello World!");
}
```

## RTC (Real-Time Clock)

The PCF8563 RTC shares the I2C bus with the OLED (D4/D5).

### Display Time on OLED

```cpp
#include <Arduino.h>
#include <U8x8lib.h>
#include <PCF8563.h>
#include <Wire.h>

PCF8563 pcf;
U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);

void setup() {
  Serial.begin(115200);
  u8x8.begin();
  u8x8.setFlipMode(1);
  Wire.begin();
  pcf.init();
  pcf.stopClock();
  // Set initial time (year, month, day, hour, minute, second)
  pcf.setYear(20);
  pcf.setMonth(10);
  pcf.setDay(23);
  pcf.setHour(17);
  pcf.setMinut(33);
  pcf.setSecond(0);
  pcf.startClock();
}

void loop() {
  Time nowTime = pcf.getTime();
  u8x8.setFont(u8x8_font_chroma48medium8_r);
  u8x8.setCursor(0, 0);
  u8x8.print(nowTime.day);
  u8x8.print("/");
  u8x8.print(nowTime.month);
  u8x8.print("/20");
  u8x8.print(nowTime.year);
  u8x8.setCursor(0, 1);
  u8x8.print(nowTime.hour);
  u8x8.print(":");
  u8x8.print(nowTime.minute);
  u8x8.print(":");
  u8x8.println(nowTime.second);
  delay(1000);
}
```

## SD Card

The SD card CS pin is connected to D2. Use the built-in SD library for most boards.

### Basic Read/Write (ESP32/SAMD21/RP2040)

```cpp
#include <SPI.h>
#include <SD.h>
#include "FS.h"

File myFile;

void setup() {
  Serial.begin(115200);
  while(!Serial);
  delay(500);

  Serial.print("Initializing SD card...");

  pinMode(D2, OUTPUT);
  if (!SD.begin(D2)) {
    Serial.println("initialization failed!");
    return;
  }
  Serial.println("initialization done.");

  // Write to file
  myFile = SD.open("/test.txt", FILE_WRITE);
  if (myFile) {
    Serial.print("Writing to test.txt...");
    myFile.println("testing 1, 2, 3.");
    myFile.close();
    Serial.println("done.");
  }

  // Read from file
  myFile = SD.open("/test.txt");
  if (myFile) {
    Serial.println("test.txt:");
    while (myFile.available()) {
      Serial.write(myFile.read());
    }
    myFile.close();
  }
}

void loop() {}
```

### SD Card for nRF52840 (SdFat)

```cpp
#include <SPI.h>
#include "SdFat.h"
SdFat SD;

#define SD_CS_PIN D2
File myFile;

void setup() {
  Serial.begin(9600);
  while (!Serial);

  Serial.print("Initializing SD card...");

  if (!SD.begin(SD_CS_PIN)) {
    Serial.println("initialization failed!");
    return;
  }
  Serial.println("initialization done.");

  myFile = SD.open("/test.txt", FILE_WRITE);
  if (myFile) {
    myFile.println("testing 1, 2, 3.");
    myFile.close();
  }
}

void loop() {}
```

## Buzzer

The passive buzzer is on D3 (A3). Use PWM to generate tones.

### Play Happy Birthday

```cpp
int speakerPin = D3;
int length = 28;
char notes[] = "GGAGcB GGAGdc GGxecBA yyecdc";
int beats[] = { 2, 2, 8, 8, 8, 16, 1, 2, 2, 8, 8, 8, 16, 1, 2, 2, 8, 8, 8, 8, 16, 1, 2, 2, 8, 8, 8, 16 };
int tempo = 150;

void playTone(int tone, int duration) {
  for (long i = 0; i < duration * 1000L; i += tone * 2) {
    digitalWrite(speakerPin, HIGH);
    delayMicroseconds(tone);
    digitalWrite(speakerPin, LOW);
    delayMicroseconds(tone);
  }
}

void playNote(char note, int duration) {
  char names[] = {'C', 'D', 'E', 'F', 'G', 'A', 'B', 'c', 'd', 'e', 'f', 'g', 'a', 'b', 'x', 'y'};
  int tones[] = { 1915, 1700, 1519, 1432, 1275, 1136, 1014, 956, 834, 765, 593, 468, 346, 224, 655, 715 };

  for (int i = 0; i < 16; i++) {
    if (names[i] == note) {
      int newduration = duration / 5;
      playTone(tones[i], newduration);
    }
  }
}

void setup() {
  pinMode(speakerPin, OUTPUT);
}

void loop() {
  for (int i = 0; i < length; i++) {
    if (notes[i] == ' ') {
      delay(beats[i] * tempo);
    } else {
      playNote(notes[i], beats[i] * tempo);
    }
    delay(tempo);
  }
}
```

## User Button

The user button is on D1, active LOW (uses internal pull-up).

### Button Controls LED

```cpp
const int buttonPin = D1;
int buttonState = 0;

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
  pinMode(buttonPin, INPUT_PULLUP);
}

void loop() {
  buttonState = digitalRead(buttonPin);
  if (buttonState == LOW) {  // Button pressed
    digitalWrite(LED_BUILTIN, HIGH);
  } else {
    digitalWrite(LED_BUILTIN, LOW);
  }
}
```

## Grove I2C Port

The Grove I2C connectors are on D4 (SDA) and D5 (SCL), shared with the onboard OLED and RTC.

## Grove UART Port

The Grove UART port is on D6 (RX) and D7 (TX), accessible via Serial1.

```cpp
// Use Serial1 for Grove UART devices
void setup() {
  Serial.begin(115200);
  Serial1.begin(9600);  // Grove UART
}

void loop() {
  if (Serial1.available()) {
    Serial.write(Serial1.read());
  }
}
```

## ESP32 Servo Note

If using XIAO ESP32 series, use ESP32Servo library instead of Servo:

```cpp
// For ESP32C3/C6/S3 only
#include <ESP32Servo.h>  // NOT #include <Servo.h>
```

## Examples

See `expansion-boards/examples/` for complete project examples:
- `expansion-base-oled.md` - OLED display examples
- `expansion-base-rtc-sd.md` - RTC and SD card data logging

## Troubleshooting

### OLED Not Displaying

**Symptom:** OLED remains dark or shows garbage

**Possible Causes:**
1. I2C address wrong - Check if display responds to 0x3C
2. Wrong pins - Verify D4/D5 connections
3. Power issue - Ensure USB connected

**Solution:**
```cpp
// Scan I2C bus
#include <Wire.h>
void setup() {
  Wire.begin();
  Serial.begin(115200);
  for (byte addr = 1; addr < 127; addr++) {
    Wire.beginTransmission(addr);
    if (Wire.endTransmission() == 0) {
      Serial.print("Found device at 0x");
      Serial.println(addr, HEX);
    }
  }
}
```

### SD Card Not Detected

**Symptom:** "initialization failed!" message

**Possible Causes:**
1. Wrong CS pin - Must use D2
2. SD card not formatted - Format as FAT32
3. Loose connection - Reseat SD card

**Solution:**
```cpp
// Verify D2 is set as output
pinMode(D2, OUTPUT);
if (!SD.begin(D2)) {
  Serial.println("SD failed - check card and connections");
}
```

### RTC Loses Time

**Symptom:** Time resets when power removed

**Possible Causes:**
1. Dead CR1220 battery - Replace battery
2. Battery not installed - Add CR1220

**Solution:** Check battery voltage (should be ~3V)

### Buzzer Too Quiet

**Symptom:** Buzzer volume too low

**Possible Causes:**
1. Wrong pin - Must use D3 (A3)
2. PWM frequency too high/low

**Solution:**
```cpp
// Try different tone frequencies
int speakerPin = D3;
void setup() {
  analogWrite(speakerPin, 128);  // 50% duty cycle
}
```

## Battery Power

The Expansion Base supports LiPo battery via JST 2.0mm connector:
- Charging current: 460mA max
- Charge LED: Blinking = charging, Solid = charged

**Note:** Battery charging only works when USB is connected.
