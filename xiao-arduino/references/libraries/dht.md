# DHT/DHT22 Library for XIAO

## Overview

DHT library for reading temperature and humidity from DHT11, DHT22, and DHT21 sensors. Simple 1-wire digital sensor interface.

## Board Compatibility

All XIAO boards support DHT sensors via digital I/O:
- ESP32C3/C5/C6/S3: Any GPIO
- nRF52840/MG24: Any GPIO
- RP2040/RP2350: Any GPIO
- SAMD21/RA4M1: Any GPIO

## Pin Connection

```cpp
DHT Sensor    XIAO Board
-----------   -----------
VCC (Pin 1) → 3.3V
DATA (Pin 2) → D0 (or any digital pin)
NC  (Pin 3) → Not connected
GND (Pin 4) → GND

Optional: Add 10KΩ pull-up resistor between VCC and DATA
```

## Basic DHT22 Reading

```cpp
#include <DHT.h>

#define DHTPIN D0
#define DHTTYPE DHT22  // DHT11, DHT22, or DHT21

DHT dht(DHTPIN, DHTTYPE);

void setup() {
    Serial.begin(115200);
    dht.begin();
}

void loop() {
    delay(2000);  // DHT requires 2 seconds between readings

    float h = dht.readHumidity();
    float t = dht.readTemperature();

    if (isnan(h) || isnan(t)) {
        Serial.println("Failed to read from DHT sensor!");
        return;
    }

    Serial.printf("Humidity: %.1f%%\n", h);
    Serial.printf("Temperature: %.1f°C\n", t);
}
```

## DHT11 vs DHT22

```cpp
#include <DHT.h>

// DHT11: 0-50°C ±2°C, 20-90% RH ±5%
// DHT22: -40-80°C ±0.5°C, 0-100% RH ±2-5%

DHT dht11(D0, DHT11);
DHT dht22(D1, DHT22);

void compareSensors() {
    float t11 = dht11.readTemperature();
    float h11 = dht11.readHumidity();

    float t22 = dht22.readTemperature();
    float h22 = dht22.readHumidity();

    Serial.println("DHT11:");
    Serial.printf("  Temp: %.1f°C\n", t11);
    Serial.printf("  Humidity: %.1f%%\n", h11);

    Serial.println("DHT22:");
    Serial.printf("  Temp: %.1f°C\n", t22);
    Serial.printf("  Humidity: %.1f%%\n", h22);
}

void setup() {
    Serial.begin(115200);
    dht11.begin();
    dht22.begin();
}

void loop() {
    delay(2000);
    compareSensors();
}
```

## Fahrenheit

```cpp
#include <DHT.h>

#define DHTPIN D0
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

void setup() {
    Serial.begin(115200);
    dht.begin();
}

void loop() {
    delay(2000);

    float tF = dht.readTemperature(true);  // true = Fahrenheit
    float h = dht.readHumidity();

    Serial.printf("Temperature: %.1f°F\n", tF);
    Serial.printf("Humidity: %.1f%%\n", h);
}
```

## Heat Index

```cpp
#include <DHT.h>

#define DHTPIN D0
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

void setup() {
    Serial.begin(115200);
    dht.begin();
}

void loop() {
    delay(2000);

    float h = dht.readHumidity();
    float t = dht.readTemperature();
    float f = dht.readTemperature(true);

    if (isnan(h) || isnan(t) || isnan(f)) {
        return;
    }

    float hif = dht.computeHeatIndex(f, h);
    float hic = dht.computeHeatIndex(t, h, false);

    Serial.printf("Humidity: %.1f%%\n", h);
    Serial.printf("Temperature: %.1f°C (%.1f°F)\n", t, f);
    Serial.printf("Heat Index: %.1f°C (%.1f°F)\n", hic, hif);
}
```

## Error Handling

```cpp
#include <DHT.h>

#define DHTPIN D0
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

bool readDHT(float* temp, float* humidity) {
    // DHT requires minimum 2 seconds between readings
    static unsigned long lastRead = 0;
    if (millis() - lastRead < 2000) {
        return false;
    }

    float h = dht.readHumidity();
    float t = dht.readTemperature();

    if (isnan(h) || isnan(t)) {
        Serial.println("DHT read failed - check wiring!");
        return false;
    }

    *temp = t;
    *humidity = h;
    lastRead = millis();
    return true;
}

void setup() {
    Serial.begin(115200);
    dht.begin();
}

void loop() {
    float temp, humidity;
    if (readDHT(&temp, &humidity)) {
        Serial.printf("Temperature: %.1f°C\n", temp);
        Serial.printf("Humidity: %.1f%%\n", humidity);
    }
}
```

## Multiple DHT Sensors

