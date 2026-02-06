---
name: xiao-arduino
description: Complete Arduino development for XIAO boards. Use when writing Arduino sketches for XIAO (ESP32C3/C5/C6/S3, nRF52840, RP2040/RP2350, SAMD21, MG24, nRF54L15, RA4M1). Requires /xiao skill for board-specific pin definitions. Covers board setup, ESP32/nRF/RP specific APIs (WiFi, BLE, PIO), common Arduino libraries (Wire, SPI, Servo, WiFiClient, HTTPClient, PubSubClient), and project examples.
---

# XIAO Arduino Development

## Overview

Complete Arduino development guide for SeeedStudio XIAO series. This skill works with the `/xiao` skill - use `/xiao` for pin definitions, `/xiao-arduino` for platform-specific APIs and libraries.

## Arduino Project Structure (CRITICAL)

**Arduino requires a specific folder/file structure:**

1. **Folder name must match the .ino filename**
   - Example: If your sketch is named `Blink.ino`, it must be in a folder named `Blink`
   - The .ino file is the main file - Arduino IDE automatically generates other files

2. **One .ino file per folder**
   - Each Arduino project (sketch) has its own folder
   - The folder name = the .ino filename (without extension)

```
MyArduinoProjects/
├── BlinkRPM/
│   └── BlinkRPM.ino          # Folder = filename
├── CanBusOBDII/
│   └── CanBusOBDII.ino       # Folder = filename
└── WeatherStation/
    └── WeatherStation.ino    # Folder = filename
```

## Code Generation Workflow (CRITICAL)

**When a user requests Arduino code generation, you MUST:**

### Step 1: Create Project Files
- Use the `Write` tool to create the actual `.ino` file in the appropriate directory
- Create the folder structure matching the Arduino requirements
- Example: Create `XIAO_Expansion_Base_RTC/XIAO_Expansion_Base_RTC.ino`

### Step 2: Verify Compilation (If arduino-cli is Available)
- After creating the file, run `arduino-cli compile` to verify the code compiles
- Use the appropriate FQBN for the target board
- Report compilation results to the user

## Prerequisites

1. **Pin Definitions**: Read `/xiao` skill first for board-specific pin mappings
2. **Arduino IDE or CLI**: Version 2.x/1.8.x IDE, or Arduino CLI
3. **Board Package**: Install via Board Manager or Arduino CLI

### Environment Setup

For first-time Arduino environment installation, see `setup/`:
- **Arduino IDE Installation**: `setup/arduino-ide.md`
- **Arduino CLI Installation**: `setup/arduino-cli.md` (for CI/CD and automation)
- **USB Drivers**: `setup/drivers.md`
- **Troubleshooting**: `setup/troubleshooting.md`

## Quick Start by Board

| Board | Board Manager Package | Getting Started |
|-------|----------------------|-----------------|
| ESP32C3 | "esp32" by Espressif | `getting-started/esp32c3.md` |
| ESP32C5 | "esp32" by Espressif | `getting-started/esp32c5.md` |
| ESP32C6 | "esp32" by Espressif | `getting-started/esp32c6.md` |
| ESP32S3 | "esp32" by Espressif | `getting-started/esp32s3.md` |
| nRF52840 | "Adafruit nRF52" | `getting-started/nrf52840.md` |
| RP2040 | "Raspberry Pi Pico" | `getting-started/rp2040.md` |
| RP2350 | "Raspberry Pi RP2350" | `getting-started/rp2350.md` |
| SAMD21 | "Seeed SAMD Boards" | `getting-started/samd21.md` |
| MG24 | "Seeed nRF52" | `getting-started/mg24.md` |
| nRF54L15 | **NOT SUPPORTED** | See MicroPython skill (`/xiao-micropython`) |
| RA4M1 | "Seeed SAMD Boards" | `getting-started/ra4m1.md` |

## Board-Specific APIs

Each chipset has unique Arduino APIs beyond standard Arduino functions:

### ESP32 Series (C3/C5/C6/S3)
- **WiFi**: See `api/esp32-wifi.md`
- **BLE**: See `api/esp32-ble.md`
- **Hardware**: Timer, PWM, ADC in `api/esp32-hw.md`
- **Deep Sleep**: `api/esp32-sleep.md`

