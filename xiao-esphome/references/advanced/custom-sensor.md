# Custom Sensors in ESPHome

## Overview

Create custom sensors using C++ lambda functions or full custom components when ESPHome's built-in sensors don't meet your needs.

## Lambda Sensors

### Basic Template Sensor

```yaml
sensor:
  - platform: template
    name: "Custom Sensor"
    lambda: |-
      return 42.0;
    unit_of_measurement: "units"
    accuracy_decimals: 1
    update_interval: 60s
```

### Mathematical Operations

```yaml
sensor:
  # Input sensors
  - platform: dht
    pin: GPIO2
    temperature:
      name: "Temperature"
      id: temp_f
    humidity:
      name: "Humidity"
      id: hum_f

  # Convert Fahrenheit to Celsius
  - platform: template
    name: "Temperature Celsius"
    lambda: |-
      return (id(temp_f).state - 32.0) * 5.0 / 9.0;
    unit_of_measurement: "°C"
    update_interval: 60s

  # Calculate heat index
  - platform: template
    name: "Heat Index"
    lambda: |-
      float t = id(temp_f).state;
      float h = id(hum_f).state;
      float c1 = -42.379;
      float c2 = 2.04901523;
      float c3 = 10.14333127;
      float c4 = -0.22475541;
      float c5 = -6.83783e-3;
      float c6 = -5.481717e-2;
      float c7 = 1.22874e-3;
      float c8 = 8.5282e-4;
      float c9 = -1.99e-6;

      float hi = c1 + (c2 * t) + (c3 * h) + (c4 * t * h) +
                  (c5 * t * t) + (c6 * h * h) +
                  (c7 * t * t * h) + (c8 * t * h * h) +
                  (c9 * t * t * h * h);
      return hi;
    unit_of_measurement: "°F"
    update_interval: 60s
```

### Average Multiple Sensors

```yaml
sensor:
  - platform: bme280
    temperature:
      name: "Inside Temperature"
      id: temp_in

  - platform: homeassistant
    entity_id: sensor.outside_temperature
    id: temp_out

  - platform: template
    name: "Average Temperature"
    lambda: |-
      return (id(temp_in).state + id(temp_out).state) / 2.0;
    unit_of_measurement: "°C"
    update_interval: 300s
```

### Conditional Sensor

```yaml
sensor:
  - platform: template
    name: "Alert Status"
    lambda: |-
      if (id(temp_sensor).state > 30.0) {
        return 1.0;  // Alert
      }
      return 0.0;  // Normal
    update_interval: 10s
```

### Sensor with Hysteresis

```yaml
sensor:
  - platform: template
    name: "Thermostat State"
    lambda: |-
      static float last_state = 0.0;
      float temp = id(temp_sensor).state;
      float setpoint = id(setpoint).state;
      float hysteresis = 0.5;

      if (temp > setpoint + hysteresis) {
        last_state = 0.0;  // Cooling needed
      } else if (temp < setpoint - hysteresis) {
        last_state = 1.0;  // Heating needed
      }
      return last_state;
    update_interval: 1s
```

## Custom C++ Sensor

### Full Custom Sensor Class

```yaml
# Register custom sensor
esphome:
  on_boot:
    then:
      - lambda: |-
          auto custom_sensor = new MyCustomSensor();
          App.register_sensor(custom_sensor);

sensor:
  - platform: custom
    lambda: |-
      auto my_sensor = new MyCustomSensor();
      App.register_sensor(my_sensor);
      return {my_sensor};

    sensors:
      - name: "My Custom Sensor"
        unit_of_measurement: "x"
        accuracy_decimals: 2
```

### Custom Sensor with Header File

```yaml
esphome:
  includes:
    - custom_sensors.h

sensor:
  - platform: custom
    lambda: |-
      return {my_custom_sensor};

    sensors:
      name: "Custom Sensor"
```

