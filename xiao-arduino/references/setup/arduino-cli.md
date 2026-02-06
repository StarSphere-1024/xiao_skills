# Arduino CLI Installation Guide

## Overview

Arduino CLI is the official command-line tool for Arduino development. Ideal for automation, CI/CD, and developers who prefer terminal workflows. This guide covers installation and usage for XIAO boards.

## System Requirements

- **OS**: Windows 10+, macOS 10.14+, Linux (Ubuntu 20.04+)
- **Permissions**: USB device access for uploads
- **Network**: For downloading cores and libraries

## Installation

### Windows

**Method 1: MSI Installer (Recommended)**

1. Download 64-bit MSI: https://downloads.arduino.cc/arduino-cli/arduino-cli_latest_Windows_64bit.msi
2. Run installer
3. Follow installation wizard
4. Verify: `arduino-cli version`

**Method 2: ZIP Download**

1. Download ZIP: https://downloads.arduino.cc/arduino-cli/arduino-cli_latest_Windows_64bit.zip
2. Extract to folder (e.g., `C:\Tools\arduino-cli`)
3. Add to PATH:
   - Search "Environment Variables" in Windows
   - Edit PATH variable
   - Add folder path
4. Restart terminal
5. Verify: `arduino-cli version`

**Method 3: PowerShell Script**

```powershell
# Download and install
Invoke-Expression -Command (Invoke-WebRequest -Uri https://raw.githubusercontent.com/arduino/arduino-cli/master/install.ps1).Content

# Or with specific directory
Invoke-Expression -Command "BINDIR=C:\Tools\arduino-cli; $(Invoke-WebRequest -Uri https://raw.githubusercontent.com/arduino/arduino-cli/master/install.ps1).Content"
```cpp
### macOS

**Method 1: Homebrew (Recommended)**

```bash
brew update
brew install arduino-cli
```cpp
**Method 2: Binary Download**

1. Download for your architecture:
   - Intel (64-bit): https://downloads.arduino.cc/arduino-cli/arduino-cli_latest_macOS_64bit.tar.gz
   - Apple Silicon (ARM64): https://downloads.arduino.cc/arduino-cli/arduino-cli_latest_macOS_ARM64.tar.gz
2. Extract:
   ```bash
   tar -xf arduino-cli_latest_macOS_*.tar.gz
   sudo mv arduino-cli /usr/local/bin/
   ```cpp
3. Verify: `arduino-cli version`

**Method 3: Install Script**

```bash
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh
```cpp
### Linux

**Method 1: Package Manager**

Ubuntu/Debian:
```bash
# Add Arduino repository
sudo apt update
sudo apt install arduino-cli
```cpp
Fedora:
```bash
sudo dnf install arduino-cli
```cpp
**Method 2: Binary Download**

```bash
# Download 64-bit
wget https://downloads.arduino.cc/arduino-cli/arduino-cli_latest_Linux_64bit.tar.gz

# Extract
tar -xf arduino-cli_latest_Linux_64bit.tar.gz

# Install
sudo mv arduino-cli /usr/local/bin/

# Verify
arduino-cli version
```cpp
**Method 3: Install Script**

```bash
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh

# Or with specific directory
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | BINDIR=$HOME/local/bin sh
```cpp
## Initial Configuration

### 1. Initialize Configuration

```bash
arduino-cli config init
```cpp
Creates `~/.arduino15/arduino-cli.yaml` for storing settings.

### 2. Configure Additional Board URLs

Edit `~/.arduino15/arduino-cli.yaml`:

```yaml
board_manager:
  additional_urls:
    - https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
    - https://raw.githubusercontent.com/adafruit/arduino-nrf52/master/package_nrf52_boards_index.json
    - https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json
```cpp
### 3. Update Core Index

```bash
# Update all cores
arduino-cli core update-index

# Or with additional URLs
arduino-cli core update-index --additional-urls https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```cpp
## Installing XIAO Board Cores

### ESP32 Series (ESP32C3/C5/C6/S3)

```bash
# Install ESP32 core
arduino-cli core install esp32:esp32

# Verify installation
arduino-cli core list
```cpp
### nRF52 Series (nRF52840, MG24)

```bash
# Add Adafruit nRF52 URL
arduino-cli core update-index --additional-urls https://raw.githubusercontent.com/adafruit/arduino-nrf52/master/package_nrf52_boards_index.json

# Install core
arduino-cli core install adafruit:nrf52 --additional-urls https://raw.githubusercontent.com/adafruit/arduino-nrf52/master/package_nrf52_boards_index.json
```cpp
### RP Series (RP2040, RP2350)

```bash
# Add Raspberry Pi Pico URL
arduino-cli core update-index --additional-urls https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json

# Install core
arduino-cli core install rp2040:rp2040 --additional-urls https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json
```cpp
### SAMD Series (SAMD21, RA4M1)

```bash
# Add Seeed SAMD URL
arduino-cli core update-index --additional-urls https://files.seeedstudio.com/arduino/package_seeeduino_boards_index.json

# Install core
arduino-cli core install Seeeduino:samd --additional-urls https://files.seeedstudio.com/arduino/package_seeeduino_boards_index.json
```cpp
## Common Commands

