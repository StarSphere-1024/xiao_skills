# UART Configuration

## Basic UART Setup

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600
```

## Board-Specific UART Pins

### ESP32C3

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600
```

### ESP32S3

```yaml
uart:
  tx_pin: GPIO17
  rx_pin: GPIO18
  baud_rate: 9600
```

### ESP32C6

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600
```

### RP2040

```yaml
uart:
  tx_pin: GPIO8
  rx_pin: GPIO9
  baud_rate: 9600
```

## UART with GPS

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600

gps:
  uart_id: uart_0
  update_interval: 1s
```

## UART with PM2.5 Sensor

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600

sensor:
  - platform: pmsx003
    type: PMS5003ST
    pm_1_0:
      name: "PM1.0"
    pm_2_5:
      name: "PM2.5"
    pm_10_0:
      name: "PM10"
```

## UART for RFID RC522

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600

rc522:
  uart_id: uart_0
  on_tag:
    - mqtt.publish:
        topic: rfid/tag
        payload: !lambda 'return x;'
```

## UART for SDM120 Meter

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 2400

sensor:
  - platform: sdm120
    uart_id: uart_0
    voltage:
      name: "Voltage"
    current:
      name: "Current"
```

## Multiple UART Ports

```yaml
# UART 0 for GPS
uart:
  id: uart_gps
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600

# UART 1 for sensor
uart:
  id: uart_sensor
  tx_pin: GPIO17
  rx_pin: GPIO18
  baud_rate: 9600

gps:
  uart_id: uart_gps

sensor:
  - platform: pmsx003
    uart_id: uart_sensor
```

## UART Debug Output

```yaml
# Enable debug output on UART
logger:
  baud_rate: 115200
  tx_pin: GPIO6
  rx_pin: GPIO7
```

## UART Baud Rates

Common baud rates:

| Value | Use Case |
|-------|----------|
| 9600 | GPS, RFID, sensors |
| 19200 | Some meters |
| 38400 | Some displays |
| 57600 | Fast sensors |
| 115200 | Debug, fast data |

## UART with Display

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
  baud_rate: 9600

display:
  - platform: nextion
    uart_id: uart_0
    tft_file: "nextion.tft"
```

## UART Text Sensor

```yaml
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7

text_sensor:
  - platform: uart
    uart_id: uart_0
    name: "UART Output"
```

## Best Practices

1. **Match baud rates** - Device and ESPHome must match
2. **Use correct pins** - TX connects to RX, RX to TX
3. **Ground devices** - Common ground required
4. **Enable logging** - During development to debug
5. **Test connections** - Verify with loopback test

## Troubleshooting

### No Data Received

1. Check TX/RX are crossed (TX→RX, RX→TX)
2. Verify baud rate matches device
3. Check ground connection
4. Enable logger to see UART activity

### Garbled Data

1. Wrong baud rate
2. Voltage mismatch (3.3V vs 5V)
3. Noise on line (add shielding)

### UART Conflicts

Don't use same pins for multiple UARTs:

```yaml
# WRONG - Same pins
uart:
  tx_pin: GPIO6
  rx_pin: GPIO7
uart:
  tx_pin: GPIO6  # Conflict!
  rx_pin: GPIO7

# CORRECT - Different pins or IDs
uart:
  id: uart_0
  tx_pin: GPIO6
  rx_pin: GPIO7
uart:
  id: uart_1
  tx_pin: GPIO17
  rx_pin: GPIO18
```
