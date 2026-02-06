# LED Driver Board for XIAO

## Overview

The LED Driver Board for XIAO is a smart module designed to control addressable RGB LED strips (NeoPixel WS2812, WS2813, WS2815, SK6812). It supports both 5V/3A and 12V/2A LED strips with built-in power regulation. When paired with XIAO ESP32 series, it enables smart control through WLED app and Home Assistant/ESPHome integration. Features Grove ecosystem compatibility for interactive lighting projects.

## Hardware

### Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | Full support, recommended for WLED |
| ESP32C5 | ✅ | Full support |
| ESP32C6 | ✅ | Full support |
| ESP32S3 | ✅ | Full support, WLED + ESPHome |
| nRF52840 | ✅ | Full support, use ezWS2812 library |
| RP2040 | ✅ | Full support |
| RP2350 | ✅ | Full support |
| SAMD21 | ✅ | Full support |
| RA4M1 | ✅ | Full support |
| MG24 | ⚠️ | Use ezWS2812 library (not standard NeoPixel) |
| nRF54L15 | ✅ | Full support |

### Features

- **Wide LED Compatibility**: 5V/3A or 12V/2A support
- **Addressable RGB Support**: WS2811, WS2812(B), WS2813, WS2815, SK6812
- **Grove Interface**: I2C connector (D4/D5) for 400+ Grove modules
- **User Button**: D3 for manual control
- **User Pin Header**: SPI ×1, UART ×1, Digital ×2
- **Smart Home Ready**: WLED + ESPHome with ESP32 series

### Specifications

| Item | Value |
|------|-------|
| Power Input | DC 12V/2A via 5.5×2.1mm barrel jack |
| LED Power Support | DC 12V or DC 5V |
| Max Operating Current | 12V/2A or 5V/3A |
| LED Connector | 4-pin 3.81mm screw terminal (12V, 5V, A0, GND) |
| Grove I2C Connector | D5, D4, 5V, GND |
| User Button | D3 |
| User Pin Header | SPI ×1, UART ×1, Digital ×2 |

### Pin Connections

| Function | XIAO Pin | Notes |
|----------|-----------|-------|
| LED Data | A0 (Grove) or D5 (I2C) | For addressable LEDs |
| Grove I2C SDA | D5 | Default Grove I2C |
| Grove I2C SCL | D4 | Default Grove I2C |
| User Button | D3 | GPIO input |
| SPI, UART, Digital | User pin header | Extended access |

**LED Terminal Block Connection:**
```
┌─────────────────────────┐
│ 12V | 5V | A0 | GND     │
│  VCC  VCC  DATA  GND    │
└─────────────────────────┘
```

**Important:**
- Use **12V** terminal for 12V LED strips (connect to 12V, not 5V)
- Use **5V** terminal for 5V LED strips
- **A0** is the data signal pin

### Supported LED Types

**5V LED Strips (connect to 5V terminal):**
- Grove RGB LED WS2813 Mini
- Grove RGB LED Stick (10/15/20 WS2813)
- Grove RGB LED Ring (16/20/24 WS2813)
- Grove Ultimate RGB LED Ring
- WS2813B RGB LED Flexi-Strip (30/60 LED/m)
- WS2813 RGB LED Strip Waterproof (30/60 LED/m)

**12V LED Strips (connect to 12V terminal):**
- Various 12V addressable LED strips

### Power Requirements

**By LED Type:**
- 5V LEDs: Use 5V output, max 3A
- 12V LEDs: Use 12V input/output, max 2A

