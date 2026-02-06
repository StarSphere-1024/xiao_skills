---
name: xiao
description: Pin definitions and board specifications for SeeedStudio XIAO series microcontrollers. Use this skill to identify correct GPIO pins, alternate functions, power domains, strapping pins, or boot mode restrictions for XIAO boards (ESP32C3/C5/C6/S3, nRF52840, RP2040/RP2350, SAMD21, MG24, nRF54L15, RA4M1). Provides board selection guidance and links to platform sub-skills. Requires a platform sub-skill for code implementation.
---

# XIAO Board Reference

This skill provides pin definitions and board specifications for SeeedStudio XIAO series. For code implementation, use the appropriate platform sub-skill.

## Platform Sub-Skills

| Platform | Sub-Skill | When to Use |
|----------|-----------|-------------|
| Arduino | `/xiao-arduino` | Arduino/C++ framework development |
| MicroPython | `/xiao-micropython` | MicroPython development |
| ESPHome | `/xiao-esphome` | Home Assistant ESPHome YAML configs |

**First select your platform sub-skill, then read board reference below.**

## Board Selection

Read the corresponding board reference for pin definitions:

| Board | Reference | Platform Support |
|-------|-----------|------------------|
| **ESP32C3** | `boards/esp32c3.md` | Arduino, MicroPython, ESPHome |
| **ESP32C5** | `boards/esp32c5.md` | Arduino |
| **ESP32C6** | `boards/esp32c6.md` | Arduino, MicroPython, ESPHome |
| **ESP32S3** | `boards/esp32s3.md` | Arduino, MicroPython, ESPHome |
| **ESP32S3 Sense** | `boards/esp32s3.md` | Arduino, MicroPython |
| **nRF52840** | `boards/nrf52840.md` | Arduino, MicroPython |
| **nRF52840 Sense** | `boards/nrf52840.md` | Arduino, MicroPython |
| **RP2040** | `boards/rp2040.md` | Arduino, MicroPython, ESPHome |
| **RP2350** | `boards/rp2350.md` | Arduino |
| **SAMD21** | `boards/samd21.md` | Arduino |
| **MG24** | `boards/mg24.md` | Arduino |
| **MG24 Sense** | `boards/mg24.md` | Arduino |
| **nRF54L15 Sense** | `boards/nrf54l15.md` | MicroPython only |
| **RA4M1** | `boards/ra4m1.md` | Arduino |

## Board Quick Reference

### ESP32 Series
| Board | Wireless | GPIO | ADC | Special |
|-------|----------|------|-----|---------|
| ESP32C3 | WiFi, BLE | 11 | 5 | Low cost |
| ESP32C5 | WiFi, BLE | ~11 | - | WiFi 6 |
| ESP32C6 | WiFi, BLE | 13 | 5 | Zigbee, low power |
| ESP32S3 | WiFi, BLE | 11 | 4 | Dual-core, AI |

### nRF Series
| Board | Wireless | GPIO | ADC | Special |
|-------|----------|------|-----|---------|
| nRF52840 | BLE 5.0 | 21 | 6 | NFC, USB |
| nRF52840 Sense | BLE 5.0 | 21 | 6 | + PDM mic |
| nRF54L15 Sense | BLE 5.4 | 15 | 4 | Ultra low power |

### RP Series
| Board | GPIO | ADC | Special |
|-------|------|-----|---------|
| RP2040 | 30 | 4 | PIO, SWD boot |
| RP2350 | 30 | 4 | ARM M33, DSP |

### Other
| Board | GPIO | ADC | Special |
|-------|------|-----|---------|
| SAMD21 | 16 | 6 | USB HID |
| MG24 | 20 | 8 | Matter, Zigbee |
| RA4M1 | 18 | 14 | Arduino UNO R4 chip |

## Expansion Boards

XIAO expansion boards add peripherals and capabilities. For complete hardware documentation, see `references/expansion_boards/`.

