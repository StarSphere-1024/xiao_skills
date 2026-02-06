# Sense Models Guide (Camera, Sensors)

## XIAO ESP32S3 Sense

### Camera Support

Camera pin mapping and Arduino examples for XIAO ESP32S3 Sense:

`xiao-arduino:api/esp-camera.md`

### Built-in PDM Microphone

```cpp
#include <driver/i2s.h>

#define I2S_WS 41
#define I2S_SD 42
#define I2S_SCK 40
#define I2S_PORT I2S_NUM_0

void setup() {
    i2s_config_t i2s_config = {
        .mode = (i2s_mode_t)(I2S_MODE_MASTER | I2S_MODE_RX),
        .sample_rate = 16000,
        .bits_per_sample = I2S_BITS_PER_SAMPLE_32BIT,
        .channel_format = I2S_CHANNEL_FMT_ONLY_LEFT,
        .communication_format = I2S_COMM_FORMAT_I2S,
        .intr_alloc_flags = ESP_INTR_FLAG_LEVEL1,
        .dma_buf_count = 8,
        .dma_buf_len = 64,
        .use_apll = false,
        .tx_desc_auto_clear = false,
        .fixed_mclk = 0
    };
    i2s_pin_config_t pin_config = {
        .bck_io_num = I2S_SCK,
        .ws_io_num = I2S_WS,
        .data_out_num = -1,
        .data_in_num = I2S_SD
    };
    i2s_driver_install(I2S_PORT, &i2s_config, 0, NULL);
    i2s_set_pin(I2S_PORT, &pin_config);
}

void loop() {
    // Audio reading code here
}
```

### SD Card (SPI)

| SD Pin | XIAO Pin |
|--------|----------|
| CS | D4 |
| SCK | D8 |
| MOSI | D5 |
| MISO | D10 |

## XIAO nRF52840 Sense

### Built-in PDM Microphone

```cpp
#include <Adafruit_PDM.h>

#define PDM_DATA_PIN 19
#define PDM_CLK_PIN 17

Adafruit_PDM mic = Adafruit_PDM(PDM_DATA_PIN, PDM_CLK_PIN);

void setup() {
    Serial.begin(115200);
    mic.begin();
    mic.setGain(20);
}

void loop() {
    int16_t sample;
    if (mic.available()) {
        sample = mic.read();
        Serial.println(sample);
    }
}
```

## XIAO MG24 Sense

### Built-in Sensors

```cpp
// Temperature sensor (SHT40 or similar)
#include <Adafruit_SHT4x.h>

Adafruit_SHT4x sht4x = Adafruit_SHT4x();

void setup() {
    Serial.begin(115200);
    sht4x.begin();
}

void loop() {
    sensors_event_t humidity, temp;
    sht4x.getEvent(&humidity, &temp);
    Serial.printf("Temp: %.1f°C, Humidity: %.1f%%\n",
                  temp.temperature, humidity.relative_humidity);
    delay(2000);
}
```

## XIAO nRF54L15 Sense

### Built-in Sensors

Similar to MG24 Sense with:
- Temperature sensor
- Humidity sensor
- Ultra-low power operation

## Common Sensor Libraries

| Sensor | Arduino Library | CircuitPython |
|--------|-----------------|---------------|
| SHT40/31 | Adafruit SHT4x | adafruit_sht4x |
| BME280 | Adafruit BME280 | adafruit_bme280 |
| MPU6050 | Adafruit MPU6050 | adafruit_mpu6050 |
| QMC5883L | QMC5883LCompass | adafruit_qmc5883 |

## AI/ML on ESP32S3

### Edge Impulse

1. Train model on Edge Impulse
2. Download EON Tuner model
3. Use ESP-IDF or Arduino library

### TensorFlow Lite Micro

```cpp
#include "tensorflow/lite/micro/micro_interpreter.h"

// Load model
const uint8_t* model_data = g_model;
const int kTensorArenaSize = 200000;
uint8_t tensor_arena[kTensorArenaSize];

// Run inference
// (Full implementation in TensorFlow Lite examples)
```

## Image Classification (ESP32S3 Sense)

```cpp
// Basic object detection using pre-trained model
// See: https://github.com/espressif/esp-face

#include "esp_camera.h"
#include "fd_forward.h"

// Load detection model
// Run inference on camera frames
```

## Audio Classification (ESP32S3 Sense)

```cpp
// Keyword spotting or audio classification
// Uses built-in PDM microphone
// See: https://github.com/Seeed-Studio/Seeed_Arduino_KWS

#include "Seeed_Arduino_KWS.h"

void setup() {
    KWS.init();
    KWS.add_command("hello");
}

void loop() {
    if (KWS.detect("hello")) {
        Serial.println("Detected 'hello'!");
    }
}
```
