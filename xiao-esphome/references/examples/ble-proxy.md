# BLE Proxy Example

ESP32 BLE to WiFi bridge for Home Assistant.

## Features

- BLE scanning
- Device tracking
- Temperature/humidity sensors (BLE)
- MQTT forwarding
- Home Assistant BLE proxy

## Hardware

- XIAO ESP32C3/ESP32S3/ESP32C6
- BLE devices to track

## Configuration

```yaml
# XIAO BLE Proxy
substitutions:
  device_name: "xiao-ble-proxy"
  friendly_name: "XIAO BLE Proxy"
  mqtt_topic: "ble-proxy/xiao"

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
  power_save_mode: none

  ap:
    ssid: "${device_name} Fallback"
    password: !secret ap_password

captive_portal:

# Status LED
status_led:
  pin:
    number: GPIO13
    inverted: true

# Enable BLE
esp32_ble:
  on_ble_advertise:
    then:
      - lambda: |-
          ESP_LOGD("ble", "Discovered BLE device");

# BLE Tracker
ble_tracker:
  scan_interval: 5s
  duration: 1min
  active_scan: true

  # Track specific devices
  on_ble_advertise:
    - mac_address: "AA:BB:CC:DD:EE:FF"
      then:
        - lambda: |-
            ESP_LOGI("ble", "Found my device!");

# BLE RSSI Sensor
sensor:
  - platform: ble_rssi
    mac_address: "AA:BB:CC:DD:EE:FF"
    name: "My Device RSSI"
    update_interval: 5s

  # WiFi Signal
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s

  # Uptime
  - platform: uptime
    name: "Uptime"
    update_interval: 60s

# BLE Presence Detection
binary_sensor:
  - platform: ble_presence
    mac_address: "AA:BB:CC:DD:EE:FF"
    name: "My Device Present"
    device_class: presence

# Text Sensors
text_sensor:
  - platform: ble_scanner
    name: "BLE Scanner"
```

## Xiaomi MiFlora Sensor

```yaml
# Add Xiaomi MiFlora plant sensor
esp32_ble:
  on_ble_advertise:
    - mac_address: "XX:XX:XX:XX:XX:XX"
      then:
        - ble_client.find:
            address: "XX:XX:XX:XX:XX:XX"
        - lambda: |-
            auto *bt = esphome::ble_client::global_ble_client;
            bt->write_data(0x0035, {0xA0, 0x1F});

sensor:
  - platform: xiaomi_miflora
    mac_address: "XX:XX:XX:XX:XX:XX"
    temperature:
      name: "MiFlora Temperature"
    moisture:
      name: "MiFlora Moisture"
    illuminance:
      name: "MiFlora Illuminance"
    conductivity:
      name: "MiFlora Conductivity"
    battery:
      name: "MiFlora Battery"
    update_interval: 60s
```

## LYWSD02/Xiaomi Thermometer

```yaml
# Xiaomi LYWSD02 / MHO-C303
sensor:
  - platform: atc_mithermometer
    mac_address: "XX:XX:XX:XX:XX:XX"
    temperature:
      name: "ATC Temperature"
    humidity:
      name: "ATC Humidity"
    battery:
      name: "ATC Battery"
    update_interval: 60s
```

## BLE Beacons

```yaml
# Track iBeacon
esp32_ble:
  on_ble_advertise:
    - mac_address: "XX:XX:XX:XX:XX:XX"
      then:
        - lambda: |-
            ESP_LOGI("ble", "Found iBeacon");

# Create BLE beacon
ble_beacon:
  - uuid: 'd0679eb2-b343-45b1-a8a4-30361df858e5'
    major: 1
    minor: 1
    tx_power: -59
```

## Multiple Device Tracking

```yaml
# Track multiple devices
sensor:
  - platform: ble_rssi
    mac_address: "AA:BB:CC:DD:EE:FF"
    name: "Phone RSSI"
    update_interval: 5s

  - platform: ble_rssi
    mac_address: "11:22:33:44:55:66"
    name: "Watch RSSI"
    update_interval: 5s

  - platform: ble_rssi
    mac_address: "77:88:99:AA:BB:CC"
    name: "Keys RSSI"
    update_interval: 5s

binary_sensor:
  - platform: ble_presence
    mac_address: "AA:BB:CC:DD:EE:FF"
    name: "Phone Present"

  - platform: ble_presence
    mac_address: "11:22:33:44:55:66"
    name: "Watch Present"
```

## MQTT Forwarding

```yaml
# Forward BLE data to MQTT
mqtt:
  broker: !secret mqtt_broker
  username: !secret mqtt_user
  password: !secret mqtt_password

# Forward BLE advertise
esp32_ble:
  on_ble_advertise:
    then:
      - mqtt.publish:
          topic: ${mqtt_topic}/ble/advertise
          payload: !lambda |-
            char buffer[64];
            sprintf(buffer, "Found %s", x.c_str());
            return std::string(buffer);
```

## Room Presence

