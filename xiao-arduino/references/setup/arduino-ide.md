# Arduino IDE Installation

## System Requirements

- **Windows**: Windows 10 or later
- **macOS**: macOS 10.14 (Mojave) or later
- **Linux**: Ubuntu 20.04 LTS or similar

## Method 1: Arduino IDE 2.x (Recommended)

### Download

1. Visit: https://www.arduino.cc/en/software
2. Download "Arduino IDE 2.x" for your OS
3. Run installer

### Windows Installation

```cpp
1. Run ArduinoIDE-2.x.x.exe
2. Click "I Agree" to license
3. Choose installation path (default: C:\Program Files\Arduino IDE)
4. Click "Install"
5. Wait for installation to complete
6. Click "Close" when done
```

### macOS Installation

```cpp
1. Open ArduinoIDE_2.x.x_macOS.app.dmg
2. Drag Arduino IDE to Applications folder
3. Open Arduino IDE from Applications
4. Accept security prompt if appears
```

### Linux Installation

```bash
# Download AppImage
wget https://downloads.arduino.cc/arduino-ide/arduino-ide_2.3.3_linux_AppImage

# Make executable
chmod +x arduino-ide_2.3.3_linux_AppImage

# Run
./arduino-ide_2.3.3_linux_AppImage
```cpp
## Method 2: Arduino IDE 1.8.x (Legacy)

For compatibility with older projects:

### Download

1. Visit: https://www.arduino.cc/en/software/OldIDDReleases#1.8.x
2. Download "Arduino IDE 1.8.x" for your OS

### Installation (Windows)

```
1. Run arduino-1.8.19-windows.exe
2. Click "I Agree"
3. Choose installation components:
   - Arduino Software
   - USB Driver (important for XIAO boards)
4. Click "Install"
```

## Verify Installation

1. Open Arduino IDE
2. Check **File > Preferences** is accessible
3. Check **Tools > Board** shows boards list
4. Check **Tools > Port** is available

## Next Steps

After Arduino IDE is installed:

1. [Configure Board Manager](../getting-started/esp32c3.md) for XIAO ESP32C3
2. [Install Drivers](drivers.md) if needed
3. Upload your first sketch

## Troubleshooting

### IDE Won't Start (Windows)

1. Install Microsoft Visual C++ Redistributable
   - Download: https://aka.ms/vs/17/release/vc_redist.x64.exe
   - Run installer and restart

### Port Not Visible

1. Check USB cable (data + power, not power-only)
2. Install [drivers](drivers.md)
3. Try different USB port

### "Sketch Too Large" Error

1. Select correct board in Tools > Board
2. Check "Partition Scheme" in Tools
3. Select "Huge App" or "Minimal SPIFFS"

### Upload Failed

1. Hold BOOT button while clicking Upload
2. Try different USB cable
3. Reduce upload speed to 115200 in Tools

## Alternative Editors

### PlatformIO (VS Code Extension)

For advanced users:

1. Install VS Code
2. Install PlatformIO extension
3. Create new project
4. Select board: `seeed_xiao_esp32c3`

See [PlatformIO setup](https://docs.platformio.org/en/latest/integration/ide/vscode.html)

## Links

- Arduino IDE Download: https://www.arduino.cc/en/software
- XIAO Wiki: https://wiki.seeedstudio/XIAO
- ESP32 Arduino: https://github.com/espressif/arduino-esp32