### Board Management

```bash
# List connected boards
arduino-cli board list

# List all available boards
arduino-cli board listall

# Attach board to sketch (saves FQBN to sketch.yaml)
arduino-cli board attach -p /dev/ttyUSB0 MySketch
```cpp
### Core Management

```bash
# Search for cores
arduino-cli core search esp32

# Install core
arduino-cli core install esp32:esp32

# List installed cores
arduino-cli core list

# Uninstall core
arduino-cli core uninstall esp32:esp32

# Update cores
arduino-cli core upgrade
```cpp
### Library Management

```bash
# Search for libraries
arduino-cli lib search Servo

# Install library
arduino-cli lib install Servo

# List installed libraries
arduino-cli lib list

# Update library
arduino-cli lib upgrade Servo

# Uninstall library
arduino-cli lib uninstall Servo
```cpp
### Sketch Management

```bash
# Create new sketch
arduino-cli sketch new MyFirstSketch

# Compile sketch
arduino-cli compile --fqbn esp32:esp32:esp32c3 MyFirstSketch

# Upload sketch
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32c3 MyFirstSketch

# Upload with verify
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32c3 --verify MyFirstSketch
```cpp
## XIAO Board FQBNs

| Board | FQBN |
|-------|------|
| ESP32C3 | `esp32:esp32:XIAO_ESP32C3` |
| ESP32C5 | `esp32:esp32:XIAO_ESP32C5` |
| ESP32C6 | `esp32:esp32:XIAO_ESP32C6` |
| ESP32S3 | `esp32:esp32:XIAO_ESP32S3` |
| nRF52840 | `Seeeduino:nrf52:xiaonRF52840` |
| nRF52840 Sense | `Seeeduino:nrf52:xiaonRF52840Sense` |
| RP2040 | `rp2040:rp2040:seeed_xiao_rp2040` |
| RP2350 | `rp2040:rp2040:seeed_xiao_rp2350` |
| SAMD21 | `Seeeduino:samd:seeed_XIAO_m0` |
| MG24 | `SiliconLabs:silabs:xiao_mg24` |
| RA4M1 | `Seeeduino:renesas_uno:XIAO_RA4M1` |

## Compilation Examples

### Basic Compilation

```bash
# Compile for XIAO ESP32C3
arduino-cli compile --fqbn esp32:esp32:esp32c3 MySketch

# Show verbose output
arduino-cli compile --fqbn esp32:esp32:esp32c3 --verbose MySketch

# Save build output
arduino-cli compile --fqbn esp32:esp32:esp32c3 --build-path build MySketch

# Export binaries
arduino-cli compile --fqbn esp32:esp32:esp32c3 --output-dir binaries MySketch
```cpp
### Compilation with Libraries

```bash
# Auto-install missing libraries
arduino-cli compile --fqbn esp32:esp32:esp32c3 --libraries MySketch

# Specify library path
arduino-cli compile --fqbn esp32:esp32:esp32c3 --libraries /path/to/libraries MySketch
```cpp
### Compilation Options

```bash
# Optimize for size
arduino-cli compile --fqbn esp32:esp32:esp32c3 --build-property="compiler.cpp.flags=-Os" MySketch

# Set build properties
arduino-cli compile --fqbn esp32:esp32:esp32c3 --build-property="build.partitions=default" MySketch
```cpp
## Upload Examples

### USB Upload

```bash
# Detect board port
arduino-cli board list

# Upload
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32c3 MySketch

# Upload with input file
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32c3 --input-dir build MySketch
```cpp
### Using sketch.yaml

Create `MySketch/sketch.yaml`:
```yaml
default_fqbn: esp32:esp32:esp32c3
default_port: /dev/ttyUSB0
```cpp
Then compile/upload without specifying FQBN:
```bash
arduino-cli compile MySketch
arduino-cli upload MySketch
```cpp
## Serial Monitor

```bash
# Open monitor (default baud: 9600)
arduino-cli monitor -p /dev/ttyUSB0

# Specify baud rate
arduino-cli monitor -p /dev/ttyUSB0 -c baudrate=115200

# Monitor with echo
arduino-cli monitor -p /dev/ttyUSB0 --echo
```cpp
## CI/CD Integration

### GitHub Actions Example

```yaml
name: Arduino Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install Arduino CLI
        run: |
          curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh

      - name: Install ESP32 Core
        run: |
          arduino-cli core update-index
          arduino-cli core install esp32:esp32

      - name: Install Libraries
        run: |
          arduino-cli lib install WiFi

      - name: Compile Sketch
        run: |
          arduino-cli compile --fqbn esp32:esp32:esp32c3 MySketch
```cpp
### Automated Testing

