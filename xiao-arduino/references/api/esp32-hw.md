# ESP32 Hardware APIs for XIAO

## Overview

XIAO ESP32C3/C5/C6/S3 boards provide additional hardware-specific APIs beyond standard Arduino functions.

## Memory Configuration

### PSRAM (ESP32S3 only)

```cpp
// Configure PSRAM in setup()
void setup() {
    // PSRAM is usually auto-detected
    // Manual configuration if needed:
    psramFound();

    // Check PSRAM availability
    if (psramFound()) {
        Serial.println("PSRAM detected");

        Serial.printf("PSRAM size: %d bytes\n", ESP.getPsramSize());
    }
}
```

### Heap Allocation

```cpp
// Use PSRAM for large allocations
void* large_buffer = ps_malloc(1024 * 1024);  // 1MB from PSRAM
void* large_buffer_cstr = ps_calloc(1024, sizeof(char));  // Clear memory
```

## PWM (Pulse Width Modulation)

### LEDC PWM (ESP32)

```cpp
// ESP32C3/C6 have 8 LEDC channels
// ESP32S3 has 8 channels

const int PWM_PIN = D7;  // PWM capable pin
const int PWM_FREQ = 5000;  // 5kHz
const int PWM_RES = 10;  // 10-bit resolution (0-1023)

void setup() {
    ledcSetup(0, PWM_FREQ, PWM_RES);  // Channel 0

    ledcAttachPin(PWM_PIN, 0);  // Attach pin to channel 0
}

void setBrightness(int brightness) {
    // brightness: 0-1023
    ledcWrite(0, brightness);
}

void loop() {
    // Fade in/out

    for (int i = 0; i <= 1023; i++) {
        ledcWrite(0, i);
        delay(1);
    }
    for (int i = 1023; i >= 0; i--) {
        ledcWrite(0, i);
        delay(1);
    }
}
```

### AnalogWrite (uses LEDC)

```cpp
// analogWrite uses LEDC on ESP32
const int LED_PIN = D10;

void setup() {
    // No setup needed, analogWrite uses LEDC by default
}

void loop() {
    // Fade LED (0-255)
    for (int i = 0; i <= 255; i++) {
        analogWrite(LED_PIN, i);
        delay(10);
    }
}
```

## ADC (Analog-to-Digital Converter)

### ESP32C3/C6/S3 ADC

```cpp
// ESP32 has 12-bit ADC (0-4095)
// Attenuation options:
// - ADC_0db: 0-1.1V (no attenuation)
// - ADC_2_5db: 0-1.5V
// - ADC_6db: 0-2.2V
// - ADC_11db: 0-3.3V (full range)

const int ADC_PIN = A0;  // D0


void setup() {
    Serial.begin(115200);
    analogSetPinAttenuation(ADC_PIN, ADC_11db);  // 0-3.3V range
}

void loop() {
    int rawValue = analogRead(ADC_PIN);  // 0-4095
    float voltage = rawValue * 3.3 / 4095.0;

    Serial.printf("Raw: %d, Voltage: %.2fV\n", rawValue, voltage);
    delay(1000);
}
```

### Hall Effect Sensor (ESP32C3/C6/S3)

```cpp
// Built-in hall effect sensor
void setup() {
    Serial.begin(115200);
}

void loop() {
    int hall = hallRead();  // Read hall sensor

    Serial.printf("Hall sensor: %d\n", hall);
    delay(1000);
}
```

## Temperature Sensor

```cpp
// Internal temperature sensor
void setup() {
    Serial.begin(115200);
}

void loop() {
    // Read internal temperature (in Celsius)
    float temp = temperatureRead();
    Serial.printf("Temperature: %.2f°C\n", temp / 10.0);
    delay(1000);
}
```

## Touch Sensors (ESP32C3/C6/S3)

```cpp
// ESP32C3/C6/S3 have touch-capable pins
// XIAO ESP32C3 touch pins: D1, D2, D3, D4, D5, D6, D7, D8, D9, D10

const int TOUCH_PIN = T1;  // D1 is T1

void setup() {
    Serial.begin(115200);
}

void loop() {

    int touchValue = touchRead(TOUCH_PIN);  // Lower = more touch
    Serial.printf("Touch: %d\n", touchValue);

    if (touchValue < 50) {
        Serial.println("Touched!");
    }

    delay(100);
}
```

## Deep Sleep

```cpp
// ESP32 deep sleep configuration
#include <esp_sleep.h>

#define uS_TO_S_FACTOR 1000000ULL  // Conversion factor


void setup() {
    Serial.begin(115200);

    // Configure wake up source
    esp_sleep_enable_timer_wakeup(60 * uS_TO_S_FACTOR);  // 60 seconds

    // Or wake up on touch pin
    esp_sleep_pd_config(ESP_PD_DOMAIN_RTC_PERIPH, ESP_PD_OPTION_AUTO);
    esp_sleep_enable_touchpad_wakeup(T1);

    Serial.println("Going to sleep...");
    esp_deep_sleep_start();  // Sleep (code stops here)
}

void loop() {
    // Never reached

}
```

## Reset and Restart

```cpp
void setup() {
    Serial.begin(115200);
}

void loop() {
    Serial.println("Restarting in 5 seconds...");
    delay(5000);
    ESP.restart();  // Software restart
}
```

## CPU Frequency Control

```cpp
// Change CPU frequency (ESP32C3: 80MHz or 160MHz)
void setup() {
    setCpuFrequencyMhz(160);  // 160MHz (max performance)
    // setCpuFrequencyMhz(80);   // 80MHz (lower power)

    Serial.print("CPU Frequency: ");
    Serial.print(getCpuFrequencyMhz());
    Serial.println(" MHz");
}
```

## Flash Storage

```cpp
// Partition SPIFFS/LittleFS
#include <FS.h>

#include <SPIFFS.h>

void setup() {
    if (!SPIFFS.begin(true)) {
        Serial.println("SPIFFS mount failed");
        return;
    }

    // Write file
    File file = SPIFFS.open("/test.txt", "w");
    file.println("Hello XIAO!");
    file.close();

    // Read file
    file = SPIFFS.open("/test.txt", "r");
    Serial.println(file.readString());
    file.close();
}
```

## EEPROM (Emulated in Flash)

```cpp
#include <EEPROM.h>

#define EEPROM_SIZE 512

void setup() {
    EEPROM.begin(EEPROM_SIZE);

    // Write

    EEPROM.write(0, 42);
    EEPROM.commit();

    // Read
    int value = EEPROM.read(0);
    Serial.println(value);
}

void loop() {
    // Not used
}
```

## Hardware Timer

```cpp
// ESP32 hardware timers
hw_timer_t *timer = NULL;


volatile bool timer_flag = false;

void IRAM_ATTR timer_isr() {
    timer_flag = true;
}

void setup() {
    timer = timerBegin(0, 80, true);  // Timer 0, 80MHz prescaler
    timerAttachInterrupt(timer, &timer_isr);
    timerAlarmWrite(timer, 1000000, true);  // 1 second
    timerEnable(timer);
}


void loop() {
    if (timer_flag) {
        timer_flag = false;
        Serial.println("Timer triggered!");
    }
}
```

## References

- [ESP32 Arduino Core](https://github.com/espressif/arduino-esp32)
- [ESP32 Datasheet](https://www.espressif.com/sites/default/files/documentation/esp32-c3_datasheet_en.pdf)
- [LEDC Control](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/apis/ledc.html)
