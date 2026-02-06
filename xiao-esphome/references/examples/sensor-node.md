# Sensor Node Example

Multi-sensor node with MQTT for distributed monitoring.

## Features

- BME280 (temperature, humidity, pressure)
- MQTT integration
- Battery monitoring
- Deep sleep
- Home Assistant auto-discovery

## Hardware

- XIAO ESP32C3/ESP32C6
- BME280 sensor
- Battery (optional)
- Solar panel (optional)

## Configuration

```yaml
# XIAO Sensor Node
substitutions:
  device_name: "xiao-sensor-node"
  friendly_name: "XIAO Sensor Node"
  mqtt_topic: "home/xiao/sensor"
  # Battery voltage divider (if using battery)
  # Vbat * (R2 / (R1 + R2))
  batt_divider: "2.0"

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  platform: ESP32
  board: esp32-c3-devkitm-1

# Enable logging
logger:

# MQTT
mqtt:
  broker: !secret mqtt_broker
  username: !secret mqtt_user
  password: !secret mqtt_password
  discovery: true  # Home Assistant discovery
  topic_prefix: ${mqtt_topic}
  birth_message:
    topic: ${mqtt_topic}/status
    payload: "online"
  will_message:
    topic: ${mqtt_topic}/status
    payload: "offline"

# Enable OTA updates
ota:
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
    address: 0x76
    temperature:
      name: "Temperature"
      id: temperature
      state_topic: ${mqtt_topic}/temperature
    pressure:
      name: "Pressure"
      id: pressure
      state_topic: ${mqtt_topic}/pressure
    humidity:
      name: "Humidity"
      id: humidity
      state_topic: ${mqtt_topic}/humidity
    update_interval: 60s

  # Battery Voltage
  - platform: adc
    pin: GPIO1
    name: "Battery Voltage"
    id: battery_voltage
    state_topic: ${mqtt_topic}/battery_voltage
    attenuation: 11db
    accuracy_decimals: 3
    filters:
      - multiply: ${batt_divider}
    update_interval: 60s

  # Battery Percentage
  - platform: template
    name: "Battery Percentage"
    id: battery_percent
    state_topic: ${mqtt_topic}/battery_percent
    unit_of_measurement: "%"
    accuracy_decimals: 0
    lambda: |-
      float voltage = id(battery_voltage).state;
      int percent = (voltage - 3.0) / (4.2 - 3.0) * 100;
      if (percent < 0) return 0;
      if (percent > 100) return 100;
      return percent;
    update_interval: 60s

  # WiFi Signal
  - platform: wifi_signal
    name: "WiFi Signal"
    state_topic: ${mqtt_topic}/wifi_signal
    update_interval: 60s

  # Uptime
  - platform: uptime
    name: "Uptime"
    state_topic: ${mqtt_topic}/uptime
    update_interval: 60s

# Text Sensors
text_sensor:
  - platform: template
    name: "Device State"
    id: device_state
    lambda: |-
      if (id(battery_percent).state < 20) {
        return {"low_battery"};
      }
      return {"ok"};
    update_interval: 60s

  - platform: wifi_info
    ip_address:
      name: "IP Address"
      state_topic: ${mqtt_topic}/ip

# Binary Sensors
binary_sensor:
  # Low battery alert
  - platform: template
    name: "Low Battery"
    id: low_battery
    lambda: |-
      return id(battery_percent).state < 20;
    state_topic: ${mqtt_topic}/low_battery

# Switches
switch:
  - platform: restart
    name: "Restart"

# Deep Sleep
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s  # 5 minutes
  wakeup_pin: GPIO13
  wakeup_pin_mode: KEEP_AWAKE
```

## Multiple Sensors

```yaml
# Add more sensors
sensor:
  - platform: bme280
    # ... existing config ...

  # Light sensor (BH1750)
  - platform: bh1750
    i2c_id: bus_i2c
    name: "Light"
    state_topic: ${mqtt_topic}/light
    update_interval: 60s

  # Soil moisture
  - platform: adc
    pin: GPIO2
    name: "Soil Moisture"
    state_topic: ${mqtt_topic}/soil_moisture
    unit_of_measurement: "%"
    accuracy_decimals: 0
    filters:
      - calibrate_linear:
          - 0.5 -> 100
          - 2.5 -> 0
    update_interval: 60s

  # Distance sensor (HC-SR04)
  - platform: ultrasonic
    trigger_pin: GPIO6
    echo_pin: GPIO7
    name: "Distance"
    state_topic: ${mqtt_topic}/distance
    update_interval: 60s
```

## MQTT Commands

