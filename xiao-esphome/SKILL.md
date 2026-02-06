---
name: xiao-esphome
description: |
  ESPHome YAML configuration for SeeedStudio XIAO series microcontrollers.
  Use when writing ESPHome YAML configs for XIAO boards to integrate with Home Assistant.
  Supports ESP32C3, ESP32C6, ESP32S3, and RP2040 XIAO boards.
  Requires /xiao for pin definitions and board specifications.
---

# ESPHome for XIAO

ESPHome is a YAML-based firmware framework for creating custom firmware for ESP devices, with native Home Assistant integration.

## Supported XIAO Boards

| Board | Platform | WiFi | Notes |
|-------|----------|------|-------|
| ESP32C3 | ESP32-C3 | ✅ | Cost-effective, WiFi + BLE |
| ESP32C6 | ESP32-C6 | ✅ | WiFi 6, low power |
| ESP32S3 | ESP32-C3 | ✅ | Dual-core, AI, camera support |
| RP2040 | RP2040 | ❌ | No WiFi, use with external WiFi |

**Not supported:** nRF52840, SAMD21, nRF54L15 (ESPHome doesn't support these platforms)

## Quick Start

### 1. Installation

Choose your ESPHome installation method:

- **CLI (Recommended)**: See `setup/cli.md`
- **Docker**: See `setup/docker.md`
- **Home Assistant Add-on**: See `setup/add-on.md`

**Quick install** (CLI):
```bash
pip install esphome
```

### 2. Create New Configuration

```bash
esphome create config.yaml
```

### 3. Select Your Board Template

Choose the appropriate board configuration from [references/boards/](references/boards/):

- **ESP32C3**: [esp32c3.md](references/boards/esp32c3.md)
- **ESP32C6**: [esp32c6.md](references/boards/esp32c6.md)
- **ESP32S3**: [esp32s3.md](references/boards/esp32s3.md)
- **RP2040**: [rp2040.md](references/boards/rp2040.md)

### 4. Upload to Board

```bash
# Via USB
esphome run config.yaml

# Via OTA (after initial upload)
esphome upload config.yaml
```

## Board Base Configurations

Copy the base configuration for your XIAO board and customize:

```yaml
# Example: XIAO ESP32C3 base config
substitutions:
  device_name: "xiao-sensor"
  friendly_name: "XIAO Sensor"

esphome:
  name: ${device_name}
  platform: ESP32
  board: esp32-c3-devkitm-1

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

# See references/boards/esp32c3.md for complete config
```

## Common Components

Add components as needed:

### Sensors
- **BME280**: Temperature, humidity, pressure (I2C)
- **DHT**: Temperature, humidity (digital)
- **AHT20**: Temperature, humidity (I2C)
- See [references/components/sensors.md](references/components/sensors.md)

### Displays
- **SSD1306**: OLED display (I2C/SPI)
- **ST7789**: TFT display (SPI)
- See [references/components/displays.md](references/components/displays.md)

### Communication
- **I2C Bus**: [references/components/i2c.md](references/components/i2c.md)
- **SPI Bus**: [references/components/spi.md](references/components/spi.md)
- **UART**: [references/components/uart.md](references/components/uart.md)

### Connectivity
- **WiFi**: [references/components/wifi.md](references/components/wifi.md) - ESP32 only
- **MQTT**: [references/components/mqtt.md](references/components/mqtt.md)
- **API**: [references/components/api.md](references/components/api.md) - Home Assistant integration

## Complete Examples

Ready-to-use configurations:

- **Weather Station**: [references/examples/weather-station.md](references/examples/weather-station.md) - BME280 + display + OTA
- **Smart Switch**: [references/examples/smart-switch.md](references/examples/smart-switch.md) - Relay + button + LED
- **Sensor Node**: [references/examples/sensor-node.md](references/examples/sensor-node.md) - Multi-sensor + MQTT
- **BLE Proxy**: [references/examples/ble-proxy.md](references/examples/ble-proxy.md) - BLE to WiFi bridge (ESP32)

## XIAO Pin Mappings

### I2C (Most boards)

| Board | SDA | SCL |
|-------|-----|-----|
| ESP32C3 | D4 | D5 |
| ESP32S3 | D7 | D6 |
| RP2040 | D6 | D7 |

### SPI

| Board | MOSI | MISO | SCLK |
|-------|------|------|------|
| ESP32C3 | D10 | D9 | D8 |
| ESP32S3 | D11 | D9 | D10 |
| RP2040 | D10 | D9 | D8 |

See board-specific files for complete pin mappings.

## Special Features

### Deep Sleep (ESP32)

```yaml
# Enable deep sleep
deep_sleep:
  run_duration: 60s
  sleep_duration: 300s

# Wake on button
deep_sleep:
  wakeup_pin: D13
  wakeup_pin_mode: KEEP_AWAKE
```

See [references/advanced/deep-sleep.md](references/advanced/deep-sleep.md)

### OTA Updates

```yaml
# OTA enabled by default in base configs
ota:
  - platform: esphome
    password: !secret ota_password
```

### Camera (ESP32S3 Sense)

```yaml
esp32_camera:
  # See references/boards/esp32s3.md for camera config
```

## Home Assistant Integration

### Automatic Discovery

ESPHome automatically discovers devices in Home Assistant when using the `api:` component.

### Secrets

Use Home Assistant secrets:

```yaml
# In ESPHome config
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

api:
  password: !secret api_password
```

### Dashboard

Access ESPHome dashboard: `http://esphome.local:6052`

## Troubleshooting

### Upload Fails

1. Check board selection in `esphome: platform`
2. Enter boot mode: Hold BOOT button, press RESET
3. Try different USB cable
4. Check port: `esphome config.yaml show-ports`

### WiFi Not Connecting

1. Verify credentials
2. Enable `fast_connect:` for quicker connection
3. Check 2.4GHz only (5GHz not supported)

### I2C/SPI Not Working

1. Check pin mappings for your board
2. Enable `scan: true` for I2C
3. Verify pull-up resistors for I2C

See `/xiao` for authoritative pin definitions.

## Code Verification

**After generating ESPHome YAML, validate it using esphome CLI.**

### Quick Verification (If ESPHome CLI is Already Set Up)

```bash
# Validate and compile
esphome config your_device.yaml
esphome compile your_device.yaml

# Upload to device
esphome run your_device.yaml
```

### Full Verification (First-Time Setup)

**Step 1: Install ESPHome CLI** (if not installed)

```bash
pip install esphome

# Verify installation
esphome version
```

**Step 2: Create Secrets File** (if not exists)

```bash
# Create secrets.yaml in same directory as config
cat > secrets.yaml <<'EOF'
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"
api_password: "YourAPIPassword"
ota_password: "YourOTAPassword"
EOF
```

**Step 3: Validate Configuration**

```bash
# Check configuration validity
esphome config your_device.yaml

# Expected output:
# ✔ Config valid
# ➜ Platform: ESP32
# ➜ Board: esp32-c3-devkitm-1
```

**Step 4: Compile and Upload**

```bash
# Compile firmware
esphome compile your_device.yaml

# Upload to device (first time via USB)
esphome run your_device.yaml

# Or via OTA (after first upload)
esphome upload your_device.yaml
```

### Common Errors and Solutions

| Error | Solution |
|-------|----------|
| `Invalid YAML syntax` | Check indentation (use 2 spaces) |
| `esphome: command not found` | Run `pip install esphome` |
| `Board not found` | Verify board name (e.g., `esp32-c3-devkitm-1`) |
| `I2C bus not found` | Check I2C pins with `/xiao` skill |

### Common esphome CLI Commands

| Command | Purpose |
|---------|---------|
| `esphome config <file>` | Validate YAML configuration |
| `esphome compile <file>` | Compile firmware |
| `esphome run <file>` | Compile and upload |
| `esphome logs <file>` | View device logs |

### Example: Complete Verification

```yaml
# xiao_sensor.yaml - XIAO ESP32C3

substitutions:
  device_name: "xiao-sensor"

esphome:
  name: ${device_name}
  platform: ESP32
  board: esp32-c3-devkitm-1

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

**Verification command:**
```bash
esphome config xiao_sensor.yaml && esphome compile xiao_sensor.yaml
```

## Best Practices

1. **Use substitutions** for device names and common values
2. **Enable OTA** for wireless updates
3. **Use secrets** for sensitive data
4. **Enable logging** during development
5. **Add status LED** for visual feedback
6. **Use packages** for reusable configurations
7. **Test components** individually before combining

## Advanced Topics

- **Packages**: [references/advanced/packages.md](references/advanced/packages.md) - Reusable config snippets
- **Automations**: [references/advanced/automation.md](references/advanced/automation.md) - ESPHome scripting
- **Web Server**: [references/advanced/web-server.md](references/advanced/web-server.md) - Web interface
- **Custom Sensor**: [references/advanced/custom-sensor.md](references/advanced/custom-sensor.md) - Custom C++ sensors

## Migration from Arduino

Key differences:

| Arduino | ESPHome |
|---------|---------|
| C++ code | YAML config |
| Manual polling | Automated updates |
| No HA integration | Native HA integration |
| Complex OTA | Built-in OTA |
| Manual WiFi | ESPHome WiFi |

## Resources

### setup/
ESPHome environment installation:
- `setup/cli.md` - ESPHome CLI installation via pip
- `setup/docker.md` - ESPHome in Docker container
- `setup/add-on.md` - Home Assistant ESPHome add-on
- `setup/troubleshooting.md` - Common ESPHome issues and solutions

- ESPHome Documentation: https://esphome.io/
- ESPHome Components: https://esphome.io/components/
- Home Assistant: https://www.home-assistant.io/
- XIAO Documentation: SeeedStudio Wiki

---

**Requires:** `/xiao` skill for pin definitions
**Compatible with:** Home Assistant 2021.12+
**Platform:** ESPHome 2021.12+
