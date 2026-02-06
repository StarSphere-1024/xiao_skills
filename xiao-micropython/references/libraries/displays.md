# Display Libraries for XIAO (MicroPython)

## Overview

Complete guide for using displays with XIAO boards in MicroPython. Covers I2C OLED, SPI TFT, E-paper, and 7-segment displays.

---

## SSD1306 OLED (I2C)

The SSD1306 is a popular 128x64 I2C OLED display.

### Basic Setup

```python
from machine import Pin, I2C
import ssd1306
import time

# I2C setup (XIAO I2C pins: SDA=D4, SCL=D5)
i2c = I2C(0, sda=Pin(4), scl=Pin(5), freq=400000)

# Initialize display
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

# Clear display
oled.fill(0)
oled.show()

print("SSD1306 initialized")
```

### Drawing Functions

```python
from machine import Pin, I2C
import ssd1306

i2c = I2C(0, sda=Pin(4), scl=Pin(5))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

# Clear screen
oled.fill(0)

# Draw pixel
oled.pixel(64, 32, 1)
oled.show()

# Draw line
oled.line(0, 0, 127, 63, 1)
oled.show()

# Draw rectangle
oled.rect(10, 10, 50, 30, 1)
oled.show()

# Draw filled rectangle
oled.fill_rect(10, 10, 50, 30, 1)
oled.show()

# Draw circle
import math
def draw_circle(x0, y0, radius, color=1):
    x = radius
    y = 0
    err = 0

    while x >= y:
        oled.pixel(x0 + x, y0 + y, color)
        oled.pixel(x0 + y, y0 + x, color)
        oled.pixel(x0 - y, y0 + x, color)
        oled.pixel(x0 - x, y0 + y, color)
        oled.pixel(x0 - x, y0 - y, color)
        oled.pixel(x0 - y, y0 - x, color)
        oled.pixel(x0 + y, y0 - x, color)
        oled.pixel(x0 + x, y0 - y, color)

        y += 1
        if err <= 0:
            err += 2*y + 1
        if err > 0:
            x -= 1
            err -= 2*x + 1

draw_circle(64, 32, 20)
oled.show()
```

### Text Display

```python
from machine import Pin, I2C
import ssd1306

i2c = I2C(0, sda=Pin(4), scl=Pin(5))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

oled.fill(0)

# Simple text
oled.text("Hello XIAO!", 0, 0)
oled.text("Line 2", 0, 16)
oled.text("Line 3", 0, 32)
oled.show()

# Text wrapping
def text_wrap(text, x, y, max_width):
    words = text.split(' ')
    line = ''
    line_y = y

    for word in words:
        test_line = line + word + ' '
        if len(test_line) * 8 <= max_width:  # 8 pixels per char
            line = test_line
        else:
            oled.text(line, x, line_y)
            line_y += 10
            line = word + ' '

    if line:
        oled.text(line, x, line_y)

oled.fill(0)
text_wrap("This is a long text that wraps to multiple lines", 0, 0)
oled.show()
```

### Progress Bar

```python
from machine import Pin, I2C
import ssd1306
import time

i2c = I2C(0, sda=Pin(4), scl=Pin(5))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

def draw_progress(x, y, width, height, percent):
    # Draw outline
    oled.rect(x, y, width, height, 1)

    # Draw fill
    fill_width = int(width * percent / 100)
    if fill_width > 0:
        oled.fill_rect(x + 1, y + 1, fill_width - 1, height - 2, 1)

# Animate progress
for i in range(101):
    oled.fill(0)
    draw_progress(10, 20, 108, 20, i)
    oled.text(f"{i}%", 55, 25)
    oled.show()
    time.sleep(0.05)
```

### Bitmap Display

```python
from machine import Pin, I2C
import ssd1306

i2c = I2C(0, sda=Pin(4), scl=Pin(5))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

# 8x8 smiley face bitmap
SMILEY = bytearray([
    0b00111100,
    0b01000010,
    0b10100101,
    0b10000001,
    0b10100101,
    0b10011001,
    0b01000010,
    0b00111100
])

oled.fill(0)
oled.blit(ssd1306.SSD1306(8, 8, SMILEY), 60, 28)
oled.show()
```

