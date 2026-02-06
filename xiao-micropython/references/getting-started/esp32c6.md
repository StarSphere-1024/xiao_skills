# MicroPython on XIAO ESP32C6

## Overview

The SeeedStudio XIAO ESP32C6 features the ESP32-C6 chip with WiFi 6, BLE 5.4, and 802.15.4 support.

## Board Specifications

| Feature | Value |
|---------|-------|
| MCU | ESP32-C6 (RISC-V, 160MHz single-core) |
| Flash | 4 MB |
| RAM | 512 KB |
| WiFi | 802.11b/g/n (2.4GHz, WiFi 6 ready) |
| BLE | Bluetooth 5.4 |
| 802.15.4 | Zigbee/Thread ready |
| GPIO | 13 pins |
| ADC | 12-bit, 5 channels |
| PWM | 6 channels |

## Pin Mapping for MicroPython

| XIAO Pin | GPIO | MicroPython | Functions |
|----------|------|-------------|-----------|
| D0 | GPIO8 | Pin(8) | USB-, BOOT |
| D1 | GPIO9 | Pin(9) | USB+, BOOT |
| D2 | GPIO10 | Pin(10) | SPI MOSI |
| D3 | GPIO11 | Pin(11) | SPI MISO |
| D4 | GPIO12 | Pin(12) | I2C SDA (alt) |
| D5 | GPIO13 | Pin(13) | I2C SCL (alt), LED |
| D6 | GPIO18 | Pin(18) | UART TX |
| D7 | GPIO19 | Pin(19) | UART RX |
| D8 | GPIO20 | Pin(20) | SPI SCK |
| D9 | GPIO21 | Pin(21) | SPI MISO |
| D10 | GPIO10 | Pin(10) | SPI MOSI |

Built-in LED: GPIO13 (white)

## Flashing MicroPython Firmware

### 1. Download Firmware

Download ESP32-C6 MicroPython firmware from:
https://micropython.org/download/ESP32_GENERIC_C6/

### 2. Install esptool

```bash
pip install esptool
```

### 3. Enter Boot Mode

1. Hold BOOT button
2. Connect USB cable
3. Release BOOT button

### 4. Erase Flash

```bash
esptool.py --chip esp32c6 --port COM3 erase_flash
```

### 5. Flash Firmware

```bash
esptool.py --chip esp32c6 --port COM3 write_flash -z 4MB 0 firmware.bin
```

### 6. Connect with Thonny

1. Open Thonny IDE
2. Tools > Options > Interpreter
3. Select "MicroPython (ESP32)"
4. Select port and click OK

## First Script

```python
from machine import Pin
import time

# Built-in LED on D13
led = Pin(13, Pin.OUT)

while True:
    led.value(1)
    print("LED ON")
    time.sleep(1)

    led.value(0)
    print("LED OFF")
    time.sleep(1)
```

## I2C Configuration

```python
from machine import Pin, I2C
import time

# I2C on D4 (SDA) and D5 (SCL)
i2c = I2C(0, scl=Pin(5), sda=Pin(4), freq=100000)

# Scan for devices
devices = i2c.scan()
print(f"I2C devices: {[hex(d) for d in devices]}")

# Read from device (e.g., BME280 at 0x76)
i2c.writeto(0x76, b'\x00')  # Write register
data = i2c.readfrom(0x76, 8)  # Read 8 bytes
print(f"Data: {data}")
```

## SPI Configuration

```python
from machine import Pin, SPI

# SPI on D8 (SCK), D9 (MISO), D10 (MOSI)
spi = SPI(1, sck=Pin(8), miso=Pin(9), mosi=Pin(10), baudrate=1000000)

# Select device
cs = Pin(2, Pin.OUT)
cs.value(0)

# Transfer data
data = spi.read(8)  # Read 8 bytes
print(f"SPI data: {data}")

cs.value(1)
```

## UART Configuration

```python
from machine import UART, Pin

# UART on D6 (TX) and D7 (RX)
uart = UART(1, tx=Pin(18), rx=Pin(19), baudrate=115200)

uart.write('Hello XIAO ESP32C6!\n')

while True:
    if uart.any():
        data = uart.read()
        print(f"Received: {data}")
```

## PWM Output

```python
from machine import Pin, PWM
import time

# PWM on D10
pwm = PWM(Pin(10))
pwm.freq(1000)  # 1kHz

# Fade LED
for duty in range(0, 1024, 16):
    pwm.duty(duty)
    time.sleep(0.01)

for duty in range(1023, -1, -16):
    pwm.duty(duty)
    time.sleep(0.01)
```

## ADC Reading

```python
from machine import ADC, Pin
import time

# ADC on A0
adc = ADC(Pin(0))
adc.atten(ADC.ATTN_11DB)  # 0-3.3V range

while True:
    value = adc.read_u16()  # 0-65535
    voltage = value * 3.3 / 65535
    print(f"ADC: {value}, Voltage: {voltage:.2f}V")
    time.sleep(0.5)
```

## WiFi Connection

```python
import network
import time

wlan = network.WLAN(network.STA_IF)
wlan.active(True)

# Connect to WiFi
wlan.connect('YourSSID', 'YourPassword')

# Wait for connection
while not wlan.isconnected():
    print('Connecting...')
    time.sleep(1)

print('Connected!')
print(f'IP: {wlan.ifconfig()[0]}')
print(f'Subnet: {wlan.ifconfig()[1]}')
print(f'Gateway: {wlan.ifconfig()[2]}')
print(f'DNS: {wlan.ifconfig()[3]}')
```

## HTTP Request

```python
import network
import urequests

# Connect to WiFi first
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('YourSSID', 'YourPassword')

while not wlan.isconnected():
    pass

# Make HTTP request
response = urequests.get('http://httpbin.org/get')
print(f"Status: {response.status_code}")
print(f"Content: {response.text}")
response.close()
```

