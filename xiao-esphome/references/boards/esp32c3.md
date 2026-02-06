# XIAO ESP32C3 ESPHome Configuration

## Base Configuration

Complete ESPHome configuration for XIAO ESP32C3.

```yaml
# XIAO ESP32C3 Base Configuration
substitutions:
  device_name: "xiao-esp32c3"
  friendly_name: "XIAO ESP32C3"
  device_description: "SeeedStudio XIAO ESP32C3"

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  comment: ${device_description}
  platform: ESP32
  board: esp32-c3-devkitm-1
  project:
    name: "SeeedStudio.XIAO_ESP32C3"
    version: "1.0.0"

# Enable logging
logger:
  level: INFO
  baud_rate: 0  # USB serial

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
  power_save_mode: none

  # Enable fallback hotspot
  ap:
    ssid: "${device_name} Fallback"
    password: !secret ap_password

# Captive portal for WiFi setup
captive_portal:

# Web server
web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password

# MDNS for local discovery
mdns:

# Status LED (built-in orange LED)
status_led:
  pin:
    number: GPIO13
    inverted: true

# I2C Bus (D4=SDA, D5=SCL)
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true
  frequency: 100kHz

# SPI Bus (D8=SCK, D9=MISO, D10=MOSI)
spi:
  clk_pin: GPIO8
  miso_pin: GPIO9
  mosi_pin: GPIO10

# UART (D6=TX, D7=RX)
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600

# Button (D13)
binary_sensor:
  - platform: gpio
    name: "Button"
    pin:
      number: GPIO13
      inverted: true
      mode:
        input: true
        pullup: true
    on_press:
      - logger.log: "Button pressed"

# Internal sensors
sensor:
  # WiFi Signal
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s

  # Uptime
  - platform: uptime
    name: "Uptime"

  # Internal Temperature
  - platform: esp32_temperature
    name: "Internal Temperature"
    update_interval: 60s

  # Internal Memory
  - platform: esp32_hall
    name: "Hall Sensor"
    update_interval: 60s

text_sensor:
  # WiFi Info
  - platform: wifi_info
    ip_address:
      name: "IP Address"
    ssid:
      name: "Connected SSID"
    bssid:
      name: "Connected BSSID"

  # Device version
  - platform: version
    name: "ESPHome Version"
    description: "ESPHome firmware version"

switch:
  # Restart switch
  - platform: restart
    name: "Restart"
    id: restart_switch

  # Safe mode
  - platform: safe_mode
    name: "Safe Mode"
    id: safe_mode_switch
```

## Pin Mapping Reference

| XIAO Pin | GPIO | ESPHome Pin | Function |
|----------|------|-------------|----------|
| D0 | GPIO8 | GPIO8 | USB-, BOOT |
| D1 | GPIO9 | GPIO9 | USB+, BOOT |
| D2 | GPIO10 | GPIO10 | SPI MOSI |
| D3 | GPIO11 | GPIO11 | SPI MISO |
| D4 | GPIO12 | GPIO12 | I2C SDA (alt) |
| D5 | GPIO13 | GPIO13 | I2C SCL (alt), LED |
| D6 | GPIO18 | GPIO18 | UART TX |
| D7 | GPIO19 | GPIO19 | UART RX |
| D8 | GPIO20 | GPIO20 | SPI SCK |
| D9 | GPIO21 | GPIO21 | SPI MISO |
| D10 | GPIO10 | GPIO10 | SPI MOSI |

**Alternative I2C:** Use GPIO4 (D4) and GPIO5 (D5) for I2C

## Board Specifications

| Feature | Value |
|---------|-------|
| MCU | ESP32-C3 |
| Core | RISC-V 32-bit @ 160MHz |
| Flash | 4 MB |
| RAM | 400 KB |
| WiFi | 802.11b/g/n |
| BLE | Bluetooth 5.0 |
| GPIO | 11 pins |
| ADC | 12-bit, 5 channels |
| PWM | 6 channels |

## Common Component Examples

### BME280 Sensor (I2C)

```yaml
sensor:
  - platform: bme280
    i2c_id: bus_i2c
    address: 0x76
    temperature:
      name: "Temperature"
      oversampling: 16x
    pressure:
      name: "Pressure"
      oversampling: 16x
    humidity:
      name: "Humidity"
      oversampling: 16x
    update_interval: 60s
```

### DHT22 Sensor

```yaml
sensor:
  - platform: dht
    pin: GPIO2
    model: DHT22
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"
    update_interval: 60s
```

### SSD1306 OLED Display (I2C)

```yaml
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    lambda: |-
      it.printf(0, 0, id(font), "Hello XIAO!");
```

### Relay Control

```yaml
switch:
  - platform: gpio
    name: "Relay"
    pin: GPIO2
    id: relay_switch
```

### PWM Output

```yaml
output:
  - platform: ledc
    pin: GPIO2
    frequency: 1000 Hz
    id: pwm_output

light:
  - platform: monochromatic
    output: pwm_output
    name: "PWM Light"
```

## Deep Sleep Configuration

```yaml
# Deep sleep for battery powered projects
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s  # 5 minutes
  wakeup_pin: GPIO13
  wakeup_pin_mode: KEEP_AWAKE
```

## BLE Beacon (ESP32C3)

```yaml
esp32_ble:
  on_ble_advertise:
    then:
      - logger.log: "BLE advertising"

ble beacon:
  - uuid: 'd0679eb2-b343-45b1-a8a4-30361df858e5'
    major: 1
    minor: 1
```

## Troubleshooting

### Upload Issues

1. Hold BOOT button while connecting USB
2. Check device manager for COM port
3. Use manual baud rate: `baud_rate: 115200`

### WiFi Issues

1. Use 2.4GHz only
2. Enable `fast_connect: true`
3. Check AP credentials

### I2C Scan

```yaml
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true
```

Check logs for detected devices.

## Complete Example: Weather Station

```yaml
substitutions:
  device_name: "xiao-weather"

esphome:
  name: ${device_name}
  platform: ESP32
  board: esp32-c3-devkitm-1

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true

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

display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
    lambda: |-
      it.printf(0, 0, id(font), "T: %.1f°C", id(bme280_temperature).state);
      it.printf(0, 16, id(font), "P: %.0fhPa", id(bme280_pressure).state);
      it.printf(0, 32, id(font), "H: %.1f%%", id(bme280_humidity).state);
```

## Secrets File

```yaml
# secrets.yaml
wifi_ssid: "YourWiFi"
wifi_password: "YourPassword"
api_password: "YourAPIPassword"
api_encryption_key: "YourEncryptionKey"
ota_password: "YourOTAPassword"
ap_password: "FallbackAPPassword"
web_password: "WebAdminPassword"
```
