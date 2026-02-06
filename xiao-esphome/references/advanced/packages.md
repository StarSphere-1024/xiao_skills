# ESPHome Packages - Reusable Configuration

## Overview

Packages allow you to split your configuration into reusable, modular YAML files that can be shared across multiple devices.

## Basic Package Structure

```
esphome/
├── packages/
│   ├── base.yaml
│   ├── wifi.yaml
│   ├── sensors/
│   │   ├── bme280.yaml
│   │   └── dht.yaml
│   └── displays/
│       └── oled.yaml
└── xiao-sensor.yaml
```

## Creating Packages

### base.yaml - Common Base Configuration

```yaml
# packages/base.yaml
esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  comment: ${device_comment}
  project:
    name: "seeed.xiao"
    version: "1.0.0"

# Enable logging
logger:
  level: INFO

# Enable API
api:
  password: !secret api_password
  encryption:
    key: !secret api_encryption_key

# Enable OTA
ota:
  password: !secret ota_password

# Enable Web Server
web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password
```

### wifi.yaml - WiFi Configuration

```yaml
# packages/wifi.yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  power_save_mode: HIGH

  # Enable fallback hotspot
  ap:
    ssid: ${device_name}_Fallback
    password: !secret ap_password

# captive_portal:

# Status LED for WiFi
status_led:
  pin:
    number: GPIO13
    inverted: true
```

### bme280_sensor.yaml - BME280 Sensor Package

```yaml
# packages/sensors/bme280.yaml
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true

sensor:
  - platform: bme280
    i2c_id: bus_i2c
    address: 0x76
    temperature:
      name: "${friendly_name} Temperature"
      id: bme280_temp
      filters:
        - offset: -1.2  # Calibration
    pressure:
      name: "${friendly_name} Pressure"
      id: bme280_press
    humidity:
      name: "${friendly_name} Humidity"
      id: bme280_hum
    update_interval: 60s

# Dew point sensor
sensor:
  - platform: template
    name: "${friendly_name} Dew Point"
    unit_of_measurement: "°C"
    accuracy_decimals: 1
    lambda: |-
      const float a = 17.27;
      const float b = 237.7;
      float temp = id(bme280_temp).state;
      float hum = id(bme280_hum).state;
      float alpha = ((a * temp) / (b + temp)) + log(hum / 100.0);
      return (b * alpha) / (a - alpha);
    update_interval: 60s
```

### oled_display.yaml - SSD1306 Display Package

```yaml
# packages/displays/oled.yaml
i2c:
  sda: GPIO4
  scl: GPIO5

display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
    id: oled_display
    update_interval: 5s
    lambda: |-
      it.printf(0, 0, id(font_small), "Temp: %.1f°C", id(bme280_temp).state);
      it.printf(0, 16, id(font_small), "Hum: %.0f%%", id(bme280_hum).state);
      it.printf(0, 32, id(font_small), "Press: %.0fhPa", id(bme280_press).state);
```

## Using Packages

### Main Configuration File

```yaml
# xiao-sensor.yaml
substitutions:
  device_name: "xiao-sensor"
  friendly_name: "XIAO Sensor"
  device_comment: "XIAO ESP32C3 Weather Station"

esphome:
  platform: ESP32
  board: esp32-c3-devkitm-1

# Import packages
packages:
  base: !include packages/base.yaml
  wifi: !include packages/wifi.yaml
  bme280: !include packages/sensors/bme280.yaml
  oled: !include packages/displays/oled.yaml

# Device-specific overrides
binary_sensor:
  - platform: gpio
    pin: GPIO0
    name: "${friendly_name} Button"
    on_press:
      - logger.log: "Button pressed"
```

## Package Inheritance

Packages can override previous configurations:

```yaml
# packages/base.yaml
logger:
  level: INFO

# In main config
packages:
  base: !include packages/base.yaml

# Override for debug
logger:
  level: DEBUG
```

## Conditional Packages

```yaml
# Main config
substitutions:
  use_display: "true"  # or "false"

packages:
  base: !include packages/base.yaml
  wifi: !include packages/wifi.yaml
  # Conditional display package
  display: !include
    file: packages/displays/oled.yaml
    vars:
      use_it: "${use_display}"
```

## Multi-Device Project

### Project Structure

```
esphome/
├── packages/
│   ├── xiao/
│   │   ├── base.yaml
│   │   ├── wifi.yaml
│   │   └── esp32c3.yaml
│   └── sensors/
│       └── bme280.yaml
├── devices/
│   ├── sensor-1.yaml
│   ├── sensor-2.yaml
│   └── switch-1.yaml
└── secrets.yaml
```

### xiao/base.yaml

```yaml
# packages/xiao/base.yaml
esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  project:
    name: "seeed.xiao"
    version: "1.0.0"

logger:
  level: INFO

api:
  password: !secret api_password

ota:
  password: !secret ota_password
```

