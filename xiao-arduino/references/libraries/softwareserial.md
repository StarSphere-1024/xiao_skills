# SoftwareSerial Library for XIAO

## Overview

Software serial provides a UART-like interface on GPIO pins.

In most cases, XIAO boards already provide at least one hardware UART (typically `Serial1` on the unified XIAO pins `D6/D7`). If you can use a hardware UART, prefer it.

- For hardware UART usage and common external module examples (GPS / Bluetooth / LoRa / passthrough), see [Serial/HardwareSerial](serial.md).

## Choosing an approach

- **Prefer `HardwareSerial` first**: better reliability, higher baud rates.
- **Use `SoftwareSerial` when you must**: for boards with limited UART options, or when you need an extra UART-like interface.

:::note
On ESP32, the built-in Arduino `SoftwareSerial` is generally not reliable. If you must do software serial on ESP32-based XIAO boards, use **EspSoftwareSerial**.
:::

## ESP32 Series (C3/C5/C6/S3)

To use software serial, install the [EspSoftwareSerial](https://github.com/plerup/espsoftwareserial) library.

:::tip
Currently we recommend version 7.0.0 of the EspSoftwareSerial library. Other versions may have varying degrees of problems that prevent the soft serial port from working properly.
:::

:::caution
If you have other soft serial libraries installed, they may conflict. Please check your environment if compilation fails.
:::

:::tip
If you only need a second UART on ESP32, you can often avoid software serial entirely by using `HardwareSerial` with custom pins (see [Serial/HardwareSerial](serial.md)).
:::

### Example (ESP32C3 / ESP32C6)

```cpp
#include <SoftwareSerial.h>

SoftwareSerial mySerial(D7, D6); // RX, TX

void setup() {
  Serial.begin(9600);
  mySerial.begin(9600);
}

void loop() {
  if (mySerial.available()) {
    char data = mySerial.read();
    Serial.print("Received via software serial: ");
    Serial.println(data);
  }

  if (Serial.available()) {
    char data = Serial.read();
    mySerial.print("Received via hardware serial: ");
    mySerial.println(data);
  }
}
```

This example sets up software serial on pins `D7 (RX)` and `D6 (TX)` at 9600 baud. It monitors both the hardware serial (USB) and software serial ports, echoing received data between them.

### Example (ESP32S3)

```cpp
#include <SoftwareSerial.h>

SoftwareSerial mySerial(2, 3); // RX, TX

void setup() {
  Serial.begin(9600);
  while (!Serial);

  mySerial.begin(9600);
}

void loop() {
  if (mySerial.available()) {
    char data = mySerial.read();
    Serial.print("Received data: ");
    Serial.println(data);
  }

  mySerial.print("Hello World!");
  delay(1000);
}
```

:::note
Select RX/TX pins that are not used for other purposes. In the reference example, pins 9 and 10 are used for RX and TX.
:::

## Non-ESP32 XIAO boards (Arduino `SoftwareSerial`)

Many non-ESP32 XIAO boards can use Arduino's `SoftwareSerial` directly (no extra library install is required). This is commonly used when the board has limited UART options.

### Example (Simple echo to USB)

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

:::note
If your use case is a serial bridge (USB \u2194 UART), you may find it simpler to use hardware UART passthrough instead. See [Serial/HardwareSerial](serial.md).
:::

## External module example (GNSS)

If you're using a GNSS module that outputs NMEA sentences (e.g. L76K / L76-L series), a common pattern is to read from `SoftwareSerial` and feed bytes into a parser (like TinyGPSPlus).

### Example (TinyGPSPlus + SoftwareSerial)

```cpp
#include <TinyGPSPlus.h>
#include <SoftwareSerial.h>

static const int RXPin = D7, TXPin = D6;
static const uint32_t GPSBaud = 9600;

TinyGPSPlus gps;
SoftwareSerial ss(RXPin, TXPin);

void setup() {
  Serial.begin(115200);
  ss.begin(GPSBaud);
}

void loop() {
  while (ss.available() > 0) {
    gps.encode(ss.read());
  }
}
```

:::tip
For complete GNSS wiring + full examples, see the XIAO expansion board GNSS references in this repo (e.g. `Ref/SeeedStudio_XIAO/SeeedStudio_XIAO_Expansion_board/gnss-for-xiao.md` and `Ref/SeeedStudio_XIAO/SeeedStudio_XIAO_Expansion_board/GPS_Module_For_XIAO/L76K_GNSS_Module.md`).
:::

## References

- [EspSoftwareSerial](https://github.com/plerup/espsoftwareserial)
