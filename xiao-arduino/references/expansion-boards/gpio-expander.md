# GPIO Expander for XIAO - Arduino

## Overview

Arduino implementation for the XIAO GPIO Expander using MCP23017. Adds 16 additional digital I/O pins via I2C. Each pin can be input or output with optional pull-up resistors and interrupt capability.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/gpio-expander.md`.

## Pin Connections

| Function | XIAO Pin | Notes |
|----------|----------|-------|
| I2C SDA | D4 | Default Grove I2C |
| I2C SCL | D5 | Default Grove I2C |
| Interrupt | Optional | Any available GPIO pin |

**MCP23017 Pin Mapping:**
```
Port A: GPA0-GPA7 (bits 0-7)
Port B: GPB0-GPB7 (bits 8-15)
```

**I2C Address:** 0x20-0x27 (configurable via A0/A1/A2 jumpers)

## Required Libraries

| Library | Purpose |
|---------|---------|
| **Adafruit MCP23017** | MCP23017 driver |

## Basic Digital Output

```cpp
#include <Adafruit_MCP23X17.h>

Adafruit_MCP23X17 mcp;

void setup() {
  Wire.begin();
  if (!mcp.begin_I2C()) {
    Serial.println("MCP23017 not found!");
    while (1);
  }

  // Set GPA0 as output
  mcp.pinMode(0, OUTPUT);
}

void loop() {
  mcp.digitalWrite(0, HIGH);
  delay(1000);
  mcp.digitalWrite(0, LOW);
  delay(1000);
}
```

## Multiple Outputs

```cpp
#include <Adafruit_MCP23X17.h>

Adafruit_MCP23X17 mcp;

void setup() {
  Wire.begin();
  mcp.begin_I2C();

  // Set pins 0-7 as outputs
  for (int i = 0; i < 8; i++) {
    mcp.pinMode(i, OUTPUT);
  }
}

void loop() {
  // Blink all 8 pins
  for (int i = 0; i < 8; i++) {
    mcp.digitalWrite(i, HIGH);
    delay(100);
  }

  for (int i = 0; i < 8; i++) {
    mcp.digitalWrite(i, LOW);
    delay(100);
  }

  delay(1000);
}
```

## Digital Input with Pull-up

```cpp
#include <Adafruit_MCP23X17.h>

Adafruit_MCP23X17 mcp;

void setup() {
  Serial.begin(115200);
  Wire.begin();
  mcp.begin_I2C();

  // Set GPA0 as input with pull-up
  mcp.pinMode(0, INPUT_PULLUP);
}

void loop() {
  int state = mcp.digitalRead(0);
  Serial.print("Pin 0: ");
  Serial.println(state);

  if (state == LOW) {
    Serial.println("Button pressed!");
  }
  delay(100);
}
```

## Read All Inputs

```cpp
#include <Adafruit_MCP23X17.h>

Adafruit_MCP23X17 mcp;

void setup() {
  Serial.begin(115200);
  Wire.begin();
  mcp.begin_I2C();

  // Set all Port A pins as inputs
  for (int i = 0; i < 8; i++) {
    mcp.pinMode(i, INPUT_PULLUP);
  }
}

void loop() {
  // Read all Port A pins at once
  uint16_t pins = mcp.readGPIOAB();

  Serial.print("Port A: ");
  for (int i = 0; i < 8; i++) {
    Serial.print((pins >> i) & 1);
  }
  Serial.println();

  delay(500);
}
```

## Interrupt-Driven Input

```cpp
#include <Adafruit_MCP23X17.h>

Adafruit_MCP23X17 mcp;
volatile bool interruptFlag = false;

#define INT_PIN D2  // Connect INT to D2

void IRAM_ATTR handleInterrupt() {
  interruptFlag = true;
}

