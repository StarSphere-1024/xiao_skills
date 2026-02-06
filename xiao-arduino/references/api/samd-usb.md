# SAMD USB API for Arduino

## Overview

Native USB support on SAMD21 XIAO for creating custom USB devices.

## USB HID (Keyboard/Mouse)

### Keyboard

```cpp
#include <Keyboard.h>

void setup() {
    Keyboard.begin();
}

void loop() {
    // Type "Hello"
    Keyboard.press(KEY_LEFT_GUI);
    Keyboard.press('r');
    delay(100);
    Keyboard.releaseAll();
    delay(500);

    Keyboard.println("notepad");
    delay(1000);

    Keyboard.println("Hello from XIAO SAMD21!");

    delay(5000);
}
```

### Keyboard with Modifiers

```cpp
#include <Keyboard.h>

void sendShortcut() {
    // Ctrl+C (Copy)
    Keyboard.press(KEY_LEFT_CTRL);
    Keyboard.press('c');
    delay(100);
    Keyboard.releaseAll();
    delay(1000);

    // Ctrl+V (Paste)
    Keyboard.press(KEY_LEFT_CTRL);
    Keyboard.press('v');
    delay(100);
    Keyboard.releaseAll();
}

void setup() {
    Keyboard.begin();
}

void loop() {
    sendShortcut();
    delay(3000);
}
```

### Mouse Control

```cpp
#include <Mouse.h>

void setup() {
    Mouse.begin();
}

void loop() {
    // Move mouse in circle
    for (int i = 0; i < 360; i += 10) {
        int x = cos(i * PI / 180) * 10;
        int y = sin(i * PI / 180) * 10;
        Mouse.move(x, y);
        delay(50);
    }

    // Click
    Mouse.click(MOUSE_LEFT);
    delay(1000);

    // Scroll
    Mouse.move(0, 0, 2);
    delay(1000);
}
```

### Combined Keyboard + Mouse

```cpp
#include <Keyboard.h>
#include <Mouse.h>

void setup() {
    Keyboard.begin();
    Mouse.begin();
}

void loop() {
    // Move mouse and type
    Mouse.move(10, 0);
    Keyboard.print("Moving right");
    delay(500);

    Mouse.move(0, 10);
    Keyboard.print("Moving down");
    delay(500);

    Mouse.move(-10, 0);
    Keyboard.print("Moving left");
    delay(500);

    Mouse.move(0, -10);
    Keyboard.print("Moving up");
    delay(500);
}
```

## USB MIDI

### Basic MIDI

```cpp
#include <MIDI.h>

MIDI_CREATE_DEFAULT_INSTANCE();

void setup() {
    MIDI.begin();
}

void loop() {
    // Send note on
    MIDI.sendNoteOn(60, 127, 1);  // Note 60, velocity 127, channel 1
    delay(500);

    // Send note off
    MIDI.sendNoteOff(60, 0, 1);
    delay(500);
}
```

### MIDI Controller

```cpp
#include <MIDI.h>

MIDI_CREATE_DEFAULT_INSTANCE();

void setup() {
    pinMode(A0, INPUT);
    MIDI.begin();
}

void loop() {
    // Read potentiometer
    int value = analogRead(A0);
    int ccValue = map(value, 0, 1023, 0, 127);

    // Send control change
    MIDI.sendControlChange(1, ccValue, 1);

    delay(10);
}
```

## USB Serial

### Multiple Serial Ports

```cpp
#define SerialSerial Serial1  // Native USB Serial
#define HardwareSerial Serial2  // Hardware UART

void setup() {
    SerialSerial.begin(115200);  // USB Serial
    HardwareSerial.begin(9600);  // Hardware Serial
}

void loop() {
    // USB to Hardware bridge
    if (SerialSerial.available()) {
        char c = SerialSerial.read();
        HardwareSerial.write(c);
    }

    if (HardwareSerial.available()) {
        char c = HardwareSerial.read();
        SerialSerial.write(c);
    }
}
```

## USB Mass Storage

### SD Card as USB Drive

```cpp
#include <SPI.h>
#include <SD.h>
#include <Mouse.h>
#include <Keyboard.h>
#include "MassStorage.h"

#define SD_CS 4

MassStorage MSD;

void setup() {
    SPI.begin();
    SD.begin(SD_CS);

    // Initialize USB mass storage
    MSD.begin();
    MSD.setDrive(&SD);
}

void loop() {
    // Handle USB events
    MSD.task();
}
```

## USB HID Custom

### Custom HID Device

