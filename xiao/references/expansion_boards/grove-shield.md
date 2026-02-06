# Grove Shield for XIAO

## Overview

The Grove Shield for XIAO is a plug-and-play Grove extension board that bridges Seeed Studio XIAO with Seeed's Grove ecosystem. It features 8 Grove connectors (2x I2C, 1x UART), on-board lithium battery charging and management, all 14 GPIO pins broken out, and a breakable design for size-constrained projects (25×39mm after break-off).

## Hardware

### Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | Full support |
| ESP32C5 | ✅ | Full support |
| ESP32C6 | ✅ | Full support |
| ESP32S3 | ✅ | Full support |
| nRF52840 | ✅ | Full support |
| RP2040 | ✅ | Full support |
| RP2350 | ✅ | Full support |
| SAMD21 | ✅ | Full support |
| RA4M1 | ✅ | Full support |
| MG24 | ✅ | Full support |
| nRF54L15 | ✅ | Full support |

### Features

- **8 Grove Connectors**: 2x I2C, 1x UART, and pin access for more Grove connections
- **Battery Management**: On-board charging IC with JST 2.0mm battery connector
- **Power Switch**: Convenient power control
- **Charging Status LED**: Indicates charging state
- **All GPIO Broken Out**: All 14 XIAO pins accessible
- **Breakable Design**: Can be reduced to 25×39mm for compact projects
- **SPI Flash Bonding Pad**: Reserved for memory expansion (advanced users)

### Specifications

| Item | Value |
|------|-------|
| Operating Voltage | 3.3V / 3.7V Lithium Battery |
| Load Capacity | 800mA |
| Charging Current | 400mA (max) |
| Operating Temperature | -40°C to 85°C |
| Storage Temperature | -55°C to 150°C |
| Grove Interfaces | I2C × 2, UART × 1 |
| Dimensions (full) | 60×39mm |
| Dimensions (break-off) | 25×39mm |
| Weight (full) | 13g |
| Weight (break-off) | 10g |

### Pin Connections

| Grove Connector | XIAO Pins | Function |
|----------------|-----------|----------|
| Grove I2C #1 | D4 (SDA), D5 (SCL) | Default I2C |
| Grove I2C #2 | D4 (SDA), D5 (SCL) | Shared with #1 |
| Grove UART | D6 (TX), D7 (RX) | UART communication |
| GPIO Headers | All 14 pins | Full GPIO access |

**Pinout (all 14 pins broken out):**
- D0, D1, D2, D3, D4, D5, D6, D7, D8, D9, D10, D11, D12, D13 (varies by XIAO board)

**Power Pins:**
- 3V3, 5V, GND available on header

### Board Layout

```
+------------------------------------------+
|  [Power Switch]  [Charging LED]          |
|                                          |
|  [Grove I2C]  [Grove I2C]  [Grove UART]  |
|                                          |
|  [GPIO Header - All 14 Pins]             |
|                                          |
|  [BAT Connector]                          |
|                                          |
|  [XIAO Socket]                           |
|                                          |
|  [Break-off line] ═══════════════        |
+------------------------------------------+
```

**Break-off Section** (25×39mm):
- Can be separated for ultra-compact projects
- Reduces weight from 13g to 10g

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/grove-shield.md`
  - Grove sensor integration
  - I2C/UART communication examples

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## Power Options

1. **USB Power**: 5V via Type-C cable
2. **Battery Power**: 3.7V Li-Po battery via JST 2.0mm connector
3. **Auto-charging**: Battery charges when USB is connected and power switch is ON

**Charging Indicator:**
- LED ON = Charging
- LED OFF = Not charging (no battery or fully charged)

## Usage Notes

1. **Grove Ecosystem**: Connect any Grove sensor/actuator to appropriate connector
2. **I2C Sharing**: Both I2C connectors share the same bus (D4/D5)
3. **UART Communication**: Default UART on D6 (TX) and D7 (RX)
4. **Break-off**: Use PCB stamp holes to break off smaller section if needed
5. **Flash Memory**: SOIC8 bonding pad for advanced memory expansion

## Applications

- **Wearable Devices**: Battery-powered, compact form factor
- **Rapid Prototyping**: No soldering required for Grove modules
- **Sensor Testing**: Quick connection to Grove sensor ecosystem
- **Size-Constrained Projects**: Break-off design for ultra-compact builds
- **Battery-Powered Projects**: On-board charging and management

## Grove Ecosystem Compatibility

The Grove Shield works with hundreds of Grove modules including:

**Sensors:**
- Temperature & Humidity (DHT series, BME280)
- Motion (PIR, Accelerometer, Gyroscope)
- Light (Light sensor, UV sensor)
- Air Quality (Gas, Dust, CO2)

**Actuators:**
- Servo motors
- Relays
- Buzzer
- LED strips
- Displays (OLED, LCD)

**Communication:**
- UART GPS
- I2C RTC
- SPI SD card
- LoRa radio
- Bluetooth modules

## Troubleshooting

### Grove Sensor Not Detected

**Symptom**: I2C sensor not found

**Possible Causes:**
1. Wrong I2C address - Check sensor documentation
2. Loose connection - Verify Grove cable is seated
3. Power issue - Check power switch is ON

**Solution**: Run I2C scanner sketch, check connections

### Battery Not Charging

**Symptom**: Charging LED not lit

**Possible Causes:**
1. Power switch OFF - Turn switch to ON position
2. No battery connected - Verify JST connector is seated
3. Dead battery - Replace battery

**Solution**: Check power switch, verify battery connection

### GPIO Not Working

**Symptom**: Pin reads incorrect values

**Possible Causes:**
1. Wrong pin number - Check XIAO board pinout
2. Pin conflict - Verify pin is not used by Grove connector
3. Wrong mode - Set pinMode correctly

**Solution**: Verify pin mapping, check pinMode configuration

## Comparison with Expansion Base

| Feature | Grove Shield | Expansion Base |
|---------|-------------|----------------|
| Grove Connectors | 8 (2×I2C, 1×UART) | 4 (2×I2C, 1×UART, 1×A0/D0) |
| On-board Display | None | OLED (SSD1306) |
| RTC | None | PCF8563 with battery |
| SD Card | Via SPI header | Built-in slot |
| Buzzer | None | Built-in |
| Buttons | None | User + RESET |
| GPIO Access | All 14 pins broken out | All 14 pins broken out |
| Size | 60×39mm (25×39mm break) | 50×39mm |
| Best For | Grove ecosystem projects | All-in-one prototyping |

**Choose Grove Shield if:** You want maximum Grove connectivity or need ultra-compact size
**Choose Expansion Base if:** You want built-in display, RTC, and SD card for data logging
