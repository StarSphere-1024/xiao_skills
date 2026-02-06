# Wire Library (I2C) for XIAO

## Overview

The Wire library allows communication with I2C/TWI devices.

## XIAO Unified I2C Pin Standard

**All XIAO boards use the same default I2C pins:**

| Pin | Signal | Purpose |
|-----|--------|---------|
| **D4** | SDA | Default I2C data line |
| **D5** | SCL | Default I2C clock line |

This unified mapping ensures compatibility across all XIAO expansion boards (OLED displays, sensors, etc.).

Simply call `Wire.begin()` and it will use D4/D5 automatically.

## Multiple I2C Buses

Many XIAO boards support multiple I2C interfaces. Here's what each board supports:

| Board | I2C Interfaces | Default Pins | Secondary I2C Pins |
|-------|----------------|--------------|-------------------|
| **ESP32C3** | 2 | D4/D5 (Wire) | Any GPIO with `Wire1.begin(sda, scl)` |
| **ESP32C5** | 2 | D4/D5 (Wire) | Any GPIO with `Wire1.begin(sda, scl)` |
| **ESP32C6** | 2 | D4/D5 (Wire) | Any GPIO with `Wire1.begin(sda, scl)` |
| **ESP32S3** | 2 | D4/D5 (Wire) | Any GPIO with `Wire1.begin(sda, scl)` |
| **nRF52840** | 1 (fixed) | D4/D5 | Not available |
| **RP2040** | 2 | D4/D5 (Wire0) | D13/D14 (Wire1) |
| **RP2350** | 2 | D4/D5 (Wire1) | D13/D14 (Wire0) |
| **SAMD21** | 2 | D4/D5 (Wire) | Not available |
| **MG24** | 2 | D4/D5 | D13/D14 |
| **RA4M1** | 3 | D4/D5 (Wire) | D6/D7, D15/D16 |

**Note**: nRF54L15 does NOT support Arduino. Use MicroPython for nRF54L15 development.

## Custom I2C Pins

**ESP32 and RP2040** can use ANY GPIO pins for I2C:

```cpp
#include <Wire.h>

// ESP32: Use any pins for I2C
#define CUSTOM_SDA D2  // Any GPIO
#define CUSTOM_SCL D3  // Any GPIO

void setup() {
    Wire.begin(CUSTOM_SDA, CUSTOM_SCL);
}
```

This is useful when:
- Default pins conflict with other hardware
- You need multiple I2C buses
- Pin layout requires different routing

## Basic Usage

### Using Default I2C (D4/D5)

```cpp
#include <Wire.h>

void setup() {
    Wire.begin();  // Automatically uses D4 (SDA) and D5 (SCL)
    Serial.begin(115200);
}

void loop() {
    // Write to device at address 0x48
    Wire.beginTransmission(0x48);
    Wire.write(0x00);
    Wire.endTransmission();
    delay(100);
}
```

### Using Custom I2C Pins (ESP32/RP2040)

```cpp
#include <Wire.h>

#define SDA_PIN D2
#define SCL_PIN D3

void setup() {
    Wire.begin(SDA_PIN, SCL_PIN);
}

void loop() {
    Wire.beginTransmission(0x48);
    Wire.write(0x00);
    Wire.endTransmission();
    delay(100);
}
```

### Using Multiple I2C Buses (ESP32)

```cpp
#include <Wire.h>

TwoWire I2C_0 = TwoWire(0);  // First I2C bus
TwoWire I2C_1 = TwoWire(1);  // Second I2C bus

void setup() {
    I2C_0.begin(D4, D5);   // Default XIAO pins
    I2C_1.begin(D2, D3);   // Custom pins for second bus
}

void loop() {
    // Use I2C_0 for first device
    I2C_0.beginTransmission(0x48);
    I2C_0.write(0x00);
    I2C_0.endTransmission();

    // Use I2C_1 for second device
    I2C_1.beginTransmission(0x76);
    I2C_1.write(0x00);
    I2C_1.endTransmission();

    delay(100);
}
```

### Using Multiple I2C Buses (RP2040)

