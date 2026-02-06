# Expansion Board Base for XIAO

## Overview

The Expansion Board Base is a comprehensive development board for XIAO that adds multiple peripherals in a compact form factor (half the size of Raspberry Pi 4). It features an OLED display, RTC with battery backup, MicroSD card slot, passive buzzer, user button, Grove connectors, and battery management circuitry. Ideal for sensor hubs, data loggers, mini displays, and rapid prototyping.

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
| MG24 | ❌ | Different SWD pin layout |
| nRF54L15 | ❌ | Different SWD pin layout |

### Features

- **0.96" OLED Display** (SSD1306, 128x64) - Visual data output without PC
- **High-Precision RTC** (PCF8563) - Real-time clock with CR1220 battery backup
- **MicroSD Card Slot** - Expandable memory (up to 32GB)
- **Passive Buzzer** - PWM-based audio output
- **User Button** - Configurable input
- **RESET Button** - Easy reset without jumper wires
- **SWD Debug Interface** - Male header for debugger connection
- **Grove Connectors** - 2x I2C, 1x UART, 1x A0/D0
- **Battery Management** - JST 2.0mm connector with charging circuit (460mA max)
- **5V Servo Connector** - For 5V servos and sensors

### Specifications

| Item | Value |
|------|-------|
| Operating Voltage | 5V (USB) / 3.7V (Li-Po battery) |
| Charging Current | 460mA (max) |
| RTC Precision | ±1.5 seconds/day at 25°C |
| RTC Battery | CR1220 (included) |
| Display | 0.96" OLED (SSD1306, 128x64) |
| SD Card Support | Up to 32GB (FAT/exFAT) |
| Grove Interfaces | I2C×2, UART×1, Analog/Digital×1 |
| Dimensions | Approximately 50×39mm (before break-off) |

### Pin Connections

| Expansion Feature | XIAO Pin | Function | Notes |
|-------------------|----------|----------|-------|
| OLED SDA | D4 | I2C SDA | Default Grove I2C |
| OLED SCL | D5 | I2C SCL | Default Grove I2C |
| RTC SDA | D4 | I2C SDA | Shared with OLED |
| RTC SCL | D5 | I2C SCL | Shared with OLED |
| SD Card CS | D2 | SPI CS | - |
| SD Card SCK | D8 | SPI SCK | - |
| SD Card MISO | D9 | SPI MISO | - |
| SD Card MOSI | D10 | SPI MOSI | - |
| Buzzer | A3 (D3) | PWM output | Can be cut to disable |
| User Button | D1 | GPIO input | - |
| Grove UART RX | D6 | UART RX | - |
| Grove UART TX | D7 | UART TX | - |
| Grove I2C | D4/D5 | I2C | Same as OLED/RTC |
| Grove A0/D0 | D0 | GPIO/Analog | - |
| Servo 5V | - | 5V output | Male header |

**IMPORTANT:** The following pins are used by the expansion board and not available for other uses:
- **D2**: SD card CS
- **D4/D5**: I2C bus (OLED, RTC, Grove I2C)
- **D8/D9/D10**: SPI bus (SD card)
- **A3/D3**: Buzzer
- **D1**: User button

Available free pins: D0, D6, D7, D11-D18 (varies by XIAO board)

### Board Layout

```
+------------------------------------------+
|  [OLED Display]    [RTC Battery]         |
|                                          |
|  [User BTN]  [RESET]  [SWD Header]       |
|                                          |
|  [Grove I2C]  [Grove I2C]                |
|  [Grove UART]                            |
|  [Grove A0/D0]                           |
|                                          |
|  [Buzzer]    [5V Servo]                  |
|                                          |
|  [MicroSD Slot]                          |
|                                          |
|  [XIAO Socket]                           |
+------------------------------------------+
```

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/expansion-base.md`
  - OLED display with u8g2 library
  - RTC time reading/setting
  - SD card read/write
  - Buzzer tone generation
  - Complete project examples

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## Power Options

1. **USB Power**: Connect Type-C cable for 5V input
2. **Battery Power**: Connect 3.7V Li-Po battery to JST 2.0mm connector
3. **Charging**: Battery charges automatically when USB is connected and power switch is ON

**Charging Indicator:**
- LED flashing = Battery not connected or charging error
- LED solid = Charging in progress

## Usage Notes

1. **First Connection**: Plug XIAO into the middle of the two female header connectors to avoid damage
2. **SD Card Format**: Use FAT or exFAT format for best compatibility
3. **Buzzer Disable**: Cut the trace on the board to permanently disable buzzer
4. **SWD Debug**: Use male header for debugger connection (nRF52840, RP2040)
5. **CircuitPython**: Well-supported via MicroSD card for library storage

## Applications

- **Sensor Hubs**: Display sensor data on OLED
- **Data Loggers**: Log data to SD card with RTC timestamps
- **Mini Displays**: Portable information displays
- **Rapid Prototyping**: No soldering required for most connections
- **Battery-Powered Projects**: On-board battery charging and management
- **Grove Integration**: Quick connection to Grove ecosystem

## Troubleshooting

### Display Not Working

**Symptom**: OLED screen is blank or shows garbage

**Possible Causes:**
1. I2C address conflict - Check that D4/D5 are not used by other devices
2. Power issue - Verify USB or battery is connected
3. Loose connection - Ensure XIAO is properly seated

**Solution**: Check I2C scan, verify wiring, reseat XIAO

### SD Card Not Detected

**Symptom**: SD card initialization fails

**Possible Causes:**
1. Wrong format - Must be FAT or exFAT
2. CS pin mismatch - Should be D2
3. Card size issue - Try smaller card (≤32GB)

**Solution**: Reformat card as FAT/exFAT, verify CS pin is D2

### RTC Time Incorrect

**Symptom**: Time resets or is wrong

**Possible Causes:**
1. Dead battery - CR1220 needs replacement
2. Not initialized - Set time after first power-on

**Solution**: Replace CR1220 battery, initialize RTC in code

### Buzzer Not Working

**Symptom**: No sound from buzzer

**Possible Causes:**
1. Wrong pin - Should be A3 (D3)
2. Buzzer disabled - Trace may be cut
3. PWM frequency issue - Adjust in code

**Solution**: Verify pin is A3, check trace is intact, adjust PWM
