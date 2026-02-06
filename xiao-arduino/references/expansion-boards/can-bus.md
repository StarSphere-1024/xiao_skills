# CAN Bus Expansion Board for XIAO - Arduino

## Overview

Arduino implementation for the XIAO CAN Bus Expansion Board. Uses MCP2515 CAN controller and SN65HVD230 transceiver for automotive/industrial CAN communication.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/can-bus.md`.

## Pin Connections

| MCP2515 | XIAO Pin | Function |
|---------|----------|----------|
| CS | D7 | Chip Select |
| SO | D9 | MISO |
| SI | D10 | MOSI |
| SCK | D8 | SPI Clock |
| INT | Optional | Interrupt (any GPIO) |

**Termination:** 120Ω termination via P1 pad on board back (solder jumper).

## Required Libraries

Install these libraries via Arduino Library Manager:

| Library | Purpose |
|---------|---------|
| **mcp_can** | MCP2515 CAN controller driver |

## Basic CAN Send Example

```cpp
#include <mcp_can.h>
#include <SPI.h>

MCP_CAN CAN0(D7);  // CS pin

void setup() {
  Serial.begin(115200);

  // Initialize CAN at 500kbps
  if (CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_8MHZ) == CAN_OK) {
    Serial.println("CAN Initialized Successfully!");
  } else {
    Serial.println("Error Initializing CAN...");
    while (1);
  }

  CAN0.setMode(MCP_NORMAL);
}

void loop() {
  // Send CAN message
  unsigned char stmp[8] = {0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08};

  byte sndStat = CAN0.sendMsgBuf(0x100, 0, 8, stmp);
  if (sndStat == CAN_OK) {
    Serial.println("Message Sent Successfully!");
  } else {
    Serial.println("Error Sending Message...");
  }
  delay(1000);
}
```

## Basic CAN Receive Example

```cpp
#include <mcp_can.h>
#include <SPI.h>

MCP_CAN CAN0(D7);
unsigned long canId;
byte len;
byte buf[8];

void setup() {
  Serial.begin(115200);

  if (CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_8MHZ) == CAN_OK) {
    Serial.println("CAN Initialized Successfully!");
  } else {
    Serial.println("Error Initializing CAN...");
    while (1);
  }

  CAN0.setMode(MCP_NORMAL);
  pinMode(D2, INPUT);  // Optional: Use interrupt pin
}

void loop() {
  if (CAN0.checkReceive() == CAN_MSGAVAIL) {
    CAN0.readMsgBuf(&canId, &len, buf);

    Serial.print("ID: ");
    Serial.print(canId, HEX);
    Serial.print(" Data: ");
    for (int i = 0; i < len; i++) {
      Serial.print(buf[i], HEX);
      Serial.print(" ");
    }
    Serial.println();
  }
}
```

## Interrupt-Driven Receive

```cpp
#include <mcp_can.h>
#include <SPI.h>

MCP_CAN CAN0(D7);
volatile bool messageReceived = false;

void IRAM_ATTR CAN_ISR() {
  messageReceived = true;
}

void setup() {
  Serial.begin(115200);

  if (CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_8MHZ) == CAN_OK) {
    Serial.println("CAN Initialized Successfully!");
  } else {
    Serial.println("Error Initializing CAN...");
    while (1);
  }

  // Set up interrupt on D2 (connect INT pin to D2)
  attachInterrupt(digitalPinToInterrupt(D2), CAN_ISR, FALLING);
}

void loop() {
  if (messageReceived) {
    messageReceived = false;

    unsigned long canId;
    byte len;
    byte buf[8];

    CAN0.readMsgBuf(&canId, &len, buf);

    Serial.print("ID: 0x");
    Serial.print(canId, HEX);
    Serial.print(" Len: ");
    Serial.print(len);
    Serial.print(" Data: ");
    for (int i = 0; i < len; i++) {
      Serial.print("0x");
      Serial.print(buf[i], HEX);
      Serial.print(" ");
    }
    Serial.println();
  }
}
```

## OBD-II RPM Example

```cpp
#include <mcp_can.h>
#include <SPI.h>

MCP_CAN CAN0(D7);

// OBD-II PID for RPM
#define PID_RPM 0x0C

void setup() {
  Serial.begin(115200);

  if (CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_8MHZ) == CAN_OK) {
    Serial.println("CAN Bus Ready");
  } else {
    Serial.println("CAN Init Failed");
    while (1);
  }

  CAN0.setMode(MCP_NORMAL);
}

void sendOBDRequest(byte pid) {
  byte len = 8;
  byte buf[8] = {0x02, 0x01, pid, 0x00, 0x00, 0x00, 0x00, 0x00};
  CAN0.sendMsgBuf(0x7DF, 0, len, buf);
}

void loop() {
  sendOBDRequest(PID_RPM);
  delay(100);

  if (CAN0.checkReceive() == CAN_MSGAVAIL) {
    unsigned long canId;
    byte len;
    byte buf[8];

    CAN0.readMsgBuf(&canId, &len, buf);

    // Check for response from 0x7E8 (ECU)
    if (canId == 0x7E8 && buf[2] == PID_RPM) {
      int rpm = ((buf[3] * 256) + buf[4]) / 4;
      Serial.print("RPM: ");
      Serial.println(rpm);
    }
  }
  delay(1000);
}
```

## CAN Bus Speed Selection

```cpp
// Available speeds (must match CAN network):
#define CAN_125KBPS  125000
#define CAN_250KBPS  250000
#define CAN_500KBPS  500000  // Most common
#define CAN_1000KBPS 1000000

// Initialize with desired speed
CAN0.begin(MCP_ANY, CAN_500KBPS, MCP_8MHZ);
```

## Filter Setup (Receive Specific IDs)

```cpp
void setupFilters() {
  // Accept only ID 0x100-0x1FF
  struct CAN_FILTER_MASK mask;
  mask.flags.rxn = 0;  // Receive filter
  mask.flags.ext = 0;  // Standard ID
  mask.id = 0x100;
  mask.mask = 0x1FF;  // Mask for ID range

  CAN0.initMask(MCP_RXM0, &mask, true);
  CAN0.initFilter(MCP_RXF0, &mask, true);
}
```

## Troubleshooting

### No Messages Received

**Symptom:** CAN send works, but no messages received

**Possible Causes:**
1. Bus speed mismatch
2. No termination resistor
3. Wrong filter settings

**Solution:**
- Verify all devices use same CAN speed
- Solder P1 pad for 120Ω termination
- Check wiring (CAN High, CAN Low, GND)

### CAN Init Fails

**Symptom:** "Error Initializing CAN..."

**Possible Causes:**
1. Wrong CS pin
2. MCP2515 not powered
3. Crystal frequency wrong

**Solution:** Verify D7 is CS pin, check 8MHz crystal in code

### Bus Off Error

**Symptom:** CAN stops transmitting/receiving

**Possible Causes:**
1. Bus short circuit
2. Too many errors
3. Wrong termination

**Solution:** Check wiring, reset CAN controller with `CAN0.reset();`

## Safety Notes

- CAN bus operates at automotive voltage levels
- Ensure proper grounding between XIAO and CAN devices
- Use opto-isolation if connecting to vehicle CAN bus
- Always test with isolated CAN bus first
