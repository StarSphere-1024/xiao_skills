# Arduino IDE Troubleshooting

## Common Installation Issues

### Arduino IDE Won't Install

**Windows Defender blocking:**
1. Allow Arduino IDE in Windows Security
2. Add to exclusion list if needed

**macOS "Unverified Developer":**
1. System Preferences > Security & Privacy
2. Click "Open Anyway" for Arduino IDE

**Linux permission denied:**
```bash
chmod +x arduino-ide_2.x.x_linux_AppImage
```cpp
## Board Manager Issues

### Board Manager Not Loading

1. Check internet connection
2. Check firewall/proxy settings
3. File > Preferences > Additional Boards Manager URLs
4. Remove duplicate URLs
5. Restart Arduino IDE

### ESP32 Package Installation Failed

**"Index download failed":**
```
Solution: Use alternate URL
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```cpp
**"Error downloading https://...":**
```
Solution: Disable antivirus temporarily
Or download manually from GitHub releases
```cpp
### Package Not Appearing After Installation

1. Restart Arduino IDE
2. Check Tools > Board > Board Manager
3. Search for "esp32" or "xiao"

## Upload Issues

### "Failed to Connect"

**ESP32 Series:**
1. Hold BOOT button while clicking Upload
2. Release BOOT when "Connecting..." appears
3. Ensure correct COM port selected

**RP2040/RP2350:**
1. Hold BOOTSEL button while plugging USB
2. Drive appears as "RPI-RP2"
3. Copy .uf2 file to drive

**nRF52/MG24:**
1. Double-click RESET button
2. Red LED pulsing = bootloader mode
3. Upload within 10 seconds

### "Sketch Too Large"

1. **Wrong board selected** - Tools > Board > Select correct XIAO
2. **Reduce partition size** - Not applicable, use correct board
3. **Optimize code** - Remove unused libraries
4. **Disable debug** - Tools > Core Debug Level > None

### "A programmer is required to upload"

1. **Select correct board** - Tools > Board
2. **Check Upload Speed** - Tools > Upload Speed > 921600
3. **Wrong COM port** - Select correct port

### "Permission Denied" (Linux)

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Or run with sudo (not recommended)
sudo arduino

# Or configure udev rules - see drivers.md
```cpp
## Compilation Issues

### "No such file or directory"

**Missing library:**
1. Sketch > Include Library > Manage Libraries
2. Search for missing library
3. Click Install

**Missing #include:**
```cpp
// Add at top of sketch
#include <Wire.h>
#include <SPI.h>
```cpp
### "Multiple Definitions"

**Library conflict:**
1. Check for duplicate .cpp/.h files
2. Remove duplicate library folders
3. Use proper #ifndef guards

### "Expected ';' before..."

**Syntax error:**
1. Check line number in error
2. Add missing semicolon
3. Check for unbalanced braces `{}`

## Port Issues

### Port Not Listed

**Windows:**
1. Device Manager > Ports (COM & LPT)
2. Unplug/replug USB
3. Install drivers if needed
4. Scan for hardware changes

**macOS:**
1. /dev/cu.* and /dev/tty.* should appear
2. Check System Information > USB
3. Reset USB NVRAM if needed

**Linux:**
```bash
# List ports
ls -l /dev/ttyUSB* /dev/ttyACM*

# If no output, install drivers
# Check dmesg for errors
dmesg | tail -20
```cpp
### Wrong Port Selected

1. Unplug other USB devices
2. Plug XIAO board
3. Note which new COM port appears
4. Select that port in Tools > Port

## Board-Specific Issues

### ESP32C3 Boot Failures

**Board not entering bootloader:**
1. Hold BOOT button, press RESET
2. Release both, then upload
3. Or GPIO8 low at boot

**Continuous restart:**
1. Check for short circuit
2. Reduce power supply noise
3. Add 10µF capacitor to EN pin

### RP2040 Boot Failures

**Drive not appearing:**
1. Hold BOOTSEL while plugging USB
2. Try different USB cable
3. Try different USB port
4. Check for physical damage

**Bootloop:**
1. Delete main.py or code.py
2. Or hold BOOTSEL at power-on

### nRF52840 Boot Failures

**Not entering bootloader:**
1. Double-click RESET button quickly
2. Red LED should pulse
3. Upload within 10 seconds

**SoftDevice not found:**
1. Tools > SoftDevice > S140 v7.3.0
2. Or use SoftDevice 132 for smaller code

## Serial Monitor Issues

### No Output

**Wrong baud rate:**
1. Check Serial.begin() speed in code
2. Match Serial Monitor baud rate

**Garbled text:**
1. Mismatched baud rate
2. Wrong line ending setting

**Port busy:**
1. Close Serial Monitor before upload
2. Only one app can use port at a time

## Library Issues

### Library Not Found

**PlatformIO vs Arduino IDE:**
- Arduino libraries: Documents/Arduino/libraries
- PlatformIO: ~/.platformio/libraries

**Manual library install:**
1. Download library ZIP
2. Sketch > Include Library > Add .ZIP Library
3. Navigate to downloaded file

### Library Conflicts

**Duplicate libraries:**
1. Check Documents/Arduino/libraries
2. Remove old versions
3. Keep only latest version

## Performance Issues

### IDE Slow/Laggy

**Reduce memory usage:**
1. File > Preferences > Decrease "Editor font size"
2. Disable "Check for updates on startup"
3. Close unused files

**Compilation slow:**
1. File > Preferences > "Compiler warnings" > None
2. Turn off "Verbose output" during compilation

## Advanced Troubleshooting

### Enable Verbose Output

**During upload:**
1. File > Preferences
2. Check "Show verbose output during: upload"
3. Check console for detailed error messages

**During compilation:**
1. File > Preferences
2. Check "Show verbose output during: compilation"
3. See full compiler commands

### Clear Arduino IDE Preferences

**Reset to defaults:**
1. File > Preferences
2. Note current settings
3. Close Arduino IDE
4. Delete preferences file:
   - Windows: `%APPDATA%\Arduino15`
   - macOS: `~/Library/Arduino15`
   - Linux: `~/.arduino15`
5. Restart Arduino IDE

### Check Board Definitions

1. File > Preferences
2. Note "Arduino IDE preferences.txt" location
3. Open in text editor
4. Look for boards.* entries
5. Delete corrupted entries
6. Restart

## Getting Help

### Debug Information to Collect

1. **Board model**: XIAO ESP32C3 / nRF52840 / etc.
2. **OS**: Windows 10 / macOS 13 / Ubuntu 22.04
3. **Arduino IDE version**: 2.3.3 / 1.8.19
4. **Board package version**: Check in Boards Manager
5. **Error message**: Full text from console

### Useful Resources

- **XIAO Wiki**: https://wiki.seeedstudio/XIAO
- **Arduino Forum**: https://forum.arduino.cc/
- **ESP32 Forum**: https://github.com/espressif/arduino-esp32/issues
- **Seeed Forum**: https://forum.seeedstudio.com/
- **GitHub Issues**: Check individual board repositories

### Minimal Test Case

Create minimal test sketch:

```cpp
void setup() {
    Serial.begin(115200);
    Serial.println("XIAO Test");
}

void loop() {
    Serial.println("Running...");
    delay(1000);
}
```

If this works, issue is with your code, not the board.

## Still Stuck?

1. Try different XIAO board if available
2. Try different computer
3. Test with known-good code
4. Check for hardware damage
5. Contact Seeed support with full details
