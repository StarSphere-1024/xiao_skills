# Arduino on XIAO ESP32C5

## Overview

The SeeedStudio XIAO ESP32C5 features the ESP32-C5 chip with WiFi 6 and Bluetooth 5.

## Board Manager Setup

1. Open Arduino IDE
2. File > Preferences > Additional Boards Manager URLs
3. Add: `https://espressif.github.io/arduino-esp32/package_esp32_index.json`
4. Tools > Board > Boards Manager > Search "esp32" > Install
5. Tools > Board > esp32 > Select "XIAO_ESP32C5"

## First Sketch

```cpp
#define LED_PIN D10  // Built-in LED

void setup() {
    Serial.begin(115200);
    pinMode(LED_PIN, OUTPUT);
}

void loop() {
    Serial.println("Hello XIAO ESP32C5!");
    digitalWrite(LED_PIN, HIGH);
    delay(1000);
    digitalWrite(LED_PIN, LOW);
    delay(1000);
}
```

## Pin Mapping

| Name | GPIO | Functions | Notes |
|------|------|-----------|-------|
| D0 | GPIO8 | USB-, BOOT | Strapping pin |
| D1 | GPIO9 | USB+, BOOT | Strapping pin |
| D2 | GPIO2 | ADC1_CH2, Pull-up | Boot mode |
| D3 | GPIO3 | ADC1_CH3 | - |
| D4 | GPIO4 | ADC1_CH4 | - |
| D5 | GPIO5 | ADC1_CH5 | - |
| D6 | GPIO6 | ADC1_CH6 | - |
| D7 | GPIO7 | ADC1_CH7 | - |
| D8 | GPIO10 | ADC2_CH0 | - |
| D9 | GPIO11 | ADC2_CH1 | - |
| D10 | GPIO12 | ADC2_CH2, SPI_MISO | - |

Built-in LED: GPIO39 (white, connected to USB)

## Special Pins

### Strapping Pins (D0, D1)
- Used for USB D-/D+
- Affect boot mode
- Avoid using as GPIO if possible

### Boot Pin (D2)
- Pull-up to boot normally
- Pull-down to enter download mode

### ADC
- 12-bit ADC (0-4095)
- Voltage range: 0-3.3V
- Channels: ADC1_CH0-CH9, ADC2_CH0-CH2

### PWM
- All GPIO pins support PWM
- LEDC controller with 16 channels

### I2C
- Default: D0 (SDA), D1 (SCL)
- Can use any pins with `Wire.begin(sda, scl)`

### SPI
- Default: D5 (MOSI), D10 (MISO), D4 (SCK)
- Can use custom pins

### UART
- USB Serial (CDC)
- UART0: D7 (RX), D6 (TX)
- UART1: Custom pins

## WiFi 6 (802.11ax)

```cpp
#include <WiFi.h>

const char* ssid = "your-SSID";
const char* password = "your-PASSWORD";

void setup() {
    Serial.begin(115200);
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println("\nConnected!");
    Serial.print("IP: ");
    Serial.println(WiFi.localIP());
}

void loop() {}
```

## Bluetooth 5

### BLE Peripheral

```cpp
#include <BLEDevice.h>
#include <BLEServer.h>

void setup() {
    BLEDevice::init("XIAO-C5");
    BLEServer* pServer = BLEDevice::createServer();

    BLEService* pService = pServer->createService("4fafc201-1fb5-459e-8fcc-c5c9c331914b");
    pService->createCharacteristic("beb5483e-36e1-4688-b7f5-ea07361b26a8",
                                    BLECharacteristic::PROPERTY_READ);

    pService->start();
    BLEAdvertising* pAdvertising = BLEDevice::getAdvertising();
    pAdvertising->start();
}

void loop() {}
```

## ADC Reading

```cpp
void setup() {
    Serial.begin(115200);
    analogReadResolution(12);  // 12-bit resolution
}

void loop() {
    int adc = analogRead(D2);
    float voltage = adc * 3.3 / 4095;
    Serial.printf("ADC: %d, Voltage: %.2fV\n", adc, voltage);
    delay(1000);
}
```

## Hall Effect Sensor

```cpp
void setup() {
    Serial.begin(115200);
}

void loop() {
    int hall = hallRead();
    Serial.printf("Hall sensor: %d\n", hall);
    delay(500);
}
```

## Touch Sensor

```cpp
void setup() {
    Serial.begin(115200);
}

void loop() {
    // Touch pins: T0-T5
    int touch = touchRead(T0);
    Serial.printf("Touch: %d\n", touch);
    delay(100);
}
```

## Deep Sleep

```cpp
#include "esp_sleep.h"

#define uS_TO_S_FACTOR 1000000
#define TIME_TO_SLEEP 60

void setup() {
    Serial.begin(115200);

    esp_sleep_enable_timer_wakeup(TIME_TO_SLEEP * uS_TO_S_FACTOR);
    Serial.println("Going to sleep...");
    esp_deep_sleep_start();
}

void loop() {}
```

## Brownout Detection

```cpp
#include "esp_pm.h"

void setup() {
    Serial.begin(115200);

    // Set brownout voltage
    esp_pm_configure(&esp_pm_config_t{
        .max_freq = 240000000,
        .min_freq = 40000000,
        .light_sleep_enable = false
    });

    Serial.println("Brownout configured");
}

void loop() {}
```

## Key Differences from ESP32C3

| Feature | ESP32C3 | ESP32C5 |
|---------|---------|---------|
| WiFi | WiFi 4 (802.11n) | WiFi 6 (802.11ax) |
| Bluetooth | BLE 5.0 | BLE 5.4 |
| CPU | Xtensa LX7 | RISC-V |
| Flash | Up to 16MB | Up to 8MB |
| PSRAM | Optional | Optional |
| ADC | 12-bit | 12-bit |
| GPIO | 22 | 30 |

## Known Issues

1. **D0 and D1**: Used for USB, avoid for other uses
2. **D2**: Pull-up resistor affects boot mode
3. **ADC2**: Can't use when WiFi is active
4. **Power**: Ensure adequate power supply for WiFi

## Troubleshooting

### Can't upload
- Hold BOOT button while uploading
- Check USB cable (data cable required)
- Try different USB port

### WiFi won't connect
- Check 2.4GHz only (ESP32C5 WiFi 6 is 2.4GHz)
- Verify SSID and password
- Check for interference

### ADC readings wrong
- Set resolution with `analogReadResolution(12)`
- Check input voltage range (0-3.3V)
- Use external reference if needed

### Boot loop
- Check D0/D1 state during boot
- Verify D2 pull-up is present
- Try holding BOOT button at power-on

## Best Practices

1. **Power**: Use stable 3.3V supply with at least 500mA
2. **Antenna**: Keep antenna area clear of metal
3. **Decoupling**: Add 100nF capacitor near power pins
4. **Heat**: ESP32C5 can get warm under WiFi load
5. **GPIO**: Avoid using D0/D1 for general I/O

## Performance Tips

1. **WiFi**: Use WiFi 6 features for better throughput
2. **BLE**: Use BLE 5.4 features for longer range
3. **Power**: Enable WiFi power save when not transmitting
4. **CPU**: Run at lower frequency when not needed
5. **Sleep**: Use deep sleep for battery applications
