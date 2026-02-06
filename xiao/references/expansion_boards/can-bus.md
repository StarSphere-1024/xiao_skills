# CAN Bus Expansion Board for XIAO

## Overview

The CAN Bus Expansion Board adds CAN (Controller Area Network) communication capability to XIAO boards. It features an MCP2515 CAN controller and SN65HVD230 transceiver, providing reliable automotive and industrial communication. Ideal for vehicle diagnostics, industrial control, robotics, and IoT applications requiring robust differential signaling.

## Hardware

### Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | Full support |
| ESP32C5 | ✅ | Full support |
| ESP32C6 | ✅ | Full support |
| ESP32S3 | ✅ | Full support |
| nRF52840 | ✅ | Full support |
| RP2040 | ✅ | Full support |
| RP2350 | ✅ | Full support |
| SAMD21 | ✅ | Full support |
| RA4M1 | ✅ | Full support |
| MG24 | ✅ | Full support |
| nRF54L15 | ✅ | Full support |

### Features

- **MCP2515 CAN Controller**: Handles protocol management, message filtering, and error handling
- **SN65HVD230 Transceiver**: Converts digital signals to differential CAN bus signals
- **Terminal Connection**: 3-pin terminal (CANH, CANL, GND) for easy bus connection
- **LED Indicators**: RX and TX LEDs for communication status
- **120Ω Termination**: Optional via P1 pad on board back
- **SPI Interface**: Communication with XIAO via SPI

### Specifications

| Item | Value |
|------|-------|
| Communication Interface | CAN bus (Controller Area Network) |
| CAN Controller | MCP2515 |
| CAN Transceiver | SN65HVD230 |
| Max Baud Rate | 1 Mbps |
| Connector | 3-pin terminal (CANH, CANL, GND) |
| LEDs | RX, TX indicators |

### Pin Connections

| Function | XIAO Pin | Notes |
|----------|-----------|-------|
| SPI CS | D7 | Chip Select (configurable in code) |
| SPI SCK | D8 | SPI Clock |
| SPI MISO | D9 | SPI MISO |
| SPI MOSI | D10 | SPI MOSI |
| Interrupt | Optional | Can be configured |

**IMPORTANT:** The SPI pins (D7-D10) are used by the CAN controller and not available for other uses when CAN bus is active.

**Standard SPI Pin Mapping (MCP2515):**
```
MCP2515    XIAO
-------    -----
CS    →    D7
SO    →    D9
SI    →    D10
SCK   →    D8
INT   →    Optional
GND   →    GND
```

### Termination Resistor

The board has a **P1 pad on the back** for adding a 120Ω termination resistor:
- **Default**: P1 is NOT shorted (no termination)
- **When to short**: If communication fails, try shorting P1 for termination
- **CAN bus rule**: Termination resistors needed at both ends of bus

