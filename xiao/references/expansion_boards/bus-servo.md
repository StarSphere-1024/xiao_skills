# Bus Servo Driver Board / XIAO Bus Servo Adapter

## Overview

The Bus Servo Driver Board is a hardware solution for controlling serial bus servos (ST/SC series) for robotics and automation. It supports UART communication for precise control with position and load feedback. Available in two versions: the driver board (requires external controller) and the XIAO Bus Servo Adapter (includes integrated XIAO ESP32-C3). Ideal for robotic arms, hexapods, humanoid robots, and wheeled robots.

## Hardware

### Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | Built-in on XIAO Bus Servo Adapter |
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

### Product Versions

**1. Bus Servo Driver Board (without XIAO):**
- Does NOT include XIAO ESP32-C3
- Does NOT include 3D-printed enclosure
- Requires external UART controller
- General-purpose bus servo interface

**2. XIAO Bus Servo Adapter:**
- Includes XIAO ESP32-C3 onboard
- Includes 3D-printed case
- Direct servo control via XIAO
- Ready-to-use solution

### Features

- **Serial Bus Servo Support**: ST/SC series (Feetech SCS)
- **UART Communication**: RX/TX interface
- **Multiple Servo Control**: Control multiple servos on single bus
- **Feedback Support**: Position, speed, load, temperature, voltage
- **5-12V Power**: Supports various servo voltages
- **Jumper Settings**: USB or UART mode selection

### Specifications

| Item | Value |
|------|-------|
| Power Input | DC 5-12V via 5.5×2.1mm barrel jack |
| Servo Interface | ST/SC series connector |
| Control Interface | UART (RX/TX) |
| Communication Speed | 1 Mbps |
| Baud Rate | Up to 1,000,000 bps |

### Pin Connections

| Function | XIAO Pin | Notes |
|----------|-----------|-------|
| Servo RX | D7 | Connect to TX on host |
| Servo TX | D6 | Connect to RX on host |
| Power | DC 5-12V | Must match servo voltage |

**UART Connection (for XIAO):**
```
XIAO          Bus Servo Board
────────────────────────────
D7 (TX)  ←→  RX
D6 (RX)  ←→  TX
GND      ←→  GND
```

**Important:**
- **D6 and D7** are used for UART communication with servos
- For XIAO Bus Servo Adapter, no external connection needed
- For Bus Servo Driver Board, connect XIAO D6/D7 to board's RX/TX

### Jumper Settings (Critical!)

#### UART Connection Mode (for MCUs, XIAO, ESP32)

**Wiring:**
- Board RX → XIAO D7 (TX)
- Board TX → XIAO D6 (RX)

**Jumper Setting:**
- **Do NOT** short the 2-pin header on the front
- No jumper cap needed
- Backside pads remain open

**Power:** Host device needs separate power supply

#### USB Connection Mode (for PC, Raspberry Pi 4B)

**Wiring:**
- Connect USB cable from computer to board

**Jumper Setting (Critical):**
1. Ensure backside pads have solder bridge (both versions)
2. Short the 2-pin header on front with jumper cap

**Board Versions:**
- Version 1: Check backside pads are bridged
- Version 2: Check backside pads are bridged

## Supported Servos

### Feetech SCS/STS/TTL Series

Compatible with Feetech serial bus servos:
- ST series (typically 9V operation)
- SC series (typically 12V operation)
- STS series
- TTL series

