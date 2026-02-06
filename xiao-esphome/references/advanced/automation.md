# ESPHome Automation and Scripting

## Basic Automation

### On Boot

```yaml
esphome:
  on_boot:
    then:
      - logger.log: "Booting up..."
      - switch.turn_on: relay
```

### On Shutdown

```yaml
esphome:
  on_shutdown:
    then:
      - logger.log: "Shutting down..."
      - switch.turn_off: relay
```

### On Loop

```yaml
esphome:
  on_loop:
    then:
      - lambda: |-
          // Run every loop iteration
          static int count = 0;
          count++;
```

## Interval Actions

```yaml
# Run every second
interval:
  - interval: 1s
    then:
      - logger.log: "Tick"
```

## Script Component

```yaml
script:
  - id: my_script
    mode: restart
    then:
      - logger.log: "Script started"
      - delay: 1s
      - logger.log: "Script ended"

# Run script
binary_sensor:
  - platform: gpio
    pin: GPIO2
    on_press:
      - script.execute: my_script
```

## Conditional Logic

### If Statement

```yaml
binary_sensor:
  - platform: gpio
    pin: GPIO2
    on_press:
      - if:
          condition:
            sensor.in_range:
              id: temperature
              above: 25
          then:
            - switch.turn_on: fan
          else:
            - switch.turn_off: fan
```

### Wait Until

```yaml
script:
  - id: wait_for_temp
    then:
      - wait_until:
          sensor.in_range:
            id: temperature
            above: 20
      - logger.log: "Temperature reached 20°C"
```

### While Loop

```yaml
script:
  - id: blink_led
    then:
      - while:
          condition:
            binary_sensor.is_on: button
          then:
            - light.turn_on: led
            - delay: 500ms
            - light.turn_off: led
            - delay: 500ms
```

## Switch Actions

### On Turn On/Off

```yaml
switch:
  - platform: gpio
    pin: GPIO2
    id: relay
    on_turn_on:
      then:
        - logger.log: "Relay on"
        - light.turn_on: status_led
    on_turn_off:
      then:
        - logger.log: "Relay off"
        - light.turn_off: status_led
```

### Toggle

```yaml
binary_sensor:
  - platform: gpio
    pin: GPIO3
    on_press:
      - switch.toggle: relay
```

## Sensor Thresholds

```yaml
sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
      id: temperature
      on_value_range:
        - above: 30
          then:
            - logger.log: "High temperature!"
        - below: 10
          then:
            - logger.log: "Low temperature!"
```

## Text Sensor Patterns

```yaml
text_sensor:
  - platform: template
    name: "Status"
    lambda: |-
      if (id(relay).state) {
        return {"ON"};
      } else {
        return {"OFF"};
      }
    update_interval: 1s
```

## Lambda Functions

### Custom Logic

```yaml
script:
  - id: calculate_dewpoint
    mode: restart
    then:
      - lambda: |-
          float temp = id(temperature).state;
          float hum = id(humidity).state;
          float dewpoint = 243.5 * (log(hum/100.0) + (17.67 * temp / (243.5 + temp))) / (17.67 - log(hum/100.0) - (17.67 * temp / (243.5 + temp)));
          ESP_LOGI("dewpoint", "Calculated: %.1f°C", dewpoint);
```

### Custom Sensor

```yaml
sensor:
  - platform: template
    name: "Custom Sensor"
    lambda: |-
      float value = id(sensor1).state * id(sensor2).state;
      return value;
    update_interval: 10s
```

## Time-Based Automation

```yaml
# Run at specific time
time:
  - platform: homeassistant
    id: esptime
    on_time:
      - seconds: 0
        minutes: 0
        hours: 6
        then:
          - logger.log: "6:00 AM - Wake up!"
```

## Sunrise/Sunset

```yaml
# Requires Home Assistant time
time:
  - platform: homeassistant
    id: ha_time

# Turn on at sunset
automation:
  - trigger:
      - platform: time
        at: !lambda "return id(ha_time).sunset().to_c_str();"
    action:
      - switch.turn_on: light
```

## Delay and Timing

```yaml
script:
  - id: sequence
    then:
      - switch.turn_on: relay1
      - delay: 1s
      - switch.turn_on: relay2
      - delay: 2s
      - switch.turn_off: relay1
      - switch.turn_off: relay2
```

## Repeat

```yaml
script:
  - id: blink_5_times
    then:
      - repeat:
          count: 5
          then:
            - light.turn_on: led
            - delay: 500ms
            - light.turn_off: led
            - delay: 500ms
```

## For Loop (Lambda)

```yaml
script:
  - id: fade_led
    then:
      - lambda: |-
          for (int i = 0; i <= 255; i += 5) {
            id(led_pwm).set_level(i);
            delay(10);
          }
```

## Global Variables

```yaml
esphome:
  on_boot:
    then:
      - lambda: |-
          static int counter = 0;
          global_var = &counter;
```

## HTTP Requests

```yaml
http_request:
  useragent: esphome/device
  timeout: 5s

script:
  - id: send_notification
    then:
      - http_request.post:
          url: http://server.com/notify
          headers:
            Content-Type: application/json
          json:
            message: "Hello from XIAO!"
```

## MQTT Publish

```yaml
mqtt:
  on_message:
    - topic: home/command
      then:
        - logger.log: "Received command"

script:
  - id: publish_data
    then:
      - mqtt.publish:
          topic: home/sensor/data
          payload: !lambda "return to_string(id(sensor1).state);"
```

## Complete Automation Example

```yaml
# Temperature-based fan control
esphome:
  on_boot:
    then:
      - logger.log: "System started"

script:
  - id: check_temperature
    mode: restart
    then:
      - if:
          condition:
            sensor.in_range:
              id: temperature
              above: 25
          then:
            - switch.turn_on: fan
            - light.turn_on: status_led
          else:
            - switch.turn_off: fan
            - light.turn_off: status_led

# Check every minute
interval:
  - interval: 1min
    then:
      - script.execute: check_temperature
```

## Best Practices

1. **Use scripts** for reusable logic
2. **Keep lambdas short** - prefer YAML actions
3. **Use conditions** for branching
4. **Add logging** for debugging
5. **Test incrementally** - build up complexity
6. **Document logic** with comments
7. **Use delays sparingly** - prefer wait_until