**CAN Bus Network Topology:**
```
[Node 1] ───── 120Ω ───── [Node 2] ───── ... ───── [Node N]
  │                                      │
 120Ω                                   120Ω
  │                                      │
CANH/CANL                            CANH/CANL
```

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/can-bus.md`
  - MCP2515 library setup
  - Basic send/receive examples
  - OBD-II PIDs for vehicle diagnostics
  - Mask and filter configuration

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## Supported Baud Rates

The MCP2515 supports these standard CAN bus baud rates:

| Rate | Constant | Use Case |
|------|----------|----------|
| 5 Kbps | CAN_5KBPS | Low speed, long distance |
| 10 Kbps | CAN_10KBPS | Low speed industrial |
| 20 Kbps | CAN_20KBPS | Industrial control |
| 25 Kbps | CAN_25KBPS | Industrial |
| 31.25 Kbps | CAN_31K25BPS | DeviceNet |
| 33 Kbps | CAN_33KBPS | Industrial |
| 40 Kbps | CAN_40KBPS | Industrial |
| 50 Kbps | CAN_50KBPS | General purpose |
| 80 Kbps | CAN_80KBPS | Higher speed |
| 83.3 Kbps | CAN_83K3BPS | CANopen |
| 95 Kbps | CAN_95KBPS | Industrial |
| 100 Kbps | CAN_100KBPS | Common |
| 125 Kbps | CAN_125KBPS | Common |
| 200 Kbps | CAN_200KBPS | Vehicle |
| 250 Kbps | CAN_250KBPS | Vehicle (common) |
| 500 Kbps | CAN_500KBPS | Vehicle (most common) |
| 666 Kbps | CAN_666kbps | Special |
| 1000 Kbps | CAN_1000KBPS | Maximum speed |

## CAN Bus Basics

### What is CAN Bus?

CAN (Controller Area Network) is a robust vehicle bus standard designed to allow microcontrollers and devices to communicate with each other in applications without a host computer.

### Key Features

- **Differential Signaling**: CANH and CANL for noise immunity
- **Multi-Master**: Any node can initiate communication
- **Message Prioritization**: Lower ID wins arbitration
- **Error Detection**: CRC, ACK, error framing
- **Automatic Retransmission**: Failed messages are resent

### Frame Types

1. **Data Frame**: Carries actual data
2. **Remote Frame**: Requests data from another node
3. **Error Frame**: Reports errors
4. **Overload Frame**: Introduces delay

### Standard vs Extended Frames

- **Standard Frame (11-bit ID)**: 0-0x7FF, more bandwidth efficient
- **Extended Frame (29-bit ID)**: 0-0x1FFFFFFF, more IDs available

## Usage Notes

1. **CS Pin**: Default is D7, can be changed in code
2. **Termination**: Enable P1 pad only at bus ends
3. **LED Behavior**:
   - Constant ON = No other devices on bus
   - Blinking = Active communication
4. **Maximum Cable Length**: Depends on baud rate
   - 125 Kbps: Up to 500m
   - 500 Kbps: Up to 40m
   - 1 Mbps: Up to 20m
5. **Multiple Nodes**: Can use multiple expansion boards on same bus

## Applications

- **Automotive**: OBD-II diagnostics, vehicle data logging
- **Industrial Control**: Factory automation, PLC communication
- **Robotics**: Inter-module communication in robotic systems
- **IoT**: Sensor networks with robust communication
- **Building Automation**: HVAC, lighting control

## Troubleshooting

### No Communication

**Symptom**: Cannot send or receive messages

**Possible Causes:**
1. Wrong baud rate - Match all devices on bus
2. Missing termination - Short P1 pad if at bus end
3. Wrong CS pin - Verify D7 in code
4. Wiring issue - Check CANH/CANL connection

**Solution**: Check baud rate, enable termination, verify wiring

### LEDs Always On

**Symptom**: RX/TX LEDs stay lit constantly

**Possible Causes:**
1. No other devices on bus - Normal behavior
2. Wrong termination - Only board with termination
3. Bus short circuit - Check CANH/CANL wiring

**Solution**: This is normal if only device on bus

### CRC Errors

**Symptom**: High error rate, messages not received

**Possible Causes:**
1. Baud rate mismatch - All devices must use same rate
2. Cable too long - Reduce cable length for baud rate
3. Electrical noise - Use shielded cable
4. Missing termination - Add 120Ω at both ends

**Solution**: Verify baud rate, check termination, use shielded cable

### Max Baud Rate

**Q**: What is the maximum baud rate?
**A**: 1 Mbps maximum, limited by MCP2515 controller

### Multiple Boards

**Q**: Can I use multiple expansion boards on same bus?
**A**: Yes, each should have unique node ID. Only enable termination on end devices.

### Cable Length

**Q**: What is the maximum cable length?
**A**: Depends on baud rate. At 500 Kbps: ~40m. At 125 Kbps: ~500m.

### Other Microcontrollers

**Q**: Can I use with other boards?
**A**: Designed for XIAO, but may work with proper pin mapping and SPI support.

## Comparison with RS485

| Feature | CAN Bus | RS485 |
|---------|---------|-------|
| Protocol | Standardized (J1939, etc.) | Raw serial |
| Arbitration | Automatic priority | Master/slave |
| Error Handling | Built-in, automatic | Application-level |
| Complexity | Higher | Lower |
| Cost | Higher | Lower |
| Automotive | Standard | Less common |
| Industrial | Common | Very common |

**Choose CAN Bus if:** Automotive applications, robust error handling needed
**Choose RS485 if:** Simple point-to-point, cost-sensitive, industrial networks
