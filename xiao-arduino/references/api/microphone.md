# Microphone API

## Overview

XIAO Sense boards feature built-in microphones for audio capture:

| Board | Microphone Type | Interface | Library |
|-------|----------------|-----------|---------|
| ESP32S3 Sense | PDM | I2S (GPIO41/42) | I2S.h / ESP_I2S.h |
| nRF52840 Sense | PDM | PDM (D2/D3) | Seeed_Arduino_Mic |
| MG24 Sense | Analog (I2S) | ADC (PC9/PC8) | Seeed_Arduino_Mic / SilabsMicrophoneAnalog |

## ESP32S3 Sense - PDM Microphone

The ESP32S3 Sense has a built-in PDM microphone on GPIO41 (DATA) and GPIO42 (CLK).

### Basic Setup (ESP32 2.0.x - I2S.h)

```cpp
#include <I2S.h>

void setup() {
    Serial.begin(115200);

    // Configure PDM microphone pins
    I2S.setAllPins(-1,  // MCLK
                   42,  // SCK (BCLK/CLK)
                   41,  // SD (DATA)
                   -1,  // FS (WS/LRCK)
                   -1); // amp

    // Initialize I2S in PDM mono mode
    if (!I2S.begin(PDM_MONO_MODE, 16000, 16)) {
        Serial.println("Failed to initialize I2S!");
        while (1);
    }

    Serial.println("PDM Microphone initialized");
}

void loop() {
    int16_t sample = I2S.read();
    Serial.println(sample);
    delay(10);
}
```

### ESP32 3.0.x (ESP_I2S.h)

```cpp
#include <ESP_I2S.h>

I2SClass i2s;

void setup() {
    Serial.begin(115200);

    i2s.setPins(41, 42, -1, -1, -1);
    i2s.begin(I2S_MODE_PDM, 16000, 16, I2S_SLOT_MONO);
    Serial.println("Microphone ready");
}

void loop() {
    int16_t sample;
    size_t bytesRead;
    i2s.read(&sample, sizeof(sample), &bytesRead);
    if (bytesRead > 0) {
        Serial.println(sample);
    }
}
```

### Configuration Options

```cpp
// Sample rates: 16000, 32000, 44100, 48000 Hz
// Bit depths: 16 or 32 bit
I2S.begin(PDM_MONO_MODE, 16000, 16);

// Or stereo mode
I2S.begin(PDM_STEREO_MODE, 16000, 16);
```

## nRF52840 Sense - PDM Microphone

The nRF52840 Sense has a PDM microphone on D2 (CLK) and D3 (DATA).

### Basic Setup

```cpp
#include <mic.h>

#define SAMPLES 800

mic_config_t mic_config{
    .channel_cnt = 1,
    .sampling_rate = 16000,
    .buf_size = 1600,
    .debug_pin = LED_BUILTIN
};

NRF52840_ADC_Class Mic(&mic_config);
int16_t recording_buf[SAMPLES];
volatile bool record_ready = false;

void audio_rec_callback(uint16_t *buf, uint32_t buf_len) {
    static uint32_t idx = 0;
    for (uint32_t i = 0; i < buf_len; i++) {
        recording_buf[idx++] = buf[i];
        if (idx >= SAMPLES) {
            idx = 0;
            record_ready = true;
            break;
        }
    }
}

void setup() {
    Serial.begin(115200);
    Mic.set_callback(audio_rec_callback);

    if (!Mic.begin()) {
        Serial.println("Mic initialization failed");
        while (1);
    }
    Serial.println("Microphone ready - Open Serial Monitor to start");
}

void loop() {
    if (record_ready) {
        Serial.println("Audio captured:");
        for (int i = 0; i < SAMPLES; i++) {
            Serial.println(recording_buf[i]);
        }
        record_ready = false;
    }
}
```

### Configuration Structure

```cpp
mic_config_t {
    .channel_cnt = 1,        // 1 = mono, 2 = stereo
    .sampling_rate = 16000,   // Hz: 8000, 16000, 32000, 44100
    .buf_size = 1600,        // Buffer size in samples
    .debug_pin = LED_BUILTIN // Pin toggles during ISR
}
```

### Visualizing Audio Data

```cpp
void loop() {
    if (record_ready) {
        // Print for Serial Plotter
        for (int i = 0; i < SAMPLES; i++) {
            Serial.println(recording_buf[i]);
        }
        record_ready = false;
    }
}
```

## MG24 Sense - Analog Microphone

The MG24 Sense has a MEMS analog microphone on PC9 (DATA) and PC8 (PWR).

### Seeed Studio Demo

```cpp
#include <mic.h>

#define SAMPLES 800

mic_config_t mic_config{
    .channel_cnt = 1,
    .sampling_rate = 16000,
    .buf_size = 1600,
    .debug_pin = LED_BUILTIN
};

MG24_ADC_Class Mic(&mic_config);
int16_t recording_buf[SAMPLES];
volatile bool record_ready = false;

void audio_rec_callback(uint16_t *buf, uint32_t buf_len) {
    static uint32_t idx = 0;
    for (uint32_t i = 0; i < buf_len; i++) {
        recording_buf[idx++] = buf[i];
        if (idx >= SAMPLES) {
            idx = 0;
            record_ready = true;
            break;
        }
    }
}

void setup() {
    Serial.begin(115200);
    Mic.set_callback(audio_rec_callback);

    if (!Mic.begin()) {
        Serial.println("Mic initialization failed");
        while (1);
    }
    Serial.println("Microphone ready");
}

void loop() {
    if (record_ready) {
        for (int i = 0; i < SAMPLES; i++) {
            Serial.println(recording_buf[i]);
        }
        record_ready = false;
    }
}
```