### nRF52 Series (nRF52840, MG24)
- **BLE**: See `api/nrf-ble.md`
- **Deep Sleep**: `api/nrf-sleep.md`

### Sense Boards (ESP32S3 Sense, nRF52840 Sense, MG24 Sense)

**Onboard Peripherals** - Sense versions include built-in sensors:
- **Camera** (ESP32S3 Sense only): `api/camera.md`
- **IMU Sensor** (6-axis accelerometer+gyroscope): `api/imu-sensor.md`
- **Microphone** (PDM/Analog audio): `api/microphone.md`
- **NFC** (nRF52840 only): `api/nfc.md`

**Board Reference** - For Sense-specific pin mappings:
- ESP32S3 Sense: See `xiao/references/boards/esp32s3.md` (camera, mic, SD card)
- nRF52840 Sense: See `xiao/references/boards/nrf52840.md` (IMU, mic, NFC)
- MG24 Sense: See `xiao/references/boards/mg24.md` (IMU, mic)

### RP Series (RP2040, RP2350)
- **PIO**: See `api/rp2040-pio.md`
- **Deep Sleep**: `api/rp-sleep.md`

### SAMD Series (SAMD21, RA4M1)
- **USB**: See `api/samd-usb.md`
- **ADC/DAC**: See `api/samd-adc.md`

## Common Arduino Libraries

### Communication

| Library | Reference |
|---------|-----------|
| Wire (I2C) | `libraries/wire.md` |
| SPI | `libraries/spi.md` |
| Serial/HardwareSerial | `libraries/serial.md` |
| SoftwareSerial | `libraries/softwareserial.md` |

### Networking (ESP32)

| Library | Reference |
|---------|-----------|
| WiFi | `api/esp32-wifi.md` |
| WiFiClient | `libraries/wifi-client.md` |
| HTTPClient | `libraries/http-client.md` |
| PubSubClient (MQTT) | `libraries/mqtt-client.md` |
| WebSockets | `libraries/websocket.md` |

### Actuators

| Library | Reference |
|---------|-----------|
| Servo | `libraries/servo.md` |
| Stepper | `libraries/stepper.md` |

### Storage

| Library | Reference |
|---------|-----------|
| EEPROM | `libraries/eeprom.md` |
| Preferences (ESP32) | `libraries/preferences.md` |
| SPIFFS/LittleFS | `libraries/spiffs.md` |
| SD | `libraries/sd.md` |

### Sensors

| Library | Reference |
|---------|-----------|
| DHT/DHT22 | `libraries/dht.md` |
| Adafruit BME280 | `libraries/bme280.md` |
| Adafruit MPU6050 | `libraries/mpu6050.md` |
| Adafruit Unified Sensor | `libraries/adafruit-sensor.md` |

### Displays

| Library | Reference |
|---------|-----------|
| Adafruit GFX | `libraries/adafruit-gfx.md` |
| Adafruit SSD1306 (OLED) | `libraries/ssd1306.md` |
| Adafruit ST7789 (TFT) | `libraries/st7789.md` |

## Expansion Boards

XIAO expansion boards add peripherals and capabilities. For hardware specifications and pin connections, see `/xiao/references/expansion_boards/`.

| Expansion Board | Key Features | Reference |
|----------------|--------------|-----------|
| Expansion Base | OLED, RTC, SD card, buzzer, Grove connectors | `expansion-boards/expansion-base.md` |
| Round Display | 1.28" touchscreen (240x240) | `expansion-boards/round-display.md` |
| Grove Shield | 8 Grove connectors + battery management | `expansion-boards/grove-shield.md` |
| ePaper V2 | 7 ePaper display sizes (1.54" to 7.5") | `expansion-boards/epaper-v2.md` |
| CAN Bus | MCP2515 controller for automotive/industrial | `expansion-boards/can-bus.md` |
| RS485 | Industrial communication | `expansion-boards/rs485.md` |
| LED Driver | 5V/12V LED strips with WLED support | `expansion-boards/led-driver.md` |
| Bus Servo | Serial bus servo control for robotics | `expansion-boards/bus-servo.md` |
| GPS/GNSS | L76K GNSS module for positioning | `expansion-boards/gps-gnss.md` |
| GPIO Expander | MCP23017 16-bit I/O expansion | `expansion-boards/gpio-expander.md` |
| RGB Matrix | RGB LED matrix driver | `expansion-boards/rgb-matrix.md` |
| COB LED | Chip-on-Board LED lighting | `expansion-boards/cob-led.md` |

