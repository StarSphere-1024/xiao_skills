---
name: xiao-micropython
description: Complete MicroPython development for XIAO boards. Use when writing MicroPython scripts for XIAO (ESP32C3/C5/C6/S3, RP2040/RP2350). Requires /xiao skill for board-specific pin definitions. Covers firmware flashing, ESP32/network modules (network, WiFi, BLE, MQTT), machine modules (Pin, I2C, SPI, UART, PWM, ADC), common libraries (urequests, umqtt, ssd1306, bme280, neopixel), and project examples.
---

# XIAO MicroPython Development

## Overview

Complete MicroPython development guide for SeeedStudio XIAO series. This skill works with the `/xiao` skill - use `/xiao` for pin definitions, `/xiao-micropython` for MicroPython-specific APIs and libraries.

## Prerequisites

1. **Pin Definitions**: Read `/xiao` skill first for board-specific pin mappings
2. **Thonny IDE**: Recommended for MicroPython development
3. **Firmware**: MicroPython firmware flashed to board

### Environment Setup

For first-time MicroPython environment setup, see `setup/`:
- **Thonny IDE Installation**: `setup/thonny-ide.md`
- **Firmware Flashing**: `setup/esptool.md`

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

**Note**: SAMD21, MG24 MicroPython support is limited or requires community forks.

## Quick Start

### 1. Flash Firmware

```bash
# Install esptool for ESP32 series
pip install esptool

# Download firmware from micropython.org
# Flash (ESP32C3 example):
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
    print("LED ON")
    time.sleep(1)
    led.value(0)
    print("LED OFF")
    time.sleep(1)
```

## Machine Module Reference

### GPIO (machine.Pin)

```python
from machine import Pin

# Output
led = Pin(10, Pin.OUT)
led.value(1)
led.on()
led.off()
led.toggle()

# Input with pull-up
button = Pin(6, Pin.IN, Pin.PULL_UP)
if button.value() == 0:
    print("Button pressed")

# IRQ (interrupt)
def callback(pin):
    print("Interrupt!", pin)

button.irq(trigger=Pin.IRQ_FALLING, handler=callback)
```

### PWM (machine.PWM)

```python
from machine import Pin, PWM
import time

pwm = PWM(Pin(10))
pwm.freq(1000)      # 1kHz frequency
pwm.duty(512)       # 50% duty (0-1023)

# Fade LED
for duty in range(0, 1024, 16):
    pwm.duty(duty)
    time.sleep(0.01)
```

### ADC (machine.ADC)

```python
from machine import ADC
import time

adc = ADC(Pin(0))  # A0
adc.atten(ADC.ATTN_11DB)  # 0-3.3V range (ESP32)

while True:
    value = adc.read_u16()  # 0-65535
    voltage = value * 3.3 / 65535
    print(f"ADC: {value}, Voltage: {voltage:.2f}V")
    time.sleep(0.5)
```

### I2C (machine.I2C)

```python
from machine import I2C, Pin

i2c = I2C(0, scl=Pin(1), sda=Pin(0), freq=100000)

# Scan for devices
devices = i2c.scan()
print(f"I2C devices: {[hex(d) for d in devices]}")

# Write to device
i2c.writeto(0x48, b'\x00')

# Read from device
i2c.writeto(0x48, b'\x00')
data = i2c.readfrom(0x48, 2)
print(data)
```

### SPI (machine.SPI)

```python
from machine import SPI, Pin

spi = SPI(0, baudrate=1000000, polarity=0, phase=0,
          sck=Pin(2), mosi=Pin(3), miso=Pin(4))
cs = Pin(5, Pin.OUT)

# Transfer data
cs.value(0)
spi.write(b'\x00')
data = spi.read(2)
cs.value(1)
```

### UART (machine.UART)

```python
from machine import UART

uart = UART(0, baudrate=115200, tx=Pin(6), rx=Pin(7))
uart.write(b'Hello UART\r\n')

while True:
    if uart.any():
        data = uart.read()
        print(f"Received: {data}")
```

## ESP32 Network Modules

### WiFi (network.WiFi)

