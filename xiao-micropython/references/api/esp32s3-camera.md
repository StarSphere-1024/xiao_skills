# ESP32S3 Sense Camera API (MicroPython)

## Overview

The XIAO ESP32S3 Sense features a built-in camera sensor for image capture and processing.

## Camera Specifications

| Feature | Value |
|---------|-------|
| Sensor | OV2640 |
| Resolution | Up to 1600x1200 |
| Format | JPEG, RGB565 |
| Interface | SCCB (I2C-like) |
| Flash | Built-in LED |

## Pin Mapping (Camera)

| Function | XIAO Pin | GPIO |
|----------|----------|------|
| D0 | GPIO16 | CAM_D0 |
| D1 | GPIO17 | CAM_D1 |
| D2 | GPIO18 | CAM_D2 |
| D3 | GPIO19 | CAM_D3 |
| D4 | GPIO20 | CAM_D4 |
| D5 | GPIO21 | CAM_D5 |
| D6 | GPIO22 | CAM_D6 |
| D7 | GPIO23 | CAM_D7 |
| D8 | GPIO4 | CAM_VSYNC |
| D9 | GPIO5 | CAM_HREF |
| D10 | GPIO6 | CAM_PCLK |
| D15 | GPIO15 | CAM_XCLK |
| D14 | GPIO14 | CAM_SDA (SCCB) |
| D13 | GPIO13 | CAM_SCL (SCCB) |

## Basic Camera Setup

```python
import machine
import camera

# Initialize camera
camera.init(0, format=camera.JPEG,
            fb_count=2, xclk_freq=camera.XCLK_20MHz)

# Configure pins for XIAO ESP32S3 Sense
camera.jpeg_quality(10)  # 0-63 (lower is better)
camera.framesize(camera.FRAME_SVGA)  # 800x600

print("Camera initialized")
```

## Capture Image

```python
import machine
import camera

# Initialize camera
camera.init(0, format=camera.JPEG)

# Capture image
buf = camera.capture()

# Save to file
with open('image.jpg', 'wb') as f:
    f.write(buf)

print("Image saved")
```

## Display Image Info

```python
import camera
import gc

# Capture and get info
buf = camera.capture()

print(f"Buffer size: {len(buf)} bytes")
print(f"Width: {camera.width()}")
print(f"Height: {camera.height()}")

# Free memory
del buf
gc.collect()
```

## Available Frame Sizes

```python
import camera

# Frame size options
camera.framesize(camera.FRAME_160x120)    # 160x120
camera.framesize(camera.FRAME_QQVGA)      # 160x120
camera.framesize(camera.FRAME_QQVGA2)     # 128x160
camera.framesize(camera.FRAME_QCIF)       # 176x144
camera.framesize(camera.FRAME_HQVGA)      # 240x176
camera.framesize(camera.FRAME_QVGA)       # 320x240
camera.framesize(camera.FRAME_CIF)        # 400x296
camera.framesize(camera.FRAME_VGA)        # 640x480
camera.framesize(camera.FRAME_SVGA)       # 800x600
camera.framesize(camera.FRAME_XGA)        # 1024x768
camera.framesize(camera.FRAME_HD)         # 1280x720
camera.framesize(camera.FRAME_SXGA)      # 1280x1024
camera.framesize(camera.FRAME_UXGA)      # 1600x1200
```

## JPEG Quality

```python
import camera

# JPEG quality (0-63, lower is better quality)
camera.jpeg_quality(10)   # High quality
camera.jpeg_quality(30)   # Medium quality
camera.jpeg_quality(63)   # Lowest quality, smallest file
```

## RGB565 Mode

```python
import camera

# Initialize for RGB565
camera.init(0, format=camera.RGB565,
            fb_count=2, xclk_freq=camera.XCLK_20MHz)

camera.framesize(camera.FRAME_QVGA)  # 320x240

# Capture RGB data
buf = camera.capture()

# Access pixels
print(f"First pixel: {buf[0:2]}")
```

## Camera Effects

```python
import camera

# Negative effect
camera.saturate(2)  # -2 to 2

# Special effects
camera.effect(camera.EFFECT_NONE)       # No effect
camera.effect(camera.EFFECT_NEG)       # Negative
camera.effect(camera.EFFECT_GRAYSCALE) # Grayscale
camera.effect(camera.EFFECT_RED)       # Red tint
camera.effect(camera.EFFECT_GREEN)     # Green tint
camera.effect(camera.EFFECT_BLUE)      # Blue tint
camera.effect(camera.EFFECT_RETRO)     # Retro
```

