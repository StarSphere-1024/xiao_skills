# XIAO ESP32S3 ESPHome Configuration

## Base Configuration

Complete ESPHome configuration for XIAO ESP32S3.

```yaml
# XIAO ESP32S3 Base Configuration
substitutions:
  device_name: "xiao-esp32s3"
  friendly_name: "XIAO ESP32S3"
  device_description: "SeeedStudio XIAO ESP32S3"

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  comment: ${device_description}
  platform: ESP32
  board: esp32-s3-devkitc-1
  project:
    name: "SeeedStudio.XIAO_ESP32S3"
    version: "1.0.0"

# Enable logging
logger:
  level: INFO
  baud_rate: 0

# Enable Home Assistant API
api:
  encryption:
    key: !secret api_encryption_key
  password: !secret api_password

# Enable OTA updates
ota:
  - platform: esphome
    password: !secret ota_password

# WiFi
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: true
  power_save_mode: none

  ap:
    ssid: "${device_name} Fallback"
    password: !secret ap_password

captive_portal:
web_server:
  port: 80
  auth:
    username: admin
    password: !secret web_password

mdns:

# Status LED (built-in white LED)
status_led:
  pin:
    number: GPIO39
    inverted: true

# I2C Bus (D6=SDA, D7=SCL)
i2c:
  sda: GPIO7
  scl: GPIO6
  scan: true
  frequency: 100kHz

# SPI Bus (D8=SCK, D9=MISO, D10=MOSI)
spi:
  clk_pin: GPIO8
  miso_pin: GPIO9
  mosi_pin: GPIO10

# UART (D0=TX, D1=RX)
uart:
  tx_pin: GPIO43
  rx_pin: GPIO44
  baud_rate: 9600

# BOOT button (GPIO0)
binary_sensor:
  - platform: gpio
    name: "Button"
    pin:
      number: GPIO0
      inverted: true
      mode:
        input: true
        pullup: true

# Internal sensors
sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 60s

  - platform: uptime
    name: "Uptime"

  - platform: esp32_temperature
    name: "Internal Temperature"
    update_interval: 60s

text_sensor:
  - platform: wifi_info
    ip_address:
      name: "IP Address"
    ssid:
      name: "Connected SSID"

  - platform: version
    name: "ESPHome Version"

switch:
  - platform: restart
    name: "Restart"
```

## Pin Mapping Reference

| XIAO Pin | GPIO | ESPHome Pin | Function |
|----------|------|-------------|----------|
| D0 | GPIO1 | GPIO1 | ADC |
| D1 | GPIO2 | GPIO2 | ADC |
| D2 | GPIO3 | GPIO3 | ADC (SD CS on Sense) |
| D3 | GPIO4 | GPIO4 | ADC |
| D4 | GPIO5 | GPIO5 | I2C SDA |
| D5 | GPIO6 | GPIO6 | I2C SCL |
| D6 | GPIO43 | GPIO43 | UART TX |
| D7 | GPIO44 | GPIO44 | UART RX |
| D8 | GPIO7 | GPIO7 | SPI SCK |
| D9 | GPIO8 | GPIO8 | SPI MISO |
| D10 | GPIO10 | GPIO10 | SPI MOSI |
| BOOT | GPIO0 | GPIO0 | BOOT button |
| USER_LED | GPIO21 | GPIO21 | User LED |

User LED: GPIO21

## Camera Configuration (ESP32S3 Sense)

```yaml
esp32_camera:
  name: "XIAO Camera"
  external_clock:
    pin: GPIO10
    frequency: 20MHz
  i2c_pins:
    sda: GPIO40
    scl: GPIO39
  data_pins:
    - GPIO15
    - GPIO17
    - GPIO18
    - GPIO16
    - GPIO14
    - GPIO12
    - GPIO11
    - GPIO48
  vsync_pin: GPIO38
  href_pin: GPIO47
  pclk_pin: GPIO13
  resolution: 1024x768
  jpeg_quality: 10
  max_framerate: 10fps
```

## Board Specifications

| Feature | Value |
|---------|-------|
| MCU | ESP32-S3 |
| Core | Xtensa dual-core @ 240MHz |
| Flash | 8 MB |
| RAM | 512 KB |
| WiFi | 802.11b/g/n |
| BLE | Bluetooth 5.0 |
| GPIO | 11 pins |
| ADC | 12-bit, 4 channels |
| AI | 8 MB PSRAM (Sense model) |

## ESP32S3 Sense Specific Features