### Scroll Text

```python
from machine import Pin, I2C
import ssd1306
import time

i2c = I2C(0, sda=Pin(4), scl=Pin(5))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

message = "Hello from XIAO ESP32! MicroPython rocks! "

oled.fill(0)

# Scroll horizontally
offset = 0
while True:
    oled.fill(0)

    # Draw text multiple times for seamless scroll
    for i in range(-1, 3):
        x = offset + i * len(message) * 8
        oled.text(message, x, 24)

    oled.show()

    offset -= 2
    if offset <= -len(message) * 8:
        offset = 0

    time.sleep(0.05)
```

### Multi-Page Display

```python
from machine import Pin, I2C
import ssd1306
import time

i2c = I2C(0, sda=Pin(4), scl=Pin(5))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

# Create virtual display for double buffering
class VirtualDisplay:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.buffer = bytearray(width * height // 8)

    def clear(self):
        self.buffer = bytearray(len(self.buffer))

    def text(self, str, x, y, col=1):
        # Simple text implementation
        ssd1306.SSD1306_I2C.text(self, str, x, y, col)

# Main display
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

# Buffer
buf = VirtualDisplay(128, 64)

# Animate
frame = 0
while True:
    buf.clear()
    buf.text(f"Frame: {frame}", 0, 0)
    buf.text(f"Time: {time.ticks_ms()}", 0, 16)

    # Copy to display
    oled.blit(buf, 0, 0)
    oled.show()

    frame += 1
    time.sleep(0.1)
```

---

## ST7789 TFT (SPI)

The ST7789 is a 240x240 IPS TFT display with 16-bit color.

### Basic Setup

```python
from machine import Pin, SPI
import st7789
import time

# SPI pins (XIAO ESP32S3)
SPI_SCK = Pin(36)
SPI_MOSI = Pin(35)
SPI_MISO = Pin(37)
LCD_CS = Pin(34)
LCD_DC = Pin(38)
LCD_RST = Pin(33)
LCD_BL = Pin(39)

# Initialize SPI
spi = SPI(2, baudrate=40000000, sck=SPI_SCK, mosi=SPI_MOSI, miso=SPI_MISO)

# Initialize display
lcd = st7789.ST7789(
    spi,
    240,
    240,
    reset=LCD_RST,
    dc=LCD_DC,
    cs=LCD_CS,
    backlight=LCD_BL
)

# Turn on backlight
lcd.backlight_on()

# Clear to black
lcd.fill(0)

print("ST7789 initialized")
```

### Drawing

```python
import st7789
from machine import Pin, SPI

# Initialize (see above)
lcd = st7789.ST7789(...)

# Clear with color
lcd.fill(0xFFFF)  # White
lcd.fill(0x0000)  # Black
lcd.fill(0xF800)  # Red
lcd.fill(0x07E0)  # Green
lcd.fill(0x001F)  # Blue

# Draw pixel
lcd.pixel(120, 120, 0xFFFF)

# Draw line
lcd.line(0, 0, 239, 239, 0xFFFF)

# Draw rectangle
lcd.rect(10, 10, 100, 50, 0xFFFF)

# Draw filled rectangle
lcd.fill_rect(10, 10, 100, 50, 0xF800)

# Draw circle
lcd.ellipse(120, 120, 50, 50, 0x07E0)

# Draw text
lcd.text("Hello XIAO!", 10, 10, 0xFFFF)
```

### Color Definitions

