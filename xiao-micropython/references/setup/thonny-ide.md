# Thonny IDE Installation for MicroPython

## What is Thonny?

Thonny is a beginner-friendly Python IDE for MicroPython development. Recommended for XIAO MicroPython development.

## System Requirements

- **Windows**: Windows 7 or later
- **macOS**: macOS 10.13 (High Sierra) or later
- **Linux**: Most distributions (Ubuntu, Fedora, etc.)

## Download

Visit: https://thonny.org/

Select the version for your operating system.

## Installation

### Windows

```
1. Download thonny-5.x.x.exe
2. Run installer
3. Click "Install for all users" (recommended)
4. Choose installation location (default: C:\Program Files\Thonny)
5. Click "Install"
6. Wait for completion
7. Click "Finish" to launch Thonny
```

### macOS

```
1. Download thonny-5.x.x.pkg
2. Open PKG file
3. Click "Continue" through installer
4. Click "Install"
5. Close installer when done
6. Open Thonny from Applications
```

### Linux

```bash
# Install pip dependencies
sudo apt install python3-tk python3-dev

# Download Thonny
wget https://github.com/thonny/thonny/releases/download/v5.x.x/thonny-5.x.x-linux-x86_64.tar.gz

# Extract
tar -xzf thonny-5.x.x-linux-x86_64.tar.gz

# Run
cd thonny
./thonny
```

Or install with pip:
```bash
pip3 install thonny
thonny
```

## First Run Configuration

### 1. Choose Language

Select your preferred language on first launch.

### 2. Choose Interface Theme

- "Thonny" (default) - Clean interface
- "IDLE" - Classic Python look

### 3. Select Interpreter

Click "Run" or F5, then:

1. Select "MicroPython (ESP32)"
2. Select "Try to detect port automatically"
3. Click "Install or update MicroPython" (if needed)

See [MicroPython Firmware Guide](esptool.md) for firmware installation.

## Installing MicroPython via Thonny

Thonny can flash MicroPython automatically:

### For ESP32C3/C6/S3

1. **Tools > Options > Interpreter**
2. Click "Install or update MicroPython"
3. Select port (COMx or /dev/ttyUSBx)
4. Select "ESP32-C3/C6/S3" variant
5. Click "Install"
6. Wait for download and flash

### For RP2040

1. Hold BOOTSEL button while plugging USB
2. Drive appears as "RPI-RP2"
3. **Tools > Options > Interpreter**
4. Click "Install or update MicroPython"
5. Thonny detects the drive
6. Click "Install"

## Testing Connection

Create test script:

```python
import machine
import sys

print("Hello XIAO!")
print(f"MicroPython version: {sys.version}")
print(f"Frequency: {machine.freq()} Hz")
```

Click "Run" (F5). Should see output in shell.

## Next Steps

After Thonny installation:

1. [Flash MicroPython Firmware](esptool.md)
2. [Connect to XIAO](#connecting-to-xiao)
3. [Run first MicroPython script](../getting-started/esp32c3.md#first-script)