```cpp
#include <HID.h>

// Report descriptor for custom HID
const uint8_t reportDescriptor[] = {
    0x06, 0x00, 0xFF,  // Usage Page (Vendor Defined)
    0x09, 0x01,        // Usage (Vendor Defined)
    0xA1, 0x01,        // Collection (Application)
    0x09, 0x02,        //   Usage (Vendor Defined)
    0x15, 0x00,        //   Logical Minimum (0)
    0x26, 0xFF, 0x00,  //   Logical Maximum (255)
    0x75, 0x08,        //   Report Size (8 bits)
    0x95, 0x40,        //   Report Count (64 bytes)
    0x81, 0x02,        //   Input (Data, Var, Abs)
    0xC0               // End Collection
};

class CustomHID : public HID_ {
public:
    CustomHID() : HID_(reportDescriptor, sizeof(reportDescriptor)) {
        // Initialize
    }

    void sendReport(uint8_t* data, size_t len) {
        SendReport(0, data, len);
    }
};

CustomHID HID;

void setup() {
    HID.begin();
}

void loop() {
    uint8_t data[64] = "Hello Custom HID!";
    HID.sendReport(data, 64);
    delay(1000);
}
```

## USB Raw HID

### Raw HID Communication

```cpp
#include <RawHID.h>

// Buffer for raw HID
uint8_t buf[64];

void setup() {
    RawHID.begin(buf, sizeof(buf));
}

void loop() {
    // Check for received data
    if (RawHID.available()) {
        // Read data
        uint8_t len = RawHID.recv(buf, sizeof(buf));

        // Process data
        // ...

        // Send response
        RawHID.send(buf, len);
    }

    delay(10);
}
```

## USB Web Serial

### Web Serial API

```cpp
#include <WebSerial.h>

void setup() {
    WebSerial.begin();
}

void loop() {
    WebSerial.println("Hello from Web Serial!");
    delay(1000);
}
```

## USB Configuration

### USB Descriptor Configuration

```cpp
#include <USBDescriptor.h>

void setup() {
    // Configure USB descriptors
    USBDevice.setManufacturer("SeeedStudio");
    USBDevice.setProduct("XIAO SAMD21");
    USBDevice.setSerialNumber("12345678");
    USBDevice.setVID(0x2886);  // SeeedStudio VID
    USBDevice.setPID(0x802D);  // XIAO SAMD21 PID
}

void loop() {}
```

## USB Power

### USB Power Detection

```cpp
void setup() {
    Serial.begin(115200);

    // Check if USB is connected
    if (USBDevice.connected()) {
        Serial.println("USB connected");
    } else {
        Serial.println("USB not connected");
    }

    // Check USB voltage
    int usbVoltage = USBDevice.getUSBVoltage();
    Serial.printf("USB Voltage: %d mV\n", usbVoltage);
}

void loop() {}
```

### USB Power Control

```cpp
void setup() {
    Serial.begin(115200);

    // Disable USB when not needed (for battery)
    // USBDevice.detach();
}

void loop() {
    // Enable USB when needed
    // USBDevice.attach();
    delay(1000);
}
```

## USB with Low Power

### USB + Sleep

```cpp
#include <RTCZero.h>

RTCZero rtc;

void setup() {
    Serial.begin(115200);
    rtc.begin();

    Serial.println("Going to sleep in 5 seconds...");
    delay(5000);

    // USB is disconnected during sleep
    Serial.end();

    // Enter sleep
    __WFI();
}

void loop() {}
```

## USB Events

### USB Connection Callback

```cpp
void USBDevice_Connected() {
    // Called when USB connects
}

void USBDevice_Disconnected() {
    // Called when USB disconnects
}

void setup() {
    Serial.begin(115200);
    // Callbacks are handled by framework
}

void loop() {}
```

## USB Vendor Requests

### Custom Vendor Requests

```cpp
#include <USBVendor.h>

void handleVendorRequest(uint8_t request) {
    switch (request) {
        case 0x01:
            // Custom request 1
            break;
        case 0x02:
            // Custom request 2
            break;
    }
}

void setup() {
    USBVendor.begin();
    USBVendor.setCallback(handleVendorRequest);
}

void loop() {}
```

## Troubleshooting

### USB not recognized

1. Check USB driver installation
2. Verify USB cable (data cable required)
3. Check VID/PID settings
4. Try different USB port

### HID not working

1. Ensure proper report descriptor
2. Check HID library version
3. Verify OS support for HID
4. Test with HID分析仪

### Serial monitor issues

1. Close other serial monitors
2. Check baud rate matches
3. Try different terminal program
4. Reset board after opening monitor

### USB enumeration fails

1. Check descriptor length
2. Verify VID/PID are valid
3. Check endpoint configuration
4. Test with USB protocol analyzer

## Best Practices

1. **Descriptors**: Keep descriptors minimal
2. **HID**: Use standard HID types when possible
3. **Serial**: Flush before sleep
4. **Power**: Disconnect USB for battery operation
5. **Compatibility**: Test on multiple OS platforms

## USB HID Key Codes

| Key | Code | Key | Code |
|-----|------|-----|------|
| A | 0x04 | 1 | 0x1E |
| Z | 0x1D | 0 | 0x27 |
| Enter | 0x28 | Space | 0x2C |
| Ctrl | 0xE0 | Shift | 0xE1 |
| Alt | 0xE2 | GUI | 0xE3 |

## USB MIDI Notes

| Note | MIDI | Note | MIDI |
|------|------|------|------|
| C3 | 48 | C4 | 60 (Middle C) |
| C5 | 72 | C6 | 84 |
