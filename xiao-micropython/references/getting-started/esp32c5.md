# XIAO ESP32C5 MicroPython Getting Started

## Overview

The XIAO ESP32C5 is a compact board with WiFi and Bluetooth LE. This guide covers MicroPython setup.

## Board Specifications

- **MCU**: ESP32-C5 (WiFi, BLE 5.0)
- **Flash**: 4MB
- **RAM**: 400KB SRAM
- **GPIO**: 11 pins
- **I2C**: D4 (SDA), D5 (SCL)
- **SPI**: D8 (SCK), D9 (MISO), D10 (MOSI)
- **UART**: D6 (TX), D7 (RX)
- **ADC**: A0-A5 (D0-D5)
- **PWM**: All GPIO pins

## Firmware Download

Download MicroPython firmware for ESP32-C5 from:

```bash
# https://micropython.org/download/ESP32_GENERIC_C5/
# Look for: ESP32_GENERIC_C5-*.bin
```

## Flashing Firmware

### Using esptool

```bash
# Install esptool
pip install esptool

# Enter bootloader mode:
# 1. Hold BOOT button
# 2. Connect USB
# 3. Release BOOT button

# Erase flash
esptool.py --chip esp32c5 --port COM3 erase_flash

# Flash firmware (Windows example)
esptool.py --chip esp32c5 --port COM3 --baud 460800 write_flash -z 0x2000 firmware.bin

# Flash firmware (Linux/Mac example)
esptool.py --chip esp32c5 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x2000 firmware.bin
```

### Using Thonny IDE

1. Download firmware from micropython.org
2. Open Thonny IDE
3. Tools > Options > Interpreter
4. Select "MicroPython (ESP32)"
5. Click "Install or update MicroPython"
6. Select port and firmware file

## First Connection

### Using Thonny IDE

1. Open Thonny
2. Click "Stop/Restart" or Run > Stop/Restart
3. Select "MicroPython (ESP32)" interpreter
4. Select the XIAO ESP32C5 port
5. You should see the REPL prompt `>>>`

### Using mpremote

```bash
# Install mpremote
pip install mpremote

# Connect
mpremote connect auto

# Run REPL
mpremote repl
```

## First Script

```python
# test.py - Blink the built-in LED

from machine import Pin
import time

# USER LED is on GPIO27
led = Pin(27, Pin.OUT)

print("Hello from XIAO ESP32C5!")

while True:
    led.on()
    print("LED ON")
    time.sleep(1)
    led.off()
    print("LED OFF")
    time.sleep(1)
```

## Pin Mapping

| Pin Name | GPIO | Functions |
|----------|------|-----------|
| D0 | GPIO1 | GPIO, ADC |
| D1 | GPIO0 | GPIO (BOOT) |
| D2 | GPIO25 | GPIO |
| D3 | GPIO7 | GPIO |
| D4 | GPIO23 | I2C (SDA) |
| D5 | GPIO24 | I2C (SCL) |
| D6 | GPIO11 | UART TX |
| D7 | GPIO12 | UART RX |
| D8 | GPIO8 | SPI SCK |
| D9 | GPIO9 | SPI MISO |
| D10 | GPIO10 | SPI MOSI |
| USER_LED | GPIO27 | LED |

## WiFi Connection

```python
import network
import time

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('your-ssid', 'your-password')

# Wait for connection
while not wlan.isconnected():
    print('Connecting...')
    time.sleep(1)

print(f'Connected! IP: {wlan.ifconfig()[0]}')
```

## I2C Scanner

```python
from machine import Pin, I2C

i2c = I2C(0, scl=Pin(24), sda=Pin(23), freq=100000)
devices = i2c.scan()

print(f'I2C devices found: {len(devices)}')
for d in devices:
    print(f'  {hex(d)}')
```

## Boot Scripts

### boot.py

```python
# boot.py - Runs on every boot

import machine
import network

# Connect WiFi automatically
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('your-ssid', 'your-password')

print('boot.py: WiFi connecting...')
```

### main.py

```python
# main.py - Runs after boot.py

import time

print('main.py: Starting...')

while True:
    print('Running...')
    time.sleep(5)
```

## Uploading Scripts

### Using Thonny

1. Write your script
2. File > Save As
3. Select "MicroPython device"
4. Name it `main.py` for auto-run

### Using mpremote

```bash
# Upload file
mpremote fs cp script.py :main.py

# Reset device
mpremote reset
```

## Troubleshooting

### Device not found

1. Check USB cable (data + power, not power-only)
2. Install CH340/CP2102 driver if needed
3. Try different USB port

### Flash fails

1. Hold BOOT button while connecting USB
2. Try lower baud rate: `--baud 115200`
3. Use short USB cable

### WiFi won't connect

1. Check SSID and password
2. Try 2.4GHz or 5GHz (ESP32-C5 supports dual-band Wi-Fi)
3. Check router settings

## Next Steps

- Learn about deep sleep: `references/api/esp32-sleep.md`
- BLE examples: `references/api/esp32-ble.md`
- Project examples: `references/examples/`

## References

- MicroPython downloads (ESP32-C5): https://micropython.org/download/ESP32_GENERIC_C5/
- Seeed wiki (XIAO ESP32-C5 with MicroPython): https://wiki.seeedstudio.com/xiao_esp32c5_with_micropyhton/
- Seeed wiki (XIAO ESP32-C5 getting started / pin map): https://wiki.seeedstudio.com/XIAO_ESP32C5_Getting_Started/
