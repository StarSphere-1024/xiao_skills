# Camera API (ESP32S3 Sense)

## Overview

The XIAO ESP32S3 Sense features a camera connector intended for use with OV2640 and OV5640 camera modules.

**Compatible Boards:**
- XIAO ESP32S3 Sense only

**Supported Cameras:**
- OV2640 (2MP, 1600x1200)
- OV5640 (5MP, 2592x1944)

**Requirements:**
- PSRAM is strongly recommended. Higher resolutions and multiple frame buffers typically require PSRAM.
- Uses 14 GPIO pins for camera interface

## Camera Pin Mapping

| Camera Signal | ESP32-S3 GPIO | Function |
|---------------|--------------|----------|
| XMCLK | GPIO10 | Master clock |
| DVP_Y8 | GPIO11 | Data bit 8 |
| DVP_Y7 | GPIO12 | Data bit 7 |
| DVP_PCLK | GPIO13 | Pixel clock |
| DVP_Y6 | GPIO14 | Data bit 6 |
| DVP_Y2 | GPIO15 | Data bit 2 |
| DVP_Y5 | GPIO16 | Data bit 5 |
| DVP_Y3 | GPIO17 | Data bit 3 |
| DVP_Y4 | GPIO18 | Data bit 4 |
| DVP_VSYNC | GPIO38 | Vertical sync |
| CAM_SCL | GPIO39 | I2C clock |
| CAM_SDA | GPIO40 | I2C data |
| DVP_HREF | GPIO47 | Horizontal reference |
| DVP_Y9 | GPIO48 | Data bit 9 |

## Basic Setup

### OV2640 Camera (Recommended)

```cpp
#include "esp_camera.h"

#define CAMERA_MODEL_XIAO_ESP32S3

void setup() {
    Serial.begin(115200);

    // IMPORTANT: Initialize to zero to avoid undefined fields.
    camera_config_t config = {};

    // Configure camera pins
    config.ledc_channel = LEDC_CHANNEL_0;
    config.ledc_timer = LEDC_TIMER_0;

    // D0..D7 map to Y2..Y9 (see the ESP32 Arduino core CameraWebServer example).
    config.pin_d0 = 15;  // Y2
    config.pin_d1 = 17;  // Y3
    config.pin_d2 = 18;  // Y4
    config.pin_d3 = 16;  // Y5
    config.pin_d4 = 14;  // Y6
    config.pin_d5 = 12;  // Y7
    config.pin_d6 = 11;  // Y8
    config.pin_d7 = 48;  // Y9
    config.pin_xclk = GPIO10;
    config.pin_pclk = GPIO13;
    config.pin_vsync = GPIO38;
    config.pin_href = GPIO47;
    config.pin_sccb_sda = GPIO40;
    config.pin_sccb_scl = GPIO39;
    config.pin_pwdn = -1;
    config.pin_reset = -1;
    config.xclk_freq_hz = 20000000;
    config.pixel_format = PIXFORMAT_JPEG;

    config.grab_mode = CAMERA_GRAB_WHEN_EMPTY;
    config.fb_location = CAMERA_FB_IN_PSRAM;

    // Frame settings
    config.frame_size = FRAMESIZE_UXGA;
    config.jpeg_quality = 12;
    config.fb_count = 1;

    // Adjust configuration based on PSRAM availability
    if (config.pixel_format == PIXFORMAT_JPEG) {
        if (psramFound()) {
            config.jpeg_quality = 10;
            config.fb_count = 2;
            config.grab_mode = CAMERA_GRAB_LATEST;
        } else {
            // Limit the frame size when PSRAM is not available
            config.frame_size = FRAMESIZE_SVGA;
            config.fb_location = CAMERA_FB_IN_DRAM;
        }
    }

    // Initialize camera
    esp_err_t err = esp_camera_init(&config);
    if (err != ESP_OK) {
        Serial.printf("Camera init failed with error 0x%x", err);
        return;
    }

    Serial.println("Camera ready");
}

void loop() {
}
```

### OV5640 Camera

```cpp
#include "esp_camera.h"

// Uses the same pin configuration as XIAO ESP32S3 Sense.
// The camera driver detects the sensor PID at runtime (e.g., OV5640_PID).
// Adjust frame_size and sensor settings as needed.
```

## Camera Configuration

### Frame Size

Available frame sizes are defined by the `framesize_t` enum in the Espressif `esp32-camera` driver (exact availability may depend on the Arduino core version).

