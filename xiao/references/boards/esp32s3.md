# XIAO ESP32S3 / ESP32S3 Sense Reference

**Chipset**: ESP32-S3 (Xtensa dual-core 240MHz)
**Wireless**: WiFi 2.4GHz, BLE 5.0
**Flash**: 8MB (standard) / 16MB (Sense)
**RAM**: 512KB SRAM

## Pin Definitions (Standard & Sense)

| Pin | GPIO | Arduino | Functions | Notes |
|-----|------|---------|-----------|-------|
| D0 | GPIO44 | D0 | - | - |
| D1 | GPIO43 | D1 | - | - |
| D2 | GPIO5 | D2 | - | - |
| D3 | GPIO6 | D3 | - | - |
| D4 | GPIO4 | D4 | I2C SDA | Default I2C SDA (XIAO standard) |
| D5 | GPIO7 | D5 | I2C SCL | Default I2C SCL (XIAO standard) |
| D6 | GPIO8 | D6 | UART TX | Default UART TX (XIAO standard) |
| D7 | GPIO9 | D7 | UART RX | Default UART RX (XIAO standard) |
| D8 | GPIO38 | D8 | SPI SCK | Default SPI SCK (XIAO standard) |
| D9 | GPIO39 | D9 | SPI MISO | Default SPI MISO (XIAO standard), BOOT button |
| D10 | GPIO37 | D10 | SPI MOSI | Default SPI MOSI (XIAO standard) |
| A0 | GPIO1 | A0 | ADC0 | Analog input |
| A1 | GPIO2 | A1 | ADC1 | Analog input |
| A2 | GPIO3 | A2 | ADC2 | Analog input |
| A3 | GPIO4 | A3 | ADC3 | Analog input (shared with D4) |

## XIAO Standard Peripheral Mapping

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | GPIO |
|-----------|-----|------|
| **I2C SDA** | D4 | GPIO4 |
| **I2C SCL** | D5 | GPIO7 |
| **UART TX** | D6 | GPIO8 |
| **UART RX** | D7 | GPIO9 |
| **SPI SCK** | D8 | GPIO38 |
| **SPI MISO** | D9 | GPIO39 |
| **SPI MOSI** | D10 | GPIO37 |

## ESP32S3 Sense Onboard Peripherals

The ESP32S3 Sense version includes additional onboard sensors and peripherals.

### Camera (OV2640/OV5640)

The Sense version has a camera connector that supports OV2640 and OV5640 cameras.

| Camera Signal | ESP32-S3 GPIO | Camera Signal | ESP32-S3 GPIO |
|---------------|--------------|---------------|--------------|
| XMCLK | GPIO10 | DVP_Y8 | GPIO11 |
| DVP_Y7 | GPIO12 | DVP_PCLK | GPIO13 |
| DVP_Y6 | GPIO14 | DVP_Y2 | GPIO15 |
| DVP_Y5 | GPIO16 | DVP_Y3 | GPIO17 |
| DVP_Y4 | GPIO18 | DVP_VSYNC | GPIO38 |
| CAM_SCL | GPIO39 | CAM_SDA | GPIO40 |
| DVP_HREF | GPIO47 | DVP_Y9 | GPIO48 |

**Requirements**: PSRAM must be enabled (Tools > PSRAM: OPI PSRAM)

**For Arduino usage**: See `/xiao-arduino` - Camera examples in `api/esp32-wifi.md` or `examples/`

### PDM Microphone

Built-in PDM microphone for audio input.

| Signal | ESP32-S3 GPIO |
|--------|--------------|
| PDM DATA | GPIO41 |
| PDM CLK | GPIO42 |

**Interface**: I2S (PDM mode)

**For Arduino usage**: See `/xiao-arduino` - `api/microphone.md` for complete API reference

### SD Card Slot

Built-in microSD card slot for storage (supports up to 32GB, FAT32 format).

| SD Card Signal | ESP32-S3 GPIO |
|---------------|--------------|
| CS | GPIO21 |
| CLK | Connected internally |
| MOSI | Connected internally |
| MISO | Connected internally |

**Interface**: SPI

**For Arduino usage**: See `/xiao-arduino` - `libraries/sd.md` for SD card API

## Sense Model Summary

| Feature | Details |
|---------|---------|
| Camera | OV2640/OV5640 support, uses 14 GPIO pins |
| Microphone | Built-in PDM microphone (GPIO41/42) |
| SD Card | Built-in microSD slot (GPIO21 CS) |
| PSRAM | Required for camera operation |
| AI | Built-in AI accelerator for TFLite Micro |

## Communication Interfaces

- **2x UART**: USB Serial + Serial1 (D6/D7)
- **1x I2C**: Hardware I2C on D4/D5
- **1x SPI**: Hardware SPI on D8/D9/D10
- **1x I2S**: Available for audio/camera

## ADC

- 4 channels: A0, A1, A2, A3
- Range: 0-3100mV
- 12-bit resolution (0-4095)

## PWM

All GPIO pins support PWM.

## AI Acceleration

- Built-in AI accelerator
- Supports TensorFlow Lite Micro
- Useful for image recognition, audio processing

## Important Notes

- BOOT button on D9 (GPIO39)
- More GPIO than ESP32C3
- USB-OTG support
- Larger flash for AI models (16MB on Sense)

## Power

- **Input**: 5V via USB
- **Operating Voltage**: 3.3V
- **Deep Sleep**: ~10mA (WiFi off)
- **Light Sleep**: ~3mA
- **Modem Sleep**: ~25mA (WiFi connected)

## Board Size

- 20mm x 17.5mm
- Castellated contacts for soldering

## Platform Support

| Platform | Support |
|----------|---------|
| Arduino | ✅ Full support |
| MicroPython | ✅ Full support |
| ESPHome | ✅ Full support |

For platform-specific usage:
- **Arduino**: See `/xiao-arduino` skill
- **MicroPython**: See `/xiao-micropython` skill
- **ESPHome**: See `/xiao-esphome` skill