**Note:** Hardware documentation is in `/xiao/references/expansion_boards/`. Arduino code examples are in `expansion-boards/` and `expansion-boards/examples/`.

## Code Patterns

### Basic Structure

```cpp
// Includes
#include <Arduino.h>
#include <Wire.h>

// Pin definitions (from /xiao skill)
#define LED_PIN D10
#define SENSOR_PIN A0

// Setup
void setup() {
    Serial.begin(115200);
    pinMode(LED_PIN, OUTPUT);
}

// Loop
void loop() {
    digitalWrite(LED_PIN, HIGH);
    delay(1000);
    digitalWrite(LED_PIN, LOW);
    delay(1000);
}
```

### Non-blocking Delay

```cpp
unsigned long lastUpdate = 0;
const unsigned long interval = 1000;

void loop() {
    if (millis() - lastUpdate >= interval) {
        lastUpdate = millis();
        // Do something
    }
}
```

## Code Verification

**After generating Arduino code, verify it compiles correctly.**

### Quick Verification (If Arduino CLI is Already Set Up)

```bash
# If sketch file already exists, just compile:
arduino-cli compile --fqbn <FQBN> <sketch_dir>

# Example for XIAO ESP32C3:
arduino-cli compile --fqbn esp32:esp32:esp32c3 MySketch
```

### Full Verification (First-Time Setup)

**Step 1: Create Sketch File** (if not exists)

```bash
# Create sketch directory
mkdir MySketch
cd MySketch

# Create main .ino file (must match directory name)
cat > MySketch.ino <<'EOF'
// Your generated code here
EOF
```

**Step 2: Install Required Libraries** (if not already installed)

```bash
# Check if library is installed
arduino-cli lib list

# Install missing libraries
arduino-cli lib install WiFi
arduino-cli lib install Wire
arduino-cli lib install Adafruit_BME280
```

**Step 3: Compile with Arduino CLI**

```bash
# Determine FQBN for target board:
# ESP32C3: esp32:esp32:XIAO_ESP32C3
# ESP32C5: esp32:esp32:XIAO_ESP32C5
# ESP32C6: esp32:esp32:XIAO_ESP32C6
# ESP32S3: esp32:esp32:XIAO_ESP32S3
# nRF52840: Seeeduino:nrf52:xiaonRF52840
# nRF52840 Sense: Seeeduino:nrf52:xiaonRF52840Sense
# RP2040: rp2040:rp2040:seeed_xiao_rp2040
# RP2350: rp2040:rp2040:seeed_xiao_rp2350
# SAMD21: Seeeduino:samd:seeed_XIAO_m0
# MG24: SiliconLabs:silabs:xiao_mg24
# RA4M1: Seeeduino:renesas_uno:XIAO_RA4M1

# Compile sketch
arduino-cli compile --fqbn <FQBN> MySketch
```

**Step 4: Verify Compilation Output**

✅ **Successful compilation:**
```
Sketch uses 123456 bytes (0%) of program storage space.
Global variables use 12345 bytes (0%) of dynamic memory.
```

### Common Errors and Solutions

| Error | Solution |
|-------|----------|
| `No such file or directory` | Install missing library: `arduino-cli lib install <libname>` |
| `'WiFi' was not declared` | Add `#include <WiFi.h>` at top of sketch |
| `expected ';' before '}'` | Check syntax, add missing semicolon |
| `undefined reference to` | Add required library or check function name |

### Verification Checklist

After generating code, verify:

- [ ] All required libraries have `#include` directives
- [ ] Pin definitions match the XIAO board (use `/xiao` skill)
- [ ] FQBN matches the target hardware
- [ ] Code compiles without errors
- [ ] Memory usage is within board limits

