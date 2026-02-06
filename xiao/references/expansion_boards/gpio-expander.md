# GPIO Expander for XIAO

## Overview

The GPIO Expander for XIAO adds 16 additional digital I/O pins using the MCP23017 I2C port expander. Ideal for projects requiring more GPIO pins than available on XIAO, such as controlling multiple relays, reading many sensors, or driving LED matrices. Each pin can be configured as input or output with optional pull-up resistors and interrupt capability.

## Hardware

### Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | Full support |
| ESP32C5 | ✅ | Full support |
| ESP32C6 | ✅ | Full support |
| ESP32S3 | ✅ | Full support |
| nRF52840 | ✅ | Full support |
| RP2040 | ✅ | Full support |
| RP2350 | ✅ | Full support |
| SAMD21 | ✅ | Full support |
| RA4M1 | ✅ | Full support |
| MG24 | ✅ | Full support |
| nRF54L15 | ✅ | Full support |

### Features

- **MCP23017 Chip**: 16-bit I/O expander
- **I2C Interface**: Communication via D4/D5
- **Configurable Pins**: Each pin as input or output
- **Interrupt Output**: Optional interrupt on pin change
- **Pull-up Resistors**: Configurable per pin
- **Address Selection**: Multiple expanders on same bus

### Specifications

| Item | Value |
|------|-------|
| Chip | MCP23017 |
| I/O Pins | 16 (8 per port, GPA/GPB) |
| Interface | I2C |
| I2C Address | 0x20-0x27 (configurable) |
| Supply Voltage | 3.3V (XIAO) |
| Max Current per Pin | 25mA |
| Max Total Current | 150mA |

### Pin Connections

| Function | XIAO Pin | Notes |
|----------|-----------|-------|
| I2C SDA | D4 | Default Grove I2C |
| I2C SCL | D5 | Default Grove I2C |
| Interrupt | Optional | Any available GPIO pin |

**MCP23017 Pin Mapping:**
```
Port A: GPA0-GPA7 (bits 0-7)
Port B: GPB0-GPB7 (bits 8-15)
```

**Important:** D4 and D5 are used for I2C communication. The 16 expanded GPIO pins are accessed via I2C commands.

### Board Layout

```
+------------------------------------------+
|  [GPA0] [GPA1] [GPA2] [GPA3]           |
|  [GPA4] [GPA5] [GPA6] [GPA7]           |
|  [GPB0] [GPB1] [GPB2] [GPB3]           |
|  [GPB4] [GPB5] [GPB6] [GPB7]           |
|                                          |
|  [XIAO Socket]                           |
|  [Address Jumpers]                       |
+------------------------------------------+
```

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/gpio-expander.md`
  - Adafruit MCP23017 library setup
  - Pin direction configuration
  - Input/output examples
  - Interrupt handling
  - Reading/writing multiple pins

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## I2C Address Configuration

The MCP23017 supports 8 different I2C addresses:
- Default: 0x20
- Configurable: 0x20-0x27 (via A0/A1/A2 address pins)

**Multiple Expanders:**
Up to 8 MCP23017 chips on same I2C bus for 128 additional GPIO pins.

## Usage Notes

1. **I2C Shared**: Uses D4/D5, same as Grove I2C
2. **Address Conflict**: Ensure no I2C address conflicts
3. **Current Limits**: 25mA per pin, 150mA total
4. **Speed**: I2C communication slower than native GPIO
5. **Interrupt**: Optional interrupt pin for change detection

## Applications

- **Relay Control**: Control multiple 12V/24V relays
- **LED Matrix**: Drive many LEDs without native GPIO
- **Sensor Arrays**: Read multiple sensors
- **Button Arrays**: Many user input buttons
- **Level Shifters**: Interface with 5V logic
- **Keypads**: Matrix keyboard scanning

## Troubleshooting

### Device Not Detected

**Symptom**: Cannot find MCP23017 on I2C bus

**Possible Causes:**
1. Wrong address - Check address jumpers
2. I2C conflict - Verify no other device at same address
3. Loose connection - Check XIAO seating

**Solution**: Run I2C scanner, check address configuration

### Incorrect Pin States

**Symptom**: Pins read wrong values

**Possible Causes:**
1. Not initialized - Configure pin direction first
2. Wrong port - Check GPA vs GPB
3. Interrupt mode - Clear interrupt flag

**Solution**: Initialize all pins, check port selection

### Current Limit Exceeded

**Symptom**: Unstable operation, resets

**Possible Causes:**
1. Too much current per pin - Limit to 25mA
2. Total current too high - Limit to 150mA

**Solution**: Reduce load on pins, use external drivers

## Comparison: Native GPIO vs Expander

| Feature | Native GPIO | MCP23017 Expander |
|---------|-------------|-------------------|
| Speed | Fast | Slower (I2C) |
| Current | Higher per pin | Limited (25mA) |
| Complexity | Simple | Requires I2C |
| Number | Limited (14-18) | Expandable (+16 per chip) |
| Interrupt | Hardware | Optional |
| Cost | Included | Additional |

**Choose Expander if:** Need more pins, can accept slower speed
**Choose Native if:** Need fast I/O, PWM, or high current

## Required Libraries (Arduino)

- **Adafruit MCP23017**: Recommended library
  - Available in Arduino Library Manager
  - Easy to use
  - Supports all features

**Alternative:**
- **MCP23017**: Wire library based
- **MCP23017_IO**: Non-blocking operations
