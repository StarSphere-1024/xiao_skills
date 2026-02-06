# Adafruit GFX Library for XIAO

## Overview

Adafruit GFX is the core graphics library for Adafruit displays. Provides drawing primitives, fonts, and canvas operations.

## Board Compatibility

All XIAO boards support GFX displays:
- ESP32C3/C5/C6/S3: SPI or I2C
- nRF52840/MG24: SPI or I2C
- RP2040/RP2350: SPI or I2C
- SAMD21/RA4M1: SPI or I2C

## Basic Graphics Operations

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.clearDisplay();

    // Basic drawing
    display.drawPixel(10, 10, SSD1306_WHITE);
    display.drawLine(0, 0, 127, 63, SSD1306_WHITE);
    display.drawRect(20, 20, 40, 30, SSD1306_WHITE);
    display.fillCircle(64, 32, 10, SSD1306_WHITE);

    display.display();
}

void loop() {}
```

## Text Display

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.clearDisplay();

    // Text settings
    display.setTextSize(1);      // 1x, 2x, 3x, etc.
    display.setTextColor(SSD1306_WHITE);  // or SSD1306_BLACK, or SSD1306_INVERSE

    // Text at position
    display.setCursor(0, 0);
    display.println("Hello XIAO!");

    // Larger text
    display.setTextSize(2);
    display.setCursor(0, 16);
    display.print("Big Text");

    display.display();
}

void loop() {}
```

## Shapes

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

void drawShapes() {
    display.clearDisplay();

    // Rectangle
    display.drawRect(0, 0, 40, 30, SSD1306_WHITE);

    // Filled rectangle
    display.fillRect(50, 0, 40, 30, SSD1306_WHITE);

    // Rounded rectangle
    display.drawRoundRect(0, 35, 40, 25, 5, SSD1306_WHITE);

    // Circle
    display.drawCircle(70, 47, 10, SSD1306_WHITE);

    // Filled circle
    display.fillCircle(100, 47, 10, SSD1306_WHITE);

    // Triangle
    display.drawTriangle(110, 0, 120, 20, 100, 20, SSD1306_WHITE);

    display.display();
}

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    drawShapes();
}

void loop() {}
```

## Progress Bar

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

void drawProgressBar(int x, int y, int w, int h, int progress) {
    // Draw border
    display.drawRect(x, y, w, h, SSD1306_WHITE);

    // Draw fill
    int fillWidth = (w * progress) / 100;
    display.fillRect(x + 1, y + 1, fillWidth - 2, h - 2, SSD1306_WHITE);

    // Draw percentage text
    display.setTextSize(1);
    display.setCursor(x + w / 2 - 12, y + h + 2);
    display.printf("%d%%", progress);
}

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.clearDisplay();

    for (int i = 0; i <= 100; i += 10) {
        display.clearDisplay();
        drawProgressBar(10, 20, 108, 20, i);
        display.display();
        delay(200);
    }
}

void loop() {}
```

## Bitmap Display

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

// 8x8 smiley face bitmap
const unsigned char smiley_bmp[] PROGMEM = {
    0x3C, 0x42, 0xA5, 0x81, 0xA5, 0x99, 0x42, 0x3C
};

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.clearDisplay();

    // Draw bitmap
    display.drawBitmap(10, 10, smiley_bmp, 8, 8, SSD1306_WHITE);

    // Draw larger bitmap (scaled)
    display.drawBitmap(30, 10, smiley_bmp, 8, 8, SSD1306_WHITE, 2);  // 2x scale

    display.display();
}

void loop() {}
```

## Custom Fonts

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Fonts/FreeSans9pt7b.h>
#include <Fonts/FreeSans12pt7b.h>
#include <Fonts/FreeSans18pt7b.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.clearDisplay();

    // Use custom font
    display.setFont(&FreeSans9pt7b);
    display.setTextSize(1);
    display.setCursor(0, 15);
    display.println("9pt Font");

    display.setFont(&FreeSans12pt7b);
    display.setCursor(0, 35);
    display.println("12pt Font");

    display.setFont(&FreeSans18pt7b);
    display.setCursor(0, 60);
    display.println("18pt");

    display.display();
}

void loop() {}
```

## Text Wrapping

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