### Silicon Labs Demo (SDK 2.3.0+)

```cpp
#include <SilabsMicrophoneAnalog.h>

#define MIC_DATA_PIN  PC9
#define MIC_PWR_PIN   PC8
#define NUM_SAMPLES   128

uint32_t mic_buffer[NUM_SAMPLES];
uint32_t mic_buffer_local[NUM_SAMPLES];
volatile bool data_ready_flag = false;

MicrophoneAnalog micAnalog(MIC_DATA_PIN, MIC_PWR_PIN);

void mic_samples_ready_cb() {
    memcpy(mic_buffer_local, mic_buffer, NUM_SAMPLES * sizeof(uint32_t));
    data_ready_flag = true;
}

void setup() {
    Serial.begin(115200);
    pinMode(LED_BUILTIN, OUTPUT);

    micAnalog.begin(mic_buffer, NUM_SAMPLES);
    Serial.println("Microphone initialized...");

    micAnalog.startSampling(mic_samples_ready_cb);
    Serial.println("Sampling started...");
}

void loop() {
    if (data_ready_flag) {
        data_ready_flag = false;

        // Get average volume level
        uint32_t voice_level = micAnalog.getAverage(mic_buffer_local, NUM_SAMPLES);

        // Constrain and map to LED brightness
        uint32_t level = constrain(voice_level, 735, 900);
        int brightness = map(level, 735, 900, 0, 255);

        // Control LED based on volume
        if (LED_BUILTIN_ACTIVE == LOW) {
            analogWrite(LED_BUILTIN, 255 - brightness);
        } else {
            analogWrite(LED_BUILTIN, brightness);
        }

        Serial.print("Voice level: "); Serial.println(voice_level);

        // Restart sampling
        micAnalog.startSampling(mic_samples_ready_cb);
    }
}
```

## Common Audio Processing

### Calculate Volume Level

```cpp
int calculateVolume(int16_t *buffer, int size) {
    int64_t sum = 0;
    for (int i = 0; i < size; i++) {
        sum += abs(buffer[i]);
    }
    return sum / size;
}

void loop() {
    if (record_ready) {
        int volume = calculateVolume(recording_buf, SAMPLES);
        Serial.print("Volume: "); Serial.println(volume);
        record_ready = false;
    }
}
```

### Simple Threshold Detection

```cpp
#define THRESHOLD 500

void loop() {
    if (record_ready) {
        int maxSample = 0;
        for (int i = 0; i < SAMPLES; i++) {
            if (abs(recording_buf[i]) > maxSample) {
                maxSample = abs(recording_buf[i]);
            }
        }

        if (maxSample > THRESHOLD) {
            Serial.println("Sound detected!");
        }

        record_ready = false;
    }
}
```

### Send Audio via Serial

```cpp
void loop() {
    if (record_ready) {
        // Send as binary
        Serial.write((uint8_t*)recording_buf, SAMPLES * sizeof(int16_t));
        record_ready = false;
    }
}
```

## Troubleshooting

### ESP32S3 Sense - No Audio

1. Check ESP32 version - I2S API differs between 2.0.x and 3.0.x
2. Verify GPIO pins: GPIO41 (DATA), GPIO42 (CLK)
3. Check sample rate - try 16000 Hz
4. Enable PSRAM in Tools > PSRAM

### nRF52840 Sense - No Audio

1. Must open Serial Monitor after upload to start
2. Check pins: D2 (CLK/P0.28), D3 (DATA/P0.29)
3. Ensure Seeed nRF52 mbed-enabled Boards library is used
4. Open Serial Monitor to trigger audio capture

### MG24 Sense - No Audio (Seeed Studio Demo)

1. May need to replace `gsdk.a` file in board package
2. Path: `Arduino15/packages/SiliconLabs/hardware/silabs/2.x.x/variants/xiao_mg24/ble_silabs/`
3. Download replacement: https://files.seeedstudio.com/wiki/mg24_mic/gsdk_v2.a

### Noisy Audio

1. Adjust buffer size
2. Add filtering
3. Check for nearby EMI sources
4. Try different sample rate

## Example Projects

### Voice Activity Detection

```cpp
#define THRESHOLD 1000
#define MIN_DURATION_MS 500

bool voiceActive = false;
unsigned long voiceStartTime = 0;

void loop() {
    if (record_ready) {
        int maxSample = 0;
        for (int i = 0; i < SAMPLES; i++) {
            if (abs(recording_buf[i]) > maxSample) {
                maxSample = abs(recording_buf[i]);
            }
        }

        if (maxSample > THRESHOLD && !voiceActive) {
            voiceActive = true;
            voiceStartTime = millis();
            Serial.println("Voice started");
        } else if (maxSample < THRESHOLD && voiceActive) {
            voiceActive = false;
            unsigned long duration = millis() - voiceStartTime;
            Serial.print("Voice ended, duration: ");
            Serial.print(duration); Serial.println("ms");
        }

        record_ready = false;
    }
}
```

## Resources

- [Seeed Arduino Mic Library](https://github.com/Seeed-Studio/Seeed_Arduino_Mic)
- [ESP32 I2S Documentation](https://github.com/espressif/arduino-esp32/tree/master/libraries/I2S)
- [SilabsMicrophoneAnalog Examples](Arduino IDE > File > Examples > SilabsMicrophoneAnalog)
