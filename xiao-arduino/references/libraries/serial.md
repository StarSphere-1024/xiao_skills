# Serial/HardwareSerial Library for XIAO

## Overview

UART (Universal Asynchronous Receiver-Transmitter) serial communication for XIAO boards.

## XIAO Unified Standard

All XIAO boards follow a unified peripheral pin mapping for expansion board compatibility:

| Interface | Pin | Description |
|-----------|-----|-------------|
| **UART TX** | D6 | Default UART transmit (default) |
| **UART RX** | D7 | Default UART receive (default) |

**Important**: D6 and D7 are the standard UART pins across all XIAO boards. Use these pins for compatibility with XIAO expansion boards.

## Multiple UART Interfaces

Some XIAO boards support multiple UART interfaces:

| Board | UART Interfaces | Default UART | Secondary UART Pins |
|-------|-----------------|---------------|---------------------|
| ESP32C3/C5/C6 | 2 | D6/D7 (UART0) | Any GPIO with `Serial1.begin(rx, tx)` |
| ESP32S3 | 2 | D6/D7 (UART0) | Any GPIO with `Serial1.begin(rx, tx)` |
| nRF52840 | 1 | D6/D7 | Not available |
| RP2040 | 2 | D6/D7 (UART0) | Alternate pins available |
| RP2350 | 2 | D6/D7 (UART0) | Alternate pins available |
| SAMD21 | 1 | D6/D7 | Not available |
| MG24 | 2 | D6/D7 (Primary) | D12/D11 (Secondary) |
| RA4M1 | 4 | D6/D7 (UART2) | D15/D16, D12/D11 (UART0, UART9) |

## Basic Usage

### USB Serial (All XIAO Boards)

```cpp
void setup() {
    Serial.begin(115200);
}

void loop() {
    Serial.println("Hello XIAO!");
    delay(1000);
}
```

### Hardware Serial (Default UART Pins)

```cpp
void setup() {
    Serial.begin(115200);   // USB Serial for debugging
    Serial1.begin(115200);  // Hardware UART on D6/D7
}

void loop() {
    Serial1.println("Hardware UART");

    // Echo from Serial1 to USB
    if (Serial1.available()) {
        Serial.write(Serial1.read());
    }

    delay(1000);
}
```

## Custom UART Pins (ESP32 Series)

ESP32C3, ESP32C5, ESP32C6, and ESP32S3 can use any GPIO pins for additional UART.

```cpp
#include <HardwareSerial.h>

HardwareSerial MySerial(1);  // UART1

#define RX_PIN D3
#define TX_PIN D2

void setup() {
    Serial.begin(115200);  // USB Serial

    // UART1 with custom pins
    MySerial.begin(115200, SERIAL_8N1, RX_PIN, TX_PIN);
}

void loop() {
    MySerial.println("Custom UART pins");
    delay(1000);
}
```

## Multiple UARTs (ESP32)

ESP32 series have 2 or more UART interfaces.

```cpp
#include <HardwareSerial.h>

HardwareSerial uart0(0);  // UART0 - D6/D7 (default)
HardwareSerial uart1(1);  // UART1 - custom pins

void setup() {
    Serial.begin(115200);  // USB

    uart0.begin(9600, SERIAL_8N1);     // Uses D6/D7
    uart1.begin(115200, SERIAL_8N1, D9, D10);  // Custom pins
}

void loop() {
    uart0.println("UART0 on D6/D7");
    uart1.println("UART1 on D9/D10");
    delay(1000);
}
```

## Multiple UART (RP2040/RP2350)

RP2040 and RP2350 support multiple UART with flexible pins.

```cpp
#include <HardwareSerial.h>

HardwareSerial Serial2(1);  // Secondary UART

void setup() {
    Serial.begin(115200);   // USB
    Serial1.begin(115200);  // D6/D7

    // Secondary UART with custom pins
    Serial2.begin(9600, SERIAL_8N1, D1, D0);
}

void loop() {
    Serial1.println("UART1");
    Serial2.println("UART2");
    delay(1000);
}
```

## Common UART Applications

### GPS Module

```cpp
#include <HardwareSerial.h>

HardwareSerial GPS(1);  // UART1

void setup() {
    Serial.begin(115200);   // USB for debugging
    GPS.begin(9600, SERIAL_8N1);  // GPS on D6/D7
}

void loop() {
    while (GPS.available()) {
        char c = GPS.read();
        Serial.write(c);  // Echo NMEA data to USB
    }
}
```

### Serial Passthrough (Bridge)

```cpp
void setup() {
    Serial.begin(115200);   // USB
    Serial1.begin(9600);    // Hardware UART on D6/D7
}

void loop() {
    // USB to UART1
    if (Serial.available()) {
        Serial1.write(Serial.read());
    }

    // UART1 to USB
    if (Serial1.available()) {
        Serial.write(Serial1.read());
    }
}
```

### Bluetooth Classic (HC-05/HC-06)

```cpp
#include <HardwareSerial.h>

HardwareSerial BT(1);  // UART1 for Bluetooth

void setup() {
    Serial.begin(115200);
    BT.begin(9600);  // HC-05 default baud rate

    Serial.println("Bluetooth ready");
}

void loop() {
    // USB to Bluetooth
    if (Serial.available()) {
        BT.write(Serial.read());
    }

    // Bluetooth to USB
    if (BT.available()) {
        Serial.write(BT.read());
    }
}
```

