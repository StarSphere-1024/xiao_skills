# Stepper Library for XIAO

## Overview

The Stepper library controls stepper motors for precise position control. Suitable for 3D printers, CNC machines, camera sliders, and robotics.

## Board Compatibility

All XIAO boards support stepper motor control:
- ESP32C3/C5/C6/S3: Hardware PWM + timers
- nRF52840/MG24: Hardware PWM
- RP2040/RP2350: PIO for precise timing
- SAMD21/RA4M1: Timer-based control

## Basic Stepper Control

```cpp
#include <Stepper.h>

// Motor configuration
const int STEPS_PER_REVOLUTION = 2048;  // 28BYJ-48 stepper
const int MOTOR_SPEED = 10;  // RPM

Stepper myStepper(STEPS_PER_REVOLUTION, D0, D1, D2, D3);

void setup() {
    myStepper.setSpeed(MOTOR_SPEED);
}

void loop() {
    // Rotate one revolution clockwise
    myStepper.step(STEPS_PER_REVOLUTION);
    delay(1000);

    // Rotate one revolution counter-clockwise
    myStepper.step(-STEPS_PER_REVOLUTION);
    delay(1000);
}
```

## 28BYJ-48 with ULN2003

```cpp
#include <Stepper.h>

const int STEPS_PER_REV = 2048;

// XIAO pin configuration
const int IN1 = D0;
const int IN2 = D1;
const int IN3 = D2;
const int IN4 = D3;

Stepper stepper(STEPS_PER_REV, IN1, IN3, IN2, IN4);

void setup() {
    stepper.setSpeed(10);  // 10 RPM (max reliable speed)
}

void loop() {
    // Rotate 90 degrees
    stepper.step(STEPS_PER_REV / 4);
    delay(500);

    // Rotate -90 degrees
    stepper.step(-STEPS_PER_REV / 4);
    delay(500);
}
```

## Precise Position Control

```cpp
#include <Stepper.h>

const int STEPS_PER_REV = 2048;
Stepper myStepper(STEPS_PER_REV, D0, D1, D2, D3);

long currentPosition = 0;
long targetPosition = 0;

void moveTo(long steps) {
    long delta = steps - currentPosition;

    if (delta != 0) {
        myStepper.step(delta);
        currentPosition = steps;
    }
}

void setup() {
    myStepper.setSpeed(10);
    Serial.begin(115200);
}

void loop() {
    // Move to absolute position
    moveTo(512);   // 90 degrees
    delay(1000);

    moveTo(1024);  // 180 degrees
    delay(1000);

    moveTo(0);     // Back to origin
    delay(1000);
}
```

## Acceleration Control (Custom)

```cpp
#include <Stepper.h>

const int STEPS_PER_REV = 2048;
Stepper myStepper(STEPS_PER_REV, D0, D1, D2, D3);

void acceleratedMove(int steps, int minSpeed, int maxSpeed) {
    int totalSteps = abs(steps);
    int direction = steps > 0 ? 1 : -1;

    // Acceleration phase (first half)
    for (int i = 0; i < totalSteps / 2; i++) {
        int speed = map(i, 0, totalSteps / 2, minSpeed, maxSpeed);
        myStepper.setSpeed(speed);
        myStepper.step(direction);
    }

    // Deceleration phase (second half)
    for (int i = totalSteps / 2; i < totalSteps; i++) {
        int speed = map(i, totalSteps / 2, totalSteps, maxSpeed, minSpeed);
        myStepper.setSpeed(speed);
        myStepper.step(direction);
    }
}

void setup() {
    Serial.begin(115200);
}

void loop() {
    // Move with acceleration
    acceleratedMove(STEPS_PER_REV, 5, 15);  // 1 revolution
    delay(1000);
    acceleratedMove(-STEPS_PER_REV, 5, 15);  // Reverse
    delay(1000);
}
```

## Multi-Motor Control

```cpp
#include <Stepper.h>

const int STEPS = 2048;

Stepper motor1(STEPS, D0, D1, D2, D3);
Stepper motor2(STEPS, D4, D5, D6, D7);

void setup() {
    motor1.setSpeed(10);
    motor2.setSpeed(10);
}

void loop() {
    // Both motors forward
    for (int i = 0; i < STEPS / 4; i++) {
        motor1.step(1);
        motor2.step(1);
    }
    delay(500);

    // Both motors backward
    for (int i = 0; i < STEPS / 4; i++) {
        motor1.step(-1);
        motor2.step(-1);
    }
    delay(500);

    // Motors in opposite directions
    for (int i = 0; i < STEPS / 4; i++) {
        motor1.step(1);
        motor2.step(-1);
    }
    delay(500);
}
```

## Non-Blocking Stepper (AccelStepper)

```cpp
#include <AccelStepper.h>

// Define stepper motor
// AccelStepper::FULL4WIRE for 4-wire stepper
AccelStepper stepper(AccelStepper::FULL4WIRE, D0, D1, D2, D3);

void setup() {
    // Set speed and acceleration
    stepper.setMaxSpeed(500);    // steps per second
    stepper.setAcceleration(100);  // steps per second^2

    // Move to position
    stepper.moveTo(2048);
}

void loop() {
    // Must call run() in loop for non-blocking operation
    stepper.run();

    // Check if target reached
    if (stepper.distanceToGo() == 0) {
        // Move to new position
        static long positions[] = {0, 2048, 4096, 0};
        static int index = 0;

        index = (index + 1) % 4;
        stepper.moveTo(positions[index]);
    }
}
```