### Subscribe to Commands

```yaml
# Add to config
mqtt:
  on_message:
    - topic: ${mqtt_topic}/set/interval
      then:
        - logger.log: "Update interval changed"
        # Process command

    - topic: ${mqtt_topic}/command/sleep
      then:
        - deep_sleep.enter: deep_sleep_1
```

## Home Assistant MQTT Discovery

ESPHome automatically sends discovery messages. Add this to `configuration.yaml`:

```yaml
# Home Assistant configuration
mqtt:
  sensor:
    - name: "XIAO Temperature"
      state_topic: "home/xiao/sensor/temperature"
      unit_of_measurement: "°C"
      device_class: temperature

    - name: "XIAO Humidity"
      state_topic: "home/xiao/sensor/humidity"
      unit_of_measurement: "%"
      device_class: humidity

    - name: "XIAO Pressure"
      state_topic: "home/xiao/sensor/pressure"
      unit_of_measurement: "hPa"
      device_class: pressure

    - name: "XIAO Battery"
      state_topic: "home/xiao/sensor/battery_percent"
      unit_of_measurement: "%"
      device_class: battery

  binary_sensor:
    - name: "XIAO Low Battery"
      state_topic: "home/xiao/sensor/low_battery"
      device_class: battery
```

## Solar Powered Version

```yaml
# Add solar charger monitoring
sensor:
  # Solar panel voltage
  - platform: adc
    pin: GPIO3
    name: "Solar Voltage"
    state_topic: ${mqtt_topic}/solar_voltage
    update_interval: 60s

  # Charging current (requires ACS712)
  - platform: adc
    pin: GPIO4
    name: "Charging Current"
    state_topic: ${mqtt_topic}/charging_current
    unit_of_measurement: "A"
    accuracy_decimals: 3
    update_interval: 60s
```

## Data Logging to InfluxDB

```yaml
# Add to Home Assistant automation
- alias: "Log XIAO Sensor Data"
  trigger:
    - platform: mqtt
      topic: home/xiao/sensor/+
  action:
    - service: influxdb.create
      data_template:
        measurement: xiao_sensor
        tags:
          device: xiao_sensor_node
          sensor: "{{ trigger.topic.split('/')[-1] }}"
        fields:
          value: "{{ trigger.payload }}"
```

## MQTT Retained Messages

```yaml
# Keep last known value
mqtt:
  # ... existing config ...

sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
      # Retain last value
      retain: true
```

## Node-RED Integration

```javascript
// Node-RED flow to process sensor data
[
    {
        "type": "mqtt in",
        "topic": "home/xiao/sensor/temperature"
    },
    {
        "type": "function",
        "func": "msg.payload = parseFloat(msg.payload);\nreturn msg;"
    },
    {
        "type": "debug"
    }
]
```

## Grafana Dashboard

```json
{
  "title": "XIAO Sensor Node",
  "panels": [
    {
      "title": "Temperature",
      "targets": [
        {
          "query": "SELECT mean(\"value\") FROM \"home/xiao/sensor/temperature\" WHERE $timeFilter GROUP BY time($interval)"
        }
      ]
    }
  ]
}
```

## Multi-Node Network

```yaml
# Create multiple sensor nodes
# Node 1: Indoor
substitutions:
  device_name: "xiao-sensor-indoor"
  mqtt_topic: "home/xiao/indoor"

# Node 2: Outdoor
substitutions:
  device_name: "xiao-sensor-outdoor"
  mqtt_topic: "home/xiao/outdoor"

# Node 3: Garage
substitutions:
  device_name: "xiao-sensor-garage"
  mqtt_topic: "home/xiao/garage"
```

## Low Power Optimization

```yaml
# Minimum power consumption
deep_sleep:
  run_duration: 10s
  sleep_duration: 600s  # 10 minutes

# Disable WiFi power save
wifi:
  power_save_mode: high

# Reduce logging
logger:
  level: WARN

# Remove unnecessary components
# Remove web_server, captive_portal for production
```

## Troubleshooting

### MQTT Not Connecting

1. Check broker credentials
2. Verify broker address
3. Check firewall rules
4. Test with MQTT explorer

### Sensors Not Updating

1. Check MQTT topic subscription
2. Verify update intervals
3. Check deep sleep timing
4. Review logs for errors

### Battery Drain

1. Increase sleep duration
2. Reduce update frequency
3. Enable WiFi power save
4. Use ESP32C6 for better battery life

### Deep Sleep Issues

1. Check wake pin configuration
2. Verify sleep duration
3. Ensure no blocking code
4. Monitor boot messages