### LoRa Module

```cpp
#include <HardwareSerial.h>

HardwareSerial LoRa(1);

void setup() {
    Serial.begin(115200);
    LoRa.begin(9600, SERIAL_8N1);  // D6/D7

    // Send AT command
    LoRa.println("AT");
}

void loop() {
    if (LoRa.available()) {
        Serial.write(LoRa.read());
    }
}
```

## Serial Event (Buffer)

Process serial data asynchronously.

```cpp
String inputString = "";
bool stringComplete = false;

void setup() {
    Serial.begin(115200);
    inputString.reserve(200);
}

void serialEvent() {
    while (Serial.available()) {
        char inChar = (char)Serial.read();
        inputString += inChar;

        if (inChar == '\n') {
            stringComplete = true;
        }
    }
}

void loop() {
    if (stringComplete) {
        Serial.println("Received: " + inputString);
        inputString = "";
        stringComplete = false;
    }
}
```

## Binary Data

```cpp
void setup() {
    Serial.begin(115200);
}

void loop() {
    // Send binary data
    byte buffer[4] = {0x01, 0x02, 0x03, 0x04};
    Serial.write(buffer, 4);

    delay(1000);
}
```

## JSON over Serial

```cpp
#include <ArduinoJson.h>

void setup() {
    Serial.begin(115200);
}

void loop() {
    StaticJsonDocument<200> doc;
    doc["sensor"] = "temperature";
    doc["value"] = 25.3;
    doc["unit"] = "C";

    serializeJson(doc, Serial);
    Serial.println();
    delay(1000);
}
```

## Binary Protocol Parsing

```cpp
// Simple binary protocol: [START][LEN][DATA...][CHECKSUM]
#define START_BYTE 0xAA
#define MAX_LEN 32

uint8_t buffer[MAX_LEN];
uint8_t idx = 0;
bool receiving = false;

void setup() {
    Serial.begin(115200);
}

void loop() {
    if (Serial.available()) {
        uint8_t byte = Serial.read();

        if (!receiving) {
            if (byte == START_BYTE) {
                receiving = true;
                idx = 0;
            }
        } else {
            buffer[idx++] = byte;

            // Check for complete packet
            if (idx >= 2 && idx >= buffer[1] + 2) {
                // Process complete packet
                processPacket(buffer, idx);
                receiving = false;
                idx = 0;
            }
        }
    }
}

void processPacket(uint8_t *pkt, uint8_t len) {
    Serial.print("Received packet: ");
    for (int i = 0; i < len; i++) {
        Serial.printf("%02X ", pkt[i]);
    }
    Serial.println();
}
```

## Serial Configuration

### Baud Rate

Common baud rates: 9600, 19200, 38400, 57600, 115200

```cpp
Serial.begin(9600);      // Slow, legacy
Serial.begin(115200);   // Most common
Serial.begin(921600);   // High speed (ESP32)
```

### Data Format

```cpp
// Default: 8N1
Serial.begin(115200);

// Custom format
Serial1.begin(115200, SERIAL_8N1);  // 8 data, no parity, 1 stop
Serial1.begin(115200, SERIAL_7E1);  // 7 data, even parity, 1 stop
Serial1.begin(115200, SERIAL_8O2);  // 8 data, odd parity, 2 stop
```

## Troubleshooting

### No output in Serial Monitor

1. Check baud rate matches (115200 most common)
2. Check "USB CDC On Boot" is Enabled (ESP32)
3. Try different USB cable (data + power, not power-only)
4. For nRF52840: Must open Serial Monitor after upload to run

### Garbage characters

1. Check baud rate matches on both ends
2. Check data bits/stop bits/parity match
3. Ground both devices together
4. Try lower baud rate

### ESP32 Upload Issues

1. Hold BOOT button while uploading
2. Check "USB CDC On Boot" setting
3. Try lowering upload speed

### nRF52840 Serial Issues

For nRF52840, code won't run until Serial Monitor is opened:
- Upload code
- Click Serial Monitor icon
- Code starts executing

### Data Loss at High Speed

1. Try lower baud rate
2. Add delays between writes
3. Use flow control if available
4. Check buffer size

## ESP32-Specific Notes

### USB CDC On Boot

In Arduino IDE: Tools > USB CDC On Boot
- **Enabled**: `Serial` goes to USB (recommended)
- **Disabled**: `Serial` goes to UART0 (D6/D7), need separate USB-serial

### UART Pin Mapping

For ESP32C3/C5/C6/S3 with default pins:
- UART0: D6/D7 (when USB CDC disabled)
- UART1: Custom pins via `Serial1.begin(rx, tx)`

## Software Serial

For boards without multiple hardware UART (nRF52840, SAMD21):

```cpp
#include <SoftwareSerial.h>

// RX, TX order
SoftwareSerial swSerial(D7, D6);  // D7=RX, D6=TX

void setup() {
    Serial.begin(115200);
    swSerial.begin(9600);
}

void loop() {
    if (swSerial.available()) {
        char c = swSerial.read();
        Serial.print("SW: ");
        Serial.println(c);
    }
}
```

**ESP32 note**: The built-in Arduino `SoftwareSerial` is generally not reliable on ESP32. Prefer `HardwareSerial` (e.g. `Serial1` with custom pins). If you must use software serial on ESP32-based XIAO boards, use **EspSoftwareSerial** (see `softwareserial.md`).
