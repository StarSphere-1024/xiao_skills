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

MicroPython for ESP32C5 uses the generic ESP32 firmware:

```bash
# Download from:
# https://micropython.org/download/ESP32_GENERIC_C3/

# Or use ESP32 generic firmware (compatible)
# https://micropython.org/download/ESP32_GENERIC/
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

# Flash firmware (Windows example)
esptool.py --chip esp32c5 --port COM3 write_flash -z 0x10000 firmware.bin

# Flash firmware (Linux/Mac example)
esptool.py --chip esp32c5 --port /dev/ttyUSB0 write_flash -z 0x10000 firmware.bin
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

# LED is on D10
led = Pin(10, Pin.OUT)

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
| D0 | GPIO8 | ADC1_CH0, I2C, SPI |
| D1 | GPIO9 | ADC1_CH1, I2C, SPI |
| D2 | GPIO10 | ADC1_CH2, I2C, SPI |
| D3 | GPIO11 | ADC1_CH3, I2C, SPI |
| D4 | GPIO12 | ADC1_CH4, I2C (SDA) |
| D5 | GPIO13 | ADC1_CH5, I2C (SCL) |
| D6 | GPIO18 | UART TX |
| D7 | GPIO17 | UART RX |
| D8 | GPIO15 | SPI SCK |
| D9 | GPIO16 | SPI MISO |
| D10 | GPIO19 | SPI MOSI, LED |

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

i2c = I2C(0, scl=Pin(5), sda=Pin(4), freq=100000)
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
2. Ensure 2.4GHz network (ESP32 doesn't support 5GHz)
3. Check router settings

## Next Steps

- Learn about deep sleep: `references/api/esp32-sleep.md`
- BLE examples: `references/api/esp32-ble.md`
- Project examples: `references/examples/`
