# IMU Sensor API (LSM6DS3/LSM6DS3TR-C)

## Overview

The 6-axis IMU (Inertial Measurement Unit) sensors are available on XIAO Sense boards:
- **XIAO nRF52840 Sense**: LSM6DS3
- **XIAO MG24 Sense**: LSM6DS3TR-C
- **XIAO nRF54L15 Sense**: LSM6DS3TR-C (MicroPython only)

These sensors provide:
- 3-axis accelerometer (measure acceleration in g-force)
- 3-axis gyroscope (measure angular velocity in degrees per second)
- Embedded temperature sensor

## Board Compatibility

| Board | IMU Sensor | I2C Address | Power Pin | Library |
|-------|-----------|-------------|-----------|---------|
| nRF52840 Sense | LSM6DS3 | 0x6A | None | Seeed_Arduino_LSM6DS3 |
| MG24 Sense | LSM6DS3TR-C | 0x6A | PD5 (must be HIGH) | Seeed_Arduino_LSM6DS3 (v2.0.4+) |
| nRF54L15 Sense | LSM6DS3TR-C | - | - | Zephyr sensor API |

## Basic Setup

### nRF52840 Sense

```cpp
#include <LSM6DS3.h>
#include <Wire.h>

LSM6DS3 myIMU(I2C_MODE, 0x6A);

void setup() {
    Serial.begin(115200);
    while (!Serial);

    if (myIMU.begin() != 0) {
        Serial.println("IMU initialization failed");
        while (1);
    }
    Serial.println("IMU ready");
}

void loop() {
    // Read sensor data
    float accX = myIMU.readFloatAccelX();
    float accY = myIMU.readFloatAccelY();
    float accZ = myIMU.readFloatAccelZ();

    float gyroX = myIMU.readFloatGyroX();
    float gyroY = myIMU.readFloatGyroY();
    float gyroZ = myIMU.readFloatGyroZ();

    // Print data
    Serial.print("Accel: "); Serial.print(accX); Serial.print(", ");
    Serial.print(accY); Serial.print(", "); Serial.println(accZ);
    Serial.print("Gyro: "); Serial.print(gyroX); Serial.print(", ");
    Serial.print(gyroY); Serial.print(", "); Serial.println(gyroZ);

    delay(100);
}
```

### MG24 Sense

```cpp
#include <LSM6DS3.h>
#include <Wire.h>

LSM6DS3 myIMU(I2C_MODE, 0x6A);

void setup() {
    Serial.begin(9600);
    while (!Serial);

    // IMPORTANT: Enable IMU power first
    pinMode(PD5, OUTPUT);
    digitalWrite(PD5, HIGH);
    delay(300);

    if (myIMU.begin() != 0) {
        Serial.println("IMU initialization failed");
        while (1);
    }
    Serial.println("aX,aY,aZ,gX,gY,gZ");
}

void loop() {
    // Print CSV format for Serial Plotter
    Serial.print(myIMU.readFloatAccelX(), 3); Serial.print(',');
    Serial.print(myIMU.readFloatAccelY(), 3); Serial.print(',');
    Serial.print(myIMU.readFloatAccelZ(), 3); Serial.print(',');
    Serial.print(myIMU.readFloatGyroX(), 3); Serial.print(',');
    Serial.print(myIMU.readFloatGyroY(), 3); Serial.print(',');
    Serial.println(myIMU.readFloatGyroZ(), 3);

    delay(100);
}
```

## API Reference

### Constructor

```cpp
LSM6DS3(interface_type, address)
```

**Parameters:**
- `interface_type`: `I2C_MODE` or `SPI_MODE`
- `address`: I2C address (default `0x6A`)

### Initialization

```cpp
int begin(void)
```

Initialize the IMU sensor.

**Returns:** `0` on success, non-zero on failure

### Accelerometer Functions

```cpp
float readFloatAccelX(void)
float readFloatAccelY(void)
float readFloatAccelZ(void)
```

Read accelerometer values in g-force (9.8 m/s²).

**Returns:** Acceleration in g (typical range: -2 to +2 g)

**Example:**
```cpp
float ax = myIMU.readFloatAccelX();  // -2.0 to +2.0 g
```

### Gyroscope Functions

```cpp
float readFloatGyroX(void)
float readFloatGyroY(void)
float readFloatGyroZ(void)
```

Read gyroscope values in degrees per second.

**Returns:** Angular velocity in °/s (typical range: -500 to +500 °/s)

**Example:**
```cpp
float gx = myIMU.readFloatGyroX();  // Degrees per second
```

### Temperature Function

```cpp
float readTemperatureC(void)
```

Read the embedded temperature sensor.

**Returns:** Temperature in Celsius

