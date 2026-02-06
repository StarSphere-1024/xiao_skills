# LED Driver Board for XIAO - Arduino

## Overview

Arduino implementation for the XIAO LED Driver Board. Controls 5V/12V LED strips (WS2812/WS2812B) with WLED support. Ideal for addressable LED lighting projects.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/led-driver.md`.

## Pin Connections

| LED Terminal | Connection | Notes |
|--------------|------------|-------|
| 12V | 12V LED strips | External power required |
| 5V | 5V LED strips | External power required |
| A0 | Data signal | XIAO pin |

**Power Requirements:**
- **External Power:** Required for LED strips (do NOT power from XIAO)
- **Current:** 60mA per LED at full white (plan accordingly)
- **Voltage:** 5V or 12V depending on LED strip type

## Required Libraries

| Library | Purpose |
|---------|---------|
| **Adafruit NeoPixel** | WS2812/WS2812B control |
| **FastLED** | Alternative LED library |

## Basic WS2812 Control (NeoPixel)

```cpp
#include <Adafruit_NeoPixel.h>

#define LED_PIN    D0  // Data pin
#define LED_COUNT  30  // Number of LEDs

Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();
  strip.show();  // Initialize all pixels to 'off'
  strip.setBrightness(50);  // 0-255
}

void loop() {
  // Color wipe
  colorWipe(strip.Color(255, 0, 0), 50);  // Red
  colorWipe(strip.Color(0, 255, 0), 50);  // Green
  colorWipe(strip.Color(0, 0, 255), 50);  // Blue
}

void colorWipe(uint32_t color, int wait) {
  for (int i = 0; i < strip.numPixels(); i++) {
    strip.setPixelColor(i, color);
    strip.show();
    delay(wait);
  }
}
```

## Rainbow Effect

```cpp
#include <Adafruit_NeoPixel.h>

#define LED_PIN    D0
#define LED_COUNT  30

Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

long firstPixelHue = 0;

void setup() {
  strip.begin();
  strip.setBrightness(50);
}

void loop() {
  rainbow(10);
}

void rainbow(int wait) {
  for (long firstPixelHue = 0; firstPixelHue < 5 * 65536; firstPixelHue += 256) {
    for (int i = 0; i < strip.numPixels(); i++) {
      int pixelHue = firstPixelHue + (i * 65536L / strip.numPixels());
      strip.setPixelColor(i, strip.gamma32(strip.ColorHSV(pixelHue)));
    }
    strip.show();
    delay(wait);
  }
}
```

## Theater Chase

```cpp
#include <Adafruit_NeoPixel.h>

#define LED_PIN    D0
#define LED_COUNT  30

Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();
  strip.setBrightness(50);
}

void loop() {
  theaterChase(strip.Color(127, 127, 127), 50);  // White
  theaterChase(strip.Color(127, 0, 0), 50);      // Red
  theaterChase(strip.Color(0, 0, 127), 50);      // Blue
}

void theaterChase(uint32_t color, int wait) {
  for (int a = 0; a < 10; a++) {
    for (int b = 0; b < 3; b++) {
      strip.clear();
      for (int c = b; c < strip.numPixels(); c += 3) {
        strip.setPixelColor(c, color);
      }
      strip.show();
      delay(wait);
    }
  }
}
```

## Using FastLED (Alternative)

```cpp
#include <FastLED.h>

#define LED_PIN     D0
#define LED_TYPE    WS2812B
#define COLOR_ORDER GRB
#define NUM_LEDS    30

CRGB leds[NUM_LEDS];
#define BRIGHTNESS 50

void setup() {
  FastLED.addLeds<LED_TYPE, LED_PIN, COLOR_ORDER>(leds, NUM_LEDS);
  FastLED.setBrightness(BRIGHTNESS);
}

void loop() {
  // Fill with solid color
  fill_solid(leds, NUM_LEDS, CRGB::Red);
  FastLED.show();
  delay(500);

  fill_solid(leds, NUM_LEDS, CRGB::Green);
  FastLED.show();
  delay(500);

  fill_solid(leds, NUM_LEDS, CRGB::Blue);
  FastLED.show();
  delay(500);
}
```

## Individual LED Control

```cpp
#include <Adafruit_NeoPixel.h>

#define LED_PIN    D0
#define LED_COUNT  30

Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();
  strip.setBrightness(100);
}

void loop() {
  // Set individual LEDs
  strip.setPixelColor(0, strip.Color(255, 0, 0));    // LED 0: Red
  strip.setPixelColor(1, strip.Color(0, 255, 0));    // LED 1: Green
  strip.setPixelColor(2, strip.Color(0, 0, 255));    // LED 2: Blue
  strip.setPixelColor(3, strip.Color(255, 255, 0));  // LED 3: Yellow
  strip.setPixelColor(4, strip.Color(255, 0, 255));  // LED 4: Magenta
  strip.setPixelColor(5, strip.Color(0, 255, 255));  // LED 5: Cyan

  strip.show();
  delay(1000);

  // Turn off all LEDs
  strip.clear();
  strip.show();
  delay(500);
}
```

## ESP32C3 with WLED

The LED Driver board supports WLED firmware on XIAO ESP32C3:

1. Install WLED via web installer
2. Connect LED strip data to A0 (D0)
3. Power LED strip from external supply
4. Control via WiFi using WLED web interface

## MG24 Note

For XIAO MG24, use the ezWS2812 library instead of standard NeoPixel:

```cpp
#include <ezWS2812.h>
// MG24-specific library for WS2812 control
```

## Power Calculations

**Current per LED (typical):**
- Red: 20mA
- Green: 20mA
- Blue: 20mA
- White (all on): 60mA

**For 30 LEDs at full white:** 30 × 60mA = 1.8A

**Power Supply Recommendations:**
| LED Count | Minimum Current | Recommended Supply |
|-----------|-----------------|-------------------|
| 10 | 0.6A | 1A 5V supply |
| 30 | 1.8A | 3A 5V supply |
| 60 | 3.6A | 5A 5V supply |
| 100 | 6A | 10A 5V supply |

## Troubleshooting

### LEDs Not Lighting

**Symptom:** LED strip stays dark

**Possible Causes:**
1. No external power - LED strip needs external power
2. Wrong data pin - Must be D0 (A0)
3. Ground not connected - Connect LED GND to XIAO GND

**Solution:**
- Verify external power connected (5V or 12V)
- Check data connection on A0
- Ensure common ground

### Wrong Colors

**Symptom:** Colors don't match expected

**Possible Causes:**
1. RGB order wrong - Check strip type
2. Color order setting wrong

**Solution:**
```cpp
// Try different color orders
NEO_GRB + NEO_KHZ800  // Most common
NEO_RGB + NEO_KHZ800  // Some strips
NEO_BRG + NEO_KHZ800  // Rare
```

### Flickering LEDs

**Symptom:** LEDs flicker randomly

**Possible Causes:**
1. Insufficient power
2. Loose connections
3. PWM frequency conflict

**Solution:**
- Use adequate power supply
- Check all connections
- Reduce brightness to lower current draw

### First LED Wrong Color

**Symptom:** First LED shows different color than rest

**Possible Causes:**
1. Data corruption at start
2. Power-on issue

**Solution:** Add dummy LED at start (not installed) or ignore in code

## Safety Notes

- **Always use external power** for LED strips - XIAO cannot supply enough current
- **Check LED strip voltage** (5V vs 12V) before connecting
- **Consider heat dissipation** for high-brightness applications
- **Use appropriate wire gauge** for high current (18AWG or better for 5A+)
