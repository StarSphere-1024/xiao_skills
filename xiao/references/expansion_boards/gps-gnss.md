# GPS/GNSS Module for XIAO

## Overview

The GPS/GNSS Expansion Board adds L76K GNSS positioning capability to XIAO boards. Supports GPS, GLONASS, BeiDou, and Galileo satellite systems for accurate positioning. Ideal for tracking, navigation, geolocation tagging, timing applications, and location-based IoT projects.

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

- **L76K GNSS Module**: Multi-constellation support
- **Satellite Systems**: GPS, GLONASS, BeiDou, Galileo
- **UART Interface**: Serial communication at 9600 baud
- **Low Power**: Power saving modes available
- **High Accuracy**: ~2.5m CEP accuracy

### Specifications

| Item | Value |
|------|-------|
| GNSS Module | L76K |
| Satellite Systems | GPS, GLONASS, BeiDou, Galileo |
| Update Rate | 1Hz (default) up to 10Hz |
| Accuracy | ~2.5m CEP |
| Communication | UART (9600 baud default) |
| Supply Voltage | 3.3V |
| Operating Temperature | -40°C to +85°C |

### Pin Connections

| Function | XIAO Pin | Notes |
|----------|-----------|-------|
| GNSS RX | D7 (TX) | Connect to XIAO TX |
| GNSS TX | D6 (RX) | Connect to XIAO RX |
| VCC | 3.3V | Power |
| GND | GND | Ground |

**UART Connection:**
```
XIAO          GNSS Module
────────────────────────────
D6 (RX)  ←→  TX
D7 (TX)  ←→  RX
3.3V    ←→  VCC
GND     ←→  GND
```

**Important:** D6 and D7 are used for UART communication with GPS module.

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/gps-gnss.md`
  - TinyGPS++ library setup
  - NMEA sentence parsing
  - Position, altitude, speed, course reading
  - Time synchronization
  - Data logging examples

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## NMEA Sentences

The L76K outputs standard NMEA sentences:
- **GGA**: Fix data (3D location and accuracy data)
- **RMC**: Recommended Minimum data (time, date, position, speed)
- **GSA**: Overall satellite data
- **GSV**: Detailed satellite data
- **VTG**: Track made good and ground speed

## Usage Notes

1. **Antenna**: Ensure antenna has clear view of sky
2. **Cold Start**: First fix may take 30+ seconds
3. **Indoor Use**: Will NOT work indoors (requires sky view)
4. **Baud Rate**: Default 9600 baud (configurable)
5. **Update Rate**: 1Hz default, can be increased
6. **Power**: 3.3V only, do not use 5V

## Applications

- **Asset Tracking**: Vehicle, pet, personal tracking
- **Navigation**: Hiking, marine, automotive
- **Geotagging**: Photos, sensor data with location
- **Time Synchronization**: UTC time from satellites
- **Location-Based IoT**: Trigger actions by location
- **Sports**: Running, cycling, fitness tracking
- **Fleet Management**: Vehicle tracking and monitoring

## Troubleshooting

### No GPS Fix

**Symptom**: Cannot acquire satellite lock

**Possible Causes:**
1. Indoor use - Must have clear sky view
2. Antenna issue - Check antenna connection
3. Cold start - Wait 30+ seconds for first fix
4. Poor location - Away from buildings/trees

**Solution**: Move outdoors with clear sky view, wait for cold start

### Garbled Data

**Symptom**: NMEA sentences corrupted

**Possible Causes:**
1. Wrong baud rate - Check 9600 baud
2. TX/RX reversed - Verify D6/D7 connections
3. Wrong UART - Check correct serial port

**Solution**: Verify baud rate, check TX/RX, confirm serial port

### Slow Fix Time

**Symptom**: Takes long time to acquire satellites

**Possible Causes:**
1. First use (cold start) - Normal, wait 30-60 seconds
2. Moved far from last use - Aiding data helps
3. Poor antenna - Use active antenna

**Solution**: Wait longer for cold start, use antenna with better view

## Comparison with Other Location Methods

| Method | Accuracy | Indoors | Power | Cost |
|--------|----------|---------|-------|------|
| GNSS | ~2.5m | No | Moderate | Moderate |
| WiFi | ~10-50m | Yes | Low | Low |
| BLE Beacon | ~1-5m | Yes | Low | Low |
| Cellular | ~100-1000m | Yes | Moderate | Moderate |

**Choose GNSS if:** Outdoor accuracy needed, global coverage
**Choose WiFi/BLE if:** Indoor positioning, lower accuracy acceptable

## Required Libraries (Arduino)

- **TinyGPS++**: NMEA parsing library
  - Available in Arduino Library Manager
  - Lightweight, easy to use
  - Supports all NMEA sentences

## Example Data Output

**Typical NMEA Sentence (RMC):**
```
$GNRMC,123519,A,2403.7485,N,12018.5157,E,0.1,353.6,280211,,,A*68
```

**Parsed Values:**
- Time: 12:35:19 UTC
- Status: Valid (A)
- Latitude: 24° 3.7485' N
- Longitude: 120° 18.5157' E
- Speed: 0.1 knots
- Course: 353.6°
- Date: 28/11/2011
