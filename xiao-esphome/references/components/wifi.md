# WiFi Configuration (ESP32 Only)

## Basic WiFi Setup

```yaml
wifi:
  ssid: "YourWiFiSSID"
  password: "YourWiFiPassword"
```

## Using Secrets

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

Create `secrets.yaml`:
```yaml
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"
```

## Fast Connect

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true  # Skip full scan, use saved network
```

## Power Save Mode

```yaml
# Different power save modes
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  power_save_mode: none      # No power save (default)
  # power_save_mode: light   # Light power save
  # power_save_mode: high    # Maximum power save
```

## Manual IP (Static)

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    static_ip: 192.168.1.100
    gateway: 192.168.1.1
    subnet: 255.255.255.0
    dns1: 8.8.8.8
    dns2: 8.8.4.4
```

## Multiple WiFi Networks

```yaml
wifi:
  networks:
    - ssid: "Home"
      password: !secret home_password
    - ssid: "Work"
      password: !secret work_password
    - ssid: "Guest"
      password: !secret guest_password
```

## Hidden WiFi

```yaml
wifi:
  ssid: "HiddenNetwork"
  password: !secret wifi_password
  hidden: true
```

## Fallback Hotspot

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Create AP if WiFi fails
  ap:
    ssid: "XIAO Fallback"
    password: !secret ap_password

captive_portal:
```

## WiFi Reboot Timeout

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Reboot if not connected after 1 minute
  reboot_timeout: 1min
```

## Domain Name

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  domain: .local  # Use mDNS
```

## WiFi Power (Output)

```yaml
# Set WiFi output power (ESP32)
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  output_power: 20dB  # Max power

# Lower power for battery
wifi:
  output_power: 10dB
```

## WiFi Status Sensors

```yaml
sensor:
  # WiFi Signal
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s

  # WiFi BSSID
  text_sensor:
    - platform: wifi_info
      bssid:
        name: "Connected BSSID"

  # WiFi IP Address
  - platform: wifi_info
    ip_address:
      name: "IP Address"
```

## WiFi Troubleshooting

### Not Connecting

1. **Check credentials**: Verify SSID and password
2. **Enable fast_connect**: Skip scan
3. **Check 2.4GHz**: ESP32 doesn't support 5GHz
4. **Check signal strength**: Should be > -70dBm

### Frequent Disconnects

1. **Reduce power save mode**: Use `none` or `light`
2. **Increase reboot timeout**: Allow more time to connect
3. **Check interference**: Other 2.4GHz devices
4. **Update ESPHome**: Latest version has better WiFi stability

### Can't Find Device

1. **Check mDNS**: Ensure `.local` domain works
2. **Use static IP**: Easier to find
3. **Check router logs**: See if device connects
4. **Use serial monitor**: View connection logs

## WiFi with MQTT

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

mqtt:
  broker: !secret mqtt_broker
  username: !secret mqtt_user
  password: !secret mqtt_password
```

## WiFi with API

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

api:
  encryption:
    key: !secret api_encryption_key
  password: !secret api_password
```

## WiFi for Battery Powered Devices

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  power_save_mode: high
  reboot_timeout: 5min

# Use deep sleep
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s
```

## Best Practices

1. **Use secrets** for credentials
2. **Enable fast_connect** for quick connection
3. **Use static IP** for critical devices
4. **Enable captive_portal** for fallback AP
5. **Monitor WiFi signal** for placement
6. **Use 2.4GHz only** (ESP32 limitation)
7. **Adjust power_save_mode** for battery vs responsiveness