### Camera Web Server

```yaml
esp32_camera_web_server:
  - port: 8080
    mode: stream
```

### Face Recognition

```yaml
esp32_face_recognition:
  id: face_rec
  source: esp32_camera
```

### Image Classification

```yaml
image_provider:
  - platform: esp32_camera
    id: cam_image

sensor:
  - platform: image_classifier
    source: cam_image
    model: mobilenet_v1
    confidence: 0.6
```

## Dual Core Configuration

```yaml
# Use second core for heavy tasks
esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: arduino

# Pin to second core
interval:
  - interval: 1s
    then:
      - lambda: |-
          // Run on core 1
          xTaskCreatePinnedToCore(
            [](void* pvParameters) {
              // Heavy computation here
            },
            "task1",
            4096,
            NULL,
            1,
            NULL,
            1  // Core 1
          );
```

## PSRAM Usage

```yaml
# Enable PSRAM for large buffers
esp32:
  psram: true
  framework:
    type: arduino

# Use PSRAM for camera
esp32_camera:
  psram: true
  grab_mode: esp32_camera.GRAB_LATEST
```

## Complete Example: Camera with Motion Detection

```yaml
substitutions:
  device_name: "xiao-camera"

esphome:
  name: ${device_name}
  platform: ESP32
  board: esp32-s3-devkitc-1

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

api:
ota:
logger:

esp32_camera:
  name: "XIAO Camera"
  external_clock:
    pin: GPIO15
    frequency: 20MHz
  i2c_pins:
    sda: GPIO4
    scl: GPIO5
  data_pins:
    - GPIO34
    - GPIO13
    - GPIO14
    - GPIO35
    - GPIO36
    - GPIO37
    - GPIO38
    - GPIO12
  vsync_pin: GPIO6
  href_pin: GPIO7
  pclk_pin: GPIO16
  resolution: 1024x768
  jpeg_quality: 10
  max_framerate: 10fps

# Motion detection
esp32_camera_web_server:
  - port: 8080
    mode: stream

binary_sensor:
  - platform: gpio
    name: "Motion"
    pin:
      number: GPIO18
      mode: INPUT_PULLUP
    device_class: motion
```

## BLE Configuration

```yaml
esp32_ble:
  on_ble_advertise:
    then:
      - logger.log: "BLE advertising"

ble_tracker:
  scan_interval: 60s
  on_ble_advertise:
    then:
      - logger.log: "Found BLE device"
```

## Voice Assistant (ESP32S3)

```yaml
microphone:
  - platform: esp32_adf_i2s
    id: mic
    adc_type: external
    i2s_din_pin: GPIO6
    sample_rate: 16000
    bits_per_sample: 16bit

voice_assistant:
  microphone: mic
  speaker: out_speaker
  on_listening:
    then:
      - light.turn_on: status_led
  on_stopped:
    then:
      - light.turn_off: status_led

speaker:
  - platform: esp32_adf_i2s
    id: out_speaker
    adc_type: external
    i2s_dout_pin: GPIO7
    sample_rate: 16000
```

## Troubleshooting

### Camera Issues

1. Check all GPIO connections
2. Verify I2C connections (GPIO4/5 for camera config)
3. Check PSRAM is enabled
4. Reduce resolution if timing errors

### Boot Issues

1. Check boot mode pins
2. Verify flash mode (QIO)
3. Try different USB cable

### PSRAM Detection

```yaml
text_sensor:
  - platform: template
    name: "PSRAM"
    lambda: |-
      if (psram_found()) {
        return {"PSRAM detected"};
      } else {
        return {"No PSRAM"};
      }
```

## Performance Tips

1. **Use PSRAM** for large buffers
2. **Dual core** for parallel tasks
3. **AI acceleration** with ESP-DL
4. **Camera**: Use lower resolution for faster frame rate
5. **BLE**: Use for low-power applications

## Advanced Features

### Custom AI Model

```yaml
# Import custom TensorFlow Lite model
sensor:
  - platform: tflite
    model: custom_model.tflite
    input:
      - camera_image
    output:
      - class_id
      - confidence
```

### Neural Network Acceleration

```yaml
# Use ESP-DL for neural network acceleration
esp32_dl:
  model: mobilenet_v1
  accelerator: true
```

## Migration from ESP32C3

1. Update `platform: ESP32`
2. Change `board: esp32-s3-devkitc-1`
3. Update pin mappings
4. Enable PSRAM if available
5. Add camera config if using Sense model
