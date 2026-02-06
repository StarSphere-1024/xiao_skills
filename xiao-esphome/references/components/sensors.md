# ESPHome Sensors for XIAO

## BME280 (Temperature, Humidity, Pressure)

I2C environmental sensor.

```yaml
sensor:
  - platform: bme280
    i2c_id: bus_i2c
    address: 0x76
    temperature:
      name: "Temperature"
      id: bme280_temp
      oversampling: 16x
      filters:
        - sliding_window_moving_average:
            window_size: 5
            send_every: 1
    pressure:
      name: "Pressure"
      id: bme280_press
      oversampling: 16x
    humidity:
      name: "Humidity"
      id: bme280_hum
      oversampling: 16x
    update_interval: 60s

# Display value with unit
text_sensor:
  - platform: template
    name: "Temperature Formatted"
    lambda: |-
      return { std::to_string((int)id(bme280_temp).state) + "°C" };
    update_interval: 60s
```

## AHT20/AHT10 (Temperature, Humidity)

```yaml
sensor:
  - platform: aht20
    i2c_id: bus_i2c
    address: 0x38
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"
    update_interval: 60s
```

## DHT11/DHT22/AM2302

Digital temperature and humidity sensor.

```yaml
sensor:
  - platform: dht
    pin: GPIO2
    model: DHT22  # DHT11, DHT22, AM2302, RHT03
    temperature:
      name: "Temperature"
      id: dht_temp
    humidity:
      name: "Humidity"
      id: dht_hum
    update_interval: 60s
```

## DHT12 (I2C)

```yaml
sensor:
  - platform: dht12
    i2c_id: bus_i2c
    address: 0x5C
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"
    update_interval: 60s
```

## BMP280 (Temperature, Pressure)

```yaml
sensor:
  - platform: bmp280
    i2c_id: bus_i2c
    address: 0x77
    temperature:
      name: "Temperature"
      oversampling: 16x
    pressure:
      name: "Pressure"
      oversampling: 16x
    update_interval: 60s
```

## SHT30/SHT31/SHT35 (Temperature, Humidity)

```yaml
sensor:
  - platform: sht3xd
    i2c_id: bus_i2c
    address: 0x44
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"
    update_interval: 60s
```

## ADS1115 (16-bit ADC)

```yaml
sensor:
  - platform: ads1115
    multiplexer: 'A0_GND'
    gain: 4.096
    name: "ADS1115 Channel 0"
    update_interval: 1s
```

## MQ Sensors (Gas)

### MQ-2 (Smoke, LPG, CO)

```yaml
sensor:
  - platform: mq2
    pin: GPIO2
    name: "MQ2 Sensor"
    update_interval: 60s
```

### MQ-135 (Air Quality)

```yaml
sensor:
  - platform: mq135
    pin: GPIO2
    name: "MQ135 Air Quality"
    update_interval: 60s
```

## Light Sensors

### BH1750

```yaml
sensor:
  - platform: bh1750
    i2c_id: bus_i2c
    address: 0x23
    name: "Illuminance"
    update_interval: 60s
```

### TSL2561

```yaml
sensor:
  - platform: tsl2561
    i2c_id: bus_i2c
    address: 0x39
    name: "Illuminance"
    update_interval: 60s
```

## Ultrasonic Distance (HC-SR04)

```yaml
sensor:
  - platform: ultrasonic
    trigger_pin: GPIO2
    echo_pin: GPIO3
    name: "Distance"
    update_interval: 1s
```

## Soil Moisture (Capacitive)

```yaml
sensor:
  - platform: adc
    pin: GPIO2
    name: "Soil Moisture"
    update_interval: 60s
    unit_of_measurement: "%"
    accuracy_decimals: 0
    filters:
      - calibrate_linear:
          - 0.5 -> 100
          - 2.5 -> 0
```

## Hall Sensor (ESP32)

```yaml
sensor:
  - platform: esp32_hall
    name: "Hall Sensor"
    update_interval: 60s
```

## Internal Temperature (ESP32)

```yaml
sensor:
  - platform: esp32_temperature
    name: "Internal Temperature"
    update_interval: 60s
```

## Pulse Counter

