# Smart Switch Example

Complete ESPHome smart switch with relay, button, LED feedback.

## Features

- Relay control
- Physical button with toggle
- LED status indicator
- Auto-off timer
- Power monitoring (optional)
- Home Assistant integration

## Hardware

- XIAO ESP32C3/ESP32S3
- Relay module (active low or high)
- Push button
- LED indicator

## Wiring

| XIAO Pin | Component |
|----------|-----------|
| D2 | Relay IN |
| D13 | Button (other side to GND) |
| D3 | LED (with 220Ω resistor) |
| 3.3V | Relay VCC |
| GND | Relay GND, Button, LED cathode |

## Configuration

```yaml
# XIAO Smart Switch
substitutions:
  device_name: "xiao-switch"
  friendly_name: "XIAO Smart Switch"
  # Relay type (active_high or active_low)
  relay_type: "active_low"
  # Auto-off timer in seconds (0 to disable)
  auto_off_timer: "3600"  # 1 hour

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  platform: ESP32
  board: esp32-c3-devkitm-1

# Enable logging
logger:

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

  ap:
    ssid: "${device_name} Fallback"
    password: !secret ap_password

captive_portal:

# Status LED (built-in)
status_led:
  pin:
    number: GPIO13
    inverted: true

# Binary Sensor (Button)
binary_sensor:
  - platform: gpio
    name: "Button"
    id: button
    pin:
      number: GPIO13
      inverted: true
      mode:
        input: true
        pullup: true
    on_press:
      then:
        - switch.toggle: relay
        - light.toggle: status_led

  # Restart button (long press)
  - platform: gpio
    name: "Restart Button"
    pin:
      number: GPIO13
      inverted: true
      mode:
        input: true
        pullup: true
    on_press:
      then:
        - delay: 5s
        - if:
            condition:
              binary_sensor.is_on: button
            then:
              - switch.restart: restart_switch

# Switch (Relay)
switch:
  - platform: gpio
    name: "Relay"
    id: relay
    pin:
      number: GPIO2
      inverted: ${relay_type == "active_low"}
    on_turn_on:
      then:
        - light.turn_on: status_led
        - if:
            condition:
              lambda: 'return ${auto_off_timer} > 0;'
            then:
              - script.execute: auto_off_script
    on_turn_off:
      then:
        - light.turn_off: status_led
        - script.stop: auto_off_script

  - platform: restart
    name: "Restart"
    id: restart_switch

# Light (Status LED)
light:
  - platform: binary
    name: "Status LED"
    id: status_led
    output: status_led_output

output:
  - platform: gpio
    id: status_led_output
    pin: GPIO3

# Scripts
script:
  - id: auto_off_script
    mode: restart
    then:
      - delay: ${auto_off_timer}s
      - switch.turn_off: relay

# Sensors
sensor:
  # WiFi Signal
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s

  # Uptime
  - platform: uptime
    name: "Uptime"
    update_interval: 60s

  # Switch state timer
  - platform: template
    name: "On Time"
    id: on_time
    unit_of_measurement: "s"
    accuracy_decimals: 0
    lambda: |-
      if (id(relay).state) {
        return id(on_time).state + 1;
      } else {
        return 0;
      }
    update_interval: 1s

# Text Sensors
text_sensor:
  - platform: wifi_info
    ip_address:
      name: "IP Address"

  - platform: version
    name: "ESPHome Version"

# Select (Auto-off timer)
select:
  - platform: template
    name: "Auto-off Timer"
    id: auto_off_select
    options:
      - "Off"
      - "5 minutes"
      - "15 minutes"
      - "30 minutes"
      - "1 hour"
      - "2 hours"
      - "4 hours"
    optimistic: true
    set_action:
      - lambda: |-
          int seconds = 0;
          if (id(auto_off_select).state == "5 minutes") {
            seconds = 300;
          } else if (id(auto_off_select).state == "15 minutes") {
            seconds = 900;
          } else if (id(auto_off_select).state == "30 minutes") {
            seconds = 1800;
          } else if (id(auto_off_select).state == "1 hour") {
            seconds = 3600;
          } else if (id(auto_off_select).state == "2 hours") {
            seconds = 7200;
          } else if (id(auto_off_select).state == "4 hours") {
            seconds = 14400;
          }
          // Update auto-off timer
```

## With Power Monitoring (HLW8012)

```yaml
# Add power monitoring sensor
sensor:
  - platform: hlw8012
    sel_pin:
      number: GPIO5
      mode: OUTPUT
    cf_pin: GPIO6
    cf1_pin: GPIO7
    current_resistor: 0.001  # Current resistor
    voltage_divider: 2351    # Voltage divider
    change_mode_every: 8     # Toggle between voltage/current
    update_interval: 5s

    voltage:
      name: "Voltage"
      id: voltage
      accuracy_decimals: 1

    current:
      name: "Current"
      id: current
      accuracy_decimals: 3

    power:
      name: "Power"
      id: power
      accuracy_decimals: 1

    energy:
      name: "Energy"
      id: energy
      accuracy_decimals: 3
      unit_of_measurement: kWh
      filters:
        - multiply: 0.00005  # Convert to kWh
```

