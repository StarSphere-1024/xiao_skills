# ESPHome Displays for XIAO

## SSD1306 OLED (I2C)

Common 128x64 or 128x32 OLED display.

```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    rotation: 0°
    update_interval: 1s
    lambda: |-
      it.print(0, 0, id(font), "Hello XIAO!");
      it.printf(0, 16, id(font), "T: %.1f°C", id(bme280_temperature).state);
      it.printf(0, 32, id(font), "H: %.1f%%", id(bme280_humidity).state);

# Fonts
font:
  - file: "fonts/Roboto-Regular.ttf"
    id: font
    size: 16
```

## SSD1306 OLED (SPI)

```yaml
display:
  - platform: ssd1306_spi
    model: "SSD1306 128x64"
    spi_id: bus_spi
    cs_pin: GPIO2
    dc_pin: GPIO3
    reset_pin: GPIO4
    rotation: 0°
    lambda: |-
      it.print(0, 0, id(font), "Hello SPI");
```

## SH1106 OLED

```yaml
display:
  - platform: ssd1306_i2c
    model: "SH1106_128x64"
    i2c_id: bus_i2c
    address: 0x3C
    lambda: |-
      it.print(0, 0, id(font), "SH1106 Display");
```

## ST7789 TFT (SPI)

Color TFT display.

```yaml
display:
  - platform: st7789v
    model: "TFT_2.4"
    spi_id: bus_spi
    cs_pin: GPIO2
    dc_pin: GPIO3
    reset_pin: GPIO4
    rotation: 0°
    update_interval: 1s
    lambda: |-
      it.fill(0xFFFF);  // White background
      it.print(0, 0, id(font_color), "Color TFT");
      it.printf(0, 20, id(font_color), "T: %.1f°C", id(temp).state);

# Color fonts require different format
```

## ILI9341 TFT (SPI)

```yaml
display:
  - platform: ili9341
    model: "TFT 2.4"
    spi_id: bus_spi
    cs_pin: GPIO2
    dc_pin: GPIO3
    reset_pin: GPIO4
    rotation: 90°
    lambda: |-
      it.fill(COLOR_BLACK);
      it.print(0, 0, id(font_color), "ILI9341");
```

## PCD8544 (Nokia 5110)

```yaml
display:
  - platform: pcd8544
    spi_id: bus_spi
    cs_pin: GPIO2
    dc_pin: GPIO3
    reset_pin: GPIO4
    contrast: 60
    lambda: |-
      it.print(0, 0, id(font), "Nokia 5110");
```

## HD44780 LCD (I2C)

Character LCD with I2C backpack.

```yaml
display:
  - platform: lcd_pcf8574
    i2c_id: bus_i2c
    address: 0x27
    dimensions: 16x2
    lambda: |-
      it.print("Line 1");
      it.print(0, 1);
      it.print("Line 2");
```

## MAX7219 7-Segment

```yaml
display:
  - platform: max7219
    cs_pin: GPIO2
    num_chips: 4
    lambda: |-
      it.printf(0, id(font_7seg), "%4.1f", id(temp).state);
```

## Drawing Primitives

```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    lambda: |-
      // Print text
      it.print(0, 0, id(font), "Hello");

      // Formatted text
      it.printf(0, 16, id(font), "T: %.1f°C", id(temp).state);

      // Draw line
      it.line(0, 32, 128, 32);

      // Draw rectangle
      it.rectangle(0, 40, 128, 60);

      // Draw filled rectangle
      it.filled_rectangle(10, 50, 50, 60);

      // Draw circle
      it.circle(64, 32, 20);

      // Draw image
      it.image(0, 0, id(my_image));
```

## Progress Bar

```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    lambda: |-
      // Progress bar
      int width = 100;
      int height = 10;
      int x = 10;
      int y = 30;
      float progress = id(humidity).state / 100.0;

      // Outline
      it.rectangle(x, y, x + width, y + height);

      // Fill based on progress
      it.filled_rectangle(x + 1, y + 1, x + (width * progress) - 1, y + height - 1);
```

## Graphs

