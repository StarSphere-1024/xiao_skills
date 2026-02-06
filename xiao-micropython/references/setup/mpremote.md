# mpremote CLI Installation Guide

## Overview

mpremote is the official MicroPython CLI tool for remote interaction, file management, and device automation. Essential for testing and deploying MicroPython code to XIAO boards.

## System Requirements

- **Python**: Python 3.7 or newer
- **OS**: Windows 10+, macOS 10.14+, Linux (Ubuntu 20.04+)
- **USB**: For XIAO board connection

## Installation

### Method 1: pip (Recommended)

```bash
# Install for current user
pip install --user mpremote

# Verify installation
mpremote --version
```

### Method 2: pipx (Isolated)

```bash
# Install pipx first (if not available)
pip install pipx

# Install mpremote in isolated environment
pipx install mpremote

# Verify
mpremote --version
```

### Method 3: Run Without Installation

```bash
# Run directly via pipx
pipx run mpremote --help
```

## Initial Connection

### Connect to XIAO

```bash
# Auto-detect and connect
mpremote connect auto

# Or specify port
mpremote connect /dev/ttyUSB0  # Linux
mpremote connect COM3         # Windows

# List available devices
mpremote connect list
```

### Enter REPL

```bash
# Connect and enter REPL
mpremote

# Or explicitly
mpremote repl
```

**REPL Commands:**
- `Ctrl-C` - Interrupt running program
- `Ctrl-D` - Soft reset
- `Ctrl-X` or `Ctrl-]` - Exit REPL

## Common Commands

### File Management

```bash
# List files on device
mpremote fs ls
mpremote ls  # Short form

# Upload file to device
mpremote fs cp local.py :main.py
mpremote cp local.py :  # Short form (copies to root)

# Download file from device
mpremote fs cp :main.py ./
mpremote cp :main.py ./  # Short form

# Delete file on device
mpremote fs rm :main.py
mpremote rm :main.py  # Short form

# Upload directory recursively
mpremote fs cp -r project/ :project/

# Create directory
mpremote mkdir :lib

# View file tree
mpremote tree
```

### Code Execution

```bash
# Run script in RAM (doesn't save to filesystem)
mpremote run script.py

# Execute code string
mpremote exec "print('Hello XIAO!')"

# Evaluate expression
mpremote eval "1 + 1"
# Output: 2

# Execute multiple commands
mpremote cp script.py : + run script.py + reset
```

### Device Management

```bash
# Soft reset (clear heap, restart interpreter)
mpremote soft-reset

# Hard reset (reboot device)
mpremote reset

# Enter bootloader mode
mpremote bootloader

# View disk usage
mpremote df

# Get/set RTC time
mpremote rtc          # Get time
mpremote rtc --set     # Sync with host time
```

### Mount Local Directory

```bash
# Mount local directory as /remote on device
mpremote mount .

# Access device files as if local
# Device files appear in /remote/
ls /remote/

# Unmount when done
mpremote unmount
```

## Code Verification Workflow

### 1. Syntax Check

```bash
# Test script without uploading
mpremote run test.py

# If no errors, syntax is valid
# Script runs in RAM only
```

### 2. Verify Modules

```bash
# Check if module is available
mpremote eval "import machine; print('✅ machine available')"
mpremote eval "import network; print('✅ network available')"

# Check MicroPython version
mpremote eval "import sys; print(sys.version)"
```

### 3. Test Pin Configuration

```bash
# Test specific pin before running full script
mpremote exec "from machine import Pin; Pin(10, Pin.OUT).on()"

# Verify pin responds (e.g., LED turns on)
```

### 4. Deploy Code

```bash
# Upload as main.py (runs on boot)
mpremote fs cp main.py :main.py

# Upload library
mpremote fs cp sensor.py :lib/sensor.py

# Reset device to run main.py
mpremote reset
```

## XIAO-Specific Usage

### ESP32C3/C6/S3