```python
# RGB565 color converter
def rgb565(r, g, b):
    return ((r & 0xF8) << 8) | ((g & 0xFC) << 3) | (b >> 3)

# Common colors
BLACK = 0x0000
WHITE = 0xFFFF
RED = 0xF800
GREEN = 0x07E0
BLUE = 0x001F
YELLOW = 0xFFE0
CYAN = 0x07FF
MAGENTA = 0xF81F
ORANGE = rgb565(255, 165, 0)
PURPLE = rgb565(128, 0, 128)

# Use colors
lcd.fill(YELLOW)
lcd.text("Colored Text", 10, 10, BLACK)
```

### Image Display

```python
import st7789

# Initialize display
lcd = st7789.ST7789(...)

# Display bitmap from RGB565 array
def display_bitmap(x, y, data, width, height):
    for row in range(height):
        for col in range(width):
            color = data[row * width + col]
            lcd.pixel(x + col, y + row, color)

# Example 10x10 red square
red_square = [RED] * 100
display_bitmap(10, 10, red_square, 10, 10)
```

### Touch Display (XIAO ESP32S3 Sense)

```python
from machine import Pin, SPI
import st7789
import xpt2046

# LCD SPI
spi_lcd = SPI(2, baudrate=40000000, ...)
lcd = st7789.ST7789(spi_lcd, 240, 240, ...)

# Touch SPI (separate)
spi_touch = SPI(1, baudrate=2000000, sck=Pin(...), mosi=Pin(...), miso=Pin(...))
touch_cs = Pin(...)
touch_irq = Pin(...)

touch = xpt2046.XPT2046(spi_touch, cs=touch_cs, irq=touch_irq)

# Read touch
while True:
    if touch.touched():
        x, y = touch.read()
        lcd.fill_circle(x, y, 5, RED)
        print(f"Touch: {x}, {y}")
```

---

## SH1106 OLED (I2C)

Larger 128x128 variant of SSD1306.

```python
from machine import Pin, I2C
import sh1106

i2c = I2C(0, sda=Pin(4), scl=Pin(5))
oled = sh1106.SH1106_I2C(128, 128, i2c)

oled.fill(0)
oled.text("SH1106 Display", 0, 0)
oled.show()
```

---

## PCD8544 Nokia 5110 (SPI)

Classic 84x48 LCD display.

```python
from machine import Pin, SPI
import pcd8544

# SPI setup
spi = SPI(0, baudrate=2000000, sck=Pin(6), mosi=Pin(7))
dc = Pin(4, Pin.OUT)
rst = Pin(5, Pin.OUT)
cs = Pin(3, Pin.OUT)

# Initialize
lcd = pcd8544.PCD8544(spi, cs, dc, rst)
lcd.contrast(60)  # 0-127

# Draw
lcd.fill(0)
lcd.text("Nokia 5110", 0, 0)
lcd.show()
```

---

## ILI9341 TFT (SPI)

320x240 color TFT display.

```python
from machine import Pin, SPI
import ili9341

spi = SPI(2, baudrate=40000000, sck=Pin(36), mosi=Pin(35), miso=Pin(37))
lcd = ili9341.ILI9341(spi, cs=Pin(34), dc=Pin(38), rst=Pin(33))

lcd.fill(0)
lcd.text("ILI9341 Display", 10, 10, 0xFFFF)
```

---

## Waveshare E-Paper

E-paper displays retain image without power.

```python
from machine import Pin, SPI
import epaper

# SPI setup
spi = SPI(1, baudrate=2000000, sck=Pin(6), mosi=Pin(7), miso=Pin(8))
cs = Pin(3)
dc = Pin(4)
rst = Pin(5)
busy = Pin(2)

# Initialize 2.13" display
epd = epaper.EPD_2IN13(spi, cs, dc, rst, busy)

# Clear
epd.init()
epd.Clear(0xFF)

# Draw text
epd.displayStringAt(10, 10, "E-Paper", epd.BLACK)
epd.displayStringAt(10, 30, "Display", epd.BLACK)

# Refresh
epd.display()

# Sleep
epd.sleep()
```

---

## MAX7219 8x8 LED Matrix (SPI)

Cascadable LED matrix displays.