Create `custom_sensors.h`:
```cpp
#pragma once

#include "esphome.h"
#include "esphome/components/sensor/sensor.h"

class MyCustomSensor : public esphome::sensor::Sensor, public esphome::Component {
 public:
  void setup() override {
    // Initialization
  }

  void loop() override {
    // Read sensor
    float value = read_my_sensor();
    publish_state(value);
  }

 private:
  float read_my_sensor() {
    // Your sensor reading code
    return 42.0;
  }
};

static MyCustomSensor *my_custom_sensor = new MyCustomSensor();
```

## GPIO-Based Custom Sensors

### Pulse Counter Sensor

```yaml
sensor:
  - platform: pulse_counter
    pin: GPIO2
    name: "Pulse Counter"
    update_interval: 60s
    filters:
      - multiply: 0.5  # Convert to flow rate
    unit_of_measurement: "L/min"
```

### Duty Cycle Sensor

```yaml
sensor:
  - platform: duty_cycle
    pin: GPIO2
    name: "PWM Duty Cycle"
    update_interval: 1s
```

## ADC-Based Custom Sensors

### Voltage Divider

```yaml
sensor:
  - platform: adc
    pin: GPIO0
    name: "Battery Voltage"
    attenuation: 11db
    filters:
      - calibrate_linear:
          - 0.0 -> 0.0
          - 3.3 -> 4.2  # ADC to actual voltage
      - sliding_window_moving_average:
          window_size: 10
          send_every: 5
    update_interval: 60s
    unit_of_measurement: "V"
```

### Current Sensor (ACS712)

```yaml
sensor:
  - platform: adc
    pin: GPIO0
    name: "Current"
    attenuation: 11db
    filters:
      - calibrate_linear:
          - 0.0 -> 2.5    # 0A = 2.5V
          - 3.3 -> 0.5    # Max current
      - multiply: 5.0    # Scale to amps
      - offset: -2.5     # Remove offset
    update_interval: 1s
    unit_of_measurement: "A"
```

### Light Sensor (Photoresistor)

```yaml
sensor:
  - platform: adc
    pin: GPIO0
    name: "Light Level"
    attenuation: 11db
    filters:
      - calibrate_linear:
          - 0.0 -> 100.0
          - 3.3 -> 0.0
    update_interval: 5s
    unit_of_measurement: "%"
```

## UART Custom Sensors

### Decode Protocol

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600

sensor:
  - platform: custom
    lambda: |-
      auto uart_sensor = new UARTSensor();
      App.register_sensor(uart_sensor);
      return {uart_sensor};

    sensors:
      name: "UART Sensor"
```

## I2C Custom Sensors

### Read I2C Register

```yaml
sensor:
  - platform: template
    name: "I2C Register Value"
    lambda: |-
      uint8_t data[2];
      id(i2c_bus).readfrom_mem(0x40, 0x00, data, 2);
      uint16_t value = (data[0] << 8) | data[1];
      return value;
    update_interval: 5s
```

## Filtered Sensors

### Median Filter

```yaml
sensor:
  - platform: dht
    pin: GPIO2
    temperature:
      name: "Temperature Raw"
      id: temp_raw
      filters:
        - median:
            window_size: 3
            send_every: 1
            send_first_at: 1

  - platform: template
    name: "Temperature Filtered"
    lambda: |-
      return id(temp_raw).state;
    update_interval: 60s
```

### Outlier Filter

```yaml
sensor:
  - platform: template
    name: "Filtered Temperature"
    lambda: |-
      return id(temp_sensor).state;
    filters:
      - outlier_window_size: 5
        outlier_percentile: 10
    update_interval: 10s
```

### Throttle Filter

```yaml
sensor:
  - platform: template
    name: "Throttled Sensor"
    lambda: |-
      return id(raw_sensor).state;
    filters:
      - throttle: 60s  # Only send every 60s
    update_interval: 1s
