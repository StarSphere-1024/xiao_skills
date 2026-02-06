# ESPHome CLI Installation

## Overview

ESPHome Command Line Interface (CLI) is the primary tool for creating, configuring, and uploading ESPHome firmware to XIAO boards.

## System Requirements

- **Python**: Python 3.8 or newer
- **OS**: Windows 10+, macOS 10.14+, Linux (Ubuntu 20.04+)
- **USB**: For XIAO board connection

## Prerequisites

### Install Python

**Windows:**
1. Download: https://www.python.org/downloads/
2. Run installer
3. ✅ Check "Add Python to PATH"
4. Click "Install Now"

**macOS:**
```bash
# Homebrew (recommended)
brew install python@3.11

# Or download from python.org
```

**Linux:**
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### Verify Python

```bash
python --version
# Should be 3.8 or higher
```

## Installation Methods

### Method 1: pip (Recommended)

```bash
# Install ESPHome
pip install esphome

# Verify installation
esphome version
```

### Method 2: Virtual Environment (Recommended for Linux)

```bash
# Create virtual environment
python3 -m venv ~/esphome
source ~/esphome/bin/activate

# Install ESPHome
pip install esphome

# Add to PATH (optional)
echo 'source ~/esphome/bin/activate' >> ~/.bashrc
```

### Method 3: Docker (Cross-platform)

```bash
# Pull ESPHome Docker image
docker pull esphome/esphome

# Run ESPHome commands
docker run --rm -v ~/.esphome:/config -it esphome/esphome version
```

## First Run Configuration

### Wizard Configuration

```bash
esphome wizard
```

The wizard will guide you through:
1. Creating configuration directory (`~/.esphome`)
2. Setting up MQTT (Home Assistant)
3. Configuring upload options

### Manual Configuration

Create config manually:

```bash
mkdir ~/esphome
cd ~/esphome
```

Create `xiao-sensor.yaml`:
```yaml
esphome:
  name: xiao-sensor
  friendly_name: XIAO Sensor

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

api:
  password: !secret api_password

ota:
  password: !secret ota_password
```

## Secrets Configuration

ESPHome uses secrets for sensitive data:

```bash
# Create secrets file
cd ~/.esphome
cat > secrets.yaml <<EOF
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"
api_password: "YourAPIPassword"
ota_password: "YourOTAPassword"
EOF
```

## Basic Commands

### Validate Configuration

```bash
esphome config xiao-sensor.yaml
```

### Compile Firmware

```bash
esphome compile xiao-sensor.yaml
```

### Upload via USB

```bash
esphome run xiao-sensor.yaml
```

### Upload via OTA (After First Upload)

```bash
esphome upload xiao-sensor.yaml
```

### Clean Build

```bash
esphome clean xiao-sensor.yaml
esphome compile xiao-sensor.yaml
```

## XIAO Board Configuration

### ESP32C3

```yaml
esphome:
  name: xiao_esp32c3
  platform: ESP32
  board: esp32-c3-devkitm-1
```

### ESP32C6

```yaml
esphome:
  name: xiao_esp32c6
  platform: ESP32
  board: esp32-c6-devkitm-1
```

### ESP32S3

```yaml
esphome:
  name: xiao_esp32s3
  platform: ESP32
  board: esp32-s3-devkitm-1
```

### RP2040

```yaml
esphome:
  name: xiao_rp2040
  platform: RP2040
  board: rpipicow
  framework:
    - arduino
```

## Uploading to XIAO

### USB Upload (First Time)

1. Connect XIAO via USB
2. Run: `esphome run xiao-sensor.yaml`
3. Follow prompts:
   - Connect board to computer
   - Press RESET button (if prompted)
   - Wait for upload completion

### OTA Upload (Subsequent)

After initial upload:

```bash
esphome upload xiao-sensor.yaml
```

### Manual Boot Mode

If auto-upload fails:

**ESP32C3/C6:**
1. Hold BOOT button
2. Plug USB
3. Release BOOT
4. Run upload command

**ESP32S3:**
1. Hold BOOT button
2. Plug USB
3. Release BOOT
4. Run upload command

## Troubleshooting

### "esphome: command not found"

```bash
# Reinstall with pip
pip install --upgrade esphome

# Or use full path
python -m esphome version
```

### "No module named 'esptool'"

```bash
# Install esptool
pip install esphome esptool

# Or for ESP32-C6 specifically
pip install esphome-c6
```

### "Permission denied" (Linux)

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Log out and back in
```

### Upload Timeout

1. Check USB cable (data + power)
2. Enter bootloader mode manually
3. Reduce upload speed:
   ```yaml
   esphome:
     platformio_options:
       - build_flags = "-DARDUINO_USB_MODE=1"
       - build_flags = "-DARDUINO_USB_CDC_ON_BOOT=1"
   ```

### WiFi Not Connecting

1. Check SSID spelling
2. Verify 2.4GHz network (ESP32 doesn't support 5GHz)
3. Check password in secrets.yaml
4. Enable logging:
   ```yaml
   logger:
     level: DEBUG
   ```

## Integration with Home Assistant

### Add ESPHome Integration

1. Home Assistant > Settings > Devices & Services
2. Click "Add Integration"
3. Search for "ESPHome"
4. Click "Configure"
5. ESPHome devices will be auto-discovered

### Verify Connection

In Home Assistant:
```
Settings > Devices & Services > ESPHome
```

Should show your XIAO device as "Connected".

## Updating ESPHome

```bash
pip install --upgrade esphome
esphome version
```

## Uninstallation

```bash
pip uninstall esphome

# Remove config if desired
rm -rf ~/.esphome
```

## Alternative: ESPHome Add-on (Home Assistant)

If running Home Assistant Supervisor:

1. Settings > Add-ons > Add-on Store
2. Search "ESPHome"
3. Click "Install"
4. Start add-on
5. Open web UI

**Advantages:**
- Integrated with Home Assistant
- No Python installation needed
- Auto-starts with HA

**Disadvantages:**
- Limited to Home Assistant OS
- Less command-line control

## Links

- ESPHome Website: https://esphome.io/
- ESPHome Docs: https://esphome.io/guides/
- ESPHome GitHub: https://github.com/esphome/esphome
- XIAO Wiki: https://wiki.seeedstudio/XIAO

## Next Steps

After ESPHome CLI installation:

1. [Create first configuration](#first-run-configuration)
2. [Upload to XIAO](#uploading-to-xiao)
3. [Integrate with Home Assistant](#integration-with-home-assistant)
4. [Explore example configurations](../examples/weather-station.md)
