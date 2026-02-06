# Bus Servo Driver Board for XIAO - Arduino

## Overview

Arduino implementation for the XIAO Bus Servo Driver Board. Controls Feetech SC/ST series bus servos via UART. Used in robotics, automation, and animatronics.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/bus-servo.md`.

## Pin Connections

| XIAO Pin | Bus Servo Board | Function |
|----------|-----------------|----------|
| D7 (TX) | RX | Data to servos |
| D6 (RX) | TX | Data from servos |
| 5V | VCC | Servo power |
| GND | GND | Common ground |

**Important Jumper Settings:**
- **UART mode:** For XIAO control (default position)
- **USB mode:** For PC configuration (move jumper)

**Baud Rate:** 1,000,000 bps (1 Mbps)

## Board Types

Two versions available:
1. **Driver Board:** Requires separate XIAO
2. **Adapter with XIAO ESP32-C3:** Integrated solution

## Required Libraries

| Library | Purpose |
|---------|---------|
| **SCServo** | Feetech bus servo control |

Install from: https://github.com/Seeed-Studio/BusServo

## Basic Servo Control

```cpp
#include <SCServo.h>

SMS_STS st;

void setup() {
  Serial.begin(115200);
  Serial1.begin(1000000);  // 1 Mbps for bus servos
  st.pSerial = &Serial1;   // Use Serial1 (D6/D7)
  delay(1000);
}

void loop() {
  // Move servo ID 1 to position 1000
  st.WritePosEx(1, 1000, 1000, 50);
  // ID=1, position=1000, speed=1000, acceleration=50
  delay(1000);

  // Move servo ID 1 to position 0
  st.WritePosEx(1, 0, 1000, 50);
  delay(1000);
}
```

## Multiple Servos

```cpp
#include <SCServo.h>

SMS_STS st;

void setup() {
  Serial.begin(115200);
  Serial1.begin(1000000);
  st.pSerial = &Serial1;
  delay(1000);
}

void loop() {
  // Move multiple servos simultaneously
  st.WritePosEx(1, 1000, 1000, 50);  // Servo 1 to 1000
  st.WritePosEx(2, 500, 1000, 50);   // Servo 2 to 500
  st.WritePosEx(3, 2000, 1000, 50);  // Servo 3 to 2000
  delay(1000);

  // Return to center
  st.WritePosEx(1, 1000, 1000, 50);
  st.WritePosEx(2, 1000, 1000, 50);
  st.WritePosEx(3, 1000, 1000, 50);
  delay(1000);
}
```

## Read Servo Position

```cpp
#include <SCServo.h>

SMS_STS st;

void setup() {
  Serial.begin(115200);
  Serial1.begin(1000000);
  st.pSerial = &Serial1;
  delay(1000);
}

void loop() {
  // Read position of servo ID 1
  int pos = st.ReadPos(1);
  if (pos != -1) {
    Serial.print("Servo 1 Position: ");
    Serial.println(pos);
  }
  delay(100);
}
```

## Wheel Mode (Continuous Rotation)

```cpp
#include <SCServo.h>

SMS_STS st;

void setup() {
  Serial.begin(115200);
  Serial1.begin(1000000);
  st.pSerial = &Serial1;
  delay(1000);

  // Set servo to wheel mode
  st.WriteMode(1, 0);  // ID=1, mode=0 (wheel mode)
}

void loop() {
  // Forward at speed 1000
  st.WriteSpe(1, 1000);
  delay(2000);

  // Stop
  st.WriteSpe(1, 0);
  delay(1000);

  // Reverse at speed -1000
  st.WriteSpe(1, -1000);
  delay(2000);

  // Stop
  st.WriteSpe(1, 0);
  delay(1000);
}
```

## Servo Calibration

```cpp
#include <SCServo.h>

SMS_STS st;

