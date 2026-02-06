# SPI Bus Configuration

## Basic SPI Setup

```yaml
spi:
  clk_pin: GPIO8
  mosi_pin: GPIO10
  miso_pin: GPIO9

# Use with sensor
sensor:
  - platform: xxx
    spi_id: bus_spi
    cs_pin: GPIO2
```

## Board-Specific SPI Pins

### ESP32C3
```yaml
spi:
  clk_pin: GPIO20
  mosi_pin: GPIO10
  miso_pin: GPIO9
```

### ESP32S3
```yaml
spi:
  clk_pin: GPIO8
  mosi_pin: GPIO10
  miso_pin: GPIO9
```

### RP2040
```yaml
spi:
  clk_pin: GPIO2
  mosi_pin: GPIO3
  miso_pin: GPIO4
```

## Multiple SPI Buses

```yaml
# SPI Bus 0
spi:
  id: bus_spi_0
  clk_pin: GPIO8
  mosi_pin: GPIO10
  miso_pin: GPIO9

# SPI Bus 1
spi:
  id: bus_spi_1
  clk_pin: GPIO18
  mosi_pin: GPIO19
  miso_pin: GPIO20
```

## SPI Frequency

```yaml
spi:
  clk_pin: GPIO8
  mosi_pin: GPIO10
  miso_pin: GPIO9
  frequency: 1MHz  # Default 1MHz

# High speed
spi:
  frequency: 8MHz

# Low speed
spi:
  frequency: 100kHz
```

## SPI Components

### W5500 Ethernet

```yaml
ethernet:
  type: W5500
  cs_pin: GPIO2
  interrupt_pin: GPIO3
  spi_id: bus_spi
  clock_pin: GPIO8
  mosi_pin: GPIO10
  miso_pin: GPIO9
```

### ILI9341 Display

```yaml
display:
  - platform: ili9341
    spi_id: bus_spi
    cs_pin: GPIO2
    dc_pin: GPIO3
    reset_pin: GPIO4
    model: "TFT 2.4"
```

### MAX7219 7-Segment

```yaml
display:
  - platform: max7219
    spi_id: bus_spi
    cs_pin: GPIO2
    num_chips: 4
```

### SD Card

```yaml
sd_card:
  cs_pin: GPIO2
  spi_id: bus_spi
  clk_pin: GPIO8
  mosi_pin: GPIO10
  miso_pin: GPIO9
```

## SPI Chip Select

Multiple devices on same SPI bus, different CS pins:

```yaml
spi:
  clk_pin: GPIO8
  mosi_pin: GPIO10
  miso_pin: GPIO9

# Device 1
sensor:
  - platform: xxx
    spi_id: bus_spi
    cs_pin: GPIO2

# Device 2
display:
  - platform: yyy
    spi_id: bus_spi
    cs_pin: GPIO3
```

## Troubleshooting SPI

### Communication Errors

1. **Check wiring**: MOSI to MOSI, MISO to MISO, SCK to SCK
2. **Verify CS pin**: Each device needs unique CS
3. **Reduce frequency**: Try 100kHz
4. **Check voltage levels**: 3.3V or 5V

### Multiple Devices

```yaml
# Ensure different CS pins
# Device 1: CS on GPIO2
# Device 2: CS on GPIO3
# Device 3: CS on GPIO4
```

### SPI Speed Issues

```yaml
# Reduce speed for long wires
spi:
  frequency: 100kHz  # Slow down
```

## Best Practices

1. **Short wires** for high-speed SPI
2. **Unique CS pins** for each device
3. **Start with 1MHz** then increase
4. **Test individually** before combining
5. **Use decoupling capacitors** near devices
