# Adafruit Unified Sensor Library for XIAO

## Overview

Adafruit Unified Sensor provides a common abstraction layer for all Adafruit sensors. Enables consistent API across different sensor types.

## Board Compatibility

All XIAO boards:
- ESP32C3/C5/C6/S3: Full support
- nRF52840/MG24: Full support
- RP2040/RP2350: Full support
- SAMD21/RA4M1: Full support

## Unified Sensor API

All unified sensors implement these common methods:

```cpp
// Get sensor event
void getEvent(sensors_event_t* event);

// Get sensor details
void getSensor(sensor_t* sensor);

// Get sensor details as string
void printSensorDetails(void);

// Enable/disable auto-ranging
void enableAutoRange(bool enable);
```

## Event Structure

```cpp
sensors_event_t {
    int32_t version;           // Sensor type
    int32_t sensor_id;         // Unique sensor ID
    int32_t type;              // Sensor type constant
    int32_t reserved0;
    int32_t timestamp;         // Time of event
    union {
        float data[4];         // Raw data
        struct {
            float acceleration_x;
            float acceleration_y;
            float acceleration_z;
        } acceleration;        // Accelerometer
        struct {
            float magnetic_x;
            float magnetic_y;
            float magnetic_z;
        } magnetic;            // Magnetometer
        struct {
            float gyro_x;
            float gyro_y;
            float gyro_z;
        } gyro;                // Gyroscope
        struct {
            float temperature;
        } temperature;         // Temperature
        struct {
            float distance;
        } distance;            // Distance
        struct {
            float light;
        } light;               // Light
        struct {
            float pressure;
        } pressure;            // Pressure
        struct {
            float relative_humidity;
        } relative_humidity;   // Humidity
        struct {
            float voltage;
            float current;
        } voltage;             // Voltage/current
    };
};
```

## Sensor Types

```cpp
// Common sensor type constants
SENSOR_TYPE_ACCELEROMETER     // 3-axis acceleration
SENSOR_TYPE_MAGNETIC_FIELD    // 3-axis magnetic field
SENSOR_TYPE_ORIENTATION       // 3-axis orientation
SENSOR_TYPE_GYROSCOPE         // 3-axis angular rate
SENSOR_TYPE_LIGHT             // Light level
SENSOR_TYPE_PRESSURE          // Atmospheric pressure
SENSOR_TYPE_PROXIMITY         // Proximity detection
SENSOR_TYPE_GRAVITY           // Gravity vector
SENSOR_TYPE_LINEAR_ACCEL      // Linear acceleration (without gravity)
SENSOR_TYPE_ROTATION_VECTOR   // Rotation vector
SENSOR_TYPE_RELATIVE_HUMIDITY // Relative humidity
SENSOR_TYPE_AMBIENT_TEMPERATURE // Ambient temperature
SENSOR_TYPE_OBJECT_TEMPERATURE // Object temperature
SENSOR_TYPE_VOLTAGE           // Voltage
SENSOR_TYPE_CURRENT           // Current
SENSOR_TYPE_COLOR             // Color (RGB)
```

## Basic Usage with BME280

```cpp
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>

Adafruit_BME280 bme;
Adafruit_Sensor* bme_temp = bme.getTemperatureSensor();
Adafruit_Sensor* bme_pressure = bme.getPressureSensor();
Adafruit_Sensor* bme_humidity = bme.getHumiditySensor();

void setup() {
    Serial.begin(115200);

    if (!bme.begin()) {
        while (1);
    }

    // Print sensor details
    bme_temp->printSensorDetails();
}

void loop() {
    sensors_event_t temp_event, pressure_event, humidity_event;

    bme_temp->getEvent(&temp_event);
    bme_pressure->getEvent(&pressure_event);
    bme_humidity->getEvent(&humidity_event);

    Serial.printf("Temperature: %.1f°C\n", temp_event.temperature);
    Serial.printf("Pressure: %.1f hPa\n", pressure_event.pressure);
    Serial.printf("Humidity: %.1f%%\n", humidity_event.relative_humidity);

    delay(2000);
}
```

## Multi-Sensor Manager

```cpp
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>
#include <Adafruit_MPU6050.h>

// Unified sensor array
Adafruit_Sensor* sensors[5];
int sensorCount = 0;

void registerSensor(Adafruit_Sensor* sensor) {
    sensors[sensorCount++] = sensor;
}

void setup() {
    Serial.begin(115200);

    // Initialize BME280
    Adafruit_BME280 bme;
    if (bme.begin()) {
        registerSensor(bme.getTemperatureSensor());
        registerSensor(bme.getPressureSensor());
        registerSensor(bme.getHumiditySensor());
    }

    // Initialize MPU6050
    Adafruit_MPU6050 mpu;
    if (mpu.begin()) {
        registerSensor(&mpu);
        registerSensor(&mpu);
    }

    // Print all sensor details
    for (int i = 0; i < sensorCount; i++) {
        sensors[i]->printSensorDetails();
        Serial.println();
    }
}

void readAllSensors() {
    sensors_event_t event;

    for (int i = 0; i < sensorCount; i++) {
        sensors[i]->getEvent(&event);

        switch (event.type) {
            case SENSOR_TYPE_AMBIENT_TEMPERATURE:
                Serial.printf("Sensor %d: %.1f°C\n", i, event.temperature);
                break;
            case SENSOR_TYPE_PRESSURE:
                Serial.printf("Sensor %d: %.1f hPa\n", i, event.pressure);
                break;
            case SENSOR_TYPE_RELATIVE_HUMIDITY:
                Serial.printf("Sensor %d: %.1f%%\n", i, event.relative_humidity);
                break;
            case SENSOR_TYPE_ACCELEROMETER:
                Serial.printf("Sensor %d: %.2f, %.2f, %.2f m/s^2\n",
                             i, event.acceleration.x, event.acceleration.y, event.acceleration.z);
                break;
        }
    }
}

void loop() {
    readAllSensors();
    Serial.println();
    delay(2000);
}
```

