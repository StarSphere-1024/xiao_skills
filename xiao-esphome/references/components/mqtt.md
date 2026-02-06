# MQTT Configuration

## Basic MQTT Setup

```yaml
mqtt:
  broker: homeassistant.local
  username: mqtt_user
  password: mqtt_password
  discovery: true
```

## MQTT with Home Assistant

```yaml
mqtt:
  broker: homeassistant.local
  port: 1883
  username: !secret mqtt_user
  password: !secret mqtt_password
  discovery: true  # Auto-discovery in HA
  discovery_prefix: homeassistant
  birth_message:
    topic: xiao-sensor/status
    payload: online
  will_message:
    topic: xiao-sensor/status
    payload: offline
```

## MQTT over TLS

```yaml
mqtt:
  broker: mqtt.example.com
  port: 8883
  username: user
  password: pass
  ssl_fingerprints:
    - "12:34:56:78:9A:BC:DE:F0:12:34:56:78:9A:BC:DE:F0:12:34:56:78:9A:BC:DE:F0"
```

## Custom Topics

```yaml
mqtt:
  broker: homeassistant.local
  topic_prefix: xiao/sensor  # Prefix all topics

sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
      # Topic will be: xiao/sensor/temperature/state
```

## MQTT Publish on Button Press

```yaml
mqtt:
  broker: homeassistant.local

binary_sensor:
  - platform: gpio
    pin: GPIO0
    name: "Button"
    on_press:
      - mqtt.publish:
          topic: home/xiao/button
          payload: "pressed"
    on_release:
      - mqtt.publish:
          topic: home/xiao/button
          payload: "released"
```

## MQTT Subscribe

```yaml
mqtt:
  broker: homeassistant.local

  on_message:
    - topic: home/xiao/command
      payload: "on"
      then:
        - switch.turn_on: relay

    - topic: home/xiao/command
      payload: "off"
      then:
        - switch.turn_off: relay

switch:
  - platform: gpio
    pin: GPIO5
    name: "Relay"
    id: relay
```

## MQTT JSON Output

```yaml
mqtt:
  broker: homeassistant.local

sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
      id: temp
    pressure:
      name: "Pressure"
      id: press
    humidity:
      name: "Humidity"
      id: hum

  - platform: template
    name: "All Sensors JSON"
    lambda: |-
      return {};
    json_message:
      - topic: sensors/xiao
        payload: |-
          {
            "temperature": {{ id(temp).state }},
            "pressure": {{ id(press).state }},
            "humidity": {{ id(hum).state }}
          }
```

## MQTT with Templates

```yaml
mqtt:
  broker: homeassistant.local

text_sensor:
  - platform: template
    name: "MQTT Status"
    lambda: |-
      return {"Connected"};
    update_interval: 60s
    on_value:
      - mqtt.publish:
          topic: home/xiao/status
          payload: !lambda 'return x;'
```

## MQTT IDs and Retain

```yaml
mqtt:
  broker: homeassistant.local

sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
      id: mqtt_temp
      # Home Assistant entity: sensor.mqtt_temp

  - platform: dht
    pin: GPIO2
    temperature:
      name: "DHT Temperature"
      id: mqtt_dht_temp
```

## QoS Levels

```yaml
mqtt:
  broker: homeassistant.local

sensor:
  - platform: dht
    temperature:
      name: "Temperature"
      # QoS: 0=fire and forget, 1=at least once, 2=exactly once
      qos: 1
      retain: true  # Keep last message
```

## MQTT with Lambda Payloads

```yaml
mqtt:
  broker: homeassistant.local

interval:
  - interval: 60s
    then:
      - mqtt.publish:
          topic: sensors/xiao
          payload: !lambda |-
            static char buffer[100];
            snprintf(buffer, sizeof(buffer), "{\"temp\":%.1f,\"hum\":%.1f}",
              id(temp_sensor).state, id(hum_sensor).state);
            return buffer;
```

## Multiple MQTT Brokers

```yaml
mqtt:
  - id: mqtt_client1
    broker: broker1.local
    username: user1
    password: pass1

  - id: mqtt_client2
    broker: broker2.local
    username: user2
    password: pass2

sensor:
  - platform: dht
    temperature:
      name: "Temp"
      on_value:
        - mqtt.publish:
            mqtt_id: mqtt_client1
            topic: sensor/temp
            payload: !lambda 'return to_string(x);'
        - mqtt.publish:
            mqtt_id: mqtt_client2
            topic: backup/sensor/temp
            payload: !lambda 'return to_string(x);'
```

## MQTT Last Will

```yaml
mqtt:
  broker: homeassistant.local
  client_id: xiao_sensor
  keepalive: 60s
  birth_message:
    topic: xiao-sensor/status
    payload: online
  will_message:
    topic: xiao-sensor/status
    payload: offline
```

## MQTT Discovery Override

```yaml
mqtt:
  broker: homeassistant.local
  discovery: true
  discovery_prefix: homeassistant  # Default
  # Or disable discovery for specific entity
  discovery_retain: true

sensor:
  - platform: dht
    temperature:
      name: "Temperature"
      # Disable auto-discovery
      # Set discovery_unique_id: null or omit
```

## MQTT with Authentication

```yaml
# Using secrets
mqtt:
  broker: !secret mqtt_broker
  port: !secret mqtt_port
  username: !secret mqtt_username
  password: !secret mqtt_password

# secrets.yaml
mqtt_broker: homeassistant.local
mqtt_port: 1883
mqtt_username: xiao_user
mqtt_password: secret_password
```

## Best Practices

1. **Use discovery** - Automatic Home Assistant integration
2. **Set birth/will** - Track device online status
3. **Use meaningful topics** - Hierarchical naming
4. **Match QoS to use** - 0 for sensors, 1 for critical
5. **Test with tools** - Use MQTT Explorer to verify
6. **Use secrets** - Don't hardcode credentials

## Troubleshooting

### Not Connecting

```yaml
# Enable debug logging
logger:
  level: DEBUG

mqtt:
  broker: homeassistant.local
  # Check broker is reachable
  # Check username/password
  # Check port (1883 non-TLS, 8883 TLS)
```

### Messages Not Received

1. Check topic names match
2. Verify subscriber is connected
3. Check QoS level
4. Enable logger to see MQTT traffic

### Discovery Not Working

```yaml
mqtt:
  discovery: true
  discovery_prefix: homeassistant
  # Ensure Home Assistant MQTT integration enabled
```

## MQTT Publish Examples

### Publish on Interval

```yaml
interval:
  - interval: 300s
    then:
      - mqtt.publish:
          topic: xiao/heartbeat
          payload: !lambda 'return id(uptime).state;'
```

### Publish JSON

```yaml
mqtt:
  broker: homeassistant.local

sensor:
  - platform: template
    name: "JSON Output"
    lambda: |-
      return id(temp).state;
    json_message:
      topic: xiao/sensors
      payload: |-
        {
          "temp": {{ id(temp).state | float }},
          "hum": {{ id(hum).state | float }},
          "press": {{ id(press).state | float }}
        }
```

### Conditional Publish

```yaml
mqtt:
  broker: homeassistant.local

sensor:
  - platform: dht
    temperature:
      name: "Temperature"
      id: temp
      on_value_range:
        - above: 30.0
          then:
            - mqtt.publish:
                topic: xiao/alert
                payload: "High temperature"
```
