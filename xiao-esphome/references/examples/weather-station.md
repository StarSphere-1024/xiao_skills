# Weather Station Example

Complete ESPHome weather station configuration for XIAO boards.

## Features

- BME280 sensor (temperature, humidity, pressure)
- SSD1306 OLED display
- WiFi connectivity
- Home Assistant API integration
- OTA updates
- Deep sleep option

## Hardware

- XIAO ESP32C3/ESP32S3/ESP32C6
- BME280 sensor (I2C)
- SSD1306 OLED display (I2C, optional)

## Wiring

### ESP32C3/ESP32S3

| XIAO Pin | BME280 | SSD1306 |
|----------|--------|---------|
| D4 | SDA | SDA |
| D5 | SCL | SCL |
| 3.3V | VCC | VCC |
| GND | GND | GND |

### RP2040

| XIAO Pin | BME280 | SSD1306 |
|----------|--------|---------|
| D6 | SDA | SDA |
| D7 | SCL | SCL |
| 3.3V | VCC | VCC |
| GND | GND | GND |

## Configuration

```yaml
# XIAO Weather Station
substitutions:
  device_name: "xiao-weather"
  friendly_name: "XIAO Weather Station"
  # BME280 I2C address (0x76 or 0x77)
  bme280_address: "0x76"

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  platform: ESP32
  board: esp32-c3-devkitm-1  # Change for your board

# Enable logging
logger:
  level: INFO

# Enable Home Assistant API
api:
  encryption:
    key: !secret api_encryption_key
  password: !secret api_password

# Enable OTA updates
ota:
  password: !secret ota_password

# WiFi
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  power_save_mode: none

  # Fallback hotspot
  ap:
    ssid: "${device_name} Fallback"
    password: !secret ap_password

captive_portal:

# Web server
web_server:
  port: 80

# Status LED
status_led:
  pin:
    number: GPIO13
    inverted: true

# I2C Bus
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true

# Sensors
sensor:
  # BME280 Temperature
  - platform: bme280
    i2c_id: bus_i2c
    address: ${bme280_address}
    temperature:
      name: "Temperature"
      id: bme280_temperature
      oversampling: 16x
      filters:
        - sliding_window_moving_average:
            window_size: 5
            send_every: 1
    pressure:
      name: "Pressure"
      id: bme280_pressure
      oversampling: 16x
    humidity:
      name: "Humidity"
      id: bme280_humidity
      oversampling: 16x
    update_interval: 60s

  # Dew Point
  - platform: template
    name: "Dew Point"
    id: dew_point
    unit_of_measurement: "°C"
    lambda: |-
      const float a = 17.27;
      const float b = 237.7;
      float temp = id(bme280_temperature).state;
      float hum = id(bme280_humidity).state;
      float alpha = ((a * temp) / (b + temp)) + log(hum / 100.0);
      return (b * alpha) / (a - alpha);
    update_interval: 60s

  # Absolute Humidity
  - platform: template
    name: "Absolute Humidity"
    id: absolute_humidity
    unit_of_measurement: "g/m³"
    accuracy_decimals: 2
    lambda: |-
      float temp = id(bme280_temperature).state;
      float hum = id(bme280_humidity).state;
      float abs_hum = 216.7 * (hum / 100.0 * 6.112 * exp((17.62 * temp) / (243.12 + temp)) / (273.15 + temp));
      return abs_hum;
    update_interval: 60s

  # Sea Level Pressure
  - platform: template
    name: "Sea Level Pressure"
    id: sea_level_pressure
    unit_of_measurement: "hPa"
    lambda: |-
      float pressure = id(bme280_pressure).state;
      float altitude = 100;  // Set your altitude
      return pressure / pow(1 - 0.0065 * altitude / 288.15, 5.255);
    update_interval: 60s

  # WiFi Signal
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s

  # Uptime
  - platform: uptime
    name: "Uptime"
    update_interval: 60s

  # Internal Temperature (ESP32)
  - platform: esp32_temperature
    name: "Internal Temperature"
    update_interval: 300s

# Text Sensors
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

  # Formatted temperature
  - platform: template
    name: "Temperature Text"
    lambda: |-
      char str[20];
      sprintf(str, "%.1f °C", id(bme280_temperature).state);
      return {str};
    update_interval: 60s

# Switches
switch:
  # Restart
  - platform: restart
    name: "Restart"

# Display (Optional - SSD1306)
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    i2c_id: bus_i2c
    address: 0x3C
    update_interval: 5s
    lambda: |-
      // Title
      it.printf(64, 0, id(font_title), TextAlign::CENTER, "Weather");

      // Temperature
      it.printf(0, 16, id(font_data), "T: %.1f°C / %.1f°F",
        id(bme280_temperature).state,
        id(bme280_temperature).state * 9/5 + 32);

      // Humidity
      it.printf(0, 32, id(font_data), "H: %.1f%%", id(bme280_humidity).state);

      // Pressure
      it.printf(0, 48, id(font_data), "P: %.0fhPa", id(bme280_pressure).state);

# Fonts
font:
  - file: "fonts/Roboto-Bold.ttf"
    id: font_title
    size: 16

  - file: "fonts/Roboto-Regular.ttf"
    id: font_data
    size: 12
```