### xiao/esp32c3.yaml

```yaml
# packages/xiao/esp32c3.yaml
esphome:
  platform: ESP32
  board: esp32-c3-devkitm-1

# I2C pins for ESP32C3
i2c:
  sda: GPIO4
  scl: GPIO5

# Status LED
status_led:
  pin:
    number: GPIO13
    inverted: true
```

### Device Configuration

```yaml
# devices/sensor-1.yaml
substitutions:
  device_name: "xiao-sensor-1"
  friendly_name: "XIAO Sensor 1"

packages:
  xiao_base: !include packages/xiao/base.yaml
  xiao_esp32c3: !include packages/xiao/esp32c3.yaml
  wifi: !include packages/wifi.yaml
  bme280: !include packages/sensors/bme280.yaml
```

## Remote Packages

Load packages from HTTP:

```yaml
packages:
  base: !include
    url: https://raw.githubusercontent.com/your-repo/esphome-packages/main/base.yaml
    type: json
```

## Package Variables

Pass variables to packages:

```yaml
# packages/sensor_with_i2c.yaml
substitutions:
  sensor_name: "My Sensor"
  i2c_sda: "GPIO4"
  i2c_scl: "GPIO5"

i2c:
  sda: ${i2c_sda}
  scl: ${i2c_scl}

sensor:
  - platform: bme280
    temperature:
      name: "${sensor_name} Temperature"

# In main config
packages:
  sensor: !include
    file: packages/sensor_with_i2c.yaml
    vars:
      sensor_name: "Living Room"
      i2c_sda: "GPIO6"
      i2c_scl: "GPIO7"
```

## Complete Example

### packages/base.yaml

```yaml
substitutions:
  device_name: "xiao-device"
  friendly_name: "XIAO Device"
  log_level: "INFO"

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}

logger:
  level: ${log_level}

api:
  password: !secret api_password

ota:
  password: !secret ota_password

web_server:
  port: 80

button:
  - platform: restart
    name: "${friendly_name} Restart"
```

### packages/sensors/bme280.yaml

```yaml
substitutions:
  temp_offset: "0"
  press_offset: "0"
  hum_offset: "0"

i2c:
  sda: GPIO4
  scl: GPIO5

sensor:
  - platform: bme280
    address: 0x76
    temperature:
      name: "${friendly_name} Temperature"
      filters:
        - offset: ${temp_offset}
    pressure:
      name: "${friendly_name} Pressure"
      filters:
        - offset: ${press_offset}
    humidity:
      name: "${friendly_name} Humidity"
      filters:
        - offset: ${hum_offset}
```

### devices/living-room-sensor.yaml

```yaml
substitutions:
  device_name: "xiao-living-room"
  friendly_name: "Living Room"

esphome:
  platform: ESP32
  board: esp32-c3-devkitm-1

packages:
  base: !include packages/base.yaml
  bme280: !include
    file: packages/sensors/bme280.yaml
    vars:
      temp_offset: "-1.2"
      press_offset: "0"
      hum_offset: "5"
```

## Best Practices

1. **Use substitution variables** - Make packages reusable
2. **Organize by function** - Group related configs
3. **Version control** - Use git for packages
4. **Document packages** - Add comments explaining usage
5. **Test locally** - Verify before deploying
6. **Keep packages small** - Single responsibility
7. **Use naming convention** - Clear, descriptive names

## Troubleshooting

### Package Not Found

```yaml
# Check file path is correct
packages:
  base: !include packages/base.yaml  # Relative to main config
```

### Variable Not Substituted

```yaml
# Define substitutions before including packages
substitutions:
  device_name: "my-device"

packages:
  base: !include packages/base.yaml
```

### Conflicting Components

```yaml
# Packages can override - last one wins
packages:
  base: !include packages/base.yaml  # Has logger: INFO
  debug: !include packages/debug.yaml  # Has logger: DEBUG
# Result: logger will be DEBUG
```

## Package Templates

### Temperature Sensor Template

```yaml
# packages/templates/temp_sensor.yaml
substitutions:
  sensor_name: "Temperature"
  sensor_id: "temp_sensor"
  update_interval: "60s"

sensor:
  - platform: template
    name: "${sensor_name}"
    id: ${sensor_id}
  update_interval: ${update_interval}
```

### Display Template

```yaml
# packages/templates/display.yaml
substitutions:
  display_model: "SSD1306 128x64"
  display_address: "0x3C"

display:
  - platform: ssd1306_i2c
    model: "${display_model}"
    address: ${display_address}
```

## Advanced: Dynamic Package Loading

```yaml
# Load different packages based on substitution
substitutions:
  sensor_type: "bme280"  # or "dht", "aht20"

packages:
  !include
    file: packages/${sensor_type}.yaml
```