**For more details:** [Feetech SCS/STS/TTL Series Official Website](https://www.feetechrc.com/en/scs_ttl_Servo.html)

### Servo Selection

**Voltage Matching:**
- ST series: Usually 9V
- SC series: Usually 12V
- Must match power supply voltage to servo requirements

**Power Supply:**
- Use appropriate voltage (5-12V DC)
- Ensure adequate current for all servos
- Calculate: Stall current × number of servos

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/bus-servo.md`
  - SCServo library setup
  - Position, speed, acceleration control
  - Multi-servo synchronization
  - Feedback reading (position, load, temperature)

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## Library Functions (Arduino)

**Key Functions:**
- `WritePos(id, position, time, speed)` - Set servo position
- `ReadPos(id)` - Read current position
- `ReadLoad(id)` - Read current load
- `ReadVoltage(id)` - Read servo voltage
- `ReadTemper(id)` - Read servo temperature
- `SyncWritePosEx()` - Synchronize multiple servos

**Feedback Capabilities:**
- Position (0-4095)
- Speed
- Load (torque)
- Voltage
- Temperature
- Movement status
- Current draw

## Usage Notes

1. **Voltage Matching**: Critical! Use correct voltage for your servo type
2. **Jumper Settings**: MUST be correct for your connection mode
3. **Pin Connections**: D6=RX, D7=TX for XIAO UART
4. **Servo IDs**: Each servo must have unique ID
5. **Power Supply**: Must handle combined stall current
6. **Initialization**: Servos move on power-up (normal behavior)

### Connection Examples

**Single Servo:**
```
[XIAO] ←UART→ [Driver Board] ←→ [Servo]
  5V                  9-12V
```

**Multiple Servos (Daisy-Chain):**
```
                [Servo 1]
                    ↓
[XIAO] ←UART→ [Driver Board] ←→ [Servo 2]
                    ↓
                [Servo 3]
```

## Applications

- **Robotic Arms**: Precise multi-joint control with feedback
- **Hexapod Robots**: 6+ leg servos with coordinated movement
- **Humanoid Robots**: Multiple servos for natural motion
- **Wheeled Robots**: Steering and positioning
- **Camera Gimbals**: Pan/tilt with stabilization
- **Robot Grippers:** Force feedback for grasping

## Troubleshooting

### Servos Not Responding

**Symptom**: No movement from servos

**Possible Causes:**
1. Wrong jumper setting - Check UART vs USB mode
2. Wrong baud rate - Should be 1,000,000 bps
3. Wrong pins - D6/RX, D7/TX
4. No power - Check DC input matches servo voltage

**Solution**: Verify jumper settings, check baud rate, verify wiring

### Servo Moves Randomly on Power-Up

**Symptom**: Servos move when powered on

**Possible Causes:**
1. Normal initialization - This is expected behavior
2. Wrong servo ID - Check ID in code matches servo

**Solution**: Accept as normal, or reposition after initialization

### Communication Errors

**Symptom**: Cannot communicate with servos

**Possible Causes:**
1. Wrong mode - UART vs USB jumper issue
2. TX/RX reversed - Check D6/D7 connections
3. Baud rate mismatch - Use 1,000,000 bps

**Solution**: Check jumper settings, verify TX/RX, confirm baud rate

### Incorrect Jumper Setting

**Q**: What happens if jumper is wrong?
**A**: Communication will fail. USB mode requires BOTH backside solder bridge AND front jumper cap.

### Multiple Servo Support

**Q**: Can I connect multiple servos?
**A**: Yes, multiple servos supported. Ensure power supply handles combined current.

### Power Supply Voltage

**Q**: What if voltage doesn't match servo?
**A**: Servo may malfunction or be damaged. Always match voltage to servo requirements.

## Safety Warnings

1. **Disconnect power** before connecting/disconnecting servos
2. **Match voltage** to servo specifications
3. **Check current** requirements for all servos
4. **Secure wiring** to prevent shorts
5. **Test with small movements** first

## Comparison: Driver Board vs Adapter

| Feature | Driver Board | XIAO Bus Servo Adapter |
|---------|--------------|------------------------|
| XIAO Included | No | Yes (ESP32-C3) |
| Enclosure | No | Yes (3D-printed) |
| Requires External MCU | Yes | No |
| Use Case | Custom integration | Ready-to-use |
| Form Factor | Board only | Complete solution |

**Choose Driver Board if:** You want custom integration with your own controller
**Choose Adapter if:** You want a complete, ready-to-use solution with XIAO included