## Advanced Usage

### Motion Detection

Detect significant motion using accelerometer threshold:

```cpp
const float accelerationThreshold = 2.5; // g-force threshold

void detectMotion() {
    float aX = myIMU.readFloatAccelX();
    float aY = myIMU.readFloatAccelY();
    float aZ = myIMU.readFloatAccelZ();

    float aSum = fabs(aX) + fabs(aY) + fabs(aZ);

    if (aSum >= accelerationThreshold) {
        Serial.println("Motion detected!");
        // Trigger action
    }
}
```

### Orientation Detection

Calculate pitch and roll angles from accelerometer:

```cpp
void calculateOrientation() {
    float ax = myIMU.readFloatAccelX();
    float ay = myIMU.readFloatAccelY();
    float az = myIMU.readFloatAccelZ();

    // Calculate pitch (rotation around Y-axis)
    float pitch = atan2(ay, sqrt(ax*ax + az*az)) * 180.0 / PI;

    // Calculate roll (rotation around X-axis)
    float roll = atan2(-ax, sqrt(ay*ay + az*az)) * 180.0 / PI;

    Serial.print("Pitch: "); Serial.print(pitch); Serial.println("°");
    Serial.print("Roll: "); Serial.print(roll); Serial.println("°");
}
```

### Data Logging to Serial Plotter

Use CSV format for Arduino Serial Plotter:

```cpp
void loop() {
    Serial.print("AccelX:"); Serial.print(myIMU.readFloatAccelX());
    Serial.print(",AccelY:"); Serial.print(myIMU.readFloatAccelY());
    Serial.print(",AccelZ:"); Serial.println(myIMU.readFloatAccelZ());
    delay(50);
}
```

## Configuration Options

### Accelerometer Range

The default accelerometer range is typically ±2g. To change:

```cpp
// Note: See library header for specific configuration functions
// Default: ±2g (most sensitive)
// Options: ±4g, ±8g, ±16g (less sensitive, higher range)
```

### Gyroscope Range

The default gyroscope range is typically ±500 dps:

```cpp
// Note: See library header for specific configuration functions
// Default: ±500 dps
// Options: ±125, ±245, ±500, ±1000, ±2000 dps
```

## Data Output Modes

### Raw Values (CSV)

```cpp
void loop() {
    Serial.print(myIMU.readFloatAccelX()); Serial.print(",");
    Serial.print(myIMU.readFloatAccelY()); Serial.print(",");
    Serial.print(myIMU.readFloatAccelZ()); Serial.print(",");
    Serial.print(myIMU.readFloatGyroX()); Serial.print(",");
    Serial.print(myIMU.readFloatGyroY()); Serial.print(",");
    Serial.println(myIMU.readFloatGyroZ());
    delay(100);
}
```

### Serial Plotter Format

```cpp
void loop() {
    Serial.print("X:"); Serial.print(myIMU.readFloatAccelX());
    Serial.print(",Y:"); Serial.print(myIMU.readFloatAccelY());
    Serial.print(",Z:"); Serial.println(myIMU.readFloatAccelZ());
    delay(50);
}
```

## Troubleshooting

### IMU Not Responding

1. **Check I2C connection**: Ensure D4/D5 are connected properly
2. **Check I2C address**: Verify address is 0x6A
3. **For MG24**: Ensure PD5 is set HIGH to enable IMU power
4. **Check library version**: For MG24, use LSM6DS3 library v2.0.4 or higher

### Always Returns Zero

1. IMU may not be initialized - check `begin()` return value
2. I2C communication issue - check Wire.begin() is called
3. Wrong I2C pins - verify D4/D5 are used

### Noisy Readings

1. Add averaging/filtering to smooth data
2. Check for mechanical vibrations
3. Adjust sample rate
4. Use lower range for better resolution

## Examples

### Simple Orientation Indicator

```cpp
void loop() {
    float ax = myIMU.readFloatAccelX();
    float ay = myIMU.readFloatAccelY();
    float az = myIMU.readFloatAccelZ();

    if (az > 0.8) {
        Serial.println("Face up");
    } else if (az < -0.8) {
        Serial.println("Face down");
    } else if (ax > 0.8) {
        Serial.println("Tilted right");
    } else if (ax < -0.8) {
        Serial.println("Tilted left");
    }

    delay(200);
}
```

## Resources

- [LSM6DS3 Datasheet](https://www.st.com/resource/en/datasheet/lsm6ds3.pdf)
- [LSM6DS3TR-C Datasheet](https://www.st.com/resource/en/datasheet/lsm6ds3tr-c.pdf)
- [Seeed Arduino LSM6DS3 Library](https://github.com/Seeed-Studio/Seeed_Arduino_LSM6DS3)
