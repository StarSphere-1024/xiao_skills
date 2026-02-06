# RGB Matrix Board for XIAO - Arduino

## Overview

Arduino implementation for the XIAO RGB Matrix Board. Drives RGB LED matrix panels (32x32, 64x64, etc.) for displaying graphics, animations, text, and video content.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/rgb-matrix.md`.

## Pin Connections

| Function | XIAO Pin | Notes |
|----------|----------|-------|
| Data | D8/D9/D10 | Usually SPI or dedicated pins |
| Clock | D8 | Often used as SPI SCK |
| Latch | D9 or D10 | Panel specific |
| Enable | D3 | Panel specific |

**Important:** RGB matrices require significant current (2A+). Always use external power supply - do NOT power from XIAO's 3.3V regulator.

## Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | Recommended (WiFi, adequate RAM) |
| ESP32S3 | ✅ | Recommended (more RAM for larger matrices) |
| nRF52840 | ✅ | Full support (BLE applications) |
| RP2040 | ✅ | Full support (good PIO support) |
| SAMD21 | ⚠️ | Limited RAM for larger matrices |

## Required Libraries

Install via Arduino Library Manager:

| Library | Purpose |
|---------|---------|
| **PxMatrix** | 32x32 and 64x64 matrix driver |
| **FastLED** | Alternative for matrix control |

## Basic 32x32 Display (PxMatrix)

```cpp
#include <PxMatrix.h>

// Pin configuration
#define P_LAT  D9
#define P_A    D0
#define P_B    D1
#define P_C    D2
#define P_D    D3
#define P_E    -1  // Not used for 32x32
#define P_OE   D8

PxMATRIX matrix(32, 32, P_LAT, P_OE, P_A, P_B, P_C, P_D, P_E);

void setup() {
  matrix.begin(8);  // 8 = color depth
  matrix.flushDisplay();
}

void loop() {
  matrix.fillScreen(matrix.color565(0, 0, 0));
  matrix.drawRect(0, 0, 32, 32, matrix.color565(255, 0, 0));
  matrix.setCursor(5, 10);
  matrix.print("XIAO");
  matrix.showBuffer();
  delay(1000);
}
```

## Scrolling Text

```cpp
#include <PxMatrix.h>

#define P_LAT  D9
#define P_A    D0
#define P_B    D1
#define P_C    D2
#define P_OE   D8

PxMATRIX matrix(32, 32, P_LAT, P_OE, P_A, P_B, P_C);
int x = 32;

void setup() {
  matrix.begin(8);
  matrix.flushDisplay();
}

void loop() {
  matrix.fillScreen(0);
  matrix.setCursor(x, 10);
  matrix.setTextColor(matrix.color565(0, 255, 0));
  matrix.print("Hello XIAO!");
  matrix.showBuffer();

  x--;
  if (x < -100) x = 32;

  delay(50);
}
```

## Drawing Graphics

```cpp
#include <PxMatrix.h>

#define P_LAT  D9
#define P_A    D0
#define P_B    D1
#define P_C    D2
#define P_OE   D8

PxMATRIX matrix(32, 32, P_LAT, P_OE, P_A, P_B, P_C);

void setup() {
  matrix.begin(8);
}

void drawGraphics() {
  matrix.fillScreen(0);

  // Draw circle
  matrix.fillCircle(16, 16, 10, matrix.color565(255, 0, 0));

  // Draw rectangle
  matrix.fillRect(5, 5, 10, 10, matrix.color565(0, 255, 0));

  // Draw line
  matrix.drawLine(0, 0, 31, 31, matrix.color565(0, 0, 255));

  matrix.showBuffer();
}

void loop() {
  drawGraphics();
  delay(1000);
}
```

## Color Patterns

```cpp
#include <PxMatrix.h>

#define P_LAT  D9
#define P_A    D0
#define P_B    D1
#define P_C    D2
#define P_OE   D8

PxMATRIX matrix(32, 32, P_LAT, P_OE, P_A, P_B, P_C);

void setup() {
  matrix.begin(8);
}