## White Balance

```python
import camera

# White balance modes
camera.whitebalance(camera.WB_NONE)     # No white balance
camera.whitebalance(camera.WB_SUNNY)    # Sunny
camera.whitebalance(camera.WB_CLOUDY)   # Cloudy
camera.whitebalance(camera.WB_OFFICE)   # Office
camera.whitebalance(camera.WB_HOME)     # Home
```

## Saturation and Brightness

```python
import camera

# Saturation (-2 to 2)
camera.saturate(0)   # Normal
camera.saturate(1)   # More saturated
camera.saturate(-1)  # Less saturated

# Brightness (-2 to 2)
camera.brightness(0)  # Normal
camera.brightness(1)  # Brighter
camera.brightness(-1) # Darker
```

## Contrast and Sharpness

```python
import camera

# Contrast (-2 to 2)
camera.contrast(0)   # Normal
camera.contrast(1)   # More contrast
camera.contrast(-1)  # Less contrast

# Sharpness (-2 to 2)
camera.sharpness(0)  # Normal
camera.sharpness(1)  # Sharper
camera.sharpness(-1) # Softer
```

## Complete Camera Example

```python
import machine
import camera
import time
import gc

# Initialize camera
camera.init(0, format=camera.JPEG,
            fb_count=2, xclk_freq=camera.XCLK_20MHz)

# Configure settings
camera.framesize(camera.FRAME_SVGA)    # 800x600
camera.jpeg_quality(10)                # Good quality

# White balance and effects
camera.whitebalance(camera.WB_SUNNY)
camera.saturate(0)
camera.brightness(0)

print("Camera ready")
print(f"Resolution: {camera.width()}x{camera.height()}")

# Capture loop
counter = 0

while True:
    # Capture image
    buf = camera.capture()

    # Save to file
    filename = f'capture_{counter}.jpg'
    with open(filename, 'wb') as f:
        f.write(buf)

    print(f"Saved {filename} ({len(buf)} bytes)")

    # Cleanup
    del buf
    gc.collect()

    counter += 1
    time.sleep(5)
```

## Motion Detection (Simple)

```python
import machine
import camera
import time
import gc

# Initialize
camera.init(0, format=camera.RGB565,
            fb_count=2, x_clk_freq=camera.XCLK_20MHz)

camera.framesize(camera.FRAME_QQVGA)  # 160x120 for speed

# Capture reference
ref_buf = camera.capture()

while True:
    # Capture new frame
    new_buf = camera.capture()

    # Simple difference (first 100 pixels)
    diff = sum(abs(ref_buf[i] - new_buf[i]) for i in range(100))

    if diff > 1000:  # Motion threshold
        print("Motion detected!")

    # Update reference
    ref_buf = new_buf
    gc.collect()
    time.sleep(0.1)
```

## HTTP Image Upload

```python
import machine
import camera
import network
import urequests
import time

# Connect to WiFi
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('YourSSID', 'YourPassword')
while not wlan.isconnected():
    time.sleep(1)
print('Connected')

# Initialize camera
camera.init(0, format=camera.JPEG)
camera.framesize(camera.FRAME_SVGA)
camera.jpeg_quality(10)

# Capture and upload
while True:
    buf = camera.capture()

    # Upload via HTTP POST
    response = urequests.post(
        'http://your-server.com/upload',
        data=buf,
        headers={'Content-Type': 'image/jpeg'}
    )

    print(f"Uploaded: {response.status_code}")
    response.close()

    del buf
    time.sleep(60)  # Every minute
```

## FTP Upload

```python
import machine
import camera
import network
import time
from ftp import FTP

# WiFi connection
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('YourSSID', 'YourPassword')
while not wlan.isconnected():
    time.sleep(1)

# Initialize camera
camera.init(0, format=camera.JPEG)
camera.framesize(camera.FRAME_SVGA)
camera.jpeg_quality(10)

# Capture and upload via FTP
ftp = FTP('ftp.server.com', 'username', 'password')
buf = camera.capture()
ftp.storbinary('STOR image.jpg', buf)
ftp.quit()
```

