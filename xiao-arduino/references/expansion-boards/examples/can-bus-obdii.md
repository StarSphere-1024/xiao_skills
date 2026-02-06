# CAN Bus OBD-II Examples - Arduino

## Overview

Complete Arduino examples for using the XIAO CAN Bus Expansion Board with OBD-II (On-Board Diagnostics) to read vehicle data. Examples demonstrate RPM reading, speed, temperature, and diagnostic trouble codes.

**Hardware Reference:** `/xiao/references/expansion_boards/can-bus.md`
**Arduino Implementation:** `/xiao-arduino/references/expansion-boards/can-bus.md`

## Required Libraries

Install **mcp_can** library via Arduino Library Manager.

## Example 1: Basic RPM Reader

```cpp
#include <mcp_can.h>
#include <SPI.h>

MCP_CAN CAN0(D7);  // CS pin

// OBD-II PID for RPM
#define PID_RPM 0x0C

unsigned long canId;
byte len;
byte buf[8];

void setup() {
  Serial.begin(115200);

  // Initialize CAN at 500kbps (standard OBD-II speed)
  if (CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_8MHZ) == CAN_OK) {
    Serial.println("CAN Bus Initialized - OBD-II Mode");
  } else {
    Serial.println("CAN Init Failed - Check Wiring");
    while (1);
  }

  CAN0.setMode(MCP_NORMAL);
  pinMode(D2, INPUT);  // Interrupt pin (optional)

  Serial.println("Waiting for vehicle connection...");
}

void sendOBDRequest(byte pid) {
  byte len = 8;
  byte buf[8] = {0x02, 0x01, pid, 0x00, 0x00, 0x00, 0x00, 0x00};
  CAN0.sendMsgBuf(0x7DF, 0, len, buf);
}

void loop() {
  // Request RPM
  sendOBDRequest(PID_RPM);
  delay(50);  // Give ECU time to respond

  if (CAN0.checkReceive() == CAN_MSGAVAIL) {
    CAN0.readMsgBuf(&canId, &len, buf);

    // Check for response from ECU (typically 0x7E8)
    if (canId == 0x7E8 && len >= 4 && buf[2] == PID_RPM) {
      // Calculate RPM: ((A * 256) + B) / 4
      int rpm = ((buf[3] * 256) + buf[4]) / 4;

      Serial.print("RPM: ");
      Serial.println(rpm);
    }
  }

  delay(500);  // Update twice per second
}
```

## Example 2: Multi-Sensor Dashboard

```cpp
#include <mcp_can.h>
#include <SPI.h>

MCP_CAN CAN0(D7);

// OBD-II PIDs
#define PID_RPM        0x0C
#define PID_SPEED      0x0D
#define PID_COOLANT    0x05
#define PID_THROTTLE   0x11
#define PID_FUEL_STATUS 0x03

struct VehicleData {
  int rpm;
  int speed;      // km/h
  int coolant;    // °C
  int throttle;   // %
  int fuelStatus;
};

VehicleData vehicle;

void setup() {
  Serial.begin(115200);

  if (CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_8MHZ) == CAN_OK) {
    Serial.println("OBD-II Dashboard Started");
  } else {
    Serial.println("CAN Init Failed");
    while (1);
  }

  CAN0.setMode(MCP_NORMAL);
}

void sendOBDRequest(byte pid) {
  byte buf[8] = {0x02, 0x01, pid};
  CAN0.sendMsgBuf(0x7DF, 0, 8, buf);
}

void readResponse(byte expectedPid) {
  unsigned long canId;
  byte len;
  byte buf[8];

  delay(50);

  if (CAN0.checkReceive() == CAN_MSGAVAIL) {
    CAN0.readMsgBuf(&canId, &len, buf);

    if (canId == 0x7E8 && buf[2] == expectedPid) {
      switch (expectedPid) {
        case PID_RPM:
          vehicle.rpm = ((buf[3] * 256) + buf[4]) / 4;
          break;
        case PID_SPEED:
          vehicle.speed = buf[3];
          break;
        case PID_COOLANT:
          vehicle.coolant = buf[3] - 40;
          break;
        case PID_THROTTLE:
          vehicle.throttle = (buf[3] * 100) / 255;
          break;
        case PID_FUEL_STATUS:
          vehicle.fuelStatus = buf[3];
          break;
      }
    }
  }
}

void updateDashboard() {
  Serial.println("=== Vehicle Dashboard ===");

  Serial.print("Engine:    ");
  Serial.print(vehicle.rpm);
  Serial.println(" RPM");

  Serial.print("Speed:     ");
  Serial.print(vehicle.speed);
  Serial.println(" km/h");

  Serial.print("Coolant:   ");
  Serial.print(vehicle.coolant);
  Serial.println(" C");

  Serial.print("Throttle:  ");
  Serial.print(vehicle.throttle);
  Serial.println(" %");

  Serial.print("Fuel:      ");
  switch (vehicle.fuelStatus) {
    case 0x01: Serial.println("Open loop"); break;
    case 0x02: Serial.println("Closed loop"); break;
    case 0x04: Serial.println("Open loop - fault"); break;
    case 0x08: Serial.println("Closed loop - fault"); break;
    default: Serial.println("Unknown"); break;
  }

  Serial.println();
}

void loop() {
  // Update all sensors
  sendOBDRequest(PID_RPM);
  readResponse(PID_RPM);

  sendOBDRequest(PID_SPEED);
  readResponse(PID_SPEED);

  sendOBDRequest(PID_COOLANT);
  readResponse(PID_COOLANT);

  sendOBDRequest(PID_THROTTLE);
  readResponse(PID_THROTTLE);

  sendOBDRequest(PID_FUEL_STATUS);
  readResponse(PID_FUEL_STATUS);

  updateDashboard();

  delay(1000);  // Update every second
}
```

