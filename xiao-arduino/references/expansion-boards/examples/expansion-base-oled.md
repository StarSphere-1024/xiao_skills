# Expansion Base OLED Examples - Arduino

## Overview

Complete Arduino examples for using the OLED display on the XIAO Expansion Base board. The 0.96" OLED (SSD1306 controller) uses I2C on pins D4/D5.

**Hardware Reference:** `/xiao/references/expansion_boards/expansion-base.md`
**Arduino Implementation:** `/xiao-arduino/references/expansion-boards/expansion-base.md`

## Required Library

Install **U8g2** library via Arduino Library Manager.

## Example 1: Hello World

```cpp
#include <Arduino.h>
#include <U8x8lib.h>

U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);

void setup(void) {
  u8x8.begin();
  u8x8.setFlipMode(1);  // Rotate 180° if needed
}

void loop(void) {
  u8x8.setFont(u8x8_font_chroma48medium8_r);
  u8x8.setCursor(0, 0);
  u8x8.print("Hello XIAO!");
  delay(1000);
}
```

## Example 2: Text Scrolling

```cpp
#include <Arduino.h>
#include <U8x8lib.h>

U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);
const char* message = "This is a scrolling text demo for XIAO Expansion Base OLED display!    ";
int scrollPos = 0;

void setup() {
  u8x8.begin();
  u8x8.setFlipMode(1);
  u8x8.setFont(u8x8_font_chroma48medium8_r);
}

void loop() {
  u8x8.clearDisplay();

  // Display 16 characters at a time
  for (int i = 0; i < 16; i++) {
    u8x8.setCursor(i, 3);
    u8x8.print(message[(scrollPos + i) % strlen(message)]);
  }

  scrollPos = (scrollPos + 1) % strlen(message);
  delay(200);
}
```

## Example 3: Multiple Text Lines

```cpp
#include <Arduino.h>
#include <U8x8lib.h>

U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);

void setup() {
  u8x8.begin();
  u8x8.setFlipMode(1);
  u8x8.setFont(u8x8_font_chroma48medium8_r);
}

void loop() {
  u8x8.clearDisplay();

  // Display multiple lines of text
  u8x8.setCursor(0, 0);
  u8x8.print("Line 1: XIAO");

  u8x8.setCursor(0, 1);
  u8x8.print("Line 2: OLED");

  u8x8.setCursor(0, 2);
  u8x8.print("Line 3: Display");

  u8x8.setCursor(0, 3);
  u8x8.print("Line 4: Demo");

  delay(5000);
}
```

## Example 4: Progress Bar

```cpp
#include <Arduino.h>
#include <U8x8lib.h>

U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);

void setup() {
  u8x8.begin();
  u8x8.setFlipMode(1);
  u8x8.setFont(u8x8_font_chroma48medium8_r);
}

void drawProgressBar(int row, int progress) {
  int barWidth = 16;  // 16 characters wide
  int filled = (progress * barWidth) / 100;

  u8x8.setCursor(0, row);
  for (int i = 0; i < barWidth; i++) {
    if (i < filled) {
      u8x8.print("=");  // Filled portion
    } else {
      u8x8.print(" ");  // Empty portion
    }
  }
}

void loop() {
  u8x8.clearDisplay();
  u8x8.setCursor(0, 0);
  u8x8.print("Loading...");

  // Animate progress bar
  for (int i = 0; i <= 100; i += 5) {
    drawProgressBar(2, i);

    u8x8.setCursor(0, 4);
    u8x8.print("Progress: ");
    u8x8.setCursor(10, 4);
    if (i < 10) u8x8.print(" ");
    if (i < 100) u8x8.print(" ");
    u8x8.print(i);
    u8x8.print("%");

    delay(100);
  }

  delay(2000);
}
```

## Example 5: Display Sensor Data