```yaml
# History component for graphs
sensor:
  - platform: homeassistant
    id: ha_temperature
    entity_id: sensor.temperature
    on_value:
      then:
        - logger.log: "Temperature updated"

# Display graph
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    lambda: |-
      // Simple line graph
      auto temp = id(ha_temperature).state;
      static float points[128];
      static int index = 0;

      points[index] = temp;
      index = (index + 1) % 128;

      for (int i = 0; i < 128; i++) {
        int idx = (index + i) % 128;
        int y = 63 - (points[idx] * 2);
        if (i == 0) {
          it.draw_pixel_at(i, y);
        } else {
          int prev_idx = (index + i - 1) % 128;
          int prev_y = 63 - (points[prev_idx] * 2);
          it.line(i - 1, prev_y, i, y);
        }
      }
```

## Animations

```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    lambda: |-
      static int frame = 0;
      frame++;

      // Animated text
      it.printf((frame % 128), 32, id(font), "Scrolling");

      // Blinking
      if (frame % 60 < 30) {
        it.print(0, 0, id(font), "ON");
      } else {
        it.print(0, 0, id(font), "OFF");
      }
```

## Multiple Pages

```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    pages:
      - id: page1
        lambda: |-
          it.print(0, 0, id(font), "Page 1");
      - id: page2
        lambda: |-
          it.print(0, 0, id(font), "Page 2");

# Cycle pages
interval:
  - interval: 5s
    then:
      - display.page.show_next: display1
```

## Touch Display (Nextion)

```yaml
display:
  - platform: nextion
    tfa: 0
    update_interval: 1s
    lambda: |-
      it.set_component_text("t0", "Hello");

binary_sensor:
  - platform: nextion
    page: 0
    component_id: 1
    name: "Button 1"
```

## Fonts

```yaml
# Load custom font
font:
  - file: "fonts/Roboto-Regular.ttf"
    id: font_16
    size: 16

  - file: "fonts/Roboto-Bold.ttf"
    id: font_24
    size: 24

# Use in display
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    lambda: |-
      it.print(0, 0, id(font_16), "Small");
      it.print(0, 20, id(font_24), "Large");
```

## Images

```yaml
# Define image
image:
  - file: "images/logo.bmp"
    id: my_logo
    resize: 50x50
    type: RGB24

# Display image
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    lambda: |-
      it.image(0, 0, id(my_logo));
```

## Complete Weather Station Display

```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    update_interval: 5s
    lambda: |-
      // Title
      it.printf(0, 0, id(font_title), "XIAO Weather");

      // Temperature
      it.printf(0, 16, id(font_data), "T: %.1f°C", id(bme280_temperature).state);

      // Humidity
      it.printf(0, 32, id(font_data), "H: %.1f%%", id(bme280_humidity).state);

      // Pressure
      it.printf(0, 48, id(font_data), "P: %.0fhPa", id(bme280_pressure).state);

      // WiFi signal
      it.printf(0, 60, id(font_small), "WiFi: %ddBm", id(wifi_signal).state);

font:
  - file: "fonts/Roboto-Bold.ttf"
    id: font_title
    size: 14

  - file: "fonts/Roboto-Regular.ttf"
    id: font_data
    size: 12

  - file: "fonts/Roboto-Regular.ttf"
    id: font_small
    size: 8
```

## Best Practices

1. **Optimize update intervals** - Don't update too frequently
2. **Use appropriate font sizes** - Fit content to display
3. **Test contrast** - Ensure readability
4. **Cache static content** - Redraw only what changes
5. **Use pages** - For multiple screens of information

## Troubleshooting

### Display Not Showing

1. Check I2C address (0x3C or 0x3D)
2. Verify wiring (SDA/SCL)
3. Enable I2C scan
4. Check display is powered (5V or 3.3V)

### Garbled Display

1. Check I2C frequency
2. Verify correct model
3. Try different I2C pins
4. Check pull-up resistors

### Partial Display

1. Check model size (128x32 vs 128x64)
2. Verify rotation setting
3. Check font size

### SPI Display Issues

1. Verify all SPI connections
2. Check CS/DC/RESET pins
3. Try lower SPI frequency