## Example 3: DTC (Diagnostic Trouble Code) Reader

```cpp
#include <mcp_can.h>
#include <SPI.h>

MCP_CAN CAN0(D7);

// OBD-II PIDs
#define PID_DTC_COUNT  0x01  // Mode 3, PID 1
#define PID_DTC_FETCH  0x02  // Mode 3, PID 2

void setup() {
  Serial.begin(115200);

  if (CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_8MHZ) == CAN_OK) {
    Serial.println("DTC Reader Ready");
  } else {
    Serial.println("CAN Init Failed");
    while (1);
  }

  CAN0.setMode(MCP_NORMAL);
}

void requestDTC() {
  // Mode 3 = Show diagnostic trouble codes
  byte buf[8] = {0x02, 0x03, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};
  CAN0.sendMsgBuf(0x7DF, 0, 8, buf);
}

void parseDTC(byte code1, byte code2) {
  // Convert two bytes to DTC
  char letter = 'P';
  byte first = code1;

  // Determine the letter
  if ((first & 0xC0) == 0x00) letter = 'P';
  else if ((first & 0xC0) == 0x40) letter = 'C';
  else if ((first & 0xC0) == 0x80) letter = 'B';
  else if ((first & 0xC0) == 0xC0) letter = 'U';

  byte second = code2;
  int digit1 = (first & 0x30) >> 4;
  int digit2 = first & 0x0F;
  int digit3 = (second & 0xF0) >> 4;
  int digit4 = second & 0x0F;

  Serial.print(letter);
  Serial.print(digit1);
  Serial.print(digit2, HEX);
  Serial.print(digit3, HEX);
  Serial.println(digit4, HEX);
}

void readDTC() {
  unsigned long canId;
  byte len;
  byte buf[8];

  delay(100);

  if (CAN0.checkReceive() == CAN_MSGAVAIL) {
    CAN0.readMsgBuf(&canId, &len, buf);

    if (canId == 0x7E8 && buf[1] == 0x43) {  // Mode 3 response
      int dtcCount = buf[2] / 2;  // Each DTC is 2 bytes

      if (dtcCount == 0) {
        Serial.println("No DTCs stored");
        return;
      }

      Serial.print("Found ");
      Serial.print(dtcCount);
      Serial.println(" DTC(s):");

      // Parse DTCs (starting from byte 3)
      for (int i = 0; i < dtcCount * 2; i += 2) {
        Serial.print("DTC #");
        Serial.print((i / 2) + 1);
        Serial.print(": ");
        parseDTC(buf[3 + i], buf[4 + i]);
      }
    }
  }
}

void clearDTC() {
  // Mode 4 = Clear diagnostic trouble codes
  byte buf[8] = {0x02, 0x04, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};
  CAN0.sendMsgBuf(0x7DF, 0, 8, buf);
  Serial.println("DTC Clear Command Sent");
}

void loop() {
  Serial.println("Reading DTCs...");
  requestDTC();
  readDTC();

  delay(3000);

  // Uncomment to clear DTCs (use with caution)
  // clearDTC();
  // while (1);  // Stop after clearing

  delay(5000);
}
```

## Example 4: Performance Logger