```cpp
#include <DHT.h>

#define DHT1_PIN D0
#define DHT2_PIN D1
#define DHT3_PIN D2

DHT dht1(DHT1_PIN, DHT22);
DHT dht2(DHT2_PIN, DHT22);
DHT dht3(DHT3_PIN, DHT11);

void setup() {
    Serial.begin(115200);
    dht1.begin();
    dht2.begin();
    dht3.begin();
}

void loop() {
    delay(2000);

    float t1 = dht1.readTemperature();
    float h1 = dht1.readHumidity();

    float t2 = dht2.readTemperature();
    float h2 = dht2.readHumidity();

    float t3 = dht3.readTemperature();
    float h3 = dht3.readHumidity();

    Serial.println("Sensor 1:");
    Serial.printf("  %.1f°C, %.1f%%\n", t1, h1);

    Serial.println("Sensor 2:");
    Serial.printf("  %.1f°C, %.1f%%\n", t2, h2);

    Serial.println("Sensor 3:");
    Serial.printf("  %.1f°C, %.1f%%\n", t3, h3);
}
```

## Low Power Mode

```cpp
#include <DHT.h>

#define DHTPIN D0
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

void setup() {
    Serial.begin(115200);
    dht.begin();
}

void lowPowerRead() {
    // Power DHT from GPIO to save power
    pinMode(D1, OUTPUT);  // D1 = VCC control
    digitalWrite(D1, HIGH);

    delay(2000);  // Wait for sensor to stabilize

    float t = dht.readTemperature();
    float h = dht.readHumidity();

    Serial.printf("Temp: %.1f°C, Humidity: %.1f%%\n", t, h);

    // Cut power
    digitalWrite(D1, LOW);
}

void loop() {
    lowPowerRead();
    delay(10000);  // Read every 10 seconds
}
```

## Data Logging

```cpp
#include <DHT.h>
#include <LittleFS.h>

#define DHTPIN D0
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);
File logFile;

void logSensorData() {
    float t = dht.readTemperature();
    float h = dht.readHumidity();

    if (isnan(t) || isnan(h)) {
        return;
    }

    logFile = LittleFS.open("/dhtlog.txt", "a");
    if (logFile) {
        logFile.printf("%lu,%.2f,%.1f\n", millis(), t, h);
        logFile.close();
    }

    Serial.printf("Logged: %.1f°C, %.1f%%\n", t, h);
}

void setup() {
    Serial.begin(115200);
    LittleFS.begin();
    dht.begin();
}

void loop() {
    delay(5000);
    logSensorData();
}
```

## OLED Display Output

```cpp
#include <DHT.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define DHTPIN D0
#define DHTTYPE DHT22

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_ADDR 0x3C

DHT dht(DHTPIN, DHTTYPE);
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

void setup() {
    Serial.begin(115200);
    dht.begin();

    if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)) {
        Serial.println("SSD1306 allocation failed");
        while(1);
    }

    display.clearDisplay();
    display.setTextSize(1);
    display.setTextColor(SSD1306_WHITE);
}

void loop() {
    delay(2000);

    float t = dht.readTemperature();
    float h = dht.readHumidity();

    display.clearDisplay();
    display.setCursor(0, 0);

    display.println("DHT22 Sensor");
    display.println();
    display.printf("Temp: %.1f C\n", t);
    display.printf("Humidity: %.1f%%\n", h);

    float f = dht.readTemperature(true);
    display.printf("Temp: %.1f F\n", f);

    display.display();
}
```

## Best Practices

```cpp
#include <DHT.h>

#define DHTPIN D0
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

void setup() {
    Serial.begin(115200);

    // Best practice 1: Initialize in setup()
    dht.begin();
}

// Best practice 2: Respect minimum delay
void reliableRead() {
    static unsigned long lastRead = 0;
    const unsigned long MIN_DELAY = 2000;

    if (millis() - lastRead < MIN_DELAY) {
        return;  // Too soon
    }

    float t = dht.readTemperature();
    float h = dht.readHumidity();

    lastRead = millis();
}

// Best practice 3: Always validate readings
void safeRead() {
    float t = dht.readTemperature();
    float h = dht.readHumidity();

    if (isnan(t) || isnan(h)) {
        // Handle error
        return;
    }

    // Check for reasonable values
    if (t < -40 || t > 80) {
        Serial.println("Invalid temperature");
        return;
    }

    Serial.printf("Temperature: %.1f°C\n", t);
}

// Best practice 4: Add pull-up resistor for long wires
void setupWithPullup() {
    pinMode(DHTPIN, INPUT_PULLUP);
    dht.begin();
}

// Best practice 5: Use power control for battery
void powerManagedRead() {
    pinMode(D1, OUTPUT);  // VCC control
    digitalWrite(D1, HIGH);

    delay(100);  // Sensor warm-up

    float t = dht.readTemperature();

    digitalWrite(D1, LOW);  // Power off
}

void loop() {
    delay(2000);
    reliableRead();
    safeRead();
}
```

## References

- [DHT Library](https://github.com/adafruit/DHT-sensor-library)
- [DHT22 Datasheet](https://www.sparkfun.com/datasheets/Sensors/Temperature/DHT22.pdf)