```cpp
#include <Arduino.h>
#include <U8x8lib.h>

U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);

void setup() {
  u8x8.begin();
  u8x8.setFlipMode(1);
  u8x8.setFont(u8x8_font_chroma48medium8_r);
  Serial.begin(115200);
}

void loop() {
  // Read sensor (example with analog input)
  int sensorValue = analogRead(A0);
  float voltage = sensorValue * (3.3 / 4095.0);

  // Display sensor data
  u8x8.clearDisplay();

  u8x8.setCursor(0, 0);
  u8x8.print("Sensor Monitor");

  u8x8.setCursor(0, 2);
  u8x8.print("Raw: ");
  u8x8.setCursor(5, 2);
  u8x8.print(sensorValue);

  u8x8.setCursor(0, 4);
  u8x8.print("Volt: ");
  u8x8.setCursor(6, 4);
  u8x8.print(voltage, 2);
  u8x8.print("V");

  // Also print to serial
  Serial.print("Sensor: ");
  Serial.print(sensorValue);
  Serial.print(", Voltage: ");
  Serial.print(voltage, 2);
  Serial.println("V");

  delay(500);
}
```

## Example 6: Simple Menu System

```cpp
#include <Arduino.h>
#include <U8x8lib.h>

U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);

const int buttonPin = D1;
int menuIndex = 0;
const char* menuItems[] = {"Option 1", "Option 2", "Option 3", "Option 4"};
const int menuSize = 4;

void setup() {
  u8x8.begin();
  u8x8.setFlipMode(1);
  u8x8.setFont(u8x8_font_chroma48medium8_r);
  pinMode(buttonPin, INPUT_PULLUP);
}

void displayMenu() {
  u8x8.clearDisplay();

  // Show title
  u8x8.setCursor(0, 0);
  u8x8.print("Main Menu:");

  // Show menu items
  for (int i = 0; i < menuSize && i < 5; i++) {
    u8x8.setCursor(0, i + 1);
    if (i == menuIndex) {
      u8x8.print(">");  // Selection indicator
    } else {
      u8x8.print(" ");
    }
    u8x8.print(menuItems[i]);
  }
}

void loop() {
  displayMenu();

  // Check button press
  static int lastButtonState = HIGH;
  int buttonState = digitalRead(buttonPin);

  if (buttonState == LOW && lastButtonState == HIGH) {
    menuIndex = (menuIndex + 1) % menuSize;
    delay(200);  // Debounce
  }

  lastButtonState = buttonState;
  delay(50);
}
```

## Example 7: Digital Clock Display

```cpp
#include <Arduino.h>
#include <U8x8lib.h>

U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);

unsigned long lastTime = 0;
int hours = 12;
int minutes = 0;
int seconds = 0;

void setup() {
  u8x8.begin();
  u8x8.setFlipMode(1);
}

void loop() {
  if (millis() - lastTime >= 1000) {
    lastTime = millis();

    seconds++;
    if (seconds >= 60) {
      seconds = 0;
      minutes++;
      if (minutes >= 60) {
        minutes = 0;
        hours++;
        if (hours >= 24) {
          hours = 0;
        }
      }
    }

    // Display time
    u8x8.clearDisplay();
    u8x8.setFont(u8x8_font_profont29_2x3_f);

    // Format time as HH:MM:SS
    char timeStr[9];
    sprintf(timeStr, "%02d:%02d:%02d", hours, minutes, seconds);

    // Center the time display
    u8x8.setCursor(0, 2);
    u8x8.print(timeStr);
  }
}
```

## Troubleshooting

### OLED Not Displaying

**Possible Causes:**
1. Wrong I2C address
2. I2C wiring issue
3. Power problem

**Solution:**
```cpp
// Scan I2C bus to find OLED
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

### Display Orientation Wrong

**Solution:**
```cpp
// Try different flip modes
u8x8.setFlipMode(0);  // Normal
u8x8.setFlipMode(1);  // Rotated 180°
```

## See Also

- **RTC Integration:** See `expansion-base-rtc-sd.md` for time display with RTC
- **Hardware Docs:** `/xiao/references/expansion_boards/expansion-base.md`
