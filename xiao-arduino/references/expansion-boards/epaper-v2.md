# ePaper V2 for XIAO - Arduino

## Overview

Arduino implementation for the Seeed Studio ePaper V2 Expansion Board. Supports 7 ePaper display sizes from 1.54" to 7.5". Low-power e-paper technology retains images without power.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/epaper-v2.md`.

## Pin Connections

| ePaper Pin | XIAO Pin | Function |
|------------|----------|----------|
| RST | D0 | Reset |
| CS | D1 | Chip Select |
| DC | D3 | Data/Command |
| BUSY | D2 | Busy signal |
| SCK | D8 | SPI Clock |
| MOSI | D10 | SPI MOSI |

**Important:** SAMD21 limited - RAM overflow for 4.26", 5.65", 7.5" displays. Use ESP32 or RP2040 for larger displays.

## Supported Display Sizes

| Display | Resolution | Recommended Board |
|---------|------------|-------------------|
| 1.54" | 200×200 | All XIAO boards |
| 2.13" | 250×122 | All XIAO boards |
| 2.13" B | 250×122 | All XIAO boards |
| 2.66" | 296×152 | ESP32/RP2040 recommended |
| 2.9" | 296×128 | All XIAO boards |
| 4.2" | 400×300 | ESP32/RP2040 recommended |
| 4.26" | 480×800 | ESP32/RP2040 only |
| 5.65" | 600×448 | ESP32/RP2040 only |
| 7.5" | 640×384 | ESP32/RP2040 only |

## Required Libraries

Install these libraries via Arduino Library Manager:

| Library | Purpose |
|---------|---------|
| **GxEPD2** | ePaper display driver |
| **SPI** | Built-in SPI library |

## Basic 2.13" Display Example

```cpp
#include <GxEPD2_BW.h>
#include <GxEPD2_3C.h>
#include <Fonts/FreeSans9pt7b.h>

// 2.13" BW display
GxEPD2_BW<GxEPD2_213_BN, GxEPD2_213_BN::HEIGHT>
  display(GxEPD2_213_BN(D1, D3, D0, D2, D8, D10));
  // CS, DC, RST, BUSY, SCK, MOSI

void setup() {
  display.init(115200);
  display.setRotation(1);
  display.setFont(&FreeSans9pt7b);
}

void loop() {
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setCursor(10, 50);
    display.print("Hello XIAO!");
  } while (display.nextPage());
  delay(5000);
}
```

## 2.9" Display Example

```cpp
#include <GxEPD2_BW.h>

GxEPD2_BW<GxEPD2_290_T94, GxEPD2_290_T94::HEIGHT>
  display(GxEPD2_290_T94(D1, D3, D0, D2, D8, D10));

void setup() {
  display.init();
  display.setRotation(1);
}

void loop() {
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setCursor(20, 100);
    display.print("2.9 inch Display");
  } while (display.nextPage());
  delay(10000);
}
```

## 4.2" Display Example (ESP32/RP2040 Only)

```cpp
#include <GxEPD2_BW.h>

GxEPD2_BW<GxEPD2_420, GxEPD2_420::HEIGHT>
  display(GxEPD2_420(D1, D3, D0, D2, D8, D10));

void setup() {
  display.init();
  display.setRotation(1);
}

void loop() {
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setCursor(50, 200);
    display.setTextSize(2);
    display.print("4.2 inch");
  } while (display.nextPage());
  delay(30000);
}
```

## Partial Refresh (Faster Updates)

```cpp
#include <GxEPD2_BW.h>

GxEPD2_BW<GxEPD2_213_BN, GxEPD2_213_BN::HEIGHT>
  display(GxEPD2_213_BN(D1, D3, D0, D2, D8, D10));

void setup() {
  display.init(115200);
  display.setRotation(1);
}

void loop() {
  // Full refresh (slower)
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setCursor(10, 50);
    display.print("Full Refresh");
  } while (display.nextPage());

  delay(5000);

  // Partial refresh (faster, but ghosting)
  display.setPartialWindow(0, 0, display.width(), 30);
  display.firstPage();
  do {
    display.fillRect(0, 0, display.width(), 30, GxEPD_WHITE);
    display.setCursor(10, 20);
    display.print("Partial Update");
  } while (display.nextPage());

  delay(1000);
}
```

## Three-Color Display

```cpp
#include <GxEPD2_3C.h>

GxEPD2_3C<GxEPD2_213_Z19c, GxEPD2_213_Z19c::HEIGHT>
  display(D1, D3, D0, D2, D8, D10);

void setup() {
  display.init();
  display.setRotation(1);
}

void loop() {
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setCursor(10, 50);
    display.print("Black Text");
    display.setTextColor(GxEPD_RED);
    display.setCursor(10, 80);
    display.print("Red Text");
  } while (display.nextPage());
  delay(10000);
}
```

## Drawing Graphics

```cpp
#include <GxEPD2_BW.h>

GxEPD2_BW<GxEPD2_213_BN, GxEPD2_213_BN::HEIGHT>
  display(GxEPD2_213_BN(D1, D3, D0, D2, D8, D10));

void setup() {
  display.init();
  display.setRotation(1);
}

void loop() {
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);

    // Draw rectangle
    display.fillRect(10, 10, 100, 50, GxEPD_BLACK);

    // Draw circle
    display.fillCircle(170, 35, 25, GxEPD_BLACK);

    // Draw line
    display.drawLine(10, 80, 230, 80, GxEPD_BLACK);

    // Draw text
    display.setTextColor(GxEPD_BLACK);
    display.setCursor(10, 110);
    display.print("Graphics Demo");
  } while (display.nextPage());
  delay(10000);
}
```

## Low Power Deep Sleep

```cpp
#include <GxEPD2_BW.h>

// ESP32 only
#include <esp_sleep.h>

GxEPD2_BW<GxEPD2_213_BN, GxEPD2_213_BN::HEIGHT>
  display(GxEPD2_213_BN(D1, D3, D0, D2, D8, D10));

void setup() {
  display.init();
  display.setRotation(1);

  // Update display once
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setCursor(10, 50);
    display.print("Sleeping...");
  } while (display.nextPage());

  // Enter deep sleep (ePaper retains image)
  esp_deep_sleep_start();
}

void loop() {}
```

## Troubleshooting

### Display Shows Nothing

**Symptom:** Display remains blank after upload

**Possible Causes:**
1. Wrong display type selected
2. SPI pins incorrect
3. Insufficient RAM (SAMD21 with large display)

**Solution:** Verify display class matches your ePaper model, check SPI connections

### Ghosting / Afterimages

**Symptom:** Previous image faintly visible

**Possible Causes:**
1. Too many partial refreshes
2. Need full refresh

**Solution:** Run a full refresh cycle to clear ghosting

### RAM Overflow on SAMD21

**Symptom:** Compilation error or sketch too large

**Possible Causes:**
1. Display too large for SAMD21 RAM
2. Font buffer too big

**Solution:** Use ESP32 or RP2040 for 4.26", 5.65", 7.5" displays

### Slow Refresh

**Symptom:** Display update takes 10+ seconds

**Normal:** ePaper refresh is inherently slow (2-15 seconds depending on size)

**Solution:** Use partial refresh for faster updates (with some ghosting)