## Battery Powered Version (Deep Sleep)

```yaml
# Add to above config
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s  # 5 minutes

# Remove display (uses power)
# Reduce update intervals
sensor:
  - platform: bme280
    update_interval: 300s  # Match deep sleep
```

## Home Assistant Dashboard Cards

### Sensor Card

```yaml
type: entities
entities:
  - entity: sensor.xiao_weather_temperature
    name: Temperature
  - entity: sensor.xiao_weather_humidity
    name: Humidity
  - entity: sensor.xiao_weather_pressure
    name: Pressure
  - entity: sensor.xiao_weather_dew_point
    name: Dew Point
  - entity: sensor.xiao_weather_wifi_signal
    name: WiFi
```

### Graph Card

```yaml
type: custom:config-template-card
variables:
  TEMPERATURE: states['sensor.xiao_weather_temperature'].entity_id
entities:
  - ${TEMPERATURE}
card:
  type: history-graph
  entities:
    - ${TEMPERATURE}
  hours_to_show: 24
```

### Weather Card (HACS)

```yaml
type: custom:weather-card
entity: weather.xiao_weather
```

## Home Assistant Automations

### High Temperature Alert

```yaml
- alias: "High Temperature Alert"
  trigger:
    - platform: numeric_state
      entity_id: sensor.xiao_weather_temperature
      above: 30
  action:
    - service: notify.mobile_app
      data:
        title: "High Temperature"
        message: "Temperature is {{ states('sensor.xiao_weather_temperature') }}°C"
```

### Pressure Drop Warning

```yaml
- alias: "Pressure Drop Warning"
  trigger:
    - platform: numeric_state
      entity_id: sensor.xiao_weather_pressure
      below: 1000
  action:
    - service: notify.mobile_app
      data:
        title: "Pressure Drop"
        message: "Barometric pressure dropping - possible storm"
```

### Data Logging to InfluxDB

```yaml
- alias: "Log Weather Data"
  trigger:
    - platform: state
      entity_id:
        - sensor.xiao_weather_temperature
        - sensor.xiao_weather_humidity
        - sensor.xiao_weather_pressure
  action:
    - service: influxdb.create
      data_template:
        measurement: weather
        tags:
          device: xiao_weather
        fields:
          temperature: "{{ states('sensor.xiao_weather_temperature') }}"
          humidity: "{{ states('sensor.xiao_weather_humidity') }}"
          pressure: "{{ states('sensor.xiao_weather_pressure') }}"
```

## Alternative: BMP280

```yaml
# Replace BME280 with BMP280 (no humidity)
sensor:
  - platform: bmp280
    i2c_id: bus_i2c
    address: 0x77
    temperature:
      name: "Temperature"
    pressure:
      name: "Pressure"
    update_interval: 60s
```

## Alternative: DHT22 + BMP280

```yaml
# Use separate sensors
sensor:
  - platform: dht
    pin: GPIO2
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"

  - platform: bmp280
    i2c_id: bus_i2c
    pressure:
      name: "Pressure"
```

## Calibration

### Temperature Offset

```yaml
sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
      filters:
        - offset: -1.5  # Adjust as needed
```

### Pressure Calibration

```yaml
sensor:
  - platform: bme280
    pressure:
      name: "Pressure"
      filters:
        - calibrate_linear:
            - 1000 -> 1013  # Calibrate to local station
```

## Multiple Stations

```yaml
# Create multiple weather stations
substitutions:
  device_name: "xiao-weather-indoor"
  # or
  #device_name: "xiao-weather-outdoor"

# Same config, different device name
```

## Troubleshooting

### BME280 Not Detected

1. Check I2C address (0x76 or 0x77)
2. Verify wiring
3. Enable I2C scan in logs

### Incorrect Readings

1. Calibrate with offset
2. Check for heat sources near sensor
3. Allow sensor to stabilize

### WiFi Dropping

1. Enable `fast_connect: true`
2. Check signal strength
3. Use static IP

### Deep Sleep Not Working

1. Ensure no blocking code
2. Check wake interval
3. Verify pin is held low
