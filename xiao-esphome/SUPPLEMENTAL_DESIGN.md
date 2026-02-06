# XIAO ESPHome Skill Design

## Overview

ESPHome is a YAML-based firmware framework for Home Assistant integration. This skill provides board-specific configurations for XIAO series boards.

## Supported Boards

| Board | ESPHome Platform | Status |
|-------|-----------------|--------|
| ESP32C3 | ESP32-C3 | ✅ Native support |
| ESP32C6 | ESP32-C6 | ✅ Native support |
| ESP32S3 | ESP32-S3 | ✅ Native support |
| RP2040 | RP2040 | ✅ Native support |
| nRF52840 | ❌ Not supported | ❌ No ESPHome support |
| SAMD21 | ❌ Not supported | ❌ No ESPHome support |

## Skill Architecture

```
/xiao-esphome
├── SKILL.md (main skill file)
└── references/
    ├── boards/ (board-specific base configs)
    │   ├── esp32c3.md
    │   ├── esp32c6.md
    │   ├── esp32s3.md
    │   └── rp2040.md
    ├── components/ (common ESPHome components)
    │   ├── sensors.md (BME280, DHT, etc.)
    │   ├── binary-sensors.md
    │   ├── switches.md
    │   ├── outputs.md (PWM, GPIO)
    │   ├── i2c.md
    │   ├── spi.md
    │   ├── uart.md
    │   ├── displays.md
    │   ├── wifi.md
    │   ├── mqtt.md
    │   └── api.md
    ├── examples/ (complete examples)
    │   ├── weather-station.md
    │   ├── smart-switch.md
    │   ├── sensor-node.md
    │   └── ble-proxy.md
    └── advanced/
        ├── deep-sleep.md
        ├── ota.md
        ├── web-server.md
        └── automation.md
```

## Design Principles

### 1. Board-Specific Base Configurations

Each board gets a base YAML template with:
- Board name and platform
- Correct pin definitions for I2C, SPI, UART
- Built-in LED configuration
- Board-specific features (e.g., WiFi for ESP32)

### 2. Progressive Disclosure

- **SKILL.md**: Quick start, board selection, component overview
- **Boards**: Base configurations for each XIAO board
- **Components**: ESPHome component documentation
- **Examples**: Complete working configurations

### 3. Dependency Chain

`/xiao-esphome` → `/xiao` (for pin definitions and board specs)

## Implementation Plan

### Phase 1: Board Configurations
- ESP32C3 base config
- ESP32C6 base config
- ESP32S3 base config
- RP2040 base config

### Phase 2: Common Components
- I2C/SPI/UART bus setup
- Common sensors (BME280, DHT, AHT20)
- Binary sensors
- Switches/outputs
- Display support (SSD1306, ST7789)
- WiFi/MQTT/API setup

### Phase 3: Complete Examples
- Weather station (BME280 + display + OTA)
- Smart switch (relay + button + LED)
- Sensor node (multi-sensor + MQTT)
- BLE proxy (ESP32-only)

### Phase 4: Advanced Topics
- Deep sleep configuration
- OTA updates
- Web server for configuration
- Automation and scripting

## ESPHome XIAO Pin Mappings

### ESP32C3
```
i2c:
  sda: D4
  scl: D5

spi:
  mosi: D10
  miso: D9
  clk: D8

uart:
  tx_pin: D6
  rx_pin: D7
```

### ESP32S3
```
i2c:
  sda: D7
  scl: D6

spi:
  mosi: D11
  miso: D9
  clk: D10

# Camera support (Sense model)
esp32_camera:
  data_pins: [D15, D14, D13, D12, D11, D10, D9, D8]
  vsync_pin: D6
  href_pin: D5
  pclk_pin: D4
  xclk_pin: D7
```

### RP2040
```
i2c:
  sda: D6
  scl: D7

spi:
  mosi: D10
  miso: D9
  clk: D8

uart:
  tx_pin: D0
  rx_pin: D1
```

## Key ESPHome Features for XIAO

### WiFi (ESP32 boards only)
- Native WiFi support
- Fast_connect for power saving
- AP mode for configuration

### Sensors
- BME280 (I2C)
- DHT11/22/AM2302
- AHT20/10 (I2C)
- Internal temperature (ESP32)

### Displays
- SSD1306 OLED (I2C/SPI)
- ST7789 TFT (SPI)
- HD44780 LCD (I2C)

### Special Features
- Deep sleep (ESP32)
- OTA updates
- Home Assistant API
- MQTT discovery
- Web server

## Example: Base Configuration Template

```yaml
# XIAO ESP32C3 Base Configuration
substitutions:
  device_name: "xiao-esp32c3"
  friendly_name: "XIAO ESP32C3"

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  platform: ESP32
  board: esp32-c3-devkitm-1

# WiFi (ESP32 only)
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

# Enable logging
logger:

# Enable Home Assistant API
api:

# Enable OTA updates
ota:

# Enable Web Server
web_server:

# I2C Bus
i2c:
  sda: D4
  scl: D5
  scan: true

# SPI Bus
spi:
  mosi: D10
  miso: D9
  clk: D8

# Built-in LED
status_led:
  pin:
    number: D13
    inverted: true
```

## Validation Checklist

- [ ] Board pin mappings match XIAO pinout reference
- [ ] ESPHome platform names are correct
- [ ] Component examples are tested
- [ ] WiFi configuration uses fast_connect
- [ ] OTA is enabled for all boards
- [ ] Deep sleep examples include wake sources
- [ ] Camera configuration (ESP32S3 Sense)
- [ ] BLE proxy (ESP32 boards only)

## References

- ESPHome Official Docs: https://esphome.io/
- ESPHome Components: https://esphome.io/components/index.html
- XIAO Pinout Reference: Ref/PinOut/XIAO_Pinout_Reference.md
