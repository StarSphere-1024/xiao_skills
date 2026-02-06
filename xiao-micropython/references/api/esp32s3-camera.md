# ESP32S3 Sense Camera API (MicroPython)

## Overview

The Seeed Studio XIAO ESP32S3 Sense uses an expansion board with a camera connector. The camera module and the MicroPython camera API depend on the firmware build you flash.

## Camera Specifications

| Feature | Value |
|---------|-------|
| Camera module | OV2640 by default; OV5640 is also supported; some documentation also mentions OV3660 variants |
| Resolution | Depends on the camera module and selected frame size |
| Format | JPEG; RGB565; (some firmware also supports GRAYSCALE) |
| Control bus | SCCB (I2C-like), via CAM_SDA/CAM_SCL |
| Flash | Optional (some examples use an LED flash if defined for the camera pins) |

## Pin Mapping (Camera)

The XIAO ESP32S3 Sense camera connector occupies 14 GPIOs on the ESP32-S3.

| Camera Signal | ESP32-S3 GPIO | Notes |
|--------------|--------------|------|
| XMCLK | GPIO10 | Master clock |
| DVP_Y8 | GPIO11 | Data |
| DVP_Y7 | GPIO12 | Data |
| DVP_PCLK | GPIO13 | Pixel clock |
| DVP_Y6 | GPIO14 | Data |
| DVP_Y2 | GPIO15 | Data |
| DVP_Y5 | GPIO16 | Data |
| DVP_Y3 | GPIO17 | Data |
| DVP_Y4 | GPIO18 | Data |
| DVP_VSYNC | GPIO38 | Vertical sync |
| CAM_SCL | GPIO39 | SCCB/I2C clock |
| CAM_SDA | GPIO40 | SCCB/I2C data |
| DVP_HREF | GPIO47 | Horizontal reference |
| DVP_Y9 | GPIO48 | Data |

## Basic Camera Setup

```python
import camera

# Initialize camera
camera.init(0, format=camera.JPEG,
            fb_location=camera.PSRAM,
            fb_count=2, xclk_freq=camera.XCLK_20MHz)

# Configure settings
camera.quality(10)  # 0-63 (lower is better)
camera.framesize(camera.FRAME_SVGA)  # 800x600

print("Camera initialized")
```

## Capture Image

```python
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

# Free memory
del buf
gc.collect()
```

## Available Frame Sizes

```python
import camera

# Common frame size options (availability depends on firmware)
camera.framesize(camera.FRAME_QVGA)  # 320x240
camera.framesize(camera.FRAME_VGA)   # 640x480
camera.framesize(camera.FRAME_SVGA)  # 800x600
camera.framesize(camera.FRAME_XGA)   # 1024x768
camera.framesize(camera.FRAME_HD)    # 1280x720
```

## Quality

```python
import camera

# Quality (0-63, lower is better quality)
camera.quality(10)   # High quality
camera.quality(30)   # Medium quality
camera.quality(63)   # Lowest quality, smallest file
```

## RGB565 Mode

```python
import camera

# Initialize for RGB565
camera.init(0, format=camera.RGB565,
            fb_location=camera.PSRAM,
            fb_count=2, xclk_freq=camera.XCLK_20MHz)

camera.framesize(camera.FRAME_QVGA)  # 320x240

# Capture RGB data
buf = camera.capture()

# Access pixels
print(f"First pixel: {buf[0:2]}")
```

## Brightness and Contrast

```python
import camera

# Brightness
camera.brightness(0)

# Contrast
camera.contrast(0)
```

## Complete Camera Example

```python
import camera
import time
import gc

# Initialize camera
camera.init(0, format=camera.JPEG,
            fb_location=camera.PSRAM,
            fb_count=2, xclk_freq=camera.XCLK_20MHz)

# Configure settings
camera.framesize(camera.FRAME_SVGA)    # 800x600
camera.quality(10)                      # Good quality

# Optional adjustments (availability depends on firmware)
camera.brightness(0)
camera.contrast(0)

print("Camera ready")

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
            fb_location=camera.PSRAM,
            fb_count=2, xclk_freq=camera.XCLK_20MHz)

camera.framesize(camera.FRAME_QVGA)  # 320x240 (use smaller sizes if your firmware provides them)

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

## Timelapse Capture

```python
import machine
import camera
import time
import gc

# Initialize
camera.init(0, format=camera.JPEG, fb_location=camera.PSRAM)
camera.framesize(camera.FRAME_SVGA)
camera.quality(10)

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
            fb_location=camera.PSRAM,
            fb_count=4, xclk_freq=camera.XCLK_20MHz)

# Small(er) resolution for speed
camera.framesize(camera.FRAME_QVGA)  # 320x240
camera.quality(20)

# Fast capture
for i in range(10):
    buf = camera.capture()
    print(f"Captured {i+1}: {len(buf)} bytes")
    time.sleep(0.1)  # 10 fps
```

## Camera Troubleshooting

### Camera not detected

1. Check camera is connected properly
2. Verify camera module is XIAO ESP32S3 Sense
3. Try resetting board
4. Check pins configuration

### Images are corrupted

1. Reduce frame size
2. Adjust quality/frame size trade-offs
3. Check for memory/PSRAM issues
4. Try lower clock frequency (if supported by your firmware)

### Out of memory

1. Reduce frame size
2. Reduce fb_count
3. Call gc.collect() regularly
4. Delete buffers after use

### Slow frame rate

1. Reduce frame size
2. Reduce fb_count
3. Avoid long blocking network operations in the capture loop

### Color issues

1. Verify lighting conditions
2. If your firmware exposes sensor controls, adjust them accordingly

## Best Practices

1. **Use appropriate resolution** - match to your needs
2. **Adjust quality** - balance quality vs size
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
camera.init(0, format=camera.JPEG, fb_location=camera.PSRAM, fb_count=1)

# Always free after use
buf = camera.capture()
# ... use buf ...
del buf  # Important!
gc.collect()
```

## Performance Tips

1. **Lower resolution** for faster capture
2. **Use RGB565** for processing (no compression)
3. **Increase clock** if needed (firmware-dependent): `xclk_freq=camera.XCLK_20MHz`
4. **Minimize buffer count**: `fb_count=1`

## References

- Seeed Studio Wiki: Camera usage (GPIO mapping, OV2640/OV5640 compatibility): https://wiki.seeedstudio.com/xiao_esp32s3_camera_usage/
- Local mirror: Ref/XIAO_WIKI/SeeedStudio_XIAO/SeeedStudio_XIAO_ESP32S3_Sense/XIAO_ESP32S3_Sense_camera.md
- Repository MicroPython getting-started guide used for `camera.*` API examples: xiao-micropython/references/getting-started/esp32s3.md
- Seeed Studio Wiki: MicroPython firmware (custom build used in official guide): https://wiki.seeedstudio.com/XIAO_ESP32S3_Micropython/
- Seeed Studio Wiki: Hardware note that Sense kits may include OV2640 or OV3660: https://wiki.seeedstudio.com/xiao_esp32s3_microblocks/
