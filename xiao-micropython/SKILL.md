---
name: xiao-micropython
description: Complete MicroPython development for XIAO boards. Use when writing MicroPython scripts for XIAO (ESP32C3/C5/C6/S3, RP2040/RP2350, nRF52840, nRF54L15). Requires /xiao skill for board-specific pin definitions. Covers firmware flashing, ESP32/network modules (network, WiFi, BLE, MQTT), machine modules (Pin, I2C, SPI, UART, PWM, ADC), common libraries (urequests, umqtt, ssd1306, bme280, neopixel), and project examples.
---

# XIAO MicroPython Development

## Overview

Complete MicroPython development guide for SeeedStudio XIAO series. This skill works with the `/xiao` skill - use `/xiao` for pin definitions, `/xiao-micropython` for MicroPython-specific APIs and libraries.

## Prerequisites

1. **Pin Definitions**: Read `/xiao` skill first for board-specific pin mappings
2. **Thonny IDE**: Recommended for MicroPython development
3. **Firmware**: MicroPython firmware flashed to board
4. **Code Verification**: Install `mpy-cross` for fast offline syntax checking (recommended)

### Environment Setup

For first-time MicroPython environment setup, see `setup/`:
- **Thonny IDE Installation**: `setup/thonny-ide.md`
- **Firmware Flashing**: `setup/esptool.md`
- **Code Verification**: `setup/mpy-cross.md` (recommended for syntax validation)

## Supported Boards

