# Bootloader Reflash Procedures

## ESP32 Series (C3/C5/C6/S3)

### Prerequisites
- Download ESP RF Test Tool: https://www.espressif.com/en/support/download/other-tools
- Download factory firmware (board-specific)

### Steps

1. **Enter Bootloader Mode**
   - Hold BOOT button
   - Connect USB to PC
   - Release BOOT button

2. **Open ESP RF Test Tool**
   - Select Chip Type (ESP32C3/C5/C6/S3)
   - Select COM port
   - Set Baud Rate to 115200
   - Click "Open"

3. **Flash Factory Firmware**
   - Select "Flash" tab
   - Click "Select Bin"
   - Choose factory firmware file
   - Click "Load Bin"
   - Wait for completion

### Factory Firmware URLs
- ESP32C3: https://files.seeedstudio.com/wiki/XIAO_WiFi/Resources/ESP32-C3_RFTest_108_2b9b157_20211014.bin

## nRF52840

### Method 1: UF2 Bootloader (Default)
1. Double-click RESET button to enter bootloader
2. RPI-RP2 drive appears
3. Drag and drop UF2 file

### Method 2: nRFutil Recovery
```bash
nrfutil pkg generate --hw-version 52 --sd-req 0x00C4 --application firmware.hex --application-version 1 output.zip
nrfutil dfu usb-serial -pkg output.zip -p /dev/ttyUSB0
```

## RP2040

### Standard Bootloader Entry
1. Hold BOOTSEL button
2. Connect USB (or press RESET while holding)
3. RPI-RP2 drive appears
4. Drag and drop .uf2 file

### Reinstall Bootloader
```bash
# Install picotool
picotool load bootloader.uf2

# Or use OpenOCD
openocd -f interface/rp2040.cfg -f target/rp2040.cfg -c "program bootloader.bin verify reset exit"
```

## SAMD21

### Method 1: Arduino IDE
1. Select "Seeed XIAO SAMD21" board
2. Tools > Burn Bootloader

### Method 2: OpenOCD + edbg
```bash
edbg -bpv -t samd21 -e -pv -f firmware.bin
```

## MG24

### Method 1: Arduino IDE
1. Install Seeed nRF52 boards
2. Select board and burn bootloader

### Method 2: J-Link
```bash
JLinkExe -device MGM240 -if SWD -speed 4000 -autoconnect 1
loadbin bootloader.bin 0x0
r
g
exit
```

## Troubleshooting

### ESP32 won't enter bootloader
- Try holding BOOT button before USB connection
- Try different USB cable (power-only cables won't work)
- Check drivers: Install CH340 or CP2102 drivers

### nRF52840 not detected
- Install Adafruit nRF52 drivers
- Try double-click RESET button rapidly
- Check Windows Device Manager for DFU device

### RP2040 not showing as drive
- Hold BOOTSEL while connecting USB
- Try different USB port
- Check cable (data + power required)

### Upload fails
- Check correct board selected
- Check correct COM port
- Try pressing BOOT button during upload
- Check for antivirus blocking

## Factory Reset

Most XIAO boards can be factory reset by:
1. Reflash bootloader (above)
2. Clear EEPROM/Flash:
   - ESP32: `ESP.eraseFlash()`
   - nRF52: `NRF.clearPreferences()`
   - RP2040: Delete all files via USB MSC