## Sensor Information

```cpp
#include <Adafruit_Sensor.h>

void printSensorInfo(Adafruit_Sensor* sensor) {
    sensor_t sensor_info;
    sensor->getSensor(&sensor_info);

    Serial.printf("Name: %s\n", sensor_info.name);
    Serial.printf("Version: %d\n", sensor_info.version);
    Serial.printf("ID: %d\n", sensor_info.sensor_id);
    Serial.printf("Type: %d\n", sensor_info.type);
    Serial.printf("Max Value: %.2f\n", sensor_info.max_value);
    Serial.printf("Min Value: %.2f\n", sensor_info.min_value);
    Serial.printf("Resolution: %.2f\n", sensor_info.resolution);
    Serial.printf("Min Delay: %d us\n", sensor_info.min_delay);
}

void setup() {
    Serial.begin(115200);

    Adafruit_BME280 bme;
    if (bme.begin()) {
        printSensorInfo(bme.getTemperatureSensor());
    }
}

void loop() {}
```

## Sensor Calibration

```cpp
#include <Adafruit_Sensor.h>

struct Calibration {
    float offset;
    float scale;
};

void calibrateSensor(Adafruit_Sensor* sensor, Calibration* cal) {
    sensor_t info;
    sensor->getSensor(&info);

    // Simple zero calibration
    sensors_event_t event;
    float sum = 0;
    int samples = 100;

    for (int i = 0; i < samples; i++) {
        sensor->getEvent(&event);
        sum += event.data[0];  // First channel
        delay(10);
    }

    float average = sum / samples;
    cal->offset = -average;
    cal->scale = 1.0;

    Serial.printf("Calibration offset: %.3f\n", cal->offset);
}

float applyCalibration(float raw, Calibration cal) {
    return (raw + cal.offset) * cal.scale;
}
```

## Sensor Auto-Ranging

```cpp
#include <Adafruit_Sensor.h>

void setupAutoRange(Adafruit_Sensor* sensor) {
    // Enable auto-ranging if supported
    sensor->enableAutoRange(true);
}

void setup() {
    Serial.begin(115200);

    Adafruit_MPU6050 mpu;
    if (mpu.begin()) {
        // Some sensors support automatic range adjustment
        // based on measured values
        setupAutoRange(&mpu);
    }
}

void loop() {
    delay(1000);
}
```

## Sensor Fusion Helper

```cpp
#include <Adafruit_Sensor.h>

class SensorFusion {
private:
    Adafruit_Sensor* accel;
    Adafruit_Sensor* gyro;
    Adafruit_Sensor* mag;

public:
    SensorFusion(Adafruit_Sensor* a, Adafruit_Sensor* g, Adafruit_Sensor* m)
        : accel(a), gyro(g), mag(m) {}

    void readFusedData(float* pitch, float* roll, float* yaw) {
        sensors_event_t a_event, g_event, m_event;

        accel->getEvent(&a_event);
        gyro->getEvent(&g_event);
        mag->getEvent(&m_event);

        // Implement sensor fusion algorithm
        // (complementary filter, Kalman filter, etc.)

        // Simple example - accelerometer tilt
        *pitch = atan2(a_event.acceleration.x, a_event.acceleration.z) * 180.0 / PI;
        *roll = atan2(a_event.acceleration.y, a_event.acceleration.z) * 180.0 / PI;
        *yaw = 0;  // Requires magnetometer
    }
};

void setup() {
    Serial.begin(115200);

    Adafruit_MPU6050 mpu;
    if (mpu.begin()) {
        SensorFusion fusion(&mpu, &mpu, nullptr);

        float pitch, roll, yaw;
        fusion.readFusedData(&pitch, &roll, &yaw);

        Serial.printf("Pitch: %.1f°, Roll: %.1f°, Yaw: %.1f°\n", pitch, roll, yaw);
    }
}

void loop() {}
```

## Best Practices

```cpp
#include <Adafruit_Sensor.h>

void setup() {
    Serial.begin(115200);

    // Best practice 1: Always check sensor initialization
    Adafruit_BME280 bme;
    if (!bme.begin()) {
        Serial.println("Sensor initialization failed");
        while(1);
    }

    // Best practice 2: Use sensor details to configure reading rate
    sensor_t info;
    bme.getTemperatureSensor()->getSensor(&info);
    unsigned long minDelay = info.min_delay;  // microseconds
    Serial.printf("Minimum delay: %lu us\n", minDelay);

    // Best practice 3: Validate sensor data range
    float max = info.max_value;
    float min = info.min_value;
    Serial.printf("Valid range: %.1f to %.1f\n", min, max);

    // Best practice 4: Use unified sensor type detection
    if (info.type == SENSOR_TYPE_AMBIENT_TEMPERATURE) {
        Serial.println("This is a temperature sensor");
    }

    // Best practice 5: Handle sensor-specific events correctly
    sensors_event_t event;
    bme.getTemperatureSensor()->getEvent(&event);

    switch (event.type) {
        case SENSOR_TYPE_AMBIENT_TEMPERATURE:
            Serial.printf("Temperature: %.1f°C\n", event.temperature);
            break;
        default:
            Serial.println("Unknown sensor type");
    }
}

void loop() {
    delay(1000);
}
```

## References

- [Adafruit Unified Sensor Library](https://github.com/adafruit/Adafruit_Sensor)
- [Adafruit Sensor Learning Guide](https://learn.adafruit.com/using-the-adafruit-unified-sensor-driver)