```bash
# Connect to ESP32 XIAO
mpremote connect auto

# Run MicroPython script
mpremote run script.py

# Upload and reset
mpremote fs cp script.py :main.py + reset
```

### RP2040/RP2350

```bash
# Connect to RP2040 XIAO
mpremote connect /dev/ttyACM0  # Linux
mpremote connect COM3          # Windows

# RP2040 enters bootloader automatically
# No special boot mode needed
```

## Package Management (mip)

```bash
# Install package from MicroPython package index
mpremote mip install aiofile

# Install from GitHub
mpremote mip install github:org/repo@branch

# Install to specific location
mpremote mip install --target /lib umqtt.simple
```

## Configuration

### Custom Commands

Create `~/.config/mpremote/config.py`:

```python
commands = {
    "deploy": ["fs", "cp", "main.py", ":", "reset"],
    "test": ["run", "test.py"],
    "list": ["fs", "ls"],
}
```

Use custom commands:
```bash
mpremote deploy  # Equivalent to: mpremote fs cp main.py : + reset
mpremote test    # Equivalent to: mpremote run test.py
```

## Troubleshooting

### "mpremote: command not found"

```bash
# Add to PATH (Linux/macOS)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Windows: Add %USERPROFILE%\AppData\Local\Programs\Python\Python311\Scripts to PATH
```

### "Failed to connect"

```bash
# List available ports
mpremote connect list

# Check device is connected
ls /dev/ttyUSB*  # Linux
Get-PnpDevice -Class Ports  # PowerShell

# Check permissions (Linux)
sudo usermod -a -G dialout $USER
# Log out and back in
```

### "No module named 'xxx'"

```bash
# The module might not be in the firmware
# Check available modules:
mpremote eval "help('modules')"

# Or install via mip if available
mpremote mip install <package>
```

### "OSError: [Errno 5] Input/output error"

**I2C/SPI error:**
- Check I2C/SPI pins are correct for your board
- Use `/xiao` skill to verify pin definitions
- Check pull-up resistors for I2C

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Test MicroPython

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install mpremote
        run: pip install mpremote

      - name: Test syntax
        run: |
          # Syntax check (would need device connected)
          # For CI, use py_compile for basic validation
          python3 -m py_compile main.py
```

### Automated Testing Script

```bash
#!/bin/bash
# test_micropython.sh

SCRIPTS=(
  "main.py"
  "sensor.py"
  "network.py"
)

for script in "${SCRIPTS[@]}"; do
  echo "Testing $script..."

  if mpremote run "$script"; then
    echo "✅ $script passed"
  else
    echo "❌ $script failed"
    exit 1
  fi
done

echo "✅ All tests passed"
```

## Comparison: mpremote vs Thonny

| Feature | mpremote | Thonny |
|---------|----------|--------|
| Interface | CLI | GUI |
| Automation | ✅ Excellent | ⚠️ Limited |
| CI/CD | ✅ Native | ❌ No |
| File Transfer | ✅ Fast | ✅ Easy |
| Debugging | Basic | Advanced |
| REPL | ✅ | ✅ |
| Learning Curve | Medium | Easy |

## When to Use mpremote

**Use mpremote when:**
- Automating deployments
- Running tests in CI/CD
- Batch file operations
- Quick syntax verification
- Remote device management
- Prefer command-line workflows

**Use Thonny when:**
- Learning MicroPython
- Interactive debugging
- Visual file management
- Step-through debugging

## Links

- mpremote Documentation: https://docs.micropython.org/en/latest/reference/mpremote.html
- MicroPython GitHub: https://github.com/micropython/micropython
- Package Index: https://mpy.im/

## Next Steps

After mpremote installation:

1. [Connect to XIAO](#initial-connection)
2. [Run first script](#code-execution)
3. [Verify code syntax](#code-verification-workflow)
4. [Automate deployments](#ci-cd-integration)
