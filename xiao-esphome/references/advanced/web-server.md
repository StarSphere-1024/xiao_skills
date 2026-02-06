# ESPHome Web Server

## Basic Web Server Setup

```yaml
web_server:
  port: 80
```

Access at: `http://xiao-sensor.local/`

## Web Server with Authentication

```yaml
web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password
```

Add to secrets.yaml:
```yaml
web_password: your_secure_password
```

## Web Server with SSL

```yaml
web_server:
  port: 80
  ssl:
    certificate: !secret ssl_cert
    key: !secret ssl_key
```

## Web Server Dashboard

The web server provides a dashboard showing:

- Device status
- All sensors and their values
- Switches and binary sensors
- Controls for buttons, selects, numbers
- Log viewer
- Actions (restart, safe mode, etc.)

## Custom Web Server Pages

### Static Content

```yaml
web_server:
  port: 80

http_request:
  - resource: /info
    on_response:
      - logger.log: "Info page accessed"
```

### JSON Endpoint

```yaml
web_server:
  port: 80

sensor:
  - platform: template
    name: "JSON Data"
    json_message:
      topic: sensors/data
      payload: |-
        {
          "temp": {{ id(temp_sensor).state | float }},
          "hum": {{ id(hum_sensor).state | float }}
        }
```

## Web Server with OTA

```yaml
web_server:
  port: 80

ota:
  - platform: esphome
    password: !secret ota_password

# Web server enables OTA updates through browser
```

## Web Server with File Serving

```yaml
web_server:
  port: 80

# Serve static files from SPIFFS
spiFFS:
  - platform: esphome
    # Upload files via esphome run
    # Access at http://device.local/files/
```

## Web Server and API Together

```yaml
api:
  password: !secret api_password

web_server:
  port: 80
  # API and web server can run together
  # Web UI uses API for control
```

## Web Server for Local Control

```yaml
substitutions:
  device_name: "xiao-controller"
  friendly_name: "XIAO Controller"

web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password

# Expose controls
switch:
  - platform: gpio
    pin: GPIO5
    name: "${friendly_name} Relay 1"
    id: relay_1

  - platform: gpio
    pin: GPIO6
    name: "${friendly_name} Relay 2"
    id: relay_2

sensor:
  - platform: dht
    pin: GPIO2
    temperature:
      name: "${friendly_name} Temperature"
    humidity:
      name: "${friendly_name} Humidity"
```

Access at `http://xiao-controller.local/` to control relays and view sensors.

## Web Server with Captive Portal

```yaml
wifi:
  ap:
    ssid: ${device_name}_Setup
    password: !secret ap_password

captive_portal:

web_server:
  port: 80
  # Shows when device creates AP
```

## Web Server Status Indicators

```yaml
web_server:
  port: 80

# Add status indicators
sensor:
  - platform: wifi_signal
    name: "${friendly_name} WiFi Signal"
    web_server:
      ordering: 1

  - platform: uptime
    name: "${friendly_name} Uptime"
    web_server:
      ordering: 2

text_sensor:
  - platform: wifi_info
    ip_address:
      name: "${friendly_name} IP Address"
      web_server:
        ordering: 0
```

## Web Server with Logging

```yaml
web_server:
  port: 80

logger:
  level: INFO
  # Logs available in web UI
```

## Web Server Actions

```yaml
web_server:
  port: 80

# Actions available in web UI
button:
  - platform: restart
    name: "Restart Device"
    web_server:
      ordering: 100

  - platform: safe_mode
    name: "Safe Mode"
    web_server:
      ordering: 101
```

## Web Server with Custom CSS

```yaml
web_server:
  port: 80
  # Custom styling for web UI
  css_url: "http://your-server.com/custom.css"
  # Or embed:
  css: |
    body { background: #f0f0f0; }
```

## Web Server for Firmware Updates

```yaml
web_server:
  port: 80

ota:
  - platform: esphome
    password: !secret ota_password

# Navigate to http://device.local/
# Click "Update" to upload new firmware
```

## Web Server with CORS

```yaml
web_server:
  port: 80
  # Enable CORS for API access
  cors: true
  cors_allowed_origins:
    - https://homeassistant.local
    - http://localhost:8123
```