```

## Complex Example: Weather Calculations

```yaml
sensor:
  # Base sensors
  - platform: bme280
    temperature:
      name: "Temperature"
      id: temperature
    humidity:
      name: "Humidity"
      id: humidity
    pressure:
      name: "Pressure"
      id: pressure

  # Dew point
  - platform: template
    name: "Dew Point"
    id: dew_point
    lambda: |-
      const float a = 17.27;
      const float b = 237.7;
      float temp = id(temperature).state;
      float hum = id(humidity).state;
      float alpha = ((a * temp) / (b + temp)) + log(hum / 100.0);
      return (b * alpha) / (a - alpha);
    unit_of_measurement: "°C"
    update_interval: 60s

  # Absolute humidity
  - platform: template
    name: "Absolute Humidity"
    lambda: |-
      float temp = id(temperature).state;
      float hum = id(humidity).state;
      float abs_hum = 216.7 * (hum / 100.0 * 6.112 * exp((17.62 * temp) / (243.12 + temp)) / (273.15 + temp));
      return abs_hum;
    unit_of_measurement: "g/m³"
    update_interval: 60s

  # Vapor pressure deficit
  - platform: template
    name: "VPD"
    lambda: |-
      float temp = id(temperature).state;
      float hum = id(humidity).state;
      float dew = id(dew_point).state;
      float svp = 6.1078 * exp((17.27 * temp) / (temp + 237.3));
      float avp = 6.1078 * exp((17.27 * dew) / (dew + 237.3));
      return svp - avp;
    unit_of_measurement: "kPa"
    update_interval: 60s

  # Pressure trend
  - platform: template
    name: "Pressure Trend"
    lambda: |-
      static float pressures[3] = {0, 0, 0};
      static int index = 0;

      pressures[index] = id(pressure).state;
      index = (index + 1) % 3;

      if (pressures[0] < pressures[1] && pressures[1] < pressures[2])
        return 1.0;  // Rising
      else if (pressures[0] > pressures[1] && pressures[1] > pressures[2])
        return -1.0;  // Falling
      else
        return 0.0;  // Stable
    update_interval: 300s
```

## Best Practices

1. **Use template sensors** - For simple calculations
2. **Cache values** - Use `id()` for efficiency
3. **Add units** - Always specify unit_of_measurement
4. **Set accuracy** - Match sensor precision
5. **Filter appropriately** - Smooth noisy readings
6. **Test thoroughly** - Verify edge cases
7. **Document logic** - Add comments explaining math

## Troubleshooting

### Sensor Not Updating

```yaml
# Check update_interval
sensor:
  - platform: template
    name: "My Sensor"
    lambda: |-
      return 42.0;
    update_interval: 60s  # Must be set
```

### Wrong Values

```yaml
# Add logging
sensor:
  - platform: template
    name: "Debug Sensor"
    lambda: |-
      ESP_LOGI("custom", "Input: %f", id(input_sensor).state);
      float result = id(input_sensor).state * 2.0;
      ESP_LOGI("custom", "Output: %f", result);
      return result;
```

### Compilation Errors

```yaml
# Check C++ syntax
# Use proper semicolons
# Include headers if needed
esphome:
  includes:
    - my_sensor.h
```

## Advanced: State Machine Sensor

```yaml
sensor:
  - platform: template
    name: "System State"
    lambda: |-
      static enum { NORMAL, WARNING, CRITICAL } state = NORMAL;
      static uint32_t last_change = 0;

      float temp = id(temp_sensor).state;

      // State transitions
      switch (state) {
        case NORMAL:
          if (temp > 25.0) {
            state = WARNING;
            last_change = millis();
          }
          break;
        case WARNING:
          if (temp > 30.0) {
            state = CRITICAL;
            last_change = millis();
          } else if (temp < 23.0) {
            state = NORMAL;
            last_change = millis();
          }
          break;
        case CRITICAL:
          if (temp < 28.0) {
            state = WARNING;
            last_change = millis();
          }
          break;
      }

      return (float)state;
    update_interval: 1s
```

## Performance Tips

1. **Minimize lambda complexity** - Keep calculations simple
2. **Use appropriate intervals** - Don't poll too fast
3. **Cache calculations** - Store intermediate results
4. **Avoid blocking calls** - Keep loop() fast
5. **Use filters** - Let ESPHome handle smoothing
