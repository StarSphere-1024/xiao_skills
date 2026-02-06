# XIAO ESP32S3 / ESP32S3 Sense Reference

**Chipset**: ESP32-S3 (Xtensa dual-core 240MHz)
**Wireless**: WiFi 2.4GHz, BLE 5.0
**Flash**: 8MB (standard) / 16MB (Sense)
**RAM**: 512KB SRAM

## Pin Definitions (Standard & Sense)

| Pin | GPIO | Arduino | Functions | Notes |
|-----|------|---------|-----------|-------|
| D0 | GPIO1 | D0 | GPIO, ADC | - |
| D1 | GPIO2 | D1 | GPIO, ADC | - |
| D2 | GPIO3 | D2 | GPIO, ADC | Onboard SD CS (Sense) |
| D3 | GPIO4 | D3 | GPIO, ADC | - |
| D4 | GPIO5 | D4 | I2C SDA, ADC | Default I2C SDA (XIAO standard) |
| D5 | GPIO6 | D5 | I2C SCL, ADC | Default I2C SCL (XIAO standard) |
| D6 | GPIO43 | D6 | UART TX | Default UART TX (XIAO standard) |
| D7 | GPIO44 | D7 | UART RX | Default UART RX (XIAO standard) |
| D8 | GPIO7 | D8 | SPI SCK, ADC | Default SPI SCK (XIAO standard) |
| D9 | GPIO8 | D9 | SPI MISO, ADC | Onboard SD MISO (Sense) |
| D10 | GPIO10 | D10 | SPI MOSI, ADC | Onboard SD MOSI (Sense) |
| D11 | GPIO42 | D11 | GPIO, ADC | Also used for PDM MIC CLK (Sense) |
| D12 | GPIO41 | D12 | GPIO, ADC | Also used for PDM MIC DATA (Sense) |
| BOOT | GPIO0 | BOOT | GPIO | BOOT button |
| USER_LED | GPIO21 | LED_BUILTIN | LED | User LED |
| MTDO | GPIO40 | MTDO | JTAG | - |
| MTDI | GPIO41 | MTDI | JTAG, ADC | Shared with D12 |
| MTCK | GPIO39 | MTCK | JTAG, ADC | - |
| MTMS | GPIO42 | MTMS | JTAG, ADC | Shared with D11 |
| A0 | GPIO1 | A0 | ADC | Same as D0 |
| A1 | GPIO2 | A1 | ADC | Same as D1 |
| A2 | GPIO3 | A2 | ADC | Same as D2 |
| A3 | GPIO4 | A3 | ADC | Same as D3 |

## XIAO Standard Peripheral Mapping

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | GPIO |
|-----------|-----|------|
| **I2C SDA** | D4 | GPIO5 |
| **I2C SCL** | D5 | GPIO6 |
| **UART TX** | D6 | GPIO43 |
| **UART RX** | D7 | GPIO44 |
| **SPI SCK** | D8 | GPIO7 |
| **SPI MISO** | D9 | GPIO8 |
| **SPI MOSI** | D10 | GPIO10 |

## ESP32S3 Sense Onboard Peripherals

The ESP32S3 Sense version includes additional onboard sensors and peripherals.

### Camera (OV2640/OV5640)

The Sense version has a camera connector that supports OV2640 and OV5640 cameras.

| Camera Signal | ESP32-S3 GPIO | Notes |
|---|---:|---|
| XCLK | GPIO10 | Camera-related clock pin |
| Y8 | GPIO11 | Camera video data pin |
| Y7 | GPIO12 | Camera video data pin |
| PCLK | GPIO13 | Camera pixel clock |
| Y6 | GPIO14 | Camera video data pin |
| Y2 | GPIO15 | Camera video data pin |
| Y5 | GPIO16 | Camera video data pin |
| Y3 | GPIO17 | Camera video data pin |
| Y4 | GPIO18 | Camera video data pin |
| CAM_SDA | GPIO40 | I2C data for camera |
| CAM_SCL | GPIO39 | I2C clock for camera |
| VSYNC | GPIO38 | Camera vertical sync |
| HREF | GPIO47 | Camera horizontal sync |
| Y9 | GPIO48 | Camera video data pin |

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
|---|---:|
| CS | GPIO3 |
| SCK | GPIO7 |
| MISO | GPIO8 |
| MOSI | GPIO10 |

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

- BOOT button is GPIO0
- USER LED is GPIO21 (`LED_BUILTIN`)
- Sense model SD card uses GPIO3/7/8/10 (CS/SCK/MISO/MOSI)

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
