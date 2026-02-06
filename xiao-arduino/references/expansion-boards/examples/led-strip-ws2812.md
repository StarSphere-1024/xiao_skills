# LED Strip Examples - Arduino

## Overview

Complete Arduino examples for controlling WS2812/WS2812B LED strips using the XIAO LED Driver Board. Examples demonstrate color patterns, animations, and WLED integration.

**Hardware Reference:** `/xiao/references/expansion_boards/led-driver.md`
**Arduino Implementation:** `/xiao-arduino/references/expansion-boards/led-driver.md`

## Required Libraries

Install via Arduino Library Manager:
- **Adafruit NeoPixel** - WS2812/WS2812B control
- **FastLED** - Alternative LED library

## Example 1: Basic Color Control

```cpp
#include <Adafruit_NeoPixel.h>

#define LED_PIN    D0  // Data pin
#define LED_COUNT  30  // Number of LEDs

Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();
  strip.show();  // Initialize all to 'off'
  strip.setBrightness(50);  // 0-255
}

void loop() {
  // Set all LEDs to red
  strip.fill(strip.Color(255, 0, 0));
  strip.show();
  delay(1000);

  // Set all LEDs to green
  strip.fill(strip.Color(0, 255, 0));
  strip.show();
  delay(1000);

  // Set all LEDs to blue
  strip.fill(strip.Color(0, 0, 255));
  strip.show();
  delay(1000);

  // Turn off all LEDs
  strip.clear();
  strip.show();
  delay(1000);
}
```

## Example 2: Rainbow Cycle

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

## Example 3: Theater Chase

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

## Example 4: Larson Scanner (KITT)

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
  larsonScanner(strip.Color(255, 0, 0), 50);
}

void larsonScanner(uint32_t color, int wait) {
  // Scan forward
  for (int i = 0; i < strip.numPixels(); i++) {
    strip.clear();
    // Main pixel
    strip.setPixelColor(i, color);
    // Trailing pixels
    if (i > 0) strip.setPixelColor(i - 1, dimColor(color, 2));
    if (i > 1) strip.setPixelColor(i - 2, dimColor(color, 4));
    if (i > 2) strip.setPixelColor(i - 3, dimColor(color, 8));
    strip.show();
    delay(wait);
  }

  // Scan backward
  for (int i = strip.numPixels() - 1; i >= 0; i--) {
    strip.clear();
    strip.setPixelColor(i, color);
    if (i < strip.numPixels() - 1) strip.setPixelColor(i + 1, dimColor(color, 2));
    if (i < strip.numPixels() - 2) strip.setPixelColor(i + 2, dimColor(color, 4));
    if (i < strip.numPixels() - 3) strip.setPixelColor(i + 3, dimColor(color, 8));
    strip.show();
    delay(wait);
  }
}

uint32_t dimColor(uint32_t color, int divisor) {
  uint8_t r = (uint8_t)(color >> 16);
  uint8_t g = (uint8_t)(color >> 8);
  uint8_t b = (uint8_t)color;
  return strip.Color(r / divisor, g / divisor, b / divisor);
}
```

## Example 5: Twinkle Effect

```cpp
#include <Adafruit_NeoPixel.h>

#define LED_PIN    D0
#define LED_COUNT  30

Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();
  strip.setBrightness(80);
}

void loop() {
  twinkle(strip.Color(255, 255, 255), 100);
}

void twinkle(uint32_t color, int wait) {
  for (int i = 0; i < 50; i++) {
    // Clear all LEDs
    strip.clear();

    // Randomly set some LEDs to the color
    for (int j = 0; j < strip.numPixels() / 3; j++) {
      int pixel = random(strip.numPixels());
      strip.setPixelColor(pixel, color);
    }

    strip.show();
    delay(wait);
  }
}
```

## Example 6: Color Temperature Adjustment

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
  // Fade from warm to cool white
  for (int kelvin = 2700; kelvin <= 6500; kelvin += 100) {
    uint32_t color = kelvinToRGB(kelvin);
    strip.fill(color);
    strip.show();
    delay(100);
  }

  delay(1000);

  // Fade back
  for (int kelvin = 6500; kelvin >= 2700; kelvin -= 100) {
    uint32_t color = kelvinToRGB(kelvin);
    strip.fill(color);
    strip.show();
    delay(100);
  }

  delay(1000);
}

uint32_t kelvinToRGB(int kelvin) {
  int r, g, b;

  if (kelvin <= 6600) {
    r = 255;
    g = 99.47 * log(kelvin / 100.0) - 161.12;
    b = 0;
  } else {
    r = 329.7 * pow(kelvin / 100.0 - 60, -0.133);
    g = 288.12 * pow(kelvin / 100.0 - 60, -0.0755);
    b = 255;
  }

  r = constrain(r, 0, 255);
  g = constrain(g, 0, 255);
  b = constrain(b, 0, 255);

  return strip.Color(r, g, b);
}
```

