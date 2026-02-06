# I2C Bus Configuration

## Basic I2C Setup

```yaml
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true
  frequency: 100kHz
```

## Board-Specific I2C Pins

### ESP32C3
```yaml
i2c:
  sda: GPIO4
  scl: GPIO5
```

### ESP32S3
```yaml
i2c:
  sda: GPIO7
  scl: GPIO6
```

### ESP32C6
```yaml
i2c:
  sda: GPIO4
  scl: GPIO5
```

### RP2040
```yaml
i2c:
  sda: GPIO6
  scl: GPIO7
```

## Multiple I2C Buses

```yaml
# I2C Bus 0
i2c:
  id: bus_i2c_0
  sda: GPIO4
  scl: GPIO5
  scan: true

# I2C Bus 1
i2c:
  id: bus_i2c_1
  sda: GPIO6
  scl: GPIO7
  scan: true

# Use with sensor
sensor:
  - platform: bme280
    i2c_id: bus_i2c_0
    # ...
```

## I2C Scan

```yaml
# Scan and display all I2C devices
i2c:
  sda: GPIO4
  scl: GPIO5
  scan: true  # Enable scan
  scan_period: 60s
```

Check logs for detected devices:
```
[I2C] Found i2c device at address 0x76
[I2C] Found i2c device at address 0x3C
```

## I2C Frequency Options

```yaml
# Standard speed (100kHz)
i2c:
  sda: GPIO4
  scl: GPIO5
  frequency: 100kHz

# Fast speed (400kHz)
i2c:
  sda: GPIO4
  scl: GPIO5
  frequency: 400kHz

# Custom frequency
i2c:
  sda: GPIO4
  scl: GPIO5
  frequency: 50kHz  # Slower for long wires
```

## I2C Device Addresses

### Common Sensor Addresses

| Sensor | Address |
|--------|---------|
| BME280 | 0x76 or 0x77 |
| BMP280 | 0x76 or 0x77 |
| AHT20 | 0x38 |
| AHT10 | 0x38 |
| DHT12 | 0x5C |
| SHT30/31/35 | 0x44 or 0x45 |
| BH1750 | 0x23 |
| TSL2561 | 0x39, 0x49, or 0x29 |
| SSD1306 | 0x3C or 0x3D |
| SH1106 | 0x3C or 0x3D |

## Troubleshooting I2C

### No Devices Found

1. **Check wiring**: SDA to SDA, SCL to SCL
2. **Check pull-up resistors**: 4.7kΩ to 3.3V
3. **Verify addresses**: Try 0x76 and 0x77 for BME280
4. **Enable scan**: See logs for detected addresses

### I2C Errors

```yaml
# Reduce frequency for long wires
i2c:
  sda: GPIO4
  scl: GPIO5
  frequency: 50kHz  # Slower

# Add delay
i2c:
  sda: GPIO4
  scl: GPIO5
  frequency: 100kHz
  delay: 10ms
```

### Multiple Devices

```yaml
# Use different addresses if available
sensor:
  - platform: bme280
    address: 0x76
    # ...

  - platform: bme280
    address: 0x77
    # ...
```

## I2C Component Examples

### BME280
```yaml
sensor:
  - platform: bme280
    i2c_id: bus_i2c
    address: 0x76
    temperature:
      name: "Temperature"
```

### SSD1306 Display
```yaml
display:
  - platform: ssd1306_i2c
    i2c_id: bus_i2c
    address: 0x3C
    model: "SSD1306 128x64"
```

### TSL2561 Light Sensor
```yaml
sensor:
  - platform: tsl2561
    i2c_id: bus_i2c
    address: 0x39
    name: "Illuminance"
```

### AHT20
```yaml
sensor:
  - platform: aht20
    i2c_id: bus_i2c
    address: 0x38
    temperature:
      name: "Temperature"
    humidity:
      name: "Humidity"
```

## Best Practices

1. **Enable scan** during development
2. **Use appropriate frequency** (100kHz standard)
3. **Check pull-up resistors** (4.7kΩ typical)
4. **Verify addresses** in datasheet
5. **Keep wires short** (<30cm) for fast speeds
6. **Test each device** individually first
7. **Use separate I2C buses** if many devices
