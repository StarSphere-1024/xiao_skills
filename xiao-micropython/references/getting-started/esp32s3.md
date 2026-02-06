# MicroPython on XIAO ESP32S3

## Overview

Getting started with MicroPython on SeeedStudio XIAO ESP32S3.

## Installation

### Download Firmware

1. Download MicroPython for ESP32S3: https://micropython.org/download/ESP32_GENERIC_S3/
2. Download `ESP32_GENERIC_S3-*.bin`

### Flash with esptool

```bash
pip install esptool

esptool.py --chip esp32s3 --port COM<X> erase_flash
esptool.py --chip esp32s3 --port COM<X> --baud 460800 write_flash -z 0x0 firmware.bin
```

### Verify Installation

```
Serial: 115200 baud
>>> print("Hello XIAO S3!")
Hello XIAO S3!
>>> import esp
>>> esp.flash_size()
```

## Thonny IDE Setup

1. Download: https://thonny.org/
2. Install and launch
3. Tools > Options > Interpreter > MicroPython (ESP32)
4. Select port and click Connect

## Pin Mapping

| Name | GPIO | Notes |
|------|------|-------|
| D0 | GPIO1 | ADC |
| D1 | GPIO2 | ADC |
| D2 | GPIO3 | ADC (SD CS on Sense) |
| D3 | GPIO4 | ADC |
| D4 | GPIO5 | I2C SDA |
| D5 | GPIO6 | I2C SCL |
| D6 | GPIO43 | UART TX |
| D7 | GPIO44 | UART RX |
| D8 | GPIO7 | SPI SCK |
| D9 | GPIO8 | SPI MISO |
| D10 | GPIO10 | SPI MOSI |
| USER_LED | GPIO21 | User LED |
| BOOT | GPIO0 | BOOT button |

User LED: GPIO21

## First Script

```python
from machine import Pin
import time

led = Pin(21, Pin.OUT)  # USER_LED

while True:
    led.value(not led.value())
    time.sleep(0.5)
```

## Troubleshooting

### Device not found / can't connect

1. Confirm you selected the correct serial port
2. Try a known good data USB cable
3. On Windows, check Device Manager for the COM port

### Flash fails

1. Enter boot mode: hold BOOT, connect USB, then release BOOT
2. Try a lower baud rate (e.g. `--baud 115200`)
3. Run `erase_flash` before `write_flash`

### LED not blinking

1. Confirm you're using `Pin(21)` for `USER_LED`
2. Make sure the script is running (REPL is responsive)

## References

- MicroPython downloads (ESP32-S3): https://micropython.org/download/ESP32_GENERIC_S3/
- MicroPython ESP32 tutorial (install/troubleshooting): https://docs.micropython.org/en/latest/esp32/tutorial/intro.html
- Seeed wiki (XIAO ESP32S3 pin map / user LED): https://wiki.seeedstudio.com/XIAO_ESP32S3_Getting_Started/
