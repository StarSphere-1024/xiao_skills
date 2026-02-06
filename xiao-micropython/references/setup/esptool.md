# MicroPython Firmware Flashing with esptool

## Overview

esptool is the official command-line tool for flashing ESP32 microcontrollers with MicroPython firmware.

## Prerequisites

### Python Installation

esptool requires Python 3.7 or later.

**Windows:**
1. Download from: https://www.python.org/downloads/
2. Install with "Add to PATH" option checked
3. Verify: `python --version`

**macOS:**
```bash
# Install with Homebrew
brew install python@3.11
```

**Linux:**
```bash
sudo apt install python3 python3-pip
```

## Installing esptool

```bash
# Install esptool
pip install esptool

# Verify installation
esptool.py version
```

## Downloading MicroPython Firmware

### ESP32C3

Visit: https://micropython.org/download/ESP32_GENERIC_C3/

Download: `ESP32_GENERIC_C3-2024xxxx-v1.22.0.bin` (or latest)

### ESP32C6

Visit: https://micropython.org/download/ESP32_GENERIC_C6/

Download: `ESP32_GENERIC_C6-2024xxxx-v1.22.0.bin`

### ESP32S3

Visit: https://micropython.org/download/ESP32_GENERIC_S3/

Download: `ESP32_GENERIC_S3-2024xxxx-v1.22.0.bin`

### RP2040/RP2350

Visit: https://micropython.org/download/RPI_PICO/

**RP2040:** Download: `RPI_PICO-2024xxxx-v1.22.0.uf2`
**RP2350:** Download: `RPI_PICO2-2024xxxx-v1.22.0.uf2`

## Flashing ESP32 Series

### Identify COM Port

**Windows:** Usually COM3, COM4
**macOS:** /dev/cu.usbserial-xxx
**Linux:** /dev/ttyUSB0

### Flash Command

**ESP32C3:**
```bash
esptool.py --chip esp32c3 --port COM3 write_flash -z 4MB 0 firmware.bin
```

**ESP32C6:**
```bash
esptool.py --chip esp32c6 --port COM3 write_flash -z 4MB 0 firmware.bin
```

**ESP32S3:**
```bash
esptool.py --chip esp32s3 --port COM3 write_flash -z 4MB 0 firmware.bin
```

### Entering Bootloader Mode

If flash fails:
1. Hold BOOT button
2. Press RESET button
3. Release both
4. Run flash command

## Flashing RP2040 Series

### RP2040/RP2350

1. Hold BOOTSEL button
2. Plug USB cable
3. Drive appears as "RPI-RP2"
4. Copy .uf2 file to the drive
5. Done

## Verification

Connect via serial/Thonny and test:

```python
print("Hello XIAO!")
import machine
print(f"CPU: {machine.freq()} Hz")
```

## Next Steps

After firmware is flashed:

1. [Install Thonny IDE](thonny-ide.md)
2. [Connect to XIAO](../getting-started/esp32c3.md#connect-with-thonny)
3. [Run first script](../getting-started/esp32c3.md#first-script)
