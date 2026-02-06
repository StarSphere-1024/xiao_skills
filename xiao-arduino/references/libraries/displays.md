# Display Libraries for XIAO

## Overview

Display support for OLED, LCD, TFT, and e-paper displays.

## SSD1306 OLED (I2C)

### Basic Text

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

void setup() {
    Wire.begin();
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.clearDisplay();
    display.setTextSize(1);
    display.setTextColor(SSD1306_WHITE);
    display.setCursor(0, 0);
    display.println("Hello XIAO!");
    display.display();
}

void loop() {}
```

### Graphics

```cpp
void drawGraphics() {
    display.clearDisplay();

    // Draw pixel
    display.drawPixel(10, 10, SSD1306_WHITE);

    // Draw line
    display.drawLine(0, 0, 127, 63, SSD1306_WHITE);

    // Draw rectangle
    display.drawRect(10, 10, 50, 30, SSD1306_WHITE);

    // Draw filled rectangle
    display.fillRect(20, 20, 30, 20, SSD1306_WHITE);

    // Draw circle
    display.drawCircle(64, 32, 15, SSD1306_WHITE);

    // Draw filled circle
    display.fillCircle(80, 32, 10, SSD1306_WHITE);

    // Draw triangle
    display.drawTriangle(30, 50, 40, 30, 50, 50, SSD1306_WHITE);

    display.display();
}
```

### Custom Fonts

```cpp
#include <Fonts/FreeSans9pt7b.h>
#include <Fonts/FreeSans18pt7b.h>

void setup() {
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.clearDisplay();
    display.setFont(&FreeSans18pt7b);
    display.setTextSize(1);
    display.setTextColor(SSD1306_WHITE);
    display.setCursor(0, 30);
    display.println("Big Text!");
    display.display();
}
```

### Progress Bar

```cpp
void drawProgressBar(int progress) {
    display.clearDisplay();
    display.drawRect(0, 30, 128, 10, SSD1306_WHITE);
    display.fillRect(0, 30, progress, 10, SSD1306_WHITE);
    display.display();
}
```

### Invert Display

```cpp
display.invertDisplay(true);
delay(1000);
display.invertDisplay(false);
```

## ST7735 TFT (SPI)

### Basic Setup

```cpp
#include <SPI.h>
#include <Adafruit_GFX.h>
#include <Adafruit_ST7735.h>

#define TFT_CS D4
#define TFT_DC D3
#define TFT_RST D2

Adafruit_ST7735 tft = Adafruit_ST7735(TFT_CS, TFT_DC, TFT_RST);

void setup() {
    SPI.begin();
    tft.initR(INITR_144GREENTAB);  // 1.44" display
    tft.fillScreen(ST7735_BLACK);
    tft.setTextColor(ST7735_WHITE);
    tft.setTextSize(2);
    tft.setCursor(10, 30);
    tft.print("XIAO TFT");
}
```

### Colors

```cpp
// Define custom colors
#define MY_COLOR tft.Color565(255, 100, 50)

void drawColors() {
    tft.fillScreen(ST7735_BLACK);
    tft.fillRect(0, 0, 40, 40, ST7735_RED);
    tft.fillRect(40, 0, 40, 40, ST7735_GREEN);
    tft.fillRect(80, 0, 40, 40, ST7735_BLUE);
    tft.fillRect(0, 40, 40, 40, ST7735_YELLOW);
    tft.fillRect(40, 40, 40, 40, ST7735_MAGENTA);
    tft.fillRect(80, 40, 40, 40, ST7735_CYAN);
}
```

### Draw Bitmap

```cpp
const unsigned char PROGMEM logo[] = {
    0x00, 0x00, 0x00, 0x00, // Bitmap data...
};

void drawBitmap() {
    tft.drawBitmap(10, 10, logo, 48, 48, ST7735_WHITE);
}
```

### Rotation

```cpp
tft.setRotation(0);  // 0-3, different orientations
```

## ILI9341 TFT

### Basic Setup

```cpp
#include <Adafruit_ILI9341.h>

#define TFT_CS D4
#define TFT_DC D3
#define TFT_RST D2

Adafruit_ILI9341 tft = Adafruit_ILI9341(TFT_CS, TFT_DC, TFT_RST);