void setup() {
  Serial.begin(115200);
  Wire.begin();
  mcp.begin_I2C();

  // Configure interrupts on GPA0-GPA3
  for (int i = 0; i < 4; i++) {
    mcp.pinMode(i, INPUT_PULLUP);
    mcp.setupInterruptPin(i, CHANGE);
  }

  // Set up interrupt on XIAO
  pinMode(INT_PIN, INPUT);
  attachInterrupt(digitalPinToInterrupt(INT_PIN), handleInterrupt, FALLING);
}

void loop() {
  if (interruptFlag) {
    interruptFlag = false;

    // Read interrupt flags
    uint8_t pin = mcp.getLastInterruptPin();
    uint8_t state = mcp.getLastInterruptPinValue();

    Serial.print("Interrupt on pin ");
    Serial.print(pin);
    Serial.print(", state: ");
    Serial.println(state);

    // Clear interrupts
    mcp.readGPIOAB();
  }
}
```

## Output Port (Write Multiple Pins)

```cpp
#include <Adafruit_MCP23X17.h>

Adafruit_MCP23X17 mcp;

void setup() {
  Wire.begin();
  mcp.begin_I2C();

  // Set all Port A pins as outputs
  for (int i = 0; i < 8; i++) {
    mcp.pinMode(i, OUTPUT);
  }
}

void loop() {
  // Binary counter on 8 LEDs
  for (int count = 0; count < 256; count++) {
    mcp.writeGPIOAB(count);
    delay(200);
  }
}
```

## Multiple MCP23017s

```cpp
#include <Adafruit_MCP23X17.h>

Adafruit_MCP23X17 mcp1;
Adafruit_MCP23X17 mcp2;

void setup() {
  Wire.begin();

  // First MCP23017 at address 0x20 (default)
  mcp1.begin_I2C(0x20);

  // Second MCP23017 at address 0x21 (A0 jumper set)
  mcp2.begin_I2C(0x21);

  // Configure both
  mcp1.pinMode(0, OUTPUT);
  mcp2.pinMode(0, OUTPUT);
}

void loop() {
  mcp1.digitalWrite(0, HIGH);
  mcp2.digitalWrite(0, LOW);
  delay(1000);

  mcp1.digitalWrite(0, LOW);
  mcp2.digitalWrite(0, HIGH);
  delay(1000);
}
```

## I2C Address Configuration

The MCP23017 supports 8 different I2C addresses:

| A0 | A1 | A2 | Address |
|----|----|----|---------|
| 0  | 0  | 0  | 0x20    |
| 1  | 0  | 0  | 0x21    |
| 0  | 1  | 0  | 0x22    |
| 1  | 1  | 0  | 0x23    |
| 0  | 0  | 1  | 0x24    |
| 1  | 0  | 1  | 0x25    |
| 0  | 1  | 1  | 0x26    |
| 1  | 1  | 1  | 0x27    |

Set jumpers on the GPIO expander board to select address.

## Current Limits

**Per Pin:** 25mA maximum
**Total:** 150mA maximum

**Example:** With 8 outputs, each LED can use ~18mA

## Troubleshooting

### Device Not Detected

**Symptom:** MCP23017 not found

**Possible Causes:**
1. Wrong address
2. I2C wiring issue
3. Power problem

**Solution:**
```cpp
// Scan I2C bus
#include <Wire.h>
void setup() {
  Wire.begin();
  Serial.begin(115200);
  for (byte addr = 1; addr < 127; addr++) {
    Wire.beginTransmission(addr);
    if (Wire.endTransmission() == 0) {
      Serial.print("Found device at 0x");
      Serial.println(addr, HEX);
    }
  }
}
```

### Incorrect Pin States

**Symptom:** Pins read wrong values

**Possible Causes:**
1. Not initialized - Configure direction first
2. Wrong port - Check GPA vs GPB
3. Interrupt mode - Clear interrupt flag

**Solution:**
- Always set pinMode() before using pins
- Check Port A (0-7) vs Port B (8-15)
- Clear interrupts with readGPIOAB()

### Current Limit Exceeded

**Symptom:** Unstable operation, resets

**Solution:**
- Limit per-pin current to 25mA
- Limit total current to 150mA
- Use external drivers for high-current loads