## End Stop Switch

```cpp
#include <AccelStepper.h>

const int END_STOP_PIN = D8;

AccelStepper stepper(AccelStepper::FULL4WIRE, D0, D1, D2, D3);
bool homingComplete = false;

void homeStepper() {
    Serial.println("Homing...");

    stepper.setSpeed(-200);  // Move toward end stop

    while (digitalRead(END_STOP_PIN) == HIGH) {
        stepper.runSpeed();
    }

    stepper.setCurrentPosition(0);
    stepper.moveTo(1000);  // Back off slightly

    while (stepper.distanceToGo() != 0) {
        stepper.run();
    }

    homingComplete = true;
    Serial.println("Homing complete");
}

void setup() {
    pinMode(END_STOP_PIN, INPUT_PULLUP);
    Serial.begin(115200);

    stepper.setMaxSpeed(500);
    stepper.setAcceleration(100);

    homeStepper();
}

void loop() {
    if (homingComplete && stepper.distanceToGo() == 0) {
        // Run sequence after homing
        stepper.moveTo(2048);
    }
    stepper.run();
}
```

## Speed Control

```cpp
#include <Stepper.h>

const int STEPS = 2048;
Stepper myStepper(STEPS, D0, D1, D2, D3);

void variableSpeedDemo() {
    // Test different speeds
    int speeds[] = {5, 10, 15};

    for (int i = 0; i < 3; i++) {
        myStepper.setSpeed(speeds[i]);
        Serial.printf("Speed: %d RPM\n", speeds[i]);

        // Rotate 90 degrees
        myStepper.step(STEPS / 4);
        delay(1000);

        // Return
        myStepper.step(-STEPS / 4);
        delay(1000);
    }
}

void setup() {
    Serial.begin(115200);
}

void loop() {
    variableSpeedDemo();
    delay(2000);
}
```

## Angle Control

```cpp
#include <Stepper.h>

const int STEPS = 2048;
Stepper myStepper(STEPS, D0, D1, D2, D3);

void setAngle(float angle) {
    // Convert angle to steps
    long steps = (angle * STEPS) / 360.0;
    myStepper.step(steps);
}

void setup() {
    myStepper.setSpeed(10);
    Serial.begin(115200);
}

void loop() {
    // Rotate to specific angles
    setAngle(0);     // 0 degrees
    delay(1000);

    setAngle(90);    // 90 degrees
    delay(1000);

    setAngle(180);   // 180 degrees
    delay(1000);

    setAngle(270);   // 270 degrees
    delay(1000);

    setAngle(360);   // Full rotation
    delay(1000);
}
```

## Continuous Rotation

```cpp
#include <Stepper.h>

const int STEPS = 2048;
Stepper myStepper(STEPS, D0, D1, D2, D3);

void continuousRotate(int revolutions, int direction) {
    long steps = revolutions * STEPS * direction;

    for (long i = 0; i < abs(steps); i++) {
        myStepper.step(direction);
    }
}

void setup() {
    myStepper.setSpeed(10);
}

void loop() {
    // Rotate 5 times clockwise
    continuousRotate(5, 1);
    delay(1000);

    // Rotate 5 times counter-clockwise
    continuousRotate(5, -1);
    delay(1000);
}
```

## Power Management

```cpp
#include <Stepper.h>

const int STEPS = 2048;
const int ENABLE_PIN = D8;  // If driver has enable pin

Stepper myStepper(STEPS, D0, D1, D2, D3);

void enableMotor() {
    pinMode(ENABLE_PIN, OUTPUT);
    digitalWrite(ENABLE_PIN, LOW);  // Enable motor (active low)
}

void disableMotor() {
    digitalWrite(ENABLE_PIN, HIGH);  // Disable motor
    delay(100);  // Let motor stop
}

void moveAndDisable(int steps) {
    enableMotor();
    myStepper.setSpeed(10);
    myStepper.step(steps);
    delay(100);  // Wait for movement to complete
    disableMotor();  // Save power
}

void setup() {
    pinMode(ENABLE_PIN, OUTPUT);
    disableMotor();  // Start disabled
}

void loop() {
    moveAndDisable(STEPS / 4);  // 90 degrees
    delay(2000);  // Stay powered off for 2 seconds
}
```

## Best Practices

```cpp
#include <Stepper.h>

const int STEPS = 2048;
Stepper myStepper(STEPS, D0, D1, D2, D3);

void setup() {
    // Best practice 1: Use appropriate speed
    myStepper.setSpeed(10);  // 5-15 RPM for 28BYJ-48

    // Best practice 2: Add delays between direction changes
    // to allow motor to stop
}

// Best practice 3: Use microsteps for precision
void preciseMove(int steps) {
    for (int i = 0; i < steps; i++) {
        myStepper.step(1);
        delay(10);  // Small delay for stability
    }
}

// Best practice 4: Monitor current (if possible)
bool checkCurrentLimit(int current) {
    const int MAX_CURRENT = 300;  // mA
    return current < MAX_CURRENT;
}

// Best practice 5: Use mechanical end stops
bool checkEndStop(int pin) {
    return digitalRead(pin) == LOW;  // Active low
}

void loop() {
}
```

## References

- [Arduino Stepper Library](https://www.arduino.cc/reference/en/libraries/stepper/)
- [AccelStepper Library](http://www.airspayce.com/mikem/arduino/AccelStepper/)
- [28BYJ-48 Datasheet](https://components101.com/motors/28byj-48-stepper-motor)
