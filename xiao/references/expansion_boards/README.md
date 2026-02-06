# XIAO Expansion Boards

XIAO expansion boards add peripherals and capabilities to XIAO development boards. This directory contains hardware documentation for all available expansion boards.

## Available Expansion Boards

| Board | Key Features | Arduino Implementation |
|-------|--------------|------------------------|
| [Expansion Base](expansion-base.md) | OLED, RTC, SD card, buzzer, Grove connectors, battery | `xiao-arduino:expansion-boards/expansion-base.md` |
| [Round Display](round-display.md) | 1.28" touchscreen (240x240), RTC, SD slot | `xiao-arduino:expansion-boards/round-display.md` |
| [Grove Shield](grove-shield.md) | 8 Grove connectors + battery management | `xiao-arduino:expansion-boards/grove-shield.md` |
| [ePaper V2](epaper-v2.md) | 7 ePaper display sizes (1.54" to 7.5") | `xiao-arduino:expansion-boards/epaper-v2.md` |
| [CAN Bus](can-bus.md) | MCP2515 controller for automotive/industrial | `xiao-arduino:expansion-boards/can-bus.md` |
| [RS485](rs485.md) | Industrial communication | `xiao-arduino:expansion-boards/rs485.md` |
| [LED Driver](led-driver.md) | 5V/12V LED strips with WLED support | `xiao-arduino:expansion-boards/led-driver.md` |
| [Bus Servo](bus-servo.md) | Serial bus servo control for robotics | `xiao-arduino:expansion-boards/bus-servo.md` |
| [GPS/GNSS](gps-gnss.md) | L76K GNSS module for positioning | `xiao-arduino:expansion-boards/gps-gnss.md` |
| [GPIO Expander](gpio-expander.md) | MCP23017 16-bit I/O expansion | `xiao-arduino:expansion-boards/gpio-expander.md` |
| [RGB Matrix](rgb-matrix.md) | RGB LED matrix driver | `xiao-arduino:expansion-boards/rgb-matrix.md` |
| [COB LED](cob-led.md) | Chip-on-Board LED lighting | `xiao-arduino:expansion-boards/cob-led.md` |

## Board Compatibility

| XIAO Board | Most Compatible Expansion Boards | Notes |
|------------|--------------------------------|-------|
| ESP32C3/C5/C6/S3 | All boards | Recommended for displays, WiFi/BLE projects |
| nRF52840 | All boards | Best for BLE applications |
| RP2040/RP2350 | All boards | Good for PIO projects |
| SAMD21 | Most boards | Some limitations with complex displays |
| MG24 | Most boards | Check specific board compatibility |
| nRF54L15 | Limited | Not compatible with Expansion Base (different SWD) |

**Note:** Always check the individual expansion board documentation for specific compatibility information.

## Platform Support

- **Arduino**: Full support in `/xiao-arduino/references/expansion-boards/`
- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`
- **ESPHome**: Select boards supported in `/xiao-esphome/references/`

## Selecting an Expansion Board

### For Display Projects
- **Simple text/UI**: Expansion Base (built-in OLED)
- **Touch interface**: Round Display
- **Low power/e-ink**: ePaper V2
- **LED effects**: RGB Matrix

### For Communication
- **Automotive/industrial**: CAN Bus
- **Industrial networks**: RS485
- **GPS tracking**: GPS/GNSS

### For Actuator Control
- **Robotics**: Bus Servo Driver
- **LED strips**: LED Driver (5V/12V)
- **COB lighting**: COB LED Driver

### For General Prototyping
- **Maximum flexibility**: Expansion Base (OLED, RTC, SD, buzzer, Grove)
- **Grove ecosystem**: Grove Shield
- **More I/O pins**: GPIO Expander

## Hardware Documentation

Each expansion board file contains:
- **Overview**: What the board does and typical use cases
- **Compatibility**: Which XIAO boards are supported
- **Specifications**: Electrical and mechanical specs
- **Pin Connections**: Which XIAO pins are used
- **Platform Implementations**: Links to platform-specific code

For code examples and platform-specific implementation, see the platform skill documentation (Arduino, MicroPython, etc.).