```python
from machine import Pin, SPI
import max7219

spi = SPI(1, baudrate=10000000, sck=Pin(6), mosi=Pin(7))
cs = Pin(3, Pin.OUT)

# Initialize 4 cascaded displays
matrix = max7219.Max7219(spi, cs, 4)

# Scroll text
matrix.scroll_text("Hello XIAO! ", delay=100)

# Display pattern
matrix.fill(1)
matrix.pixel(0, 0, 0)
matrix.show()
```

---

## HD44780 LCD (Parallel)

16x2 character LCD with I2C backpack.

```python
from machine import Pin, I2C
import hd44768_i2c

i2c = I2C(0, sda=Pin(4), scl=Pin(5))
lcd = hd44768_i2c.HD44768(i2c, 0x27, 2, 16)

lcd.clear()
lcd.putstr("Hello XIAO!")
lcd.move_to(0, 1)
lcd.putstr("Line 2")
```

---

## TM1637 4-Digit Display

Clock-style 7-segment display.

```python
from machine import Pin
import tm1637

clk = Pin(4, Pin.OUT)
dio = Pin(5, Pin.OUT)

display = tm1637.TM1637(clk, dio)

# Show number
display.number(1234)

# Show temperature
display.temperature(25)

# Show time
display.show("1234", colon=True)

# Set brightness (0-7)
display.brightness(7)
```

---

## Display Selection Guide

| Display | Resolution | Interface | Colors | Power | Best For |
|---------|------------|-----------|--------|-------|----------|
| SSD1306 | 128x64 | I2C | Monochrome | Low | Text, icons |
| SH1106 | 128x128 | I2C | Monochrome | Low | Larger display |
| ST7789 | 240x240 | SPI | 65K colors | Medium | Graphics, photos |
| ILI9341 | 320x240 | SPI | 262K colors | High | Large display |
| E-Paper | Various | SPI | 2-3 colors | Ultra-low (static) | Labels, signs |
| PCD8544 | 84x48 | SPI | Monochrome | Low | Retro display |
| MAX7219 | 8x8 per module | SPI | Monochrome (LED) | High | Scrolling text |
| TM1637 | 4-digit | 2-wire | Red (7-seg) | Low | Clocks, counters |

---

## Troubleshooting

### Display not initializing

1. Check I2C/SPI wiring
2. Verify address (I2C): scan with `i2c.scan()`
3. Check pull-up resistors on I2C (4.7k typical)
4. Try lower SPI frequency

### Flickering display

1. Add capacitor across power
2. Use shorter cables
3. Check power supply current
4. Try lower SPI frequency

### Wrong colors

1. Verify RGB565 format
2. Check byte order (MSB/LSB)
3. Test with known colors

### Slow refresh

1. Increase SPI frequency
2. Use hardware SPI
3. Optimize drawing code
4. Use partial updates

### Display corrupted

1. Reset display
2. Check for noise on SPI
3. Add decoupling capacitors
4. Verify timing

---

## Best Practices

1. **Use hardware SPI** for faster refresh rates
2. **Add capacitors** near display power pins
3. **Use short cables** for SPI signals
4. **Set appropriate frequency** - I2C: 400kHz, SPI: 10-40MHz
5. **Double buffer** for smooth animations
6. **Sleep display** when not in use to save power
7. **Limit text updates** - clear only changed areas
8. **Use monochrome** for battery-powered devices

---

## Platform Support

| Library | ESP32 | RP2040 | nRF52 |
|---------|-------|--------|-------|
| ssd1306 | ✅ | ✅ | ✅ |
| st7789 | ✅ | ✅ | ✅ |
| sh1106 | ✅ | ✅ | ✅ |
| ili9341 | ✅ | ✅ | ❌ |
| epaper | ✅ | ✅ | ❌ |
| max7219 | ✅ | ✅ | ✅ |
| tm1637 | ✅ | ✅ | ✅ |

---

## Related Resources

- `common.md` - Common library reference
- `filesystem.md` - File system operations
