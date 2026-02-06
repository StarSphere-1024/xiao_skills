# USB Drivers for XIAO Boards

## Overview

XIAO boards use USB-Serial chips that may require drivers on some systems.

## XIAO ESP32 Series (C3, C5, C6, S3)

### Windows 10/11

**No drivers needed** - Built-in USB CDC support.

If port not visible:
1. Open Device Manager
2. Look for "Other devices" > "ESP32-S3"
3. Right-click > Update driver
4. Browse my computer > Let me pick
5. Select "USB Serial Device" (or similar)
6. Click Next

### macOS

**No drivers needed** - Built-in support.

### Linux

**No drivers needed** - Built-in support.

Add user to dialout group:
```bash
sudo usermod -a -G dialout $USER
# Log out and back in
```cpp
## XIAO RP2040 / RP2350

### Windows 10/11

**No drivers needed** - USB VID/PID recognized.

### macOS

**No drivers needed** - Built-in support.

### Linux

**udev rules required** for normal user access:

```bash
# Create udev rule
sudo nano /etc/udev/rules.d/90-xiao-rp.rules
```cpp
Add:
```
# RP2040/RP2350
SUBSYSTEM=="usb", ATTR{idVendor}=="2e8a", MODE="0666"
SUBSYSTEM=="usb_device", ATTR{idVendor}=="2e8a", MODE="0666"
```cpp
Reload:
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```cpp
## XIAO nRF52840 / MG24

### Windows 10/11

**Bootloader driver needed** (for UF2 upload):

1. Download from: https://github.com/adafruit/Adafruit_Windows_Drivers/releases
2. Extract zip file
3. Run `install_drivers.bat` as Administrator
4. Click "Install" for each driver
5. Restart computer

### macOS

**No drivers needed** - Built-in support.

### Linux

**udev rules required**:

```bash
sudo nano /etc/udev/rules.d/90-xiao-nrf.rules
```cpp
Add:
```
# nRF52840/MG24
SUBSYSTEM=="usb", ATTR{idVendor}=="239a", MODE="0666"
SUBSYSTEM=="usb_device", ATTR{idVendor}=="239a", MODE="0666"
```cpp
Reload:
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```cpp
## XIAO SAMD21

### Windows

**Driver needed** - Uses ATSAMD21D18A:

1. Install Arduino IDE 1.8.x (includes driver)
   - Or download from: https://www.arduino.cc/en/Main/Software
2. During installation, select "Install USB Driver"
3. Complete installation

### macOS

**No drivers needed** - Built-in support.

### Linux

**udev rules required**:

```bash
sudo nano /etc/udev/rules.d/90-xiao-samd.rules
```cpp
Add:
```
# SAMD21
SUBSYSTEM=="usb", ATTR{idVendor}=="239a", MODE="0666"
```cpp
## XIAO RA4M1

### Windows

**Renesas driver needed**:

1. Download from: https://www.renesas.com/us/en/software-tool/usb-communications-class-device-drivers
2. Install "USB Com Port Device Driver"
3. Restart computer

### macOS

**Driver may be needed** - Use FTDI VCP driver from FTDI website.

### Linux

**Usually works** - May need udev rules similar to SAMD21.

## Troubleshooting Drivers

### Port Shows as COM1 (Windows)

COM1 is usually not the Arduino port. Try:
1. Unplug and replug USB
2. Check Device Manager for new COM port
3. Disable built-in COM1 if present

### "Device Not Recognized" (Windows)

1. Try different USB cable
2. Try different USB port
3. Uninstall driver in Device Manager
4. Scan for hardware changes
5. Reinstall driver

### "Port Not Found" (macOS)

1. Check System Information > USB for device
2. Reset USB controller: `sudo kextunload -r com.apple.driver.AppleUSBMerge`
3. Restart computer

### Permission Denied (Linux)

1. Add user to dialout group: `sudo usermod -a -G dialout $USER`
2. Or run Arduino IDE with sudo: `sudo arduino`
3. Configure udev rules (see above)

### DFU Mode Not Detected

Some boards require DFU mode for upload:
1. Double-click RESET button
2. Yellow LED on RP2040 = bootloader active
3. Drive appears as "RPI-RP2" on computer

## Driver Verification

### Check Driver Installation (Windows)

```
1. Open Device Manager
2. Expand "Ports (COM & LPT)"
3. Look for your board:
   - ESP32: "USB Serial Device" or "Silicon Labs CP210x"
   - RP2040: "RP2 Boot" or "USB Serial Device"
   - nRF52: "Adafruit Feather nRF52840" or "LPC11U35"
```cpp
### Check Driver Installation (Linux)

```bash
# List USB devices
lsusb

# Look for:
# - ESP32: "Espressif USB Serial"
# - RP2040: "Raspberry Pi Pico"
# - nRF52: "Nordic Semiconductor"

# List serial ports
ls -l /dev/ttyUSB* /dev/ttyACM*
```

## Common Driver Sources

| Board | Windows Driver Source |
|-------|----------------------|
| ESP32 Series | Built-in |
| RP2040/RP2350 | Built-in |
| nRF52840/MG24 | Adafruit Windows Drivers |
| SAMD21 | Arduino IDE or Atmel |
| RA4M1 | Renesas USB Drivers |

## Next Steps

After installing drivers:

1. [Verify port is visible](#troubleshooting-drivers)
2. [Configure Arduino IDE](../getting-started/esp32c3.md)
3. [Upload first sketch](../getting-started/esp32c3.md#first-sketch)

## Links

- Adafruit Drivers: https://github.com/adafruit/Adafruit_Windows_Drivers
- CP2102 Drivers: https://www.silabs.com/developers/usb-uart-drivers
- CH340 Drivers: https://github.com/nodemcu/nodemcu-flasher/tree/master/ch341ser
- Renesas USB Drivers: https://www.renesas.com/us/en/software-tool/usb-communications-class-device-drivers
