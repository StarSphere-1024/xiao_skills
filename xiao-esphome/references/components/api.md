# Home Assistant API Configuration

## Basic API Setup

```yaml
api:
  password: !secret api_password
```

## API with Encryption

```yaml
api:
  password: !secret api_password
  encryption:
    key: !secret api_encryption_key
```

Generate encryption key:
```bash
esphome generate-encryption-key
```

## API Services

The API enables Home Assistant to control the XIAO device:

### Call Services from Home Assistant

```yaml
# Home Assistant service call
service: esphome.xiao_sensor_turn_on
data:
  relay: 1

# In ESPHome
switch:
  - platform: gpio
    pin: GPIO5
    name: "Relay"
    id: relay
```

### Expose All Entities

```yaml
api:
  password: !secret api_password
  # All sensors, switches, etc. automatically exposed
```

## API with Services

### Custom Services

```yaml
api:
  password: !secret api_password
  services:
    - service: control_relay
      variables:
        relay_state: bool
      then:
        - if:
            condition:
              lambda: 'return relay_state;'
            then:
              - switch.turn_on: relay
            else:
              - switch.turn_off: relay

    - service: set_rgb
      variables:
        red: float
        green: float
        blue: float
      then:
        - light.addressable_set:
            id: rgb_led
            red: !lambda 'return red;'
            green: !lambda 'return green;'
            blue: !lambda 'return blue;'

switch:
  - platform: gpio
    pin: GPIO5
    name: "Relay"
    id: relay
```

## API Home Assistant Dashboard

When API is connected, device appears in ESPHome dashboard:

```yaml
api:
  password: !secret api_password
  # Enables:
  # - View logs
  # - Execute commands
  # - View states
  # - Fetch configuration
```

## API with Reboot

```yaml
api:
  password: !secret api_password

button:
  - platform: restart
    name: "Restart Device"
```

## API Statistics

```yaml
api:
  password: !secret api_password
  # View connection stats in HA:
  # - signal strength
  # - uptime
  # - free memory
  # - WiFi version
```

## API with OTA

```yaml
api:
  password: !secret api_password

ota:
  - platform: esphome
    password: !secret ota_password
  # API enables OTA updates from Home Assistant
```

## Number Component (API)

```yaml
api:
  password: !secret api_password

number:
  - platform: template
    name: "Threshold"
    min_value: 0
    max_value: 100
    step: 1
    initial_value: 50
    set_action:
      - lambda: |-
          id(threshold_var) = x;

sensor:
  - platform: template
    name: "Value"
    lambda: |-
      if (id(temp_sensor).state > id(threshold_var)) {
        return 1.0;
      }
      return 0.0;

globals:
  - id: threshold_var
    type: float
    restore_value: no
    initial_value: '50.0'
```

## Select Component (API)

```yaml
api:
  password: !secret api_password

select:
  - platform: template
    name: "Mode"
    options:
      - "Auto"
      - "Manual"
      - "Off"
    initial_option: "Auto"
    on_value:
      - lambda: |-
          if (x == "Auto") id(mode_var) = 0;
          else if (x == "Manual") id(mode_var) = 1;
          else id(mode_var) = 2;
```

## Text Component (API)

```yaml
api:
  password: !secret api_password

text:
  - platform: template
    name: "Device Name"
    mode: text
    initial_value: "XIAO Sensor"
    on_value:
      - lambda: |-
          id(device_name) = x;

globals:
  - id: device_name
    type: std::string
    restore_value: no
    initial_value: '"XIAO Sensor"'
```

## API with Climate Control

```yaml
api:
  password: !secret api_password

climate:
  - platform: thermostat
    name: "Thermostat"
    sensor: temp_sensor
    idle_action:
      - switch.turn_off: heater
    heat_action:
      - switch.turn_on: heater

sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
      id: temp_sensor

switch:
  - platform: gpio
    pin: GPIO5
    name: "Heater"
    id: heater
```

## API with Lock

```yaml
api:
  password: !secret api_password

lock:
  - platform: template
    name: "Door Lock"
    optimistic: true
    assume_state: true
    lock_action:
      - switch.turn_on: lock_relay
      - delay: 1s
      - switch.turn_off: lock_relay
    unlock_action:
      - switch.turn_on: unlock_relay
      - delay: 1s
      - switch.turn_off: unlock_relay
```

## API Web Server

```yaml
api:
  password: !secret api_password

# Enable web server for local control
web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password
```

## Best Practices

1. **Always use password** - Required for security
2. **Use encryption** - For secure communication
3. **Use secrets** - Don't hardcode passwords
4. **Enable OTA** - For wireless updates
5. **Add restart button** - For easy recovery
6. **Test services** - Verify functionality

## Secrets Configuration

```yaml
# secrets.yaml
api_password: your_secure_password_here
api_encryption_key: generated_key_here
ota_password: your_ota_password_here
web_password: your_web_password_here
```

## Troubleshooting

### API Not Connecting

```yaml
# Enable debug logging
logger:
  level: DEBUG

api:
  password: !secret api_password
  reboot_timeout: 15min
```

Check:
1. Password is correct
2. Encryption key matches
3. Home Assistant is running
4. Network is reachable

### Services Not Working

```yaml
# Verify service is defined
api:
  services:
    - service: test_service
      then:
        - logger.log: "Service called"

# Test from Home Assistant
service: esphome.xiao_sensor_test_service
```

### Encryption Issues

```yaml
# Regenerate keys if needed
api:
  encryption:
    key: !secret api_encryption_key

# In Home Assistant, re-configure device
```

## Example: Complete API Config

```yaml
esphome:
  name: xiao_sensor
  friendly_name: XIAO Sensor

api:
  password: !secret api_password
  encryption:
    key: !secret api_encryption_key
  reboot_timeout: 15min
  services:
    - service: set_led_color
      variables:
        red: int
        green: int
        blue: int
      then:
        - light.addressable_set:
            id: status_led
            red: !lambda 'return red;'
            green: !lambda 'return green;'
            blue: !lambda 'return blue;'

ota:
  password: !secret ota_password

web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password

button:
  - platform: restart
    name: "Restart"

light:
  - platform: neopixelbus
    pin: GPIO8
    num_leds: 1
    name: "Status LED"
    id: status_led
```

## API vs MQTT

| Feature | API | MQTT |
|---------|-----|------|
| Home Assistant Integration | Native | Via discovery |
| Real-time Control | Yes | Yes |
| Services | Native | Via commands |
| Encryption | Yes | Via TLS |
| Latency | Low | Low |
| Recommended | ✅ For HA | ✅ For non-HA |