```bash
#!/bin/bash
# test_compile.sh - Compile all example sketches

SKETCHES=(
  "examples/Blink/Blink.ino"
  "examples/WiFi/WiFi.ino"
  "examples/BLE/BLE.ino"
)

FQBN="esp32:esp32:esp32c3"

for sketch in "${SKETCHES[@]}"; do
  echo "Compiling $sketch..."
  if arduino-cli compile --fqbn $FQBN "$sketch"; then
    echo "✅ $sketch compiled successfully"
  else
    echo "❌ $sketch failed to compile"
    exit 1
  fi
done

echo "✅ All sketches compiled successfully"
```cpp
## Troubleshooting

### "arduino-cli: command not found"

1. Verify installation directory is in PATH
2. Restart terminal
3. Use full path: `/path/to/arduino-cli version`

### "Failed to install core"

```bash
# Check internet connection
# Update index
arduino-cli core update-index

# Clear cache and retry
rm -rf ~/.arduino15/packages/
arduino-cli core install esp32:esp32
```cpp
### "Compilation failed"

```bash
# Enable verbose output
arduino-cli compile --fqbn esp32:esp32:esp32c3 --verbose MySketch

# Check core installation
arduino-cli core list

# Verify libraries
arduino-cli lib list
```cpp
### "Upload failed - permission denied"

**Linux**:
```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Or use sudo (not recommended)
sudo arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32c3 MySketch
```cpp
**Windows**:
1. Install USB drivers (see `drivers.md`)
2. Check Device Manager for COM port
3. Close Arduino IDE if open

### "Board not detected"

```bash
# List all ports
arduino-cli board list

# Check permissions
ls -l /dev/ttyUSB*  # Linux
Get-PnpDevice -Class Ports  # PowerShell

# Try different USB cable
# Press RESET button
```cpp
## Configuration File

### arduino-cli.yaml Location

- **Windows**: `%USERPROFILE%\AppData\Local\Arduino15\arduino-cli.yaml`
- **macOS**: `~/Library/Arduino15/arduino-cli.yaml`
- **Linux**: `~/.arduino15/arduino-cli.yaml`

### Example Configuration

```yaml
# Directories
directories:
  data: ~/.arduino15
  downloads: ~/.arduino15/staging
  user: ~/Arduino

# Logging
logging:
  level: info
  format: text

# Board manager
board_manager:
  additional_urls:
    - https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
    - https://raw.githubusercontent.com/adafruit/arduino-nrf52/master/package_nrf52_boards_index.json
    - https://github.com/earlephilhower/arduino-pico/releases/download/global/package_rp2040_index.json

# Telemetry (disable)
telemetry:
  enabled: false
```cpp
## Update Arduino CLI

### Check for Updates

```bash
arduino-cli version
```cpp
### Update Methods

**Homebrew**:
```bash
brew upgrade arduino-cli
```cpp
**Manual**:
1. Download latest version
2. Replace existing binary
3. Verify: `arduino-cli version`

## Uninstallation

### Windows

```powershell
# MSI installer
# Use "Add or Remove Programs"

# Manual install
# Remove arduino-cli from PATH
# Delete installation directory
```cpp
### macOS

```bash
# Homebrew
brew uninstall arduino-cli

# Manual
sudo rm /usr/local/bin/arduino-cli
rm -rf ~/.arduino15
```cpp
### Linux

```bash
# Package manager
sudo apt remove arduino-cli  # Ubuntu/Debian
sudo dnf remove arduino-cli  # Fedora

# Manual
sudo rm /usr/local/bin/arduino-cli
rm -rf ~/.arduino15
```

## Comparison: Arduino CLI vs IDE

| Feature | CLI | IDE |
|---------|-----|-----|
| Interface | Command-line | GUI |
| Automation | ✅ Excellent | ❌ Limited |
| CI/CD | ✅ Native | ❌ Requires setup |
| Learning Curve | Steeper | Easier |
| Debugging | Basic | Advanced |
| Library Manager | ✅ | ✅ |
| Board Manager | ✅ | ✅ |
| Serial Monitor | ✅ | ✅ |
| Autocomplete | ❌ | ✅ |
| Plugin Support | ❌ | ✅ |

## When to Use Arduino CLI

**Use Arduino CLI when:**
- Building automated CI/CD pipelines
- Compiling multiple sketches in batch
- Running on headless servers
- Prefer terminal workflows
- Need programmatic access to Arduino features

**Use Arduino IDE when:**
- Learning Arduino programming
- Need visual debugging
- Prefer graphical interface
- Using advanced IDE features

## Links

- Arduino CLI GitHub: https://github.com/arduino/arduino-cli
- Arduino CLI Documentation: https://arduino.github.io/arduino-cli/
- Installation Guide: https://arduino.github.io/arduino-cli/latest/installation/
- Command Reference: https://arduino.github.io/arduino-cli/latest/commands/

## Next Steps

After Arduino CLI installation:

1. [Install XIAO board cores](#installing-xiao-board-cores)
2. [Create first sketch](#sketch-management)
3. [Compile and upload](#compilation-examples)
4. [Explore XIAO examples](../examples/)

## Testing Arduino Code with CLI

See `tests/test_arduino_compile.py` for automated compilation testing of XIAO examples.