void setup() {
    SPI.begin();
    tft.begin();
    tft.setRotation(1);
    tft.fillScreen(ILI9341_BLACK);
    tft.setCursor(0, 0);
    tft.setTextColor(ILI9341_WHITE);
    tft.setTextSize(2);
    tft.println("ILI9341");
}
```

### Touch (XPT2046)

```cpp
#include <XPT2046_Touchscreen.h>

#define TOUCH_CS D5
XPT2046_Touchscreen ts(TOUCH_CS);

void setup() {
    ts.begin();
    ts.setRotation(1);
}

void loop() {
    if (ts.touched()) {
        TS_Point p = ts.getPoint();
        // Map to display coordinates
        int x = map(p.x, 200, 3700, 0, tft.width());
        int y = map(p.y, 200, 3700, 0, tft.height());
        tft.fillCircle(x, y, 5, ILI9341_RED);
    }
}
```

## E-Paper (Waveshare)

### Basic E-Paper

```cpp
#include <GxEPD.h>
#include <GxGDEW0213I5F/GxGDEW0213I5F.h>  // 2.13" b/w/red
#include <GxIO/GxIO_SPI/GxIO_SPI.h>
#include <GxIO/GxIO.h>

GxIO_Class io(SPI, D4, D3, D2);
GxEPD_Class display(io, D2, D1);

void setup() {
    display.init();
    display.eraseDisplay();
    display.setTextColor(GxEPD_BLACK);
    display.setFont(&FreeSans9pt7b);
    display.setCursor(0, 30);
    display.println("E-Paper!");
    display.update();
}
```

### Partial Update

```cpp
void loop() {
    display.fillRect(0, 50, 50, 20, GxEPD_WHITE);
    display.setCursor(0, 65);
    display.println(millis() / 1000);
    display.updateWindow(0, 50, 50, 20);
    delay(1000);
}
```

## Max7219 7-Segment

```cpp
#include <LedControl.h>

#define DIN D5
#define CLK D7
#define CS D4

LedControl lc = LedControl(DIN, CLK, CS, 1);

void setup() {
    lc.shutdown(0, false);
    lc.setIntensity(0, 8);
    lc.clearDisplay(0);
}

void loop() {
    lc.setDigit(0, 0, random(10), false);
    lc.setDigit(0, 1, random(10), false);
    delay(100);
}
```

## HD44780 LCD (I2C)

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);  // Address, columns, rows

void setup() {
    Wire.begin();
    lcd.init();
    lcd.backlight();
    lcd.setCursor(0, 0);
    lcd.print("Hello XIAO!");
    lcd.setCursor(0, 1);
    lcd.print("LCD Display");
}

void loop() {}
```

### Custom Characters

```cpp
byte smiley[8] = {
    B00000,
    B10001,
    B00000,
    B00100,
    B00000,
    B10001,
    B01110,
    B00000
};

void setup() {
    lcd.createChar(0, smiley);
    lcd.setCursor(0, 0);
    lcd.write((byte)0);
}
```

## TM1637 4-Digit Display

```cpp
#include <TM1637Display.h>

#define CLK D6
#define DIO D5

TM1637Display display(CLK, DIO);

void setup() {
    display.setBrightness(0x0A);
}

void loop() {
    int value = 1234;
    display.showNumberDec(value);
    delay(1000);
}
```

## Display Power Considerations

```cpp
// E-Paper sleep
display.powerDown();

// TFT backlight control
#define TFT_BL D6
pinMode(TFT_BL, OUTPUT);
digitalWrite(TFT_BL, LOW);  // Off
digitalWrite(TFT_BL, HIGH); // On

// OLED off
display.ssd1306_command(SSD1306_DISPLAYOFF);
```

## Troubleshooting

### No display

1. Check I2C address with scanner
2. Verify SPI wiring (MOSI/MISO/SCK/CS)
3. Check contrast: `display.setContrast(0x7F)`

### Wrong colors

1. Check color initialization code
2. Try RGB vs BGR initialization

### Slow refresh

1. Reduce update frequency
2. Use partial update (e-paper)
3. Optimize draw calls (batch changes)
