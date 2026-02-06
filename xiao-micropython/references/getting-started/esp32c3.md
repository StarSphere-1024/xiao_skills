# XIAO ESP32C3 MicroPython Getting Started

## Firmware Download

Download from: https://micropython.org/download/ESP32_GENERIC_C3/

Look for: `ESP32_GENERIC_C3-2024xxxx-v1.22.0.bin` (or latest)

## Flash Firmware

### Method 1: esptool (Recommended)

```bash
# Install esptool
pip install esptool

# Flash firmware (adjust COM port)
esptool.py --chip esp32c3 --port COM3 write_flash -z 4MB 0 firmware.bin
```

### Method 2: Thonny IDE

1. Open Thonny
2. Tools > Options > Interpreter
3. Click "Install or update MicroPython"
4. Select "ESP32-C3" and port
5. Click "Install"

## First Steps

### Connect with Thonny

1. Open Thonny IDE
2. Click "Run" or F5
3. Select "MicroPython (ESP32)"
4. Select COM port

### First Script

```python
from machine import Pin
import time

led = Pin(10, Pin.OUT)  # D10

while True:
    led.value(not led.value())
    print("Blink!")
    time.sleep(1)
```

### REPL (Interactive)

```
MicroPython v1.22.0 on 2024-xx-xx; ESP32C3 with ESP32C3
Type "help()" for more information.
>>> print("Hello XIAO!")
Hello XIAO!
>>>
```

## Pin Mapping

| Name | GPIO | MicroPython Pin |
|------|------|-----------------|
| D0 | GPIO2 | Pin(2) |
| D1 | GPIO3 | Pin(3) |
| D2 | GPIO4 | Pin(4) |
| D3 | GPIO5 | Pin(5) |
| D4 | GPIO6 | Pin(6) |
| D5 | GPIO7 | Pin(7) |
| D6 | GPIO21 | Pin(21) |
| D7 | GPIO20 | Pin(20) |
| D8 | GPIO8 | Pin(8) |
| D9 | GPIO9 | Pin(9) |
| D10 | GPIO10 | Pin(10) |
| A0 | GPIO2 | Pin(2) |
| A1 | GPIO3 | Pin(3) |
| A2 | GPIO4 | Pin(4) |
| A3 | GPIO5 | Pin(5) |

## I2C Pins (Default)

```python
from machine import I2C, Pin

# Default I2C for XIAO ESP32C3
i2c = I2C(0, sda=Pin(6), scl=Pin(7))  # D4, D5
```

## SPI Pins (Default)

```python
from machine import SPI, Pin

# Default SPI for XIAO ESP32C3
spi = SPI(1, sck=Pin(8), mosi=Pin(10), miso=Pin(9))  # D8, D10, D9
```

## UART

```python
from machine import UART

# UART0 (USB serial)
uart0 = UART(0, baudrate=115200)

# UART1 (on pins)
uart1 = UART(1, tx=Pin(21), rx=Pin(20), baudrate=115200)  # D6, D7
```

## WiFi Quick Test

```python
import network

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

print(wlan.ifconfig())
```

## Save Scripts

### Save to Board

1. Write script in Thonny
2. File > Save as > "Select device"
3. Name it `main.py` (runs on boot)

### boot.py

Runs on every boot. Use for initialization:

```python
# boot.py
import network

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('SSID', 'password')
print("WiFi connecting...")
```

### main.py

Your main application:

```python
# main.py
import time

while True:
    print("Running...")
    time.sleep(1)
```

## Troubleshooting

### Board not detected

1. Check USB cable (data + power)
2. Install drivers: CH340/CP2102
3. Try different USB port

### Can't upload script

1. Stop any running script (Ctrl+C)
2. Click "Stop" button in Thonny
3. Try again

### ImportError

1. Check library name
2. Some libraries not available in standard firmware
3. May need custom firmware build

### WiFi won't connect

1. Check SSID/password
2. Use 2.4GHz network only
3. Check router WPA2 settings

## Performance Tips

1. **Use machine module**: Faster than pure Python
2. **Avoid blocking delays**: Use non-blocking code
3. **Use interrupts**: For time-critical tasks
4. **Optimize loops**: Unroll if needed
