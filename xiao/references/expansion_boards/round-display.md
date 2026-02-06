# Round Display for XIAO

## Overview

The Round Display for XIAO is a 1.28" round capacitive touchscreen expansion board (240×240 pixels) with built-in RTC (PCF8563), TF card slot (up to 32GB), JST 1.25 battery connector with charging (~485mA), and user button. The 39mm circular form factor makes it ideal for wearable devices, smart watches, portable instruments, and compact interactive displays.

## Hardware

### Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | Recommended (adequate RAM) |
| ESP32C5 | ✅ | Full support |
| ESP32C6 | ✅ | Full support |
| ESP32S3 | ✅ | Recommended (more RAM, camera option) |
| nRF52840 | ✅ | Recommended (BLE, good RAM) |
| RP2040 | ✅ | Full support |
| RP2350 | ✅ | Full support |
| SAMD21 | ⚠️ | Limited RAM for complex graphics |
| RA4M1 | ✅ | Full support |
| MG24 | ✅ | Full support |
| nRF54L15 | ✅ | Full support |

### Features

- **1.28" Round Touchscreen**: 240×240 pixels, capacitive touch
- **Built-in RTC**: PCF8563 with CR1220 battery backup
- **TF Card Slot**: Up to 32GB expandable memory
- **Battery Charging**: JST 1.25mm connector (~485mA)
- **User Button**: D1 GPIO input
- **Power Switch**: Battery power control
- **All Pins Broken Out**: Full access to XIAO GPIO

### Specifications

| Item | Value |
|------|-------|
| Display | 1.28" round TFT (240×240) |
| Touchscreen | Capacitive touch |
| RTC | PCF8563 (±1.5 sec/day at 25°C) |
| RTC Battery | CR1220 (included) |
| SD Card | Up to 32GB (FAT/exFAT) |
| Charging Current | ~485mA |
| Dimensions | 39mm diameter (circular) |

### Pin Connections

| Display Feature | XIAO Pin | Function |
|----------------|-----------|----------|
| Display SPI | D0/D1/D2/D3/D8/D10 | SPI interface |
| Touch I2C | D4/D5 | I2C (shared with RTC) |
| RTC I2C | D4/D5 | I2C (shared with touch) |
| SD Card CS | D2 | SPI CS |
| User Button | D1 | GPIO input |

**Important:** These pins are used by the round display and not available for other uses:
- **D0, D1, D2, D3, D8, D10** - Display SPI
- **D4, D5** - Touch and RTC (I2C shared)

Available free pins: D6, D7, D9, D11-D18 (varies by XIAO board)

### Board Layout

```
+------------------------------------------+
|              [1.28" Display]             |
|  [Touch Screen]  [RTC Battery]           |
|                                          |
|  [User Button]  [Power Switch]           |
|                                          |
|  [TF Card Slot]                          |
|                                          |
|  [XIAO Socket]                           |
|  [Battery Connector]                      |
+------------------------------------------+
```

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/round-display.md`
  - LVGL graphics library setup
  - Touch input handling
  - RTC time management
  - SD card data logging
  - Display examples (clock, widgets, UI)

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## Display Specifications

- **Resolution**: 240×240 pixels
- **Shape**: Circular (round)
- **Interface**: SPI
- **Touch**: Capacitive (I2C)
- **Color**: 65K colors
- **Backlight**: Adjustable

## Recommended XIAO Boards

**For Best Performance:**
- **ESP32S3**: More RAM for complex graphics, WiFi/BLE, optional camera
- **nRF52840**: BLE applications, good RAM, lower power
- **ESP32C3**: Cost-effective option, adequate RAM

**For Battery Operation:**
- **nRF52840**: Best for BLE wearables (low power)
- **ESP32C3/C6**: Good for WiFi-connected devices

## Usage Notes

1. **Library**: LVGL (Light and Versatile Graphics Library) recommended
2. **Memory**: Complex UI needs more RAM - use ESP32S3 or nRF52840
3. **Power**: Battery charging when USB connected and switch ON
4. **SD Card**: FAT/exFAT format, up to 32GB
5. **Touch**: I2C interface shares D4/D5 with RTC

## Applications

- **Smart Watches**: Time display, notifications, fitness tracking
- **Wearable Devices**: Compact interactive displays
- **Portable Instruments**: Measurement tools with visual feedback
- **Smart Mirrors**: Information displays with touch control
- **IoT Dashboards**: Sensor data visualization
- **Game Controllers**: Circular button layouts
- **Instrument Clusters**: Automotive/marine displays

## Troubleshooting

### Display Shows Garbage

**Symptom**: Screen shows random pixels

**Possible Causes:**
1. Wrong SPI pins - Check D0/D1/D2/D3/D8/D10
2. Wrong initialization - Verify LVGL configuration
3. Insufficient RAM - Try simpler graphics or different XIAO

**Solution**: Verify pin mapping, check LVGL setup, use XIAO with more RAM

### Touch Not Working

**Symptom**: Touch input not detected

**Possible Causes:**
1. I2C conflict - Check D4/D5 not used elsewhere
2. Wrong touch driver - Verify touch controller model
3. Calibration needed - Run touch calibration

**Solution**: Check I2C connections, verify driver, calibrate touchscreen

### SD Card Not Detected

**Symptom**: Cannot access SD card

**Possible Causes:**
1. Wrong format - Use FAT or exFAT
2. CS pin mismatch - Should be D2
3. Card too large - Use ≤32GB

**Solution**: Reformat card as FAT/exFAT, verify CS pin

### Battery Not Charging

**Symptom**: Charging LED not lit

**Possible Causes:**
1. Power switch OFF - Turn switch to ON
2. No battery - Verify JST connector seated
3. Dead battery - Replace battery

**Solution**: Check power switch, verify battery connection

## Pin Summary

| Pin | Function | Notes |
|-----|----------|-------|
| D0 | SPI MOSI | Display data |
| D1 | SPI MISO / Button | Display data / GPIO |
| D2 | SD Card CS | SPI chip select |
| D3 | SPI DC | Display data/command |
| D4 | I2C SDA | Touch + RTC |
| D5 | I2C SCL | Touch + RTC |
| D8 | SPI SCK | Display clock |
| D10 | SPI CS | Display chip select |

**Free pins for user:** D6, D7, D9, D11-D18 (varies by board)