### Example: Complete Verification

```cpp
// Blink.ino - Generated for XIAO ESP32C3

#include <Arduino.h>

#define LED_PIN D10  // Built-in LED (from /xiao skill)

void setup() {
    pinMode(LED_PIN, OUTPUT);
    Serial.begin(115200);
}

void loop() {
    digitalWrite(LED_PIN, HIGH);
    delay(1000);
    digitalWrite(LED_PIN, LOW);
    delay(1000);
}
```

**Verification command:**
```bash
arduino-cli compile --fqbn esp32:esp32:esp32c3 Blink
```

## Platform Configuration

**Arduino IDE 2.x**: Tools > Board Options

| Setting | Options | Recommended |
|---------|---------|-------------|
| CPU Frequency | 160MHz, 240MHz | 240MHz for max performance |
| Core Debug Level | None, Verbose | None (release) |
| PSRAM | OPI/QSPI | If available |
| Flash Mode | QIO, QOUT | Default |
| Partition Scheme | Default, OTA | Default for most projects |
| Upload Speed | 921600 | Max stable speed |
| USB CDC On Boot | Enabled, Disabled | Enabled for Serial |

### nRF52 Board Manager Settings

| Setting | Options | Recommended |
|---------|---------|-------------|
| SoftDevice | S140, S132 | S140 for BLE |
| Debug | None, Minimal | None (release) |
| LFO (RC Oscillator) | Enabled, Disabled | Enabled |

## Troubleshooting

### Upload Fails

1. **ESP32**: Hold BOOT button, click Upload, release when "Connecting..."
2. **nRF52**: Double-click RESET button, enters bootloader
3. **RP2040**: Hold BOOTSEL while connecting USB

### Port Not Found

1. Check USB cable (data + power, not power-only)
2. Install drivers:
   - ESP32: CH340/CP2102
   - nRF52: Adafruit nRF52 driver
   - RP2040: No driver needed
3. Try different USB port

### Compilation Errors

1. Update board package
2. Update libraries
3. Check selected board matches hardware

## Resources

### setup/
Environment installation and configuration:
- `setup/arduino-ide.md` - Arduino IDE installation (Windows/macOS/Linux)
- `setup/arduino-cli.md` - Arduino CLI installation and usage (for CI/CD)
- `setup/drivers.md` - USB driver requirements per board
- `setup/troubleshooting.md` - Common Arduino IDE issues and solutions

### getting-started/
Board-specific setup instructions and first steps.

### api/
Chipset-specific Arduino APIs (WiFi, BLE, PIO, etc.)

### libraries/
Common Arduino libraries with XIAO-specific notes.

### examples/
Complete project examples demonstrating best practices.

### expansion-boards/
Arduino implementation for XIAO expansion boards:
- `expansion-boards/expansion-base.md` - Arduino code for Expansion Base (OLED, RTC, SD, buzzer)
- `expansion-boards/round-display.md` - Arduino code for Round Display touchscreen
- `expansion-boards/grove-shield.md` - Arduino code for Grove Shield
- `expansion-boards/epaper-v2.md` - Arduino code for ePaper V2 displays
- `expansion-boards/can-bus.md` - Arduino code for CAN Bus communication
- `expansion-boards/rs485.md` - Arduino code for RS485 communication
- `expansion-boards/led-driver.md` - Arduino code for LED Driver board
- `expansion-boards/bus-servo.md` - Arduino code for Bus Servo control
- `expansion-boards/gps-gnss.md` - Arduino code for GPS/GNSS positioning
- `expansion-boards/gpio-expander.md` - Arduino code for MCP23017 GPIO expander
- `expansion-boards/rgb-matrix.md` - Arduino code for RGB LED matrix
- `expansion-boards/cob-led.md` - Arduino code for COB LED lighting

For hardware specifications, see `/xiao/references/expansion_boards/`.

## Related Skills

- `/xiao` - Core board reference with pin definitions
- `/xiao-micropython` - MicroPython development
- `/xiao-esphome` - ESPHome configuration