## Multi-Relay Version

```yaml
# Control multiple relays
switch:
  - platform: gpio
    name: "Relay 1"
    pin: GPIO2

  - platform: gpio
    name: "Relay 2"
    pin: GPIO3

  - platform: gpio
    name: "Relay 3"
    pin: GPIO4

# All on/off
switch:
  - platform: template
    name: "All On"
    turn_on_action:
      - switch.turn_on: relay_1
      - switch.turn_on: relay_2
      - switch.turn_on: relay_3
    turn_off_action:
      - switch.turn_off: relay_1
      - switch.turn_off: relay_2
      - switch.turn_off: relay_3
```

## Garage Door Opener

```yaml
# Momentary switch for garage door
switch:
  - platform: gpio
    name: "Garage Door"
    id: garage_door
    pin: GPIO2
    on_turn_on:
      - delay: 1s
      - switch.turn_off: garage_door

# Status sensors
binary_sensor:
  - platform: gpio
    name: "Door Open"
    pin: GPIO3
    device_class: garage_door

  - platform: gpio
    name: "Door Closed"
    pin: GPIO4
    device_class: garage_door
```

## Cover (Blind Control)

```yaml
# Control blinds with two relays
cover:
  - platform: template
    name: "Blinds"
    open_action:
      - switch.turn_on: relay_up
      - delay: 10s
      - switch.turn_off: relay_up
    close_action:
      - switch.turn_on: relay_down
      - delay: 10s
      - switch.turn_off: relay_down
    stop_action:
      - switch.turn_off: relay_up
      - switch.turn_off: relay_down
    has_stop: true
    optimistic: true

switch:
  - platform: gpio
    name: "Relay Up"
    id: relay_up
    pin: GPIO2

  - platform: gpio
    name: "Relay Down"
    id: relay_down
    pin: GPIO3
```

## Home Assistant Automations

### Toggle with notification

```yaml
- alias: "Smart Switch Toggled"
  trigger:
    - platform: state
      entity_id: switch.xiao_switch_relay
  action:
    - service: notify.mobile_app
      data_template:
        title: "Switch Toggled"
        message: >
          Switch is now {% if states('switch.xiao_switch_relay') == 'on' %}ON{% else %}OFF{% endif %}
```

### Auto-off notification

```yaml
- alias: "Auto-off Warning"
  trigger:
    - platform: state
      entity_id: sensor.xiao_switch_on_time
      above: 3000  # 50 minutes
  condition:
    - condition: state
      entity_id: switch.xiao_switch_relay
      state: "on"
  action:
    - service: notify.mobile_app
      data:
        title: "Auto-off Warning"
        message: "Switch will turn off in 10 minutes"
```

### Power alert

```yaml
- alias: "High Power Alert"
  trigger:
    - platform: numeric_state
      entity_id: sensor.xiao_switch_power
      above: 1000
  action:
    - service: notify.mobile_app
      data:
        title: "High Power"
        message: "Power consumption is {{ states('sensor.xiao_switch_power') }}W"
```

## Wall Switch Template

```yaml
# Physical toggle switch instead of button
binary_sensor:
  - platform: gpio
    name: "Wall Switch"
    pin:
      number: GPIO13
      mode: INPUT_PULLUP
    on_press:
      - switch.toggle: relay
    on_release:
      - switch.toggle: relay
```

## LED Brightness Control

```yaml
# PWM LED for brightness feedback
light:
  - platform: monochromatic
    name: "Status LED"
    output: status_led_pwm
    effects:
      - pulse:
          name: "Slow Pulse"
          transition_length: 1s
          update_interval: 2s

output:
  - platform: ledc
    id: status_led_pwm
    pin: GPIO3

# Change brightness based on relay state
switch:
  - platform: gpio
    name: "Relay"
    pin: GPIO2
    on_turn_on:
      - light.turn_on:
          id: status_led
          brightness: 100%
    on_turn_off:
      - light.turn_on:
          id: status_led
          brightness: 10%
```

## Troubleshooting

### Relay Not Switching

1. Check relay type (active high/low)
2. Verify pin number
3. Test with multimeter
4. Check relay power supply

### Button Not Working

1. Verify pull-up resistor
2. Check pin inversion
3. Test with serial monitor
4. Check for debounce issues

### LED Not Lighting

1. Check LED polarity
2. Verify resistor value
3. Test LED separately
4. Check GPIO voltage

### Auto-off Not Working

1. Verify script is enabled
2. Check timer value
3. Look for script errors in logs
4. Ensure relay state updates correctly