void printWrapped(const char* text, int x, int y, int maxWidth) {
    int16_t x1, y1;
    uint16_t w, h;

    String word = "";
    String line = "";
    int cursorX = x;
    int cursorY = y;

    for (int i = 0; text[i] != '\0'; i++) {
        if (text[i] == ' ') {
            // Check if word fits
            display.getTextBounds(word.c_str(), cursorX, cursorY, &x1, &y1, &w, &h);

            if (cursorX + w > x + maxWidth) {
                // Word doesn't fit, move to next line
                display.setCursor(x, cursorY);
                display.println(line);
                line = word + " ";
                cursorX = x;
                cursorY += h;
            } else {
                line += word + " ";
            }
            word = "";
        } else {
            word += text[i];
        }
    }

    // Print remaining
    display.setCursor(cursorX, cursorY);
    display.print(line + word);
}

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.clearDisplay();
    display.setTextSize(1);

    printWrapped("This is a long text that will wrap to multiple lines on the display.",
                0, 0, 128);

    display.display();
}

void loop() {}
```

## Scrolling Text

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

const char* scrollText = "This is a scrolling text message for the XIAO display! ";
int scrollPos = 0;

void scrollText() {
    display.clearDisplay();

    int16_t x1, y1;
    uint16_t w, h;

    // Get text width
    display.getTextBounds(scrollText, 0, 0, &x1, &y1, &w, &h);

    // Calculate position
    int x = 128 - scrollPos;
    if (x < -w) {
        scrollPos = 0;
        x = 128;
    }

    display.setCursor(x, 32);
    display.print(scrollText);

    scrollPos += 2;
    display.display();
}

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.setTextSize(1);
}

void loop() {
    scrollText();
    delay(50);
}
```

## Canvas (Double Buffering)

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);
GFXcanvas16 canvas(128, 64);  // 16-bit color canvas

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);

    // Draw to canvas (off-screen)
    canvas.fillScreen(0);
    canvas.setTextColor(0xFFFF);
    canvas.setTextSize(1);
    canvas.setCursor(10, 10);
    canvas.println("Canvas Buffer");

    canvas.drawCircle(64, 40, 15, 0xFFFF);

    // Copy canvas to display
    display.drawRGBBitmap(0, 0, canvas.getBuffer(), 128, 64);
    display.display();
}

void loop() {}
```

## Animated Sprite

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

Adafruit_SSD1306 display(128, 64, &Wire, -1);

// Simple 8x8 ball sprite
const uint8_t ball[] = {
    0x3C, 0x3C,  // **** ****
    0xFF, 0xFF,  // ********
    0xFF, 0xFF,  // ********
    0x3C, 0x3C   // **** ****
};

int ballX = 0;
int ballY = 28;
int velX = 2;
int velY = 1;

void animate() {
    display.clearDisplay();

    // Update position
    ballX += velX;
    ballY += velY;

    // Bounce off walls
    if (ballX <= 0 || ballX >= 120) velX *= -1;
    if (ballY <= 0 || ballY >= 56) velY *= -1;

    // Draw sprite
    display.drawBitmap(ballX, ballY, ball, 8, 8, SSD1306_WHITE);

    display.display();
    delay(30);
}

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
}

void loop() {
    animate();
}
```

## Color Display (RGB565)

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_ST7789.h>

Adafruit_ST7789 display = Adafruit_ST7789(&SPI, D7, D6, -1, D5);

void setup() {
    display.init(135, 240);  // XIAO round display
    display.setRotation(3);

    // RGB565 color values
    uint16_t red = display.color565(255, 0, 0);
    uint16_t green = display.color565(0, 255, 0);
    uint16_t blue = display.color565(0, 0, 255);

    // Draw colored shapes
    display.fillRect(0, 0, 80, 60, red);
    display.fillRect(80, 0, 80, 60, green);
    display.fillRect(0, 60, 80, 60, blue);
    display.fillRect(80, 60, 80, 60, display.color565(255, 255, 0));
}

void loop() {}
```

## Best Practices

```cpp
#include <Adafruit_GFX.h>

void setup() {
    // Best practice 1: Use clearDisplay() before new frame
    display.clearDisplay();

    // Best practice 2: Batch drawing operations
    // Draw all graphics, then call display() once
    display.drawRect(0, 0, 50, 50, WHITE);
    display.fillRect(60, 0, 50, 50, WHITE);
    display.display();  // Only call once

    // Best practice 3: Use PROGMEM for static bitmaps
    const uint8_t bitmap[] PROGMEM = { /* data */ };

    // Best practice 4: Use appropriate text size
    display.setTextSize(1);  // Don't use 2-3 unless needed

    // Best practice 5: Minimize display updates
    // Only update changed regions if possible
    display.display();
}

void loop() {}
```

## References

- [Adafruit GFX Library](https://github.com/adafruit/Adafruit-GFX-Library)
- [GFX Font Builder](https://learn.adafruit.com/adafruit-gfx-graphics-library/using-fonts)