```yaml
# Track presence in room
sensor:
  - platform: ble_rssi
    mac_address: "AA:BB:CC:DD:EE:FF"
    name: "Room RSSI"
    filters:
      - sliding_window_moving_average:
          window_size: 10
          send_every: 1

binary_sensor:
  - platform: template
    name: "In Room"
    lambda: |-
      return id(room_rssi).state > -80;
    update_interval: 5s
```

## BLE Client

```yaml
# Connect to BLE device
ble_client:
  - mac_address: "XX:XX:XX:XX:XX:XX"
    id: my_ble_client

sensor:
  - platform: ble_client
    ble_client_id: my_ble_client
    name: "BLE Client RSSI"
```

## Complete Smart Home Integration

```yaml
# Full BLE proxy with device tracking
substitutions:
  device_name: "xiao-ble-hub"
  mqtt_topic: "ble/xiao"

esphome:
  name: ${device_name}
  platform: ESP32
  board: esp32-c3-devkitm-1

logger:
api:
ota:
password:

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true

mqtt:
  broker: !secret mqtt_broker
  username: !secret mqtt_user
  password: !secret mqtt_password
  discovery: true
  topic_prefix: ${mqtt_topic}

esp32_ble:

ble_tracker:
  scan_interval: 5s
  duration: 1min

# Track all family devices
sensor:
  # Phone
  - platform: ble_rssi
    mac_address: "AA:BB:CC:DD:EE:FF"
    name: "Phone RSSI"

  # Smart watch
  - platform: ble_rssi
    mac_address: "11:22:33:44:55:66"
    name: "Watch RSSI"

  # Car keys
  - platform: ble_rssi
    mac_address: "77:88:99:AA:BB:CC"
    name: "Keys RSSI"

  # MiFlora sensors
  - platform: xiaomi_miflora
    mac_address: "11:11:11:11:11:11"
    temperature:
      name: "Plant 1 Temp"
    moisture:
      name: "Plant 1 Moisture"

  - platform: xiaomi_miflora
    mac_address: "22:22:22:22:22:22"
    temperature:
      name: "Plant 2 Temp"
    moisture:
      name: "Plant 2 Moisture"

binary_sensor:
  # Presence detection
  - platform: ble_presence
    mac_address: "AA:BB:CC:DD:EE:FF"
    name: "Phone Home"

  - platform: ble_presence
    mac_address: "11:22:33:44:55:66"
    name: "Watch Home"

# Text sensors
text_sensor:
  - platform: ble_scanner
    name: "BLE Devices"
```

## Home Assistant Automations

### Arrival Detection

```yaml
- alias: "Phone Arrived Home"
  trigger:
    - platform: state
      entity_id: binary_sensor.xiao_ble_proxy_phone_home
      to: "on"
  action:
    - service: notify.mobile_app
      data:
        title: "Welcome Home"
        message: "Your phone has been detected"
```

### Departure Detection

```yaml
- alias: "Phone Left Home"
  trigger:
    - platform: state
      entity_id: binary_sensor.xiao_ble_proxy_phone_home
      to: "off"
      for:
        minutes: 2
  action:
    - service: notify.mobile_app
      data:
        title: "Left Home"
        message: "Your phone is no longer detected"
```

### Plant Watering Alert

```yaml
- alias: "Plant Needs Water"
  trigger:
    - platform: numeric_state
      entity_id: sensor.xiao_ble_proxy_plant_1_moisture
      below: 20
  action:
    - service: notify.mobile_app
      data:
        title: "Plant Needs Water"
        message: "Plant 1 moisture is {{ states('sensor.xiao_ble_proxy_plant_1_moisture') }}%"
```

## Optimizing Battery

```yaml
# Reduce scan frequency
ble_tracker:
  scan_interval: 30s  # Less frequent scanning
  duration: 30s

# Use passive scan
ble_tracker:
  active_scan: false

# Disable when not needed
switch:
  - platform: template
    name: "BLE Scanner"
    optimistic: true
    turn_on_action:
      - ble_tracker.start:
    turn_off_action:
      - ble_tracker.stop:
```

## Troubleshooting

### Devices Not Found

1. Enable active scan
2. Increase scan duration
3. Check device MAC address
4. Verify device is advertising

### RSSI Inaccurate

1. Use sliding window filter
2. Increase update interval
3. Check for interference
4. Verify antenna not blocked

### Battery Drain

1. Reduce scan frequency
2. Use passive scan
3. Disable when not needed
4. Adjust scan duration

### Connection Issues

1. Check BLE client bonding
2. Verify device pairing
3. Monitor connection logs
4. Try restarting BLE

## Best Practices

1. **Use MAC addresses** for reliable tracking
2. **Enable RSSI filtering** to reduce false positives
3. **Set appropriate scan intervals** for battery vs responsiveness
4. **Use active scan** for more data, passive for battery
5. **Monitor multiple beacons** for better presence detection
6. **Combine with WiFi** for more reliable presence