```cpp
#include <mcp_can.h>
#include <SPI.h>
#include <SD.h>
#include <PCF8563.h>

MCP_CAN CAN0(D7);
PCF8563 pcf;
const int sdCS = D2;

struct PerformanceLog {
  unsigned long timestamp;
  int rpm;
  int speed;
  int throttle;
};

File logFile;

void setup() {
  Serial.begin(115200);
  Wire.begin();

  // Initialize CAN
  if (CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_8MHZ) == CAN_OK) {
    Serial.println("CAN Bus OK");
  } else {
    Serial.println("CAN Init Failed");
    while (1);
  }
  CAN0.setMode(MCP_NORMAL);

  // Initialize RTC
  pcf.init();

  // Initialize SD card
  pinMode(sdCS, OUTPUT);
  if (!SD.begin(sdCS)) {
    Serial.println("SD Card Failed");
    return;
  }

  // Create log file
  Time now = pcf.getTime();
  char filename[32];
  sprintf(filename, "/perf_%02d%02d%02d_%02d%02d.csv",
          now.year, now.month, now.day,
          now.hour, now.minute);

  logFile = SD.open(filename, FILE_WRITE);
  if (logFile) {
    logFile.println("Timestamp,RPM,Speed,Throttle");
    Serial.print("Logging to: ");
    Serial.println(filename);
  }
}

void sendRequest(byte pid) {
  byte buf[8] = {0x02, 0x01, pid};
  CAN0.sendMsgBuf(0x7DF, 0, 8, buf);
}

int readResponse(byte pid) {
  unsigned long canId;
  byte len;
  byte buf[8];

  delay(30);

  if (CAN0.checkReceive() == CAN_MSGAVAIL) {
    CAN0.readMsgBuf(&canId, &len, buf);

    if (canId == 0x7E8 && buf[2] == pid) {
      return ((buf[3] * 256) + buf[4]);
    }
  }
  return -1;
}

void loop() {
  if (!logFile) return;

  // Read sensors
  int rpm, speed, throttle;

  sendRequest(0x0C);  // RPM
  rpm = readResponse(0x0C) / 4;

  sendRequest(0x0D);  // Speed
  speed = readResponse(0x0D);

  sendRequest(0x11);  // Throttle
  throttle = (readResponse(0x11) * 100) / 255;

  // Get timestamp
  Time now = pcf.getTime();
  unsigned long timestamp = now.hour * 3600 + now.minute * 60 + now.second;

  // Log to SD
  logFile.print(timestamp);
  logFile.print(",");
  logFile.print(rpm);
  logFile.print(",");
  logFile.print(speed);
  logFile.print(",");
  logFile.println(throttle);
  logFile.flush();

  // Display
  Serial.print("RPM: ");
  Serial.print(rpm);
  Serial.print(" Speed: ");
  Serial.print(speed);
  Serial.print(" km/h Throttle: ");
  Serial.print(throttle);
  Serial.println("%");

  delay(200);  // 5 samples per second
}
```

## Wiring to Vehicle OBD-II Port

**OBD-II Connector Pinout:**
- Pin 4: Chassis Ground (connect to XIAO GND)
- Pin 5: Signal Ground (connect to XIAO GND)
- Pin 6: CAN High (connect to CAN H)
- Pin 14: CAN Low (connect to CAN L)
- Pin 16: Battery Power (+12V)

**XIAO Connection:**
```
Vehicle       XIAO CAN Board
────────────────────────────
CAN H    →   H terminal
CAN L    →   L terminal
GND      →   GND terminal
+12V     →   Not connected (use USB power)
```

## Safety Notes

- **Always test with vehicle turned off first**
- **Check fuse box before tapping into power**
- **Use proper gauge wire for connections**
- **Don't leave CAN bus connected when engine off for extended periods**
- **Some vehicles require ignition on for OBD-II to work**

## Troubleshooting

### No Response from ECU

**Symptom:** CAN messages sent but no response

**Possible Causes:**
1. Wrong baud rate
2. Vehicle not in ignition-on mode
3. Wiring issue

**Solution:** Verify 500kbps baud rate, check ignition, test wiring

### Garbled Data

**Symptom:** Invalid RPM or speed values

**Possible Causes:**
1. Response from wrong CAN ID
2. Data parsing error

**Solution:** Check canId matches ECU response (usually 0x7E8)

### CAN Bus Errors

**Symptom:** CAN errors, bus off state

**Possible Causes:**
1. Wrong termination
2. Wiring short/open

**Solution:** Enable 120Ω termination, check wiring

## See Also

- **CAN Basics:** `/xiao-arduino/references/expansion-boards/can-bus.md`
- **Hardware Docs:** `/xiao/references/expansion_boards/can-bus.md`