## Web Server JavaScript API

```yaml
web_server:
  port: 80

# Access device info via JavaScript
# fetch('http://device.local/json/info')
```

## Complete Web Server Example

```yaml
substitutions:
  device_name: "xiao-weather"
  friendly_name: "XIAO Weather"

esphome:
  platform: ESP32
  board: esp32-c3-devkitm-1

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  use_address: 192.168.1.100  # Static IP

api:
  password: !secret api_password

web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password
  cors: true

# Sensors
i2c:
  sda: GPIO4
  scl: GPIO5

sensor:
  - platform: bme280
    temperature:
      name: "${friendly_name} Temperature"
      web_server:
        ordering: 1
    pressure:
      name: "${friendly_name} Pressure"
      web_server:
        ordering: 2
    humidity:
      name: "${friendly_name} Humidity"
      web_server:
        ordering: 3

  - platform: template
    name: "${friendly_name} Dew Point"
    lambda: |-
      return id(dew_point_sensor).state;
    web_server:
      ordering: 4

  - platform: uptime
    name: "${friendly_name} Uptime"
    web_server:
      ordering: 10

  - platform: wifi_signal
    name: "${friendly_name} WiFi Signal"
    web_server:
      ordering: 11

text_sensor:
  - platform: wifi_info
    ip_address:
      name: "${friendly_name} IP Address"
      web_server:
        ordering: 0
    ssid:
      name: "${friendly_name} WiFi SSID"
      web_server:
        ordering: 12

button:
  - platform: restart
    name: "${friendly_name} Restart"
    web_server:
      ordering: 100

logger:
  level: INFO
```

## Web Server vs Home Assistant

| Feature | Web Server | Home Assistant API |
|---------|-----------|-------------------|
| Local Control | Yes | Via HA |
| Authentication | Basic | Encrypted |
| OTA Updates | Yes | Yes |
| Custom Pages | Limited | Via Lovelace |
| Auto-discovery | No | Yes |
| Mobile App | No | Yes |

## Best Practices

1. **Use authentication** - Always set username/password
2. **Set static IP** - Easier to access
3. **Enable OTA** - For firmware updates
4. **Organize entities** - Use web_server.ordering
5. **Combine with API** - Best of both worlds
6. **Limit exposure** - Don't expose to internet

## Troubleshooting

### Can't Access Web Server

```yaml
# Check IP address
text_sensor:
  - platform: wifi_info
    ip_address:
      name: "IP Address"

# Try IP instead of hostname
# http://192.168.1.100/
```

### Login Not Working

```yaml
# Check credentials in secrets
web_server:
  auth:
    username: admin  # Case sensitive
    password: !secret web_password
```

### Page Not Loading

1. Check device is online
2. Try different browser
3. Check firewall settings
4. Verify port (80 default)

### Slow Response

```yaml
# Reduce update intervals
sensor:
  - platform: bme280
    update_interval: 300s  # Slower = less CPU

# Disable logging in production
logger:
  level: WARN
```

## Web Server API Endpoints

```
GET  /                          - Dashboard
GET  /login                     - Login page
GET  /logout                    - Logout
GET  /json/info                 - Device info
GET  /json/sensors             - All sensors
GET  /json/sensor/:id          - Single sensor
POST /json/sensor/:id/set      - Set sensor value
GET  /json/switches            - All switches
POST /json/switch/:id/toggle   - Toggle switch
GET  /logs                      - Log viewer
```

## Security Considerations

```yaml
# Basic auth provides minimal security
# For internet exposure, use:

web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password
  # Add SSL
  ssl:
    certificate: !secret ssl_cert
    key: !secret ssl_key
  # Limit CORS
  cors: false
  # Or whitelist origins
  cors_allowed_origins:
    - https://your-domain.com
```

## Web Server for Development

```yaml
# Enable verbose logging
logger:
  level: DEBUG

# Enable all features
web_server:
  port: 80
  auth:
    username: admin
    password: admin
  # No encryption for local dev
```

## Web Server URL Access

```yaml
# Access values via URL
# http://device.local/json/sensor/temperature

text_sensor:
  - platform: template
    name: "Device URL"
    lambda: |-
      return "http://";
    update_interval: never
```