## MQTT Basic Example

```python
import network
import time
from umqtt.simple import MQTTClient

# Connect to WiFi
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('YourSSID', 'YourPassword')
while not wlan.isconnected():
    time.sleep(1)

# MQTT configuration
client = MQTTClient('xiao-esp32c6', 'broker.hivemq.com')
client.connect()

# Publish message
client.publish('xiao/sensor', 'Hello from XIAO ESP32C6')

# Subscribe
def on_message(topic, msg):
    print(f"Received: {topic} = {msg}")

client.set_callback(on_message)
client.subscribe('xiao/command')

# Loop
while True:
    client.check_msg()
    time.sleep(1)
```

## Deep Sleep

```python
import machine
import time

print("Going to deep sleep...")

# Sleep for 30 seconds
machine.deepsleep(30000)

# Code after deepsleep won't execute
```

## Low Power Optimization

```python
import machine
import network
import time

# Disable WiFi to save power
wlan = network.WLAN(network.STA_IF)
wlan.active(False)

# Lower CPU frequency
machine.freq(80_000_000)  # 80MHz

# Light sleep
machine.lightsleep(5000)

print("Woke from light sleep")
```

## Watchdog Timer

```python
import machine
import time

# Enable watchdog (5 seconds)
wdt = machine.WDT(timeout=5000)

# Feed the watchdog
while True:
    print("Watchdog fed")
    wdt.feed()
    time.sleep(1)
```

## Temperature Sensor (Internal)

```python
import machine

# Read internal temperature
temp = machine.temperature()
print(f"Internal temperature: {temp}°C")
```

## Hall Sensor (ESP32-C6)

```python
import machine

# Read hall sensor
hall = machine.hall_sensor()
print(f"Hall sensor: {hall}")
```

## Touch Pins (if available)

```python
from machine import TouchPad
import time

# Touch on T0 (if available)
touch = TouchPad(Pin(0))

while True:
    value = touch.read()
    print(f"Touch: {value}")
    time.sleep(0.1)
```

## Key Differences from ESP32C3

| Feature | ESP32C3 | ESP32C6 |
|---------|--------|--------|
| Core | RISC-V single-core | RISC-V single-core |
| Flash | 4 MB | 4 MB |
| RAM | 400 KB | 512 KB |
| WiFi | 802.11b/g/n | 802.11b/g/n (WiFi 6 ready) |
| BLE | 5.0 | 5.4 |
| 802.15.4 | No | Yes (Zigbee/Thread) |
| Low Power | Good | Better |

## Typical Current Consumption

| Mode | Current |
|------|---------|
| Active (WiFi) | ~150mA |
| Active (no WiFi) | ~50mA |
| Lightsleep | ~5mA |
| Deepsleep | ~150µA |

## Troubleshooting

### Can't upload firmware

1. Ensure BOOT button is held while connecting USB
2. Try different USB cable
3. Check COM port in device manager
4. Try lower baud rate: `--baud 115200`

### Can't connect to Thonny

1. Check device manager for COM port
2. Verify MicroPython is installed
3. Try disconnecting/reconnecting USB
4. Check Thonny interpreter settings

### WiFi won't connect

1. Check SSID and password
2. Ensure 2.4GHz network (ESP32 doesn't support 5GHz)
3. Check router settings
4. Try reset: `wlan.disconnect()`

### Scripts don't run automatically

1. Save as `main.py` on device
2. Use Thonny's "Save as" and select "MicroPython device"
3. Reset board to run `main.py`

## Best Practices

1. **Use lightsleep** for periodic wake-ups
2. **Disable WiFi** when not needed
3. **Lower CPU frequency** for battery projects
4. **Use I2C/SPI** correctly - check pin mappings
5. **Handle WiFi disconnects** gracefully
6. **Use deep sleep** for battery-powered devices
7. **Add watchdog** for critical applications

## Common Issues

### Boot loop after deep sleep

1. Check sleep duration
2. Verify wake sources configured
3. Check for hardware issues

### I2C not working

1. Verify pin numbers (D4=SDA, D5=SCL)
2. Check pull-up resistors
3. Scan for devices with `i2c.scan()`
4. Try lower frequency

### SPI not working

1. Verify all connections
2. Check CS pin control
3. Try lower baudrate
4. Check MISO/MOSI not swapped

## Migration from ESP32C3

```python
# ESP32C3 code mostly works on ESP32C6
# Changes needed:

# 1. Update WiFi (if using WiFi 6 features)
wlan.config(pm=WIFI_PS_MIN_MODEM)  # New power save modes

# 2. Update BLE (if using BLE 5.4 features)
# BLE 5.4 features may require new API

# 3. Pin mappings - same as ESP32C3

# 4. Low power - ESP32C6 has better options
machine.lightsleep(5000)  # Lower power than C3
```

## Complete Example: Weather Station

```python
from machine import Pin, I2C, ADC
import network
import time
import urequests

# I2C for BME280
i2c = I2C(0, scl=Pin(5), sda=Pin(4), freq=100000)

# WiFi connection
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('YourSSID', 'YourPassword')
while not wlan.isconnected():
    time.sleep(1)
print(f'Connected: {wlan.ifconfig()[0]}')

# Read sensor (BME280 example)
i2c.writeto(0x76, b'\x00')
data = i2c.readfrom(0x76, 8)
print(f'Sensor data: {data}')

# Send to server
response = urequests.post(
    'http://your-server.com/api/sensor',
    json={'temp': 25.5, 'hum': 60}
)
response.close()

# Sleep for 5 minutes
machine.deepsleep(300000)
```
