# Servo Library for XIAO

## Overview

The Servo library allows control of RC (hobby) servos. All XIAO GPIO pins support servo output.

## Basic Usage

```cpp
#include <Servo.h>

Servo myservo;
const int servoPin = D10;

void setup() {
    myservo.attach(servoPin);
}

void loop() {
    myservo.write(0);    // 0 degrees
    delay(1000);
    myservo.write(90);   // 90 degrees (center)
    delay(1000);
    myservo.write(180);  // 180 degrees
    delay(1000);
}
```

## Pin Selection

**Important**: ESP32 boards can use most GPIO pins, but avoid:
- Strapping pins (ESP32C3: D0, D8, D9)
- Boot pins (ESP32C3: D8, D9)
- UART TX at startup (ESP32C3: D6)

**Recommended**: D4, D5, D7, D10 for ESP32C3
**Any GPIO**: nRF52840, RP2040, SAMD21

## Multiple Servos

```cpp
#include <Servo.h>

Servo servo1;
Servo servo2;
Servo servo3;

void setup() {
    servo1.attach(D4);
    servo2.attach(D5);
    servo3.attach(D10);
}

void loop() {
    servo1.write(90);
    servo2.write(45);
    servo3.write(135);
}
```

## Read Current Angle

```cpp
int angle = myservo.read();  // Returns last written angle (0-180)
```

## Detach Servo

```cpp
myservo.detach();  // Stop PWM pulses, saves power
```

## Sweep Example

```cpp
#include <Servo.h>

Servo myservo;
int pos = 0;

void setup() {
    myservo.attach(D10);
}

void loop() {
    for (pos = 0; pos <= 180; pos += 1) {
        myservo.write(pos);
        delay(15);
    }
    for (pos = 180; pos >= 0; pos -= 1) {
        myservo.write(pos);
        delay(15);
    }
}
```

## Control with Potentiometer

```cpp
#include <Servo.h>

Servo myservo;
const int potPin = A0;

void setup() {
    myservo.attach(D10);
}

void loop() {
    int potValue = analogRead(potPin);
    int angle = map(potValue, 0, 4095, 0, 180);  // ESP32: 0-4095
    myservo.write(angle);
    delay(15);
}
```

## ESP32 LEDC Servo (Alternative)

ESP32 has native LEDC that can be used for servos:

```cpp
#include <ESP32Servo.h>

ESP32Servo myservo;

void setup() {
    myservo.setPeriodHertz(50);  // Standard 50Hz
    myservo.attach(D10, 500, 2400);  // Min/max pulse in microseconds
}

void loop() {
    myservo.write(90);
    delay(1000);
}
```

## Continuous Rotation Servo

```cpp
// Some servos rotate continuously
// 90 = stop, 0 = full speed CCW, 180 = full speed CW

Servo continuousServo;

void setup() {
    continuousServo.attach(D10);
}

void loop() {
    continuousServo.write(90);   // Stop
    delay(1000);
    continuousServo.write(180);  // Full speed CW
    delay(2000);
    continuousServo.write(90);   // Stop
    delay(1000);
    continuousServo.write(0);    // Full speed CCW
    delay(2000);
}
```

## Power Considerations

- **Servo current**: 100-500mA per servo when moving
- **XIAO 5V pin**: Can supply ~500mA (from USB)
- **Multiple servos**: Use external power supply
- **Brownout risk**: High current can cause XIAO to reset

## Wiring Multiple Servos

```cpp
     External 5V Supply
          |
    +-----+-----+-----+
    |     |     |     |
   SV1  SV2  SV3  SV4   (Servos)
    |     |     |     |
    +-----+-----+-----+
          |
         GND (Common ground with XIAO)

Signal wires to XIAO GPIO pins
```

## Smooth Movement

```cpp
#include <Servo.h>

Servo myservo;
int currentPos = 0;
int targetPos = 180;

void setup() {
    myservo.attach(D10);
}

void loop() {
    if (currentPos < targetPos) {
        currentPos++;
    } else if (currentPos > targetPos) {
        currentPos--;
    }
    myservo.write(currentPos);
    delay(20);  // Adjust for speed
}
```

## Micro Servo Limits

Most micro servos (SG90):
- **Voltage**: 4.8V - 6V
- **Pulse width**: 1000μs - 2000μs
- **Default library**: 544μs - 2400μs (wider range)

For precise control:

```cpp
myservo.attach(pin, 1000, 2000);  // Min/max microseconds
```

## Troubleshooting

### Servo jittering

1. **Power issue**: Use external 5V supply
2. **Weak signal**: Short wires
3. **Wrong pulse range**: Adjust min/max

### Servo not moving

1. **No power**: Check 5V supply
2. **Wrong pin**: Verify GPIO number
3. **Signal issue**: Check wiring

### XIAO resets

1. **Brownout**: Servo drawing too much current
2. **Solution**: External power supply