```python
import network

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('your-SSID', 'your-password')

while not wlan.isconnected():
    print('Connecting...')
    time.sleep(1)

print(f'Connected! IP: {wlan.ifconfig()[0]}')

# Check connection
wlan.ifconfig()  # (IP, subnet, gateway, DNS)
wlan.isconnected()
wlan.status()  # 1000=STAT_IDLE, 1010=STAT_GOT_IP
```

### HTTP Requests (urequests)

```python
import urequests

response = urequests.get('http://httpbin.org/get')
print(response.text)
response.close()

# POST request
data = {'key': 'value'}
response = urequests.post('http://httpbin.org/post', json=data)
print(response.text)
response.close()
```

### MQTT (umqtt.simple)

```python
from umqtt.simple import MQTTClient
import time

# Callback
def sub_cb(topic, msg):
    print((topic, msg))

# Connect
client = MQTTClient("xiao_client", "broker.hivemq.com")
client.set_callback(sub_cb)
client.connect()
client.subscribe(b"xiao/command")

# Publish
client.publish(b"xiao/sensor", b"Hello XIAO")

# Check messages
while True:
    client.check_msg()
    time.sleep(1)
```

### Socket (network.socket)

```python
import network
import socket

# Connect WiFi first
wlan = network.WLAN(network.STA_IF)
wlan.connect('SSID', 'password')
while not wlan.isconnected():
    pass

# Create socket
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('httpbin.org', 80))
sock.send(b'GET /get HTTP/1.0\r\n\r\n')

response = sock.recv(4096)
print(response)
sock.close()
```

## Common Libraries

### SSD1306 OLED

```python
from machine import Pin, I2C
import ssd1306
import time

i2c = I2C(0, sda=Pin(0), scl=Pin(1))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

oled.fill(0)
oled.text("Hello XIAO!", 0, 0)
oled.show()
```

### BME280 Sensor

```python
from machine import Pin, I2C
import bme280

i2c = I2C(0, sda=Pin(0), scl=Pin(1))
bme = bme280.BME280(i2c=i2c)

while True:
    temp, pressure, hum = bme.read_compensated_data()
    print(f"Temp: {temp/100:.1f}C, Hum: {hum/1024:.1f}%")
    time.sleep(2)
```

### Neopixel (RP2040, ESP32)

```python
from machine import Pin
from neopixel import NeoPixel
import time

pin = Pin(10, Pin.OUT)
np = NeoPixel(pin, 8)  # 8 LEDs

# Set all to red
np.fill((255, 0, 0))
np.write()

# Color wheel
for i in range(256):
    np[0] = (i, 0, 0)
    np.write()
    time.sleep(0.01)
```

### DHT Sensor

```python
from machine import Pin
import dht
import time

d = dht.DHT22(Pin(0))

while True:
    try:
        d.measure()
        print(f"Temp: {d.temperature()}C, Hum: {d.humidity()}%")
    except OSError:
        print("DHT sensor error")
    time.sleep(2)
```

## File System

```python
import os

# List files
print(os.listdir())

# Write file
with open('data.txt', 'w') as f:
    f.write('Hello XIAO!')

# Read file
with open('data.txt', 'r') as f:
    content = f.read()
    print(content)

# Get file info
print(os.stat('data.txt'))
```

## Deep Sleep (ESP32)

```python
import machine
import time

# Sleep for 60 seconds
machine.deepsleep(60000)

# Code after this won't execute until wake

# RTC memory (persists through deep sleep)
rtc = machine.RTC()
rtc.memory('saved data')
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
        print("Periodic task")

    # Other code here
```

## Boot Scripts

### boot.py
Runs on every boot. Use for initialization:

```python
# boot.py
import machine
import network

# Connect WiFi automatically
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('SSID', 'password')
```

### main.py
Runs after boot.py. Your main application:

```python
# main.py
import time

while True:
    print("Running...")
    time.sleep(1)
```

## Code Verification

**After generating MicroPython code, verify it using mpremote CLI.**

### Quick Verification (If mpremote is Already Installed)

```bash
# Test syntax by running in RAM (doesn't save to device)
mpremote run script.py

# Or upload and run
mpremote fs cp script.py :main.py && mpremote reset
```

