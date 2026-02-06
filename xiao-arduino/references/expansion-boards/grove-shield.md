# Grove Shield for XIAO - Arduino

## Overview

Arduino implementation for the Grove Shield for XIAO. Provides 8 Grove connectors with battery management. All 14 XIAO GPIO pins are broken out to Grove ports.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/grove-shield.md`.

## Grove Port Mapping

| Grove Port | XIAO Pins | Type |
|------------|-----------|------|
| I2C (x2) | D4 (SDA), D5 (SCL) | I2C |
| UART | D6 (RX), D7 (TX) | Serial |
| D0 | D0 | Digital/Analog |
| D1 | D1 | Digital |
| D2 | D2 | Digital |
| D3 | D3 | Digital/PWM |
| D4 | D4 | Digital/I2C SDA |
| D5 | D5 | Digital/I2C SCL |
| D6 | D6 | Digital/UART RX |
| D7 | D7 | Digital/UART TX |
| D8 | D8 | Digital/PWM |
| D9 | D9 | Digital/PWM |
| D10 | D10 | Digital/PWM/LED |

## Battery Management

The Grove Shield includes a battery charging circuit:

- **Charging Current:** 400mA max
- **Battery Connector:** JST 2.0mm
- **Charge LED:** Indicates charging status

```cpp
// Monitor battery (if ADC connected)
// Note: Exact pin varies by shield version
void setup() {
  Serial.begin(115200);
  pinMode(A0, INPUT);  // May be battery voltage pin
}

void loop() {
  int batteryValue = analogRead(A0);
  float voltage = batteryValue * (3.3 / 1024.0);
  // Convert based on voltage divider if present
  Serial.print("Battery: ");
  Serial.print(voltage);
  Serial.println("V");
  delay(1000);
}
```

## Grove I2C Devices

Use the standard Wire library for Grove I2C sensors and displays.

```cpp
#include <Wire.h>

void setup() {
  Wire.begin();  // D4=SDA, D5=SCL on XIAO
  Serial.begin(115200);
}

void loop() {
  // Scan I2C bus
  for (byte addr = 1; addr < 127; addr++) {
    Wire.beginTransmission(addr);
    if (Wire.endTransmission() == 0) {
      Serial.print("Found I2C device at 0x");
      Serial.println(addr, HEX);
    }
  }
  delay(5000);
}
```

## Grove UART Devices

Use Serial1 for Grove UART devices (D6/D7).

```cpp
void setup() {
  Serial.begin(115200);   // USB debug
  Serial1.begin(9600);    // Grove UART (D6/D7)
}

void loop() {
  if (Serial1.available()) {
    Serial.write(Serial1.read());
  }
  if (Serial.available()) {
    Serial1.write(Serial.read());
  }
}
```

## Grove Digital/PWM Devices

Standard digital and PWM operations on any D0-D10 pin.

```cpp
// LED Blink on D0
void setup() {
  pinMode(D0, OUTPUT);
}

void loop() {
  digitalWrite(D0, HIGH);
  delay(1000);
  digitalWrite(D0, LOW);
  delay(1000);
}
```

```cpp
// PWM LED Fading on D3
void setup() {
  pinMode(D3, OUTPUT);
}

void loop() {
  for (int i = 0; i <= 255; i++) {
    analogWrite(D3, i);
    delay(10);
  }
  for (int i = 255; i >= 0; i--) {
    analogWrite(D3, i);
    delay(10);
  }
}
```

## Grove Analog Sensors

Read analog sensors on A0 (D0).

```cpp
void setup() {
  Serial.begin(115200);
}

void loop() {
  int sensorValue = analogRead(A0);
  float voltage = sensorValue * (3.3 / 1024.0);
  Serial.print("Analog: ");
  Serial.print(sensorValue);
  Serial.print(" (");
  Serial.print(voltage);
  Serial.println("V)");
  delay(100);
}
```

## Common Grove Device Examples

### Grove Temperature Sensor (DHT)

```cpp
#include <DHT.h>

#define DHTPIN D2
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
}

void loop() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  Serial.print("Temp: ");
  Serial.print(t);
  Serial.print("C, Humidity: ");
  Serial.println(h);
  delay(2000);
}
```

### Grove Relay

```cpp
void setup() {
  pinMode(D3, OUTPUT);
}

void loop() {
  digitalWrite(D3, HIGH);  // Relay ON
  delay(1000);
  digitalWrite(D3, LOW);   // Relay OFF
  delay(1000);
}
```

### Grove Servo

```cpp
#include <Servo.h>
// For ESP32: #include <ESP32Servo.h>

Servo myservo;

void setup() {
  myservo.attach(D3);
}

void loop() {
  myservo.write(0);
  delay(1000);
  myservo.write(90);
  delay(1000);
  myservo.write(180);
  delay(1000);
}
```

## Breakable Design

The Grove Shield can be broken into two sections:
- **Full size:** 60×39mm (all 8 Grove ports)
- **Reduced size:** 25×39mm (4 Grove ports only)

## Troubleshooting

### Grove Device Not Detected

**Symptom:** I2C device not found

**Possible Causes:**
1. Wrong I2C address
2. Loose connection
3. Device not powered

**Solution:** Use I2C scanner to verify device address

### Battery Not Charging

**Symptom:** Charge LED not lit

**Possible Causes:**
1. No USB power connected
2. Battery not connected
3. Faulty charging circuit

**Solution:** Verify USB and battery connections
