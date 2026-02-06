# ePaper Driver Board V2 for XIAO

## Overview

The ePaper Driver Board V2 is a smart expansion board for driving ePaper/eInk displays with XIAO. It features a 24-pin FPC connector supporting 7 different ePaper sizes (1.54" to 7.5"), built-in battery charging IC, JST battery connector with power switch, and extension IO port for additional sensors. Ideal for WiFi-enabled digital photo frames, smart home dashboards, and low-power display applications.

## Hardware

### Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | All ePaper sizes supported |
| ESP32C5 | ✅ | All ePaper sizes supported |
| ESP32C6 | ✅ | All ePaper sizes supported |
| ESP32S3 | ✅ | All ePaper sizes supported (recommended) |
| nRF52840 | ✅ | All ePaper sizes supported |
| RP2040 | ✅ | All ePaper sizes supported |
| RP2350 | ✅ | All ePaper sizes supported |
| SAMD21 | ⚠️ | Limited - small displays only (RAM constraints) |
| RA4M1 | ✅ | All ePaper sizes supported |
| MG24 | ✅ | All ePaper sizes supported |
| nRF54L15 | ✅ | All ePaper sizes supported |

**SAMD21 Limitations**: RAM overflow for 4.26", 5.65", 7.5" displays

### Features

- **24-pin FPC Connector**: Universal interface for multiple ePaper sizes
- **Battery Management**: Built-in charging IC with JST 2-pin connector
- **Power Switch**: Control battery power for energy saving
- **Extension IO Port**: Connect additional sensors or controllers
- **XIAO Compatible**: Works with all XIAO boards (pre-soldered version)

### Specifications

| Item | Value |
|------|-------|
| Operating Voltage | 3.3V - 5V |
| Display Interface | SPI (via FPC) |
| Supported Sizes | 1.54", 2.13", 2.9", 4.2", 4.26", 5.65", 5.83", 7.5" |
| Battery Connector | JST 2-pin |
| Charging IC | Built-in |
| Extension IO | SPI + GPIO pins |

### Supported ePaper Displays

| Display Size | Resolution | Type | ESP32C3 | nRF52840 | RP2040 | SAMD21 |
|--------------|------------|------|---------|----------|---------|---------|
| 1.54" | 200×200 | Monochrome | ✅ | ✅ | ✅ | ✅ |
| 2.13" | 212×104 | Flexible Monochrome | ✅ | ✅ | ✅ | ✅ |
| 2.13" | 212×104 | Quadruple | ✅ | ✅ | ✅ | ✅ |
| 2.9" | 296×128 | Monochrome | ✅ | ✅ | ✅ | ✅ |
| 2.9" | 296×128 | Quadruple | ✅ | ✅ | ✅ | ✅ |
| 4.2" | 400×300 | Monochrome | ✅ | ✅ | ✅ | ✅ |
| 4.26" | 800×480 | Monochrome | ✅ | ✅ | ✅ | ❌ RAM overflow |
| 5.65" | 600×480 | 7-Color | ✅ | ✅ | ✅ | ❌ Flash overflow |
| 5.83" | 648×480 | Monochrome | ✅ | ✅ | ✅ | ✅ |
| 7.5" | 800×480 | Monochrome | ✅ | ✅ | ✅ | ❌ RAM overflow |
| 7.5" | 800×480 | Tri-Color | ✅ | ✅ | ✅ | ❌ RAM overflow |

### Pin Connections

| ePaper Pin | XIAO Pin | Function |
|------------|----------|----------|
| RST | D0 | Reset |
| CS | D1 | Chip Select |
| DC | D3 | Data/Command |
| BUSY | D2 | Busy signal |
| SCK | D8 | SPI Clock |
| MOSI | D10 | SPI MOSI |
| 3V3 | 3V3 | Power |
| GND | GND | Ground |

**IMPORTANT:** These pins are used by the ePaper and not available for other uses when ePaper is connected:
- **D0, D1, D2, D3, D8, D10** - Reserved for ePaper SPI

Available free pins: D4, D5, D6, D7, D9, D11-D18 (varies by XIAO board)

### Board Layout