**Power Supply:**
- Requires external 12V/2A power adapter
- 5.5×2.1mm barrel jack connector
- Voltage must match LED requirements

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/led-driver.md`
  - Adafruit NeoPixel library setup
  - Basic RGB control examples
  - Motion-color sync with IMU
  - WLED firmware for XIAO ESP32C3

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## Smart Home Integration

### WLED Support (ESP32 Series Only)

When using XIAO ESP32C3/C6/S3:
1. Install WLED firmware via [install.wled.me](https://install.wled.me/)
2. Control via web browser or WLED mobile app
3. Supports effects, colors, brightness, animations

**XIAO ESP32C3 highly recommended for WLED**

### Home Assistant + ESPHome (ESP32 Series Only)

When using XIAO ESP32C3/C6/S3:
1. Install ESPHome add-on in Home Assistant
2. Configure YAML for LED control
3. Add temperature sensors, motion sensors for interactive effects
4. Create automations and smart home scenarios

## Grove Ecosystem Compatibility

The Grove I2C connector (D4/D5) works with 400+ Grove modules:

**For AI Vision Detection:**
- Grove Vision AI Module V2
- Grove Smart IR Gesture Sensor (PAJ7660)

**For Temperature & Humidity:**
- Grove Temperature & Humidity Sensor (DHT11)
- Grove AHT20 I2C Industrial Grade
- Grove Temp and Humi Sensor (SHT31)

**For Motion Detection:**
- Grove PIR Motion Sensor
- Grove 3-Axis Analog Accelerometer
- Grove IMU 9DOF (ICM20600+AK09918)

## Usage Notes

1. **LED Voltage**: Match power supply voltage to LED requirements
2. **Data Pin**: Default is A0 (Grove connector) or D5 (I2C Grove)
3. **Power**: External 12V/2A power supply required
4. **MG24 Boards**: Use ezWS2812 library, not standard Adafruit NeoPixel
5. **WLED**: XIAO ESP32C3 recommended for WLED firmware
6. **Current Limits**: Do not exceed 3A at 5V or 2A at 12V

## Applications

- **Smart Lighting**: WiFi-controlled LED effects via WLED
- **Home Automation**: ESPHome integration with sensors
- **Ambient Lighting**: Motion-activated color changing
- **Architectural Lighting**: Color temperature adjustment
- **Entertainment**: Music-synced LED effects
- **Decor**: Holiday lighting, accent lighting

## Troubleshooting

### LEDs Not Lighting Up

**Symptom**: LED strip remains dark

**Possible Causes:**
1. No power - Check 12V power supply connected
2. Wrong voltage - Match supply to LED voltage (5V vs 12V)
3. Wrong data pin - Check A0 or D5 in code
4. Loose connection - Verify terminal block is tight

**Solution**: Check power supply, verify connections, check pin definitions

### Wrong Colors

**Symptom**: Colors don't match expected values

**Possible Causes:**
1. RGB order - Try GRB or RGB in code
2. Wrong LED type - Verify WS2812 vs WS2813 setting
3. Data corruption - Check signal quality

**Solution**: Change color order in NeoPixel constructor, verify LED type

### Flickering LEDs

**Symptom**: LEDs flicker randomly

**Possible Causes:**
1. Insufficient power - Use adequate power supply
2. Long data cable - Keep data line short (<1m)
3. Signal noise - Add capacitor near LED strip

**Solution**: Increase power supply capacity, shorten data line

### MG24 Not Working with NeoPixel Library

**Symptom**: Compilation or runtime errors on MG24

**Possible Causes:**
1. Wrong library - MG24 doesn't support standard NeoPixel

**Solution**: Use Silicon Labs ezWS2812 driver instead

## Supported LED Protocol

**1-Wire Protocol (NeoPixel):**
- Data: Single pin (A0 or D5)
- Timing: Critical (use dedicated library)
- Order: GRB for most WS2812/WS2813
- Voltage: 5V or 12V depending on LED type

**Compatible Libraries:**
- Adafruit NeoPixel (most boards)
- ezWS2812 (MG24 only)
- FastLED (alternative)

## Recommended Combinations

| Application | XIAO Board | Features |
|------------|------------|----------|
| WLED Control | ESP32C3 | Low cost, WiFi/BLE, adequate RAM |
| Home Assistant | ESP32S3 | More RAM, WiFi/BLE, camera option |
| Basic Control | nRF52840 | BLE, IMU for motion-color sync |
| Battery Powered | nRF52840 | Low power BLE applications |
| Cost-Optimized | RP2040/RP2350 | Low cost, good PWM |