void setup() {
  Serial.begin(115200);
  Serial1.begin(1000000);
  st.pSerial = &Serial1;
  delay(1000);

  // Set position limits (0-4095 range)
  st.WritePosLimit(1, 200, 3800);  // ID=1, min=200, max=3800

  // Set voltage limits (6500-8400 mV)
  st.WriteVoltageLimit(1, 6500, 8400);

  // Set max temperature (80°C)
  st.WriteMaxTemperature(1, 80);

  // Set torque limit (500/1000)
  st.WriteTorqueLimit(1, 800);
}

void loop() {
  // Servo is now calibrated and safe
  st.WritePosEx(1, 2000, 1000, 50);
  delay(1000);
}
```

## Read Servo Status

```cpp
#include <SCServo.h>

SMS_STS st;

void setup() {
  Serial.begin(115200);
  Serial1.begin(1000000);
  st.pSerial = &Serial1;
  delay(1000);
}

void loop() {
  // Read voltage
  int voltage = st.ReadVoltage(1);
  Serial.print("Voltage: ");
  Serial.print(voltage);
  Serial.println(" mV");

  // Read temperature
  int temp = st.ReadTemperature(1);
  Serial.print("Temperature: ");
  Serial.print(temp);
  Serial.println(" C");

  // Read position
  int pos = st.ReadPos(1);
  Serial.print("Position: ");
  Serial.println(pos);

  // Read speed
  int speed = st.ReadSpeed(1);
  Serial.print("Speed: ");
  Serial.println(speed);

  // Read load
  int load = st.ReadLoad(1);
  Serial.print("Load: ");
  Serial.println(load);

  Serial.println("---");
  delay(1000);
}
```

## LED Control

Many bus servos have status LEDs:

```cpp
#include <SCServo.h>

SMS_STS st;

void setup() {
  Serial.begin(115200);
  Serial1.begin(1000000);
  st.pSerial = &Serial1;
  delay(1000);
}

void loop() {
  // Set LED color (RGB)
  st.WriteLED(1, 255, 0, 0);   // Red
  delay(1000);

  st.WriteLED(1, 0, 255, 0);   // Green
  delay(1000);

  st.WriteLED(1, 0, 0, 255);   // Blue
  delay(1000);

  st.WriteLED(1, 255, 255, 0); // Yellow
  delay(1000);
}
```

## Position Range

**STS Series Servos:**
- Position range: 0-4095 (typically 0-360° range)
- Center position: ~2048
- Speed: 0-2400 steps/sec
- Torque: 0-1000 (unit dependent on servo model)

## Troubleshooting

### Servo Not Responding

**Symptom:** Servo doesn't move

**Possible Causes:**
1. Wrong baud rate - Must be 1,000,000 bps
2. Wrong ID - Default ID may not be 1
3. Jumper in wrong position - Check UART mode

**Solution:**
```cpp
// Scan for servo ID
void scanServo() {
  for (int id = 1; id <= 253; id++) {
    int pos = st.ReadPos(id);
    if (pos != -1) {
      Serial.print("Found servo at ID: ");
      Serial.println(id);
    }
  }
}
```

### Servo Jitter

**Symptom:** Servo vibrates or jitters

**Possible Causes:**
1. Insufficient power
2. Loose wiring
3. Wrong position values

**Solution:**
- Use adequate power supply (2A+ per servo)
- Check wiring connections
- Verify position within valid range (0-4095)

### Communication Errors

**Symptom:** Garbled data or no response

**Possible Causes:**
1. TX/RX reversed
2. Wrong Serial port
3. Electrical noise

**Solution:**
- Verify D6=RX, D7=TX
- Use Serial1 (not Serial)
- Add 120Ω termination if needed

## Power Requirements

**Per Servo (typical stall current):**
- SCS15: 1.5A
- SCS20: 2.0A
- SCS30: 3.0A

**Power Supply:**
- Voltage: 6V-7.4V (check servo spec)
- Current: Sum of all servos + 20% margin
- Use separate power for servos (not through XIAO)

## Notes

- Always use external power for servos
- Common ground required between XIAO and servo power
- Maximum 253 servos on single bus
- Use 4-wire cable for longer distances