| Expansion Board | Key Features | Arduino Reference |
|----------------|--------------|-------------------|
| Expansion Base | OLED, RTC, SD card, buzzer, Grove connectors | `xiao-arduino:expansion-boards/expansion-base.md` |
| Round Display | 1.28" touchscreen (240x240) | `xiao-arduino:expansion-boards/round-display.md` |
| Grove Shield | 8 Grove connectors + battery management | `xiao-arduino:expansion-boards/grove-shield.md` |
| ePaper V2 | 7 ePaper display sizes (1.54" to 7.5") | `xiao-arduino:expansion-boards/epaper-v2.md` |
| CAN Bus | MCP2515 controller for automotive/industrial | `xiao-arduino:expansion-boards/can-bus.md` |
| RS485 | Industrial communication | `xiao-arduino:expansion-boards/rs485.md` |
| LED Driver | 5V/12V LED strips with WLED support | `xiao-arduino:expansion-boards/led-driver.md` |
| Bus Servo | Serial bus servo control for robotics | `xiao-arduino:expansion-boards/bus-servo.md` |
| GPS/GNSS | L76K GNSS module for positioning | `xiao-arduino:expansion-boards/gps-gnss.md` |
| GPIO Expander | MCP23017 16-bit I/O expansion | `xiao-arduino:expansion-boards/gpio-expander.md` |
| RGB Matrix | RGB LED matrix driver | `xiao-arduino:expansion-boards/rgb-matrix.md` |
| COB LED | Chip-on-Board LED lighting | `xiao-arduino:expansion-boards/cob-led.md` |

For detailed hardware specifications, pin mappings, and compatibility, see `references/expansion_boards/`.

## Critical Pin Restrictions

### ESP32 Series Strapping Pins
These pins affect boot mode - **avoid using** or use carefully:

| Board | Pin | Function | Restriction |
|-------|-----|----------|-------------|
| ESP32C3/C6 | D8 (GPIO20/8) | SPI SCK | Must be HIGH/float at boot |
| ESP32C3/C6 | D9 (GPIO21/9) | SPI MISO | Connected to BOOT button |
| ESP32C3/C6 | D0, D1 | USB D-/D+ | Used for USB upload |
| ESP32S3 | D0, D1 | GPIO16/17 | USB D-/D+ |
| ESP32S3 | D8, D9 | GPIO8/9 | Strapping pins |

**Warning:** Using strapping pins incorrectly can prevent booting or uploading.

### RP2040 BOOTSEL
- Hold BOOTSEL button while connecting USB to enter firmware upload mode
- Drive appears as RPI-RP2 mass storage

### I2C/SPI/UART Pin Mappings
Pin mappings vary by board - **always check board reference** before wiring.

## Sense Models

For camera/microphone sensor boards, see `special/sense-models.md`:
- ESP32S3 Sense camera configuration
- nRF52840 Sense PDM microphone
- MG24 Sense built-in sensors

## Low Power

Deep sleep and power optimization are platform-specific. Use your platform sub-skill:

- **Arduino**: Use the `/xiao-arduino` skill and see the API documentation for deep sleep (ESP32, nRF52, RP2040)
- **MicroPython**: Use the `/xiao-micropython` skill and see the API documentation for deep sleep (ESP32, nRF52, RP2040)
- **ESPHome**: Use the `/xiao-esphome` skill and see the advanced documentation for deep sleep configuration

## Bootloader Reflash

For bootloader recovery or firmware flashing procedures, see `special/bootloader-reflash.md`.

## Reference Structure

```
xiao/
├── boards/              # Pin definitions for each board (READ THIS FIRST)
├── expansion_boards/    # XIAO expansion board hardware documentation
│   ├── README.md        # Expansion board overview and selection guide
│   ├── expansion-base.md
│   ├── round-display.md
│   ├── grove-shield.md
│   ├── epaper-v2.md
│   ├── can-bus.md
│   ├── rs485.md
│   ├── led-driver.md
│   ├── bus-servo.md
│   ├── gps-gnss.md
│   ├── gpio-expander.md
│   ├── rgb-matrix.md
│   └── cob-led.md
└── special/
    ├── bootloader-reflash.md  # Bootloader recovery
    └── sense-models.md        # Sense model camera/sensors
```

## Workflow

1. **Load platform sub-skill** first (`/xiao-arduino`, `/xiao-micropython`, `/xiao-esphome`)
2. **Read board reference** for pin definitions (e.g., `boards/esp32c3.md`, `boards/rp2040.md`)
3. **Check pin restrictions** for strapping/boot pins
4. **Generate code** using platform sub-skill patterns

## Important Notes

- **Always verify pin mappings** in board reference before connecting peripherals
- **Avoid strapping pins** on ESP32 boards unless necessary
- **Use appropriate sub-skill** for code examples and platform patterns
- **Check Sense model docs** for camera/microphone configuration