void loop() {
  // Red gradient
  for (int y = 0; y < 32; y++) {
    for (int x = 0; x < 32; x++) {
      int brightness = (x * 255) / 32;
      matrix.drawPixel(x, y, matrix.color565(brightness, 0, 0));
    }
  }
  matrix.showBuffer();
  delay(1000);

  // Green gradient
  for (int y = 0; y < 32; y++) {
    for (int x = 0; x < 32; x++) {
      int brightness = (y * 255) / 32;
      matrix.drawPixel(x, y, matrix.color565(0, brightness, 0));
    }
  }
  matrix.showBuffer();
  delay(1000);

  // Rainbow
  for (int i = 0; i < 255; i++) {
    matrix.fillScreen(matrix.color565(i, 255-i, (i*2)%255));
    matrix.showBuffer();
    delay(20);
  }
}
```

## Animation Effects

```cpp
#include <PxMatrix.h>

#define P_LAT  D9
#define P_A    D0
#define P_B    D1
#define P_C    D2
#define P_OE   D8

PxMATRIX matrix(32, 32, P_LAT, P_OE, P_A, P_B, P_C);

void setup() {
  matrix.begin(8);
}

void loop() {
  // Bouncing ball
  int x = 16, y = 16;
  int dx = 1, dy = 1;

  for (int i = 0; i < 200; i++) {
    matrix.fillScreen(0);

    x += dx;
    y += dy;

    if (x <= 0 || x >= 31) dx = -dx;
    if (y <= 0 || y >= 31) dy = -dy;

    matrix.fillCircle(x, y, 3, matrix.color565(255, 255, 0));
    matrix.showBuffer();
    delay(50);
  }
}
```

## Brightness Control

```cpp
#include <PxMatrix.h>

#define P_LAT  D9
#define P_A    D0
#define P_B    D1
#define P_C    D2
#define P_OE   D8

PxMATRIX matrix(32, 32, P_LAT, P_OE, P_A, P_B, P_C);
int brightness = 255;
int direction = -5;

void setup() {
  matrix.begin(8);
}

void loop() {
  matrix.fillScreen(0);
  matrix.setCursor(5, 10);
  matrix.setTextColor(matrix.color565(brightness, brightness, brightness));
  matrix.print("XIAO");
  matrix.showBuffer();

  brightness += direction;
  if (brightness <= 50 || brightness >= 255) {
    direction = -direction;
  }

  delay(100);
}
```

## Power Requirements

**Critical:** RGB LED matrices require significant power:

| Panel Size | Current | Voltage |
|------------|---------|---------|
| 32×32 | ~2A | 5V |
| 64×64 | ~4A | 5V |
| Larger | Proportional | 5V |

**Power Supply:**
- Use external 5V power supply
- Connect power directly to matrix panel
- DO NOT power through XIAO

**Current Calculation:**
- All white = maximum current
- Typical display = ~1/3 maximum
- Add 20% safety margin

## Troubleshooting

### Dim Display

**Symptom:** Matrix too dim

**Possible Causes:**
1. Insufficient power
2. Wrong voltage
3. Low brightness setting

**Solution:**
- Use adequate 5V power supply (2A+ for 32x32)
- Check brightness in code
- Verify connections

### Flickering

**Symptom:** Display flickers randomly

**Possible Causes:**
1. Refresh rate too low
2. Insufficient current
3. Poor connections

**Solution:**
- Increase refresh rate in code
- Ensure adequate power supply
- Check all wiring connections

### Wrong Colors

**Symptom:** Colors don't match expected

**Possible Causes:**
1. RGB order wrong
2. Color correction needed

**Solution:**
```cpp
// Try different RGB orders
// PxMatrix: set color order in constructor
// FastLED: define RGB ordering
#define RGB_ORDER_GRB
```

### No Display

**Symptom:** Matrix remains dark

**Possible Causes:**
1. No external power
2. Wrong pins
3. Library issue

**Solution:**
- Verify external power connected
- Check pin connections
- Verify correct library and settings

## Color Correction

Different panels may need color correction:

```cpp
// Adjust color balance
void drawCorrectedColor(int r, int g, int b) {
  // Apply correction factors
  r = r * 0.8;  // Red may be too bright
  g = g * 1.0;
  b = b * 1.2;  // Blue may be too dim

  matrix.drawPixel(x, y, matrix.color565(r, g, b));
}
```

## Panel Chaining

Multiple panels can be chained for larger displays:

```
[Panel 1] OUT → [Panel 2] IN → [Panel 3] IN...
```

**Notes:**
- Ensure adequate power supply for total current
- Update width/height in code
- Current scales with panel count