| Board | Firmware Download | Getting Started |
|-------|-------------------|-----------------|
| ESP32C3 | [micropython.org](https://micropython.org/download/ESP32_GENERIC_C3/) | `getting-started/esp32c3.md` |
| ESP32C5 | ESP32 generic (C3 works) | `getting-started/esp32c5.md` |
| ESP32C6 | [micropython.org](https://micropython.org/download/ESP32_GENERIC_C6/) | `getting-started/esp32c6.md` |
| ESP32S3 | [micropython.org](https://micropython.org/download/ESP32_GENERIC_S3/) | `getting-started/esp32s3.md` |
| RP2040 | [micropython.org](https://micropython.org/download/RPI_PICO/) | `getting-started/rp2040.md` |
| RP2350 | [micropython.org](https://micropython.org/download/RPI_PICO2/) | `getting-started/rp2350.md` |
| nRF52840 | Community port | `getting-started/nrf52840.md` |
| nRF54L15 | Seeed port | `getting-started/nrf54l15.md` |

**Note**: SAMD21, MG24 MicroPython support is limited or requires community forks.

## MicroPython Project Structure (CRITICAL)

**MicroPython projects typically use these file patterns:**

1. **`main.py`** - Main application script (auto-runs on boot)
2. **`boot.py`** - Initialization script (runs before main.py)
3. **`config.py`** - Configuration constants and settings
4. **`lib/`** - Custom library modules

```
MicroPythonProject/
├── main.py           # Main application (auto-runs)
├── boot.py           # WiFi/initialization
├── config.py         # Pin definitions, settings
└── lib/
    ├── wifi.py       # WiFi connection module
    └── sensor.py     # Sensor driver module
```

## Code Generation Workflow (CRITICAL)

**When a user requests MicroPython code generation, you MUST:**

### Step 1: Create Project Files
- Use the `Write` tool to create the actual `.py` file(s)
- Follow the MicroPython project structure shown above
- Example: Create `xiao_expansion_base_rtc.py` or a full project with `main.py`, `config.py`, etc.

### Step 2: Verify Syntax (If mpy-cross is Available)
- After creating the file, run `mpy-cross` to verify syntax
- This catches errors without needing a physical device
- Report syntax validation results to the user

## Quick Start

### 1. Flash Firmware

```bash
pip install esptool
esptool.py --chip esp32c3 --port COM3 write_flash -z 4MB 0 firmware.bin
```

### 2. Connect with Thonny

1. Open Thonny IDE
2. Tools > Options > Interpreter > MicroPython (ESP32)
3. Select port and connect

### 3. First Script

```python
from machine import Pin
import time

led = Pin(10, Pin.OUT)  # D10

while True:
    led.value(1)
    time.sleep(1)
    led.value(0)
    time.sleep(1)
```

## Machine Module Quick Reference

| Module | Reference | Usage |
|--------|-----------|-------|
| GPIO (machine.Pin) | `api/gpio.md` | Digital I/O, interrupts |
| PWM (machine.PWM) | `api/pwm.md` | Analog output, servo control |
| ADC (machine.ADC) | `api/adc.md` | Analog input reading |
| I2C (machine.I2C) | `api/i2c.md` | Sensor communication |
| SPI (machine.SPI) | `api/spi.md` | High-speed peripherals |
| UART (machine.UART) | `api/uart.md` | Serial communication |

### GPIO Example

```python
from machine import Pin

led = Pin(10, Pin.OUT)
led.on()           # Turn on
led.off()          # Turn off
led.toggle()       # Toggle

button = Pin(6, Pin.IN, Pin.PULL_UP)
if button.value() == 0:
    print("Pressed")
```

### I2C Example

```python
from machine import I2C, Pin

i2c = I2C(0, scl=Pin(1), sda=Pin(0), freq=100000)
devices = i2c.scan()
print(f"I2C devices: {[hex(d) for d in devices]}")
```

## Network Modules (ESP32)

| Module | Reference | Usage |
|--------|-----------|-------|
| WiFi (network.WLAN) | `api/wifi.md` | Wireless connectivity |
| HTTP (urequests) | `api/http.md` | Web requests |
| MQTT (umqtt) | `api/mqtt.md` | IoT messaging |
| BLE (bluetooth) | `api/ble.md` | Bluetooth Low Energy |
| Socket | `api/socket.md` | TCP/UDP networking |

### WiFi Quick Start

```python
import network

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    time.sleep(1)

print(f'IP: {wlan.ifconfig()[0]}')
```

### HTTP Request

```python
import urequests

response = urequests.get('http://httpbin.org/get')
print(response.text)
response.close()
```

## Common Libraries

| Library | Reference | Usage |
|---------|-----------|-------|
| SSD1306 | `libraries/ssd1306.md` | OLED displays |
| BME280 | `libraries/bme280.md` | Weather sensors |
| NeoPixel | `libraries/neopixel.md` | RGB LEDs |
| DHT | `libraries/dht.md` | Temp/Humidity sensors |
| SD (sdcard) | `libraries/sdcard.md` | SD card storage |

### SSD1306 OLED

```python
from machine import Pin, I2C
import ssd1306

i2c = I2C(0, sda=Pin(0), scl=Pin(1))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)
oled.fill(0)
oled.text("Hello XIAO!", 0, 0)
oled.show()
```

## Code Verification

**Quick syntax validation (no device needed):**

```bash
# Install mpy-cross
pip install mpy-cross

# Verify syntax
mpy-cross script.py
# ✅ No error = valid, ❌ SyntaxError = fix code
```

**Device testing (optional):**

```bash
# Install mpremote
pip install mpremote

# Test on device
mpremote run script.py
```

For detailed verification workflow, see `setup/mpy-cross.md` and `setup/mpremote.md`.

## Board-Specific APIs

### ESP32 Series (C3/C5/C6/S3)
- **Deep Sleep**: `api/esp32-sleep.md`
- **BLE**: `api/esp32-ble.md`
- **Camera (S3 Sense)**: `api/esp32s3-camera.md`

### nRF52 Series (nRF52840)
- **Sleep**: `api/nrf-sleep.md`
- **BLE**: `api/nrf-ble.md`

### RP Series (RP2040/RP2350)
- **Sleep**: `api/rp-sleep.md`
- **PIO**: `api/rp2040-pio.md`

## Expansion Boards

XIAO expansion boards add peripherals. For hardware specs, see `/xiao/references/expansion_boards/`.

| Expansion Board | Key Features | Reference |
|----------------|--------------|-----------|
| Expansion Base | OLED, RTC, SD card, buzzer | `expansion-boards/expansion-base.md` |
| Round Display | 1.28" touchscreen | `expansion-boards/round-display.md` |
| Grove Shield | 8 Grove connectors | `expansion-boards/grove-shield.md` |
| CAN Bus | MCP2515 controller | `expansion-boards/can-bus.md` |
| GPS/GNSS | L76K GNSS module | `expansion-boards/gps-gnss.md` |

## Expansion Board Examples

### Expansion Base (OLED + RTC)

```python
from machine import Pin, I2C
import ssd1306
import time

# OLED (I2C address 0x3C)
i2c = I2C(0, sda=Pin(0), scl=Pin(1))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

oled.fill(0)
oled.text("XIAO Base", 0, 0)
oled.show()

# RTC (DS3231, I2C address 0x68)
def set_rtc(datetime_tuple):
    i2c.writeto(0x68, bytes([0x00] + list(datetime_tuple)))

def get_rtc():
    i2c.writeto(0x68, b'\x00')
    data = i2c.readfrom(0x68, 7)
    return (data[0], data[1], data[2])

# SD Card (SPI)
import sdcard, machine
sd = sdcard.SDCard(machine.SPI(0), machine.Pin(4))
vfs = os.VfsFat(sd)
os.mount(vfs, '/sd')

with open('/sd/data.txt', 'w') as f:
    f.write('Hello from XIAO!')
```

## Code Patterns

### Main Loop with Exception Handler

```python
import time

def main():
    while True:
        try:
            # Your code here
            time.sleep(1)
        except Exception as e:
            print(f"Error: {e}")
            time.sleep(5)

if __name__ == "__main__":
    main()
```

### Non-blocking Delay

```python
import time

last_check = 0
interval = 5000  # 5 seconds

while True:
    current = time.ticks_ms()
    if time.ticks_diff(current, last_check) >= interval:
        last_check = current
        # Do periodic task

    # Other code here
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `OSError: [Errno 5]` | Check I2C/SPI pins with `/xiao` skill |
| `ImportError` | Verify module exists in firmware |
| WiFi won't connect | Ensure 2.4GHz network (ESP32 only) |
| Board won't upload | ESP32: Hold BOOT; RP2040: Hold BOOTSEL |

## Resources

### setup/
- `setup/thonny-ide.md` - Thonny IDE installation
- `setup/esptool.md` - Firmware flashing
- `setup/mpy-cross.md` - Offline syntax verification
- `setup/mpremote.md` - Device testing and deployment

### getting-started/
Board-specific setup instructions.

### api/
MicroPython module documentation (GPIO, I2C, WiFi, BLE, sleep, etc.)

### libraries/
Common libraries (SSD1306, BME280, NeoPixel, etc.)

### examples/
Complete project examples:
- `examples/weather-station.md` - BME280 weather station with MQTT
- `examples/ble-peripheral.md` - BLE environmental sensor
- `examples/mqtt-sensor.md` - MQTT temperature/humidity sensor

## Related Skills

- `/xiao` - Core board reference with pin definitions
- `/xiao-arduino` - Arduino development
- `/xiao-esphome` - ESPHome configuration
