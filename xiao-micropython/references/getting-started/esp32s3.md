# MicroPython on XIAO ESP32S3

## Overview

Getting started with MicroPython on SeeedStudio XIAO ESP32S3.

## Installation

### Download Firmware

1. Download MicroPython for ESP32S3: https://micropython.org/download/ESP32_GENERIC_S3/
2. Download `MICROPYTHON-*.bin`

### Flash with esptool

```bash
pip install esptool

esptool.py --chip esp32s3 --port COM<X> write_flash -z 0x0 MICROYPTHON.bin
```

### Verify Installation

```
Serial: 115200 baud
>>> print("Hello XIAO S3!")
Hello XIAO S3!
>>> import esp
>>> esp.flash_size()
8388608
```

## Thonny IDE Setup

1. Download: https://thonny.org/
2. Install and launch
3. Tools > Options > Interpreter > MicroPython (ESP32)
4. Select port and click Connect

## Pin Mapping

| Name | GPIO | Notes |
|------|------|-------|
| D0 | GPIO44 | USB- D- |
| D1 | GPIO43 | USB+ D+ |
| D2 | GPIO8 | User button |
| D3 | GPIO9 | Boot button |
| D4 | GPIO10 | - |
| D5 | GPIO11 | - |
| D6 | GPIO12 | - |
| D7 | GPIO13 | - |
| D8 | GPIO18 | JTAG TMS |
| D9 | GPIO19 | JTAG TCK |
| D10 | GPIO20 | JTAG TDO |

Built-in LED: GPIO39 (white, connected to USB)

## Camera Support (Sense Model)

### Initialize Camera

```python
import machine
from machine import Pin
import time

# Camera pins for XIAO ESP32S3 Sense
cam_pins = {
    'D0': 44,  # D- (USB)
    'D1': 43,  # D+ (USB)
    'D2': 5,   # SCCB
    'D3': 16,
    'D4': 4,
    'D5': 15,
    'D6': 47,
    'D7': 48,
    'D8': 45,
    'D9': 40,
    'D10': 41,
}

# Note: ESP32S3 Sense has built-in camera support
# Initialize camera driver
import camera
camera.init(0, format=camera.JPEG, fb_location=camera.PSRAM)
```

### Capture Photo

```python
import camera
import time

# Initialize camera
camera.init(0, format=camera.JPEG)

# Capture
buf = camera.capture()
print(f"Captured {len(buf)} bytes")

# Save to file
with open('photo.jpg', 'wb') as f:
    f.write(buf)

print("Photo saved!")
```

### Camera Configuration

```python
import camera

# Initialize with settings
camera.init(
    0,
    format=camera.JPEG,      # or GRAYSCALE, RGB565
    fb_location=camera.PSRAM,
    xclk_freq=camera.XCLK_20MHz,
    fb_count=2
)

# Set frame size
camera.framesize(camera.FRAME_SVGA)  # 800x600
# Options: FRAME_QVGA, FRAME_VGA, FRAME_SVGA, FRAME_XGA, FRAME_HD

# Set quality (0-63, lower is better)
camera.quality(10)

# Set contrast
camera.contrast(0)

# Set brightness
camera.brightness(0)
```

### Video Stream

```python
import camera
import network
import socket

# Initialize camera
camera.init(0, format=camera.JPEG)
camera.framesize(camera.FRAME_QVGA)

# Connect to WiFi
import network
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

print('Connected:', wlan.ifconfig()[0])

# Start streaming server
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('', 80))
s.listen(5)

print('Streaming on port 80')

while True:
    conn, addr = s.accept()
    print('Client connected from', addr)

    # Send MJPG stream
    conn.send(b'HTTP/1.1 200 OK\r\n')
    conn.send(b'Content-Type: multipart/x-mixed-replace; boundary=frame\r\n\r\n')

    while True:
        buf = camera.capture()
        conn.send(b'--frame\r\n')
        conn.send(b'Content-Type: image/jpeg\r\n\r\n')
        conn.send(buf)
        conn.send(b'\r\n')

        if not conn:
            break
```

## Dual Core Operations

### Run on Both Cores

```python
import _thread
import machine
import time

def core0_task():
    print("Core 0 starting")
    while True:
        print("Core 0 running")
        time.sleep(1)

def core1_task():
    print("Core 1 starting")
    while True:
        print("Core 1 running")
        time.sleep(0.5)

_thread.start_new_thread(core1_task, ())

# Main runs on core 0
core0_task()
```

### Check CPU ID

```python
import machine

print(f"Running on core: {machine.cpu()}")
```

## PSRAM Usage

### Check PSRAM

```python
import esp32

print(f"Total PSRAM: {esp32.psram_total() / 1024:.0f} KB")
print(f"Free PSRAM: {esp32.psram_free() / 1024:.0f} KB")
```

### Allocate Large Buffer

```python
import machine
import array

# Allocate large buffer in PSRAM
# Only works if PSRAM enabled
buf = array.array('B', [0]) * 1024000  # 1 MB buffer
print(f"Allocated {len(buf)} bytes")
```

## USB CDC (Dual Serial)

### Enable Dual Serial

```python
import machine

# USB Serial is always available
usb = machine.UART(0, 115200)

# Hardware serial on pins
hw_serial = machine.UART(1, 115200, tx=Pin(4), rx=Pin(5))

# USB to Hardware bridge
while True:
    if usb.any():
        hw_serial.write(usb.read())
    if hw_serial.any():
        usb.write(hw_serial.read())
```

## JTAG Debugging

### Disable JTAG for GPIO

```python
import machine

# By default, D8-D10 are used for JTAG
# To use them as GPIO, disable JTAG

from machine import Pin
import esp32

# Release JTAG pins
esp32.GPIO8.set_mode(esp32.GPIO.MODE_INPUT)
esp32.GPIO9.set_mode(esp32.GPIO.MODE_INPUT)
esp32.GPIO10.set_mode(esp32.GPIO.MODE_INPUT)

# Now can use as GPIO
p8 = Pin(8, Pin.OUT)
p9 = Pin(9, Pin.OUT)
p10 = Pin(10, Pin.OUT)
```

## ESP-IDF Functions

### ESP32 Specific

```python
import esp32
import machine

# Get chip info
print(f"Chip ID: {hex(esp32.hall_sensor_get_threshold())}")
print(f"CPU frequency: {machine.freq()} MHz")

# Hall sensor
hall = esp32.hall_sensor()
print(f"Hall sensor: {hall}")

# Temperature sensor (internal)
temp = esp32.raw_temperature()
print(f"Internal temp: {temp}")

# DAC output
dac1 = machine.DAC(1)  # GPIO17
dac1.write(128)  # 0-255

# Touch pins
touch0 = machine.TouchPad(0)
print(f"Touch 0: {touch0.read()}")
```

## Troubleshooting

### Camera init fails

1. Check Sense model has camera connected
2. Verify PSRAM is enabled in firmware
3. Check camera module power
4. Try different frame sizes

### PSRAM not detected

1. Use firmware with PSRAM support
2. Check board has PSRAM chip
3. Try re-flashing firmware

### USB CDC issues

1. Check `USB CDC On Boot` setting
2. Verify driver installation
3. Try different USB cable

### Boot button issues

1. D3 is boot button, avoid using
2. Can read but careful with output
3. Use D2 for user button instead

## Performance Tips

1. Use PSRAM for large buffers
2. Offload to second core for parallel tasks
3. Use camera JPEG format for faster capture
4. Lower frame rate for better performance
5. Use UART DMA for large transfers
