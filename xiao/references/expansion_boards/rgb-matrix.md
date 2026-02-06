# RGB Matrix Board for XIAO

## Overview

The RGB Matrix Board for XIAO is designed to drive RGB LED matrix displays. It provides the necessary current driving and signal conditioning to control RGB LED panels for displaying graphics, animations, text, and video content. Ideal for signage, message boards, information displays, and creative lighting projects.

## Hardware

### Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | Recommended (WiFi, adequate RAM) |
| ESP32C5 | ✅ | Full support |
| ESP32C6 | ✅ | Full support |
| ESP32S3 | ✅ | Recommended (more RAM for larger matrices) |
| nRF52840 | ✅ | Full support (BLE applications) |
| RP2040 | ✅ | Full support (good PIO support) |
| RP2350 | ✅ | Full support |
| SAMD21 | ⚠️ | Limited RAM for larger matrices |
| RA4M1 | ✅ | Full support |
| MG24 | ✅ | Full support |
| nRF54L15 | ✅ | Full support |

### Features

- **RGB Matrix Support**: Drives common RGB LED panels
- **Current Control**: Proper current limiting for LEDs
- **Signal Conditioning**: Clean data signals for matrix
- **Power Management**: Handles matrix power requirements
- **Connector**: Standard RGB matrix interface

### Specifications

| Item | Value |
|------|-------|
| Supported Panels | Various RGB matrix sizes |
| Interface | SPI or specialized protocol |
| Power | External power supply required |
| Current | Depends on panel size (can be 2A+) |

### Pin Connections

Pin connections vary by specific RGB matrix board design. Common configurations use:

| Function | XIAO Pin | Notes |
|----------|-----------|-------|
| Data | Various | Usually SPI or dedicated pins |
| Clock | D8 (SPI SCK) | Often used |
| Latch | D9 or D10 | Panel specific |
| Enable | D3 or similar | Panel specific |

**Important:** RGB matrices require significant current (2A+ for larger panels). Always use external power supply - do NOT power from XIAO's 3.3V regulator.

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/rgb-matrix.md`
  - PxMatrix or FastLED library setup
  - Basic display examples
  - Scrolling text
  - Graphics and animations
  - Color correction

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

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

## Common RGB Matrix Libraries

**PxMatrix:**
- Good for 32×32 and 64×64 panels
- Supports double buffering
- Text and graphics

**FastLED:**
- More control over individual LEDs
- Advanced effects
- Works with various LED types

**SmartMatrix:**
- For larger panels
- More features

## Usage Notes

1. **External Power**: Required for matrix (2A+)
2. **Ground Connection**: Connect XIAO GND to external supply GND
3. **Pin Mapping**: Verify panel-specific connections
4. **Refresh Rate**: Adjust for smooth animation
5. **Brightness**: High brightness = high current

## Applications

- **Information Displays**: Scrolling text, announcements
- **Signage**: Store signs, directional signs
- **Art Installations**: Animated graphics, patterns
- **Scoreboards**: Sports, game displays
- **Message Boards**: Public information
- **Party Displays**: Music visualizer, effects

## Troubleshooting

### Dim Display

**Symptom**: Matrix too dim

**Possible Causes:**
1. Insufficient power - Check external supply capacity
2. Wrong voltage - Use 5V for most panels
3. Low brightness setting - Increase in code

**Solution:** Use adequate 5V power supply (2A+), check connections

### Flickering

**Symptom:** Display flickers randomly

**Possible Causes:**
1. Refresh rate too low - Increase refresh
2. Insufficient current - Check power supply
3. Poor connections - Verify all wiring

**Solution:** Increase refresh rate, ensure adequate power, check wiring

### Wrong Colors

**Symptom:** Colors don't match expected

**Possible Causes:**
1. RGB order wrong - Adjust in code (RGB vs GRB vs BGR)
2. Color correction needed - Calibrate

**Solution:** Change color order in library configuration

### No Display

**Symptom:** Matrix remains dark

**Possible Causes:**
1. No power - Check external power supply
2. Wrong pins - Verify pin connections
3. Library issue - Check correct library and settings

**Solution:** Verify external power, check wiring, verify library setup

## Panel Types

**Common Configurations:**
- 32×32 RGB (most common)
- 64×64 RGB
- 64×32 RGB
- Various chainable configurations

**Chainable Panels:**
Multiple panels can be chained for larger displays. Ensure adequate power supply for total current.

## Comparison: RGB Matrix vs LED Strip

| Feature | RGB Matrix | LED Strip |
|---------|------------|-----------|
| Resolution | Fixed pixels | Flexible length |
| Form Factor | Rigid panel | Flexible strip |
| Applications | Signs, displays | Accent lighting |
| Power | Very high (2A+) | Moderate |
| Complexity | Higher | Lower |

**Choose RGB Matrix if:** Need pixelated display, graphics, text
**Choose LED Strip if:** Need accent lighting, flexible placement