## Timelapse Capture

```python
import machine
import camera
import time
import gc

# Initialize
camera.init(0, format=camera.JPEG)
camera.framesize(camera.FRAME_SVGA)
camera.jpeg_quality(10)

# Timelapse settings
interval = 60  # 60 seconds between shots
max_shots = 100

counter = 0
while counter < max_shots:
    # Capture
    buf = camera.capture()
    filename = f'timelapse_{counter:04d}.jpg'

    # Save
    with open(filename, 'wb') as f:
        f.write(buf)

    print(f"Shot {counter+1}/{max_shots}: {filename}")

    # Cleanup
    del buf
    gc.collect()

    counter += 1
    time.sleep(interval)

print("Timelapse complete")
```

## Low Resolution Fast Capture

```python
import machine
import camera
import time

# Initialize for speed
camera.init(0, format=camera.JPEG,
            fb_count=4, xclk_freq=camera.XCLK_20MHz)

# Small resolution for speed
camera.framesize(camera.FRAME_QQVGA)  # 160x120
camera.jpeg_quality(20)

# Fast capture
for i in range(10):
    buf = camera.capture()
    print(f"Captured {i+1}: {len(buf)} bytes")
    time.sleep(0.1)  # 10 fps
```

## Camera Flash LED

```python
from machine import Pin
import time

# Flash LED (if available)
flash = Pin(4, Pin.OUT)

def flash_on(duration=0.1):
    flash.value(1)
    time.sleep(duration)
    flash.value(0)

# Use when capturing
flash_on(0.1)
buf = camera.capture()
```

## Camera Troubleshooting

### Camera not detected

1. Check camera is connected properly
2. Verify camera module is XIAO ESP32S3 Sense
3. Try resetting board
4. Check pins configuration

### Images are corrupted

1. Reduce frame size
2. Increase JPEG quality
3. Check for memory issues
4. Try lower clock frequency

### Out of memory

1. Reduce frame size
2. Reduce fb_count
3. Call gc.collect() regularly
4. Delete buffers after use

### Slow frame rate

1. Reduce frame size
2. Increase JPEG quality (faster compression)
3. Reduce fb_count
4. Check WiFi/network not blocking

### Color issues

1. Adjust white balance
2. Check saturation settings
3. Verify camera sensor settings
4. Try different lighting conditions

## Best Practices

1. **Use appropriate resolution** - match to your needs
2. **Adjust JPEG quality** - balance quality vs size
3. **Free buffers** - delete unused buffers
4. **Use gc.collect()** - prevent memory fragmentation
5. **Test frame rates** - ensure adequate performance
6. **Handle errors** - camera may fail occasionally
7. **Reinitialize** - if camera stops working

## Memory Management

```python
import camera
import gc

# Free memory before capture
gc.collect()

# Limit frame buffer count
camera.init(0, format=camera.JPEG, fb_count=1)

# Always free after use
buf = camera.capture()
# ... use buf ...
del buf  # Important!
gc.collect()
```

## Performance Tips

1. **Lower resolution** for faster capture
2. **Reduce JPEG quality** for faster compression
3. **Use RGB565** for processing (no compression)
4. **Increase clock** if needed: `xclk_freq=camera.XCLK_20MHz`
5. **Minimize buffer count**: `fb_count=1`

## Complete Security Camera Example

```python
import machine
import camera
import network
import time
import gc

# WiFi
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('YourSSID', 'YourPassword')
while not wlan.isconnected():
    time.sleep(1)

# Camera
camera.init(0, format=camera.JPEG)
camera.framesize(camera.FRAME_VGA)  # 640x480
camera.jpeg_quality(15)
camera.whitebalance(camera.WB_AUTO)

# Flash LED
flash = Pin(4, Pin.OUT)

print("Security camera running")

while True:
    gc.collect()

    # Flash
    flash.value(1)
    time.sleep(0.05)

    # Capture
    buf = camera.capture()

    flash.value(0)

    # Save timestamp
    timestamp = time.ticks_ms()
    filename = f'sec_{timestamp}.jpg'

    # Save locally
    with open(filename, 'wb') as f:
        f.write(buf)

    print(f"Captured: {filename}")

    # Cleanup
    del buf
    time.sleep(10)  # 10 seconds between shots
```