### Full Verification (First-Time Setup)

**Step 1: Install mpremote** (if not installed)

```bash
pip install --user mpremote

# Verify installation
mpremote --version
```

**Step 2: Connect to Device**

```bash
# Auto-detect and connect
mpremote connect auto

# Or specify port
mpremote connect /dev/ttyUSB0  # Linux
mpremote connect COM3         # Windows
```

**Step 3: Test Code**

```bash
# Run script in RAM to verify syntax
mpremote run script.py

# Verify modules are available
mpremote eval "import machine; import network; print('✅ OK')"
```

**Step 4: Deploy to Device** (optional)

```bash
# Upload as main.py (runs on boot)
mpremote fs cp script.py :main.py

# Reset device
mpremote reset
```

### Common Errors and Solutions

| Error Pattern | Cause | Fix |
|---------------|-------|-----|
| `SyntaxError: invalid syntax` | CPython features in code | Remove type hints, f-strings |
| `ImportError: no module named` | Module not available on device | Check device firmware capabilities |
| `OSError: [Errno 5]` | Wrong I2C/SPI pins | Verify pins with `/xiao` skill |
| `mpremote: command not found` | mpremote not installed | Run `pip install mpremote` |

### Common mpremote Commands

| Command | Purpose |
|---------|---------|
| `mpremote connect auto` | Auto-connect to device |
| `mpremote run script.py` | Run script in RAM |
| `mpremote fs cp local.py :remote.py` | Upload file |
| `mpremote exec "code"` | Execute code string |
| `mpremote repl` | Enter interactive REPL |
| `mpremote reset` | Reset device |

### Example: Complete Verification

```python
# blink.py - Generated for XIAO ESP32C3

from machine import Pin
import time

LED_PIN = 10  # D10 (from /xiao skill)

def main():
    led = Pin(LED_PIN, Pin.OUT)
    while True:
        led.on()
        time.sleep(1)
        led.off()
        time.sleep(1)

if __name__ == "__main__":
    main()
```

**Verification command:**
```bash
mpremote run blink.py
```

## Troubleshooting

1. Check library name (MicroPython uses different names than CPython)
2. Some libraries need to be frozen into firmware

### OSError: [Errno 5] Input/output error

1. I2C/SPI pin conflict
2. Wrong I2C address
3. Sensor not powered

### WiFi won't connect

1. Check SSID and password
2. Ensure 2.4GHz network (ESP32 doesn't support 5GHz)
3. Check router settings

### Board won't enter bootloader

**ESP32**: Hold BOOT button while connecting USB
**RP2040**: Hold BOOTSEL button while connecting USB

## Resources

### setup/
Environment installation and firmware flashing:
- `setup/thonny-ide.md` - Thonny IDE installation and configuration
- `setup/esptool.md` - MicroPython firmware flashing with esptool
- `setup/mpremote.md` - mpremote CLI for code verification and deployment

### getting-started/
Board-specific firmware flashing and setup instructions.

### api/
MicroPython machine module documentation:
- `api/esp32-sleep.md` - ESP32 deep sleep and power management
- `api/esp32-ble.md` - ESP32 BLE peripheral/central API
- `api/nrf-sleep.md` - nRF52 sleep and low power modes
- `api/nrf-ble.md` - nRF52 BLE API (optimized for ultra-low power)
- `api/rp-sleep.md` - RP2040/RP2350 sleep configuration
- `api/esp32s3-camera.md` - ESP32S3 Sense camera API and examples
- `api/rp2040-pio.md` - RP2040 PIO (Programmable I/O) for custom peripherals

### libraries/
Common MicroPython libraries and usage examples.

### examples/
Complete project examples:
- `examples/weather-station.md` - BME280 weather station with MQTT
- `examples/ble-peripheral.md` - BLE environmental sensor
- `examples/mqtt-sensor.md` - MQTT temperature/humidity sensor
- `examples/data-logger.md` - SD card data logging system

## Related Skills

- `/xiao` - Core board reference with pin definitions
- `/xiao-arduino` - Arduino development
- `/xiao-esphome` - ESPHome configuration
