# XIAO ESP32C6 ESPHome Configuration

## Base Configuration

Complete ESPHome configuration for XIAO ESP32C6.

```yaml
# XIAO ESP32C6 Base Configuration
substitutions:
  device_name: "xiao-esp32c6"
  friendly_name: "XIAO ESP32C6"
  device_description: "SeeedStudio XIAO ESP32C6"

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  comment: ${device_description}
  platform: ESP32
  board: esp32-c6-devkitm-1
  project:
    name: "SeeedStudio.XIAO_ESP32C6"
    version: "1.0.0"

# Enable logging
logger:
  level: INFO
  baud_rate: 0

# Enable Home Assistant API
api:
  encryption:
    key: !secret api_encryption_key
  password: !secret api_password

# Enable OTA updates
ota:
  - platform: esphome
    password: !secret ota_password

# WiFi
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  power_save_mode: high

  ap:
    ssid: "${device_name} Fallback"
    password: !secret ap_password

captive_portal:
web_server:
  port: 80

mdns:

# Status LED (built-in white LED)
status_led:
  pin:
    number: GPIO9
    inverted: true

# I2C Bus
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true

# SPI Bus
spi:
  clk_pin: GPIO8
  miso_pin: GPIO9
  mosi_pin: GPIO10

# Internal sensors
sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s

  - platform: uptime
    name: "Uptime"

  - platform: esp32_temperature
    name: "Internal Temperature"

text_sensor:
  - platform: wifi_info
    ip_address:
      name: "IP Address"

  - platform: version
    name: "ESPHome Version"

switch:
  - platform: restart
    name: "Restart"
```

## Pin Mapping Reference

| XIAO Pin | GPIO | ESPHome Pin | Function |
|----------|------|-------------|----------|
| D0 | GPIO8 | GPIO8 | USB-, BOOT |
| D1 | GPIO9 | GPIO9 | USB+, BOOT, LED |
| D2 | GPIO10 | GPIO10 | SPI MOSI |
| D3 | GPIO11 | GPIO11 | SPI MISO |
| D4 | GPIO12 | GPIO12 | I2C SDA |
| D5 | GPIO13 | GPIO13 | I2C SCL |
| D6 | GPIO18 | GPIO18 | UART TX |
| D7 | GPIO19 | GPIO19 | UART RX |
| D8 | GPIO20 | GPIO20 | SPI SCK |
| D9 | GPIO21 | GPIO21 | SPI MISO |
| D10 | GPIO10 | GPIO10 | SPI MOSI |

## Board Specifications

| Feature | Value |
|---------|-------|
| MCU | ESP32-C6 |
| Core | RISC-V 32-bit @ 160MHz |
| Flash | 4 MB |
| RAM | 512 KB |
| WiFi | 802.11b/g/n (2.4GHz) |
| BLE | Bluetooth 5.0 |
| Zigbee | Zigbee 3.0 |
| Thread | Thread 1.3 |
| GPIO | 13 pins |
| ADC | 12-bit, 5 channels |
| PWM | 6 channels |

## Low Power Configuration

```yaml
# ESP32C6 optimized for low power
deep_sleep:
  run_duration: 30s
  sleep_duration: 600s  # 10 minutes

# WiFi power save
wifi:
  power_save_mode: high

# Disable unused features
logger:
  level: WARN
```

## Zigbee Support (Experimental)

```yaml
# ESP32C6 supports Zigbee (ESPHome development)
zigbee:
  on_frame:
    then:
      - logger.log: "Zigbee frame received"
```

## Thread Support (Experimental)

```yaml
# Thread border router (experimental)
thread:
  on_message:
    then:
      - logger.log: "Thread message"
```

## Key Differences from ESP32C3

| Feature | ESP32C3 | ESP32C6 |
|---------|--------|---------|
| RAM | 400 KB | 512 KB |
| WiFi | 802.11b/g/n | 802.11b/g/n |
| BLE | 5.0 | 5.0 |
| Zigbee | No | Yes |
| Thread | No | Yes |
| Power | Better | Much better |

## Complete Example: Low Power Sensor

```yaml
substitutions:
  device_name: "xiao-c6-sensor"

esphome:
  name: ${device_name}
  platform: ESP32
  board: esp32-c6-devkitm-1

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  power_save_mode: high

api:
ota:
logger:

i2c:
  sda: GPIO4
  scl: GPIO5

sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
    pressure:
      name: "Pressure"
    humidity:
      name: "Humidity"
    address: 0x76
    update_interval: 300s

# Deep sleep between readings
deep_sleep:
  run_duration: 30s
  sleep_duration: 600s
```

## Troubleshooting

### Upload Issues

Similar to ESP32C3:
1. Hold BOOT button
2. Check COM port
3. Try different baud rate

### Low Power Mode

```yaml
# Check current consumption
sensor:
  - platform: template
    name: "Current"
    lambda: |-
      return {};  // Requires external sensor
```

## Migration from ESP32C3

1. Update `board: esp32-c6-devkitm-1`
2. Enable `power_save_mode: high`
3. Update pin mappings if needed
4. Take advantage of Zigbee/Thread when available