```cpp
#include <Wire.h>

void setup() {
    // Wire0: Default XIAO pins
    Wire.setSDA(D4);
    Wire.setSCL(D5);
    Wire.begin();

    // Wire1: Secondary pins
    Wire1.setSDA(D14);
    Wire1.setSCL(D13);
    Wire1.begin();
}

void loop() {
    // Use Wire for first device
    Wire.beginTransmission(0x48);
    Wire.write(0x00);
    Wire.endTransmission();

    // Use Wire1 for second device
    Wire1.beginTransmission(0x76);
    Wire1.beginTransmission(0x00);
    Wire1.endTransmission();

    delay(100);
}
```

### Using Multiple I2C Buses (RP2350)

```cpp
#include <Wire.h>

void setup() {
    // Wire0: D13/D14
    Wire.setSDA(D14);
    Wire.setSCL(D13);
    Wire.begin();

    // Wire1: Default XIAO pins D4/D5
    Wire1.setSDA(D4);
    Wire1.setSCL(D5);
    Wire1.begin();
}
```

## I2C Scanner

```cpp
#include <Wire.h>

void setup() {
    Wire.begin();
    Serial.begin(115200);
    Serial.println("I2C Scanner");
}

void loop() {
    byte error, address;
    int nDevices = 0;

    Serial.println("Scanning...");

    for (address = 1; address < 127; address++) {
        Wire.beginTransmission(address);
        error = Wire.endTransmission();

        if (error == 0) {
            Serial.print("I2C device found at 0x");
            if (address < 16) Serial.print("0");
            Serial.println(address, HEX);
            nDevices++;
        }
    }

    if (nDevices == 0) {
        Serial.println("No I2C devices found");
    } else {
        Serial.println("done");
    }

    delay(5000);
}
```

## Reading from I2C

```cpp
// Request 2 bytes from device 0x48
Wire.requestFrom(0x48, 2);

while (Wire.available()) {
    byte data = Wire.read();
    Serial.println(data, HEX);
}
```

## Writing to I2C

```cpp
Wire.beginTransmission(0x48);  // Address
Wire.write(0x00);               // Register
Wire.write(0x01);               // Data
Wire.endTransmission();         // Send
```

## OLED Display (SSD1306)

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

void setup() {
    Wire.begin();  // Uses D4 (SDA) and D5 (SCL)
    display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
    display.clearDisplay();
    display.setTextSize(1);
    display.setTextColor(SSD1306_WHITE);
    display.setCursor(0, 0);
    display.println("Hello XIAO!");
    display.display();
}

void loop() {}
```

## BME280 Sensor

```cpp
#include <Wire.h>
#include <Adafruit_BME280.h>

Adafruit_BME280 bme;

void setup() {
    Wire.begin();  // Uses D4/D5
    if (!bme.begin(0x76)) {  // or 0x77
        Serial.println("BME280 not found!");
    }
}

void loop() {
    Serial.print("Temp: ");
    Serial.print(bme.readTemperature());
    Serial.println(" C");
    delay(2000);
}
```

## I2C Clock Speed

```cpp
Wire.setClock(100000);   // 100kHz (standard mode)
Wire.setClock(400000);   // 400kHz (fast mode)
Wire.setClock(1000000);  // 1MHz (ESP32 only)
```

## Common I2C Addresses

| Device | Address |
|--------|---------|
| SSD1306 OLED | 0x3C or 0x3D |
| SH1106 OLED | 0x3C |
| BME280 | 0x76 or 0x77 |
| BMP280 | 0x76 or 0x77 |
| MPU6050 | 0x68 or 0x69 |
| LSM6DS3 | 0x6A or 0x6B |
| PCF8574 (GPIO expander) | 0x20-0x27 |
| PCF8574A | 0x38-0x3F |
| BH1750 (Light) | 0x23 |
| AT24C256 (EEPROM) | 0x50-0x57 |

## Troubleshooting

### No devices found

1. **Check wiring**: SDA to SDA, SCL to SCL, VCC, GND
2. **Pull-up resistors**: Most modules have them (4.7kΩ)
3. **Wrong address**: Check device datasheet
4. **Wrong pins**: Verify you're using D4/D5 (or your custom pins)

### Random data/garbage

1. **Clock speed too high**: Try 100kHz
2. **Long wires**: Keep short (<30cm)
3. **Noise**: Add decoupling capacitors

### Device not responding after reset

1. **Add delay**: Some devices need time to initialize
2. **Check power**: Verify 3.3V supply
3. **Reset sequence**: Send reset command if available