| Frame Size | Resolution | Notes |
|------------|-----------|-------|
| FRAMESIZE_96X96 | 96x96 | Very low resolution |
| FRAMESIZE_QQVGA | 160x120 | Low resolution |
| FRAMESIZE_128X128 | 128x128 | Low resolution |
| FRAMESIZE_QCIF | 176x144 | Low resolution |
| FRAMESIZE_HQVGA | 240x176 | Medium resolution |
| FRAMESIZE_240X240 | 240x240 | Square |
| FRAMESIZE_QVGA | 320x240 | Medium resolution |
| FRAMESIZE_320X320 | 320x320 | Square |
| FRAMESIZE_CIF | 400x296 | Medium resolution |
| FRAMESIZE_HVGA | 480x320 | Wide |
| FRAMESIZE_VGA | 640x480 | Standard resolution |
| FRAMESIZE_SVGA | 800x600 | High resolution |
| FRAMESIZE_XGA | 1024x768 | High resolution |
| FRAMESIZE_HD | 1280x720 | 720p |
| FRAMESIZE_SXGA | 1280x1024 | High resolution |
| FRAMESIZE_UXGA | 1600x1200 | High resolution |
| FRAMESIZE_QXGA | 2048x1536 | 3MP class |
| FRAMESIZE_5MP | 2592x1944 | 5MP class |

**Example:**
```cpp
config.frame_size = FRAMESIZE_UXGA;  // 1600x1200
```

### JPEG Quality

Lower value = higher quality, larger file size (0-63):

```cpp
config.jpeg_quality = 10;  // High quality
config.jpeg_quality = 20;  // Medium quality
config.jpeg_quality = 30;  // Low quality
```

### Pixel Format

```cpp
// RGB565 (16-bit color)
config.pixel_format = PIXFORMAT_RGB565;

// Grayscale
config.pixel_format = PIXFORMAT_GRAYSCALE;

// JPEG (recommended)
config.pixel_format = PIXFORMAT_JPEG;
```

## Capture and Display

### Capture Single Image

```cpp
void captureImage() {
    // Acquire camera frame buffer
    camera_fb_t *fb = esp_camera_fb_get();

    if (!fb) {
        Serial.println("Camera capture failed");
        return;
    }

    Serial.printf("Image captured: %d bytes\n", fb->len);

    // Process image data here
    // fb->buf points to the image data
    // fb->len is the size of the image

    // Return frame buffer
    esp_camera_fb_return(fb);
}
```

### Save to SD Card

```cpp
#include "SD.h"
#include "esp_camera.h"

#define SD_CS 21

void saveImageToSD() {
    // Initialize SD card
    if (!SD.begin(SD_CS)) {
        Serial.println("SD card initialization failed");
        return;
    }

    // Capture image
    camera_fb_t *fb = esp_camera_fb_get();
    if (!fb) {
        Serial.println("Camera capture failed");
        return;
    }

    // Generate filename
    String filename = "/image_" + String(millis()) + ".jpg";

    // Save to SD
    File file = SD.open(filename, FILE_WRITE);
    if (file) {
        file.write(fb->buf, fb->len);
        file.close();
        Serial.print("Image saved: ");
        Serial.println(filename);
    } else {
        Serial.println("Failed to create file");
    }

    esp_camera_fb_return(fb);
}

void setup() {
    Serial.begin(115200);
    // Initialize camera...
}

void loop() {
    static unsigned long lastCapture = 0;
    if (millis() - lastCapture > 5000) {
        lastCapture = millis();
        saveImageToSD();
    }
}
```

### Send via HTTP

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include "esp_camera.h"

const char* ssid = "your_SSID";
const char* password = "your_PASSWORD";
const char* serverUrl = "http://your-server.com/upload";

void setup() {
    Serial.begin(115200);

    // Connect to WiFi
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("\nWiFi connected");

    // Initialize camera...
}

void sendImageHTTP() {
    // Capture image
    camera_fb_t *fb = esp_camera_fb_get();
    if (!fb) {
        Serial.println("Camera capture failed");
        return;
    }

    // Send via HTTP
    HTTPClient http;
    http.begin(serverUrl);
    http.addHeader("Content-Type", "image/jpeg");

    int httpResponseCode = http.POST(fb->buf, fb->len);

    if (httpResponseCode > 0) {
        Serial.printf("HTTP Response: %d\n", httpResponseCode);
    } else {
        Serial.printf("HTTP Error: %s\n", http.errorToString(httpResponseCode).c_str());
    }

    http.end();
    esp_camera_fb_return(fb);
}

void loop() {
    static unsigned long lastUpload = 0;
    if (millis() - lastUpload > 10000) {
        lastUpload = millis();
        sendImageHTTP();
    }
}
```

## Camera Settings

### Brightness

```cpp
sensor_t *sensor = esp_camera_sensor_get();