```
+------------------------------------------+
|                                          |
|  [24-pin FPC Connector]                  |
|                                          |
|  [XIAO Socket]                           |
|                                          |
|  [Power Switch]                          |
|                                          |
|  [JST BAT]                               |
|                                          |
|  [Extension IO Port]                     |
+------------------------------------------+
```

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/epaper-v2.md`
  - Seeed GFX library setup
  - Display initialization
  - Text and graphics drawing
  - Image conversion tools

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## Power Options

1. **USB Power**: 5V via Type-C cable for development
2. **Battery Power**: 3.7V Li-Po battery via JST connector for portable use
3. **Power Switch**: Turn OFF to save battery when not updating display

## ePaper Display Basics

### Advantages of ePaper

- **Ultra Low Power**: Only uses power when updating display
- **Bistable**: Image remains visible without power
- **Sunlight Readable**: Excellent contrast in bright light
- **Wide Viewing Angle**: 170° viewing angle
- **Paper-like Appearance**: Easy on the eyes

### Limitations

- **Slow Refresh**: 1-3 seconds for full refresh
- **Limited Color**: Mostly monochrome, some tri-color/7-color options
- **Refresh Cycle**: ePaper has limited refresh cycles (typically >100,000)
- **Ghosting**: May show faint remnants of previous images
- **Temperature**: Performance affected by extreme temperatures

## Usage Notes

1. **Display Not Included**: ePaper display must be purchased separately
2. **FPC Connector**: 24-pin 0.5mm pitch FPC (standard for ePaper)
3. **Orientation**: Match FPC connector polarity carefully
4. **First Time Setup**: Requires Seeed GFX library configuration
5. **Flickering**: Normal for some displays during refresh cycles
6. **Refresh Limits**: Avoid excessive full refreshes to extend display life

## Applications

- **Smart Home Dashboard**: Display weather, calendar, notifications
- **Digital Photo Frame**: WiFi-enabled photo display
- **Energy Monitoring**: Show power consumption data
- **Security Alerts**: Display security system notifications
- **Smart Thermostat**: Show temperature/humidity with controls
- **Price Tags**: Electronic shelf labels
- **Signage**: Information displays with minimal power consumption
- **E-Readers**: Book display with annotations

## Image Conversion Tools

### Recommended: Online Tool (jlamch.net)

1. Visit https://jlamch.net/MXChipWelcome/
2. Select your image (match display resolution)
3. Configure settings:
   - Canvas Size: Match your display (e.g., 800×480 for 7.5")
   - Background Color: White (usually)
   - Invert Image: As needed
   - Brightness Threshold: 0-255 (adjust for contrast)
4. Preview and copy generated code
5. Paste into Arduino sketch

### Alternative: Image2LCD Software

1. Create image at exact display resolution
2. Open Image2LCD
3. Configure for your display (see table below)
4. Save as C array (.h file)

**Image2LCD Settings by Display:**

| Display | Bit Pixel | Max Size | Reverse | Scan Mode |
|---------|-----------|----------|---------|-----------|
| 1.54" (200×200) | Monochrome | 200×200 | ✅ | Mirror |
| 2.13" (212×104) | Monochrome | 104×212 | ✅ | Normal |
| 2.13" Quad (212×104) | 4 Gray | 104×212 | - | Normal |
| 2.9" (296×128) | Monochrome | 128×296 | ✅ | Normal |
| 2.9" Quad (296×128) | 4 Gray | 128×296 | - | Normal |
| 4.2" (400×300) | Monochrome | 400×300 | ✅ | Mirror |
| 4.26" (800×480) | Monochrome | 800×480 | - | Mirror |
| 5.65" (600×480) | 256 Color | 600×448 | - | Normal |
| 5.83" (648×480) | Monochrome | 600×480 | ✅ | Mirror |
| 7.5" (800×480) | Monochrome | 800×480 | ✅ | Mirror |

## Troubleshooting

### Display Shows Nothing

**Symptom**: Blank screen after power-on

**Possible Causes:**
1. Wrong configuration - Check BOARD_SCREEN_COMBO value
2. Loose FPC connection - Reseat ePaper display
3. Insufficient power - Use external power for large displays
4. Wrong pin definitions - Verify D0, D1, D2, D3, D8, D10

**Solution**: Check configuration, reseat FPC, verify pins

### Ghosting/Afterimages

**Symptom**: Faint remnants of previous images visible

**Possible Causes:**
1. Insufficient refresh cycles
2. Partial refresh only
3. Extreme temperature

**Solution**: Run full refresh cycle, avoid temperature extremes

### Flickering During Refresh

**Symptom**: Noticeable flicker on some displays

**Possible Causes:**
1. Normal for 1.54" and 2.9" displays (driver chip characteristic)
2. Partial refresh on unsupported display

**Solution**: Use full refresh, accept as normal for these displays
**Note**: 5.83" and 7.5" displays do NOT flicker (different driver chips)

### RAM/Flash Overflow Errors

**Symptom**: Compilation error about insufficient memory

**Possible Causes:**
1. Display too large for XIAO board (see compatibility table)
2. Too large images in code

**Solution**: Use smaller display, reduce image size, use XIAO with more RAM

## Recommended XIAO Boards

- **ESP32S3**: Best overall - plenty of RAM, WiFi/BLE, fast
- **ESP32C3/C6**: Good choice - sufficient RAM, WiFi/BLE
- **RP2040**: Excellent - good RAM, PIO support, low cost
- **nRF52840**: Good for BLE applications with ePaper display

Avoid SAMD21 for displays larger than 2.9" due to RAM constraints.