## Example 7: Audio Visualizer (Simulated)

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
  // Simulated audio visualizer
  for (int i = 0; i < 100; i++) {
    strip.clear();

    // Create "wave" effect
    for (int j = 0; j < strip.numPixels(); j++) {
      int intensity = sin((j + i) * 0.3) * 127 + 128;
      uint8_t hue = (j + i) * 5;
      strip.setPixelColor(j, strip.ColorHSV(hue * 256, 255, intensity));
    }

    strip.show();
    delay(50);
  }
}
```

## Example 8: Individual LED Control

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
  strip.clear();

  for (int i = 0; i < strip.numPixels(); i++) {
    // Create gradient pattern
    uint8_t red = (i * 255) / strip.numPixels();
    uint8_t green = ((strip.numPixels() - i) * 255) / strip.numPixels();
    uint8_t blue = 128;

    strip.setPixelColor(i, strip.Color(red, green, blue));
  }

  strip.show();
  delay(2000);
}
```

## Example 9: FastLED Alternative

```cpp
#include <FastLED.h>

#define LED_PIN     D0
#define LED_TYPE    WS2812B
#define COLOR_ORDER GRB
#define NUM_LEDS    30
#define BRIGHTNESS  100

CRGB leds[NUM_LEDS];

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

  // Rainbow
  fill_rainbow(leds, NUM_LEDS, 0, 255 / NUM_LEDS);
  FastLED.show();
  delay(2000);
}
```

## Example 10: Brightness Fade

```cpp
#include <Adafruit_NeoPixel.h>

#define LED_PIN    D0
#define LED_COUNT  30

Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();
}

void loop() {
  // Fade in
  for (int brightness = 0; brightness <= 255; brightness += 5) {
    strip.setBrightness(brightness);
    strip.fill(strip.Color(0, 150, 255));
    strip.show();
    delay(30);
  }

  delay(1000);

  // Fade out
  for (int brightness = 255; brightness >= 0; brightness -= 5) {
    strip.setBrightness(brightness);
    strip.fill(strip.Color(0, 150, 255));
    strip.show();
    delay(30);
  }

  delay(1000);
}
```

## Power Calculations

**Current per LED (at full white):**
- Red: 20mA
- Green: 20mA
- Blue: 20mA
- **Total: 60mA per LED**

**For 30 LEDs at full white:** 30 × 60mA = 1.8A

**Recommended Power Supply:**
| LED Count | Min Current | Recommended Supply |
|-----------|-------------|-------------------|
| 10 | 0.6A | 1A 5V supply |
| 30 | 1.8A | 3A 5V supply |
| 60 | 3.6A | 5A 5V supply |
| 100 | 6A | 10A 5V supply |

## WLED Integration (ESP32C3 Only)

1. Install WLED via web installer
2. Connect LED strip data to A0 (D0)
3. Power LED strip from external supply
4. Control via WiFi using WLED web interface
5. Configure for WS2812B in WLED settings

## Troubleshooting

### First LED Wrong Color

**Symptom:** First LED shows different color than rest

**Solution:** Add dummy LED at start or ignore in code

### Flickering

**Symptom:** LEDs flicker randomly

**Solution:**
- Ensure adequate power supply
- Check ground connection
- Reduce brightness

### Wrong Colors

**Symptom:** Colors don't match expected

**Solution:** Try different RGB orders:
```cpp
NEO_GRB + NEO_KHZ800  // Most common
NEO_RGB + NEO_KHZ800  // Some strips
NEO_BRG + NEO_KHZ800  // Rare
```

## Safety Notes

- **Always use external power** for LED strips
- **Check LED strip voltage** (5V vs 12V) before connecting
- **Use appropriate wire gauge** for high current
- **Consider heat** for high-brightness applications