// Set brightness (-2 to +2)
sensor->set_brightness(sensor, 0);  // -2, -1, 0, 1, 2
```

### Contrast

```cpp
// Set contrast (-2 to +2)
sensor->set_contrast(sensor, 0);  // -2, -1, 0, 1, 2
```

### Saturation

```cpp
// Set saturation (-2 to +2)
sensor->set_saturation(sensor, 0);  // -2, -1, 0, 1, 2
```

### Special Effects

```cpp
// Set special effect
sensor->set_special_effect(sensor, 0);  // 0-6
```

### White Balance

```cpp
// Set white balance mode
sensor->set_whitebal(sensor, 1);  // 0 = off, 1 = on

// Set white balance mode
sensor->set_wb_mode(sensor, 0);  // 0-4
```

### Exposure

```cpp
// Set exposure
sensor->set_exposure_ctrl(sensor, 1);  // 0 = off, 1 = on

// Automatic gain control + gain value (0-30)
sensor->set_gain_ctrl(sensor, 1);   // 0 = off, 1 = on
sensor->set_agc_gain(sensor, 0);    // 0-30
```

## Stream Video

### HTTP MJPEG Stream

For a complete, working HTTP server + streaming example (including correct response handling and sensor settings), use the ESP32 Arduino core example `CameraWebServer` and select `CAMERA_MODEL_XIAO_ESP32S3`.

## Troubleshooting

### Camera Initialization Fails

1. **Check PSRAM**: Ensure PSRAM is enabled in Tools
2. **Check frame size**: Try smaller resolution (FRAMESIZE_QVGA)
3. **Check pins**: Verify all camera pins are properly connected
4. **Add delay**: Add delay(1000) before camera initialization

### Images Are Corrupted

1. **Lower JPEG quality**: Increase jpeg_quality value
2. **Check frame buffer**: Try `config.fb_count = 1` or `config.fb_count = 2`
3. **Lower resolution**: Try FRAMESIZE_VGA or lower
4. **Check XCLK frequency**: Try `config.xclk_freq_hz = 16000000`

### Brownout Issues

Camera requires significant power. If experiencing brownouts:

1. **Use quality USB cable** with good power delivery
2. **Lower XCLK frequency**: `config.xclk_freq_hz = 10000000`
3. **Reduce frame size**: Use lower resolution
4. **Increase jpeg_quality**: Reduce compression

### Memory Issues

1. **Free memory after capture**: Always call `esp_camera_fb_return(fb)`
2. **Reduce fb_count**: Set `config.fb_count = 1`
3. **Use lower resolution**: FRAMESIZE_SVGA or lower
4. **Check available memory**:
```cpp
Serial.printf("Free heap: %d bytes\n", ESP.getFreeHeap());
```

## Example Projects

### Time-Lapse Camera

```cpp
void loop() {
    static unsigned long lastShot = 0;
    const unsigned long interval = 60000;  // 1 minute

    if (millis() - lastShot >= interval) {
        lastShot = millis();
        saveImageToSD();  // Use function from above
    }
}
```

### Motion Detection Camera

```cpp
// Simple change detection
uint8_t *previousFrame = NULL;
int frameSize = 0;

bool detectMotion() {
    camera_fb_t *fb = esp_camera_fb_get();
    if (!fb) return false;

    bool motion = false;

    if (previousFrame != NULL && frameSize == fb->len) {
        int changes = 0;
        for (int i = 0; i < fb->len; i += 10) {
            if (abs(fb->buf[i] - previousFrame[i]) > 30) {
                changes++;
            }
        }

        if (changes > 100) {
            motion = true;
        }
    }

    // Store current frame
    if (previousFrame != NULL) free(previousFrame);
    frameSize = fb->len;
    previousFrame = (uint8_t *)malloc(frameSize);
    memcpy(previousFrame, fb->buf, frameSize);

    esp_camera_fb_return(fb);
    return motion;
}

void loop() {
    if (detectMotion()) {
        Serial.println("Motion detected!");
        saveImageToSD();
    }
    delay(100);
}
```

## Resources

- [ESP32 Camera Documentation](https://github.com/espressif/esp32-camera)
- [OV2640 Datasheet](https://www.ovt.com/download/OV2640_ds_1.03.pdf)
- [OV5640 Datasheet](https://www.ovt.com/download/OV5640_ds_2.03.pdf)

## References (Repo Evidence)

- `Ref/esp-arduino-core-3.3.6/libraries/ESP32/examples/Camera/CameraWebServer/CameraWebServer.ino` (pin_d0..pin_d7 mapping, PSRAM-dependent settings)
- `Ref/esp-arduino-core-3.3.6/libraries/ESP32/examples/Camera/CameraWebServer/camera_pins.h` (XIAO ESP32S3 Sense GPIO mapping)
- Seeed Studio Wiki: Camera usage (GPIO mapping, OV2640/OV5640 compatibility): https://wiki.seeedstudio.com/xiao_esp32s3_camera_usage/
