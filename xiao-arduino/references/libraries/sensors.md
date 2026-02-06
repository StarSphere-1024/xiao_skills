# Common Sensor Libraries for XIAO

## DHT11/DHT22 Temperature & Humidity

### Library
- **DHT sensor library** by Adafruit

### Installation
Arduino IDE: Sketch > Include Library > Manage Libraries > Search "DHT"

### Basic Usage

```cpp
#include "DHT.h"

#define DHTPIN D0      // Digital pin connected
#define DHTTYPE DHT22   // DHT11 or DHT22

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

    Serial.printf("Humidity: %.1f%%  Temperature: %.1f°C\n", h, t);
}
```

### XIAO Pin Notes
- Any digital GPIO works
- ESP32C3: Avoid D6, D8, D9

## BME280 (Temperature, Humidity, Pressure)

### Library
- **Adafruit BME280 Library**

### Basic Usage

```cpp
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>

Adafruit_BME280 bme;

void setup() {
    Serial.begin(115200);
    Wire.begin();  // Default SDA/SCL for your board

    if (!bme.begin(0x76)) {  // Try 0x77 if not found
        Serial.println("Could not find BME280 sensor!");
        while (1);
    }
}

void loop() {
    Serial.print("Temperature = ");
    Serial.print(bme.readTemperature());
    Serial.println(" *C");

    Serial.print("Pressure = ");
    Serial.print(bme.readPressure() / 100.0F);
    Serial.println(" hPa");

    Serial.print("Humidity = ");
    Serial.print(bme.readHumidity());
    Serial.println(" %");

    delay(2000);
}
```

### Address Selection
- **0x76**: SD0 LOW (default)
- **0x77**: SD0 HIGH

## MPU6050 IMU (Accelerometer + Gyroscope)

### Library
- **Adafruit MPU6050**

### Basic Usage

```cpp
#include <Adafruit_MPU6050.h>
#include <Adafruit_Sensor.h>

Adafruit_MPU6050 mpu;

void setup() {
    Serial.begin(115200);
    Wire.begin();

    if (!mpu.begin()) {
        Serial.println("Failed to find MPU6050 chip");
        while (1) {
            delay(10);
        }
    }

    mpu.setAccelerometerRange(MPU6050_RANGE_8_G);
    mpu.setGyroRange(MPU6050_RANGE_500_DEG);
    mpu.setFilterBandwidth(MPU6050_BAND_21_HZ);
}

void loop() {
    sensors_event_t a, g, temp;
    mpu.getEvent(&a, &g, &temp);

    Serial.print("Acceleration X: ");
    Serial.print(a.acceleration.x);
    Serial.print(", Y: ");
    Serial.print(a.acceleration.y);
    Serial.print(", Z: ");
    Serial.print(a.acceleration.z);
    Serial.println(" m/s^2");

    Serial.print("Rotation X: ");
    Serial.print(g.gyro.x);
    Serial.print(", Y: ");
    Serial.print(g.gyro.y);
    Serial.print(", Z: ");
    Serial.print(g.gyro.z);
    Serial.println(" rad/s");

    delay(100);
}
```

## DS18B20 Temperature Sensor (OneWire)

### Library
- **DallasTemperature**
- **OneWire**

### Basic Usage

```cpp
#include <OneWire.h>
#include <DallasTemperature.h>

#define ONE_WIRE_BUS D0

OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

void setup() {
    Serial.begin(115200);
    sensors.begin();
}

void loop() {
    Serial.print("Requesting temperatures...");
    sensors.requestTemperatures();
    Serial.println("DONE");

    float tempC = sensors.getTempCByIndex(0);
    Serial.print("Temperature for Device 1 is: ");
    Serial.println(tempC);

    delay(2000);
}
```

## Hall Effect Sensor (A3144)

### Direct Read (No Library)

```cpp
const int hallPin = D0;

void setup() {
    Serial.begin(115200);
    pinMode(hallPin, INPUT_PULLUP);
}

void loop() {
    if (digitalRead(hallPin) == LOW) {
        Serial.println("Magnet detected!");
    } else {
        Serial.println("No magnet");
    }
    delay(100);
}
```

## Ultrasonic Sensor (HC-SR04)

### Direct Read (No Library)

```cpp
const int trigPin = D0;
const int echoPin = D1;

void setup() {
    Serial.begin(115200);
    pinMode(trigPin, OUTPUT);
    pinMode(echoPin, INPUT);
}

void loop() {
    // Clear trigger
    digitalWrite(trigPin, LOW);
    delayMicroseconds(2);

    // Send 10μs pulse
    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);
    digitalWrite(trigPin, LOW);

    // Read echo
    long duration = pulseIn(echoPin, HIGH);

    // Calculate distance (cm)
    float distance = duration * 0.034 / 2;

    Serial.print("Distance: ");
    Serial.print(distance);
    Serial.println(" cm");

    delay(100);
}
```

## Light Sensor (BH1750)

### Library
- **BH1750**

### Basic Usage

```cpp
#include <Wire.h>
#include <BH1750.h>

BH1750 lightMeter;

void setup() {
    Serial.begin(115200);
    Wire.begin();
    lightMeter.begin();
}

void loop() {
    float lux = lightMeter.readLightLevel();
    Serial.print("Light: ");
    Serial.print(lux);
    Serial.println(" lx");
    delay(1000);
}
```

## Soil Moisture Sensor (Capacitive)

### Direct Read (Resistive/Capacitive)

```cpp
const int soilPin = A0;

void setup() {
    Serial.begin(115200);
}

void loop() {
    int moisture = analogRead(soilPin);
    Serial.print("Moisture: ");
    Serial.println(moisture);

    // Calibration needed for your sensor
    // Usually: 0 = wet, 4095 = dry (or vice versa)

    delay(1000);
}
```

## Current Sensor (ACS712)

### Direct Read

```cpp
const int acsPin = A0;

void setup() {
    Serial.begin(115200);
}

void loop() {
    int sensorValue = analogRead(acsPin);
    float voltage = sensorValue * (3.3 / 4095.0);

    // For ACS712 5A module:
    // 2.5V = 0A, 2.5V - 2.5V = -5A, 2.5V + 2.5V = +5A
    // float amps = (voltage - 2.5) / 0.185;

    // For ACS712 20A module:
    // float amps = (voltage - 2.5) / 0.100;

    // For ACS712 30A module:
    float amps = (voltage - 2.5) / 0.066;

    Serial.print("Current: ");
    Serial.print(amps);
    Serial.println(" A");

    delay(100);
}
```

## Sensor Best Practices

1. **Calibration**: Most sensors need calibration for accuracy
2. **Averaging**: Take multiple readings and average
3. **Timing**: Respect minimum reading intervals
4. **Power**: Use stable 3.3V or 5V supply
5. **Pull-ups**: I2C sensors usually have them onboard

### Averaging Example

```cpp
const int numReadings = 10;
int readings[numReadings];
int index = 0;

void setup() {
    Serial.begin(115200);
    for (int i = 0; i < numReadings; i++) {
        readings[i] = 0;
    }
}

void loop() {
    readings[index] = analogRead(A0);
    index = (index + 1) % numReadings;

    long sum = 0;
    for (int i = 0; i < numReadings; i++) {
        sum += readings[i];
    }
    float average = sum / (float)numReadings;

    Serial.println(average);
    delay(100);
}
```