```yaml
sensor:
  - platform: pulse_counter
    pin: GPIO2
    name: "Pulse Counter"
    update_interval: 60s
    filters:
      - multiply: 0.5  # Convert to flow rate
```

## Rot Encoder

```yaml
sensor:
  - platform: rotary_encoder
    pin_a: GPIO2
    pin_b: GPIO3
    name: "Rotary Encoder"
    resolution: 4
```

## WiFi Signal

```yaml
sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s
```

## Uptime

```yaml
sensor:
  - platform: uptime
    name: "Uptime"
    update_interval: 60s
```

## Template Sensor (Custom)

```yaml
sensor:
  - platform: template
    name: "Custom Sensor"
    lambda: |-
      return 42.0;
    unit_of_measurement: "°C"
    update_interval: 60s

  - platform: template
    name: "Average Temperature"
    lambda: |-
      return (id(sensor1).state + id(sensor2).state) / 2.0;
    update_interval: 60s
```

## Dew Point

```yaml
sensor:
  - platform: template
    name: "Dew Point"
    lambda: |-
      const float a = 17.27;
      const float b = 237.7;
      float temp = id(bme280_temp).state;
      float hum = id(bme280_hum).state;
      float alpha = ((a * temp) / (b + temp)) + log(hum / 100.0);
      return (b * alpha) / (a - alpha);
    unit_of_measurement: "°C"
    update_interval: 60s
```

## Absolute Humidity

```yaml
sensor:
  - platform: template
    name: "Absolute Humidity"
    lambda: |-
      float temp = id(bme280_temp).state;
      float hum = id(bme280_hum).state;
      float abs_hum = 216.7 * (hum / 100.0 * 6.112 * exp((17.62 * temp) / (243.12 + temp)) / (273.15 + temp));
      return abs_hum;
    unit_of_measurement: "g/m³"
    update_interval: 60s
```

## Filters

```yaml
sensor:
  - platform: bme280
    temperature:
      name: "Temperature"
      filters:
        # Sliding window average
        - sliding_window_moving_average:
            window_size: 5
            send_every: 1

        # Offset
        - offset: -2.0

        # Multiply
        - multiply: 1.8

        # Calibration
        - calibrate_linear:
            - 0.0 -> 0.0
            - 100.0 -> 100.0

        # Throttle
        - throttle: 10s

        # Debounce
        - debounce: 0.5s

        # Filter out outliers
        - outlier_window_size: 5
          outlier_percentile: 10
```

## Multiple Sensors Example

```yaml
# XIAO Weather Station with multiple sensors
sensor:
  - platform: bme280
    i2c_id: bus_i2c
    temperature:
      name: "BME Temperature"
    pressure:
      name: "Pressure"
    humidity:
      name: "Humidity"

  - platform: dht
    pin: GPIO2
    temperature:
      name: "DHT Temperature"
    humidity:
      name: "DHT Humidity"

  - platform: ads1115
    multiplexer: 'A0_GND'
    name: "Light Level"

  - platform: template
    name: "Temperature Difference"
    lambda: |-
      return id(bme280_temp).state - id(dht_temp).state;
```

## Publish to Home Assistant

Sensors are automatically exposed to Home Assistant when using the `api:` component.

### Automations in Home Assistant

```yaml
# Home Assistant automation.yaml
- alias: "High temperature alert"
  trigger:
    - platform: numeric_state
      entity_id: sensor.xiao_temperature
      above: 30
  action:
    - service: notify.mobile_app
      data:
        message: "Temperature is above 30°C!"
```

## Best Practices

1. **Use appropriate update intervals** - Don't poll too frequently
2. **Enable filters** - Smooth out sensor readings
3. **Use unique IDs** - For consistent entity IDs
4. **Calibrate sensors** - Use calibration filters
5. **Add error handling** - Check sensor validity

## Troubleshooting

### I2C Sensor Not Found

```yaml
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true  # Enable scan to detect devices
```

### ADC Readings Wrong

```yaml
sensor:
  - platform: adc
    pin: GPIO2
    attenuation: 11db  # Set for 3.3V range
```

### Sensor Drifting

```yaml
filters:
  - sliding_window_moving_average:
      window_size: 10
      send_every: 5
```
