# GPS/GNSS Module for XIAO - Arduino

## Overview

Arduino implementation for the XIAO GPS/GNSS Expansion Board with L76K GNSS module. Supports GPS, GLONASS, BeiDou, and Galileo satellite systems.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/gps-gnss.md`.

## Pin Connections

| Function | XIAO Pin | Notes |
|----------|----------|-------|
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

**Important:** Must have clear view of sky. Will NOT work indoors.

## Required Libraries

| Library | Purpose |
|---------|---------|
| **TinyGPS++** | NMEA sentence parsing |

## Basic Position Reading

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);  // Serial1 on D6/D7

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);  // L76K default baud rate

  Serial.println("Waiting for GPS fix...");
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  if (gps.location.isUpdated()) {
    Serial.print("Location: ");
    Serial.print(gps.location.lat(), 6);
    Serial.print(",");
    Serial.println(gps.location.lng(), 6);
  }

  if (gps.satellites.isUpdated()) {
    Serial.print("Satellites: ");
    Serial.println(gps.satellites.value());
  }

  delay(1000);
}
```

## Complete GPS Data Display

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);
  Serial.println("GPS Data Logger");
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  if (gps.location.isValid()) {
    Serial.println("=== GPS DATA ===");

    // Position
    Serial.print("Latitude: ");
    Serial.println(gps.location.lat(), 6);
    Serial.print("Longitude: ");
    Serial.println(gps.location.lng(), 6);

    // Altitude
    if (gps.altitude.isValid()) {
      Serial.print("Altitude: ");
      Serial.print(gps.altitude.meters());
      Serial.println(" meters");
    }

    // Speed
    if (gps.speed.isValid()) {
      Serial.print("Speed: ");
      Serial.print(gps.speed.kmph());
      Serial.println(" km/h");
    }

    // Course
    if (gps.course.isValid()) {
      Serial.print("Course: ");
      Serial.print(gps.course.deg());
      Serial.println(" degrees");
    }

    // Satellites
    if (gps.satellites.isValid()) {
      Serial.print("Satellites: ");
      Serial.println(gps.satellites.value());
    }

    // HDOP (accuracy)
    if (gps.hdop.isValid()) {
      Serial.print("HDOP: ");
      Serial.println(gps.hdop.value());
    }

    // Date/Time
    if (gps.date.isValid() && gps.time.isValid()) {
      Serial.print("Time: ");
      Serial.print(gps.time.hour());
      Serial.print(":");
      Serial.print(gps.time.minute());
      Serial.print(":");
      Serial.println(gps.time.second());
    }

    Serial.println("================");
  } else {
    Serial.print("Satellites: ");
    Serial.println(gps.satellites.value());
    Serial.println("Waiting for fix...");
  }

  delay(2000);
}
```

## SD Card Logging

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>
#include <SD.h>
#include <SPI.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);
File logFile;

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);

  // Initialize SD card
  if (!SD.begin(D2)) {
    Serial.println("SD init failed!");
    return;
  }

  // Create new log file
  int fileNum = 1;
  String filename;
  do {
    filename = "/gps_log_" + String(fileNum++) + ".csv";
  } while (SD.exists(filename));

  logFile = SD.open(filename, FILE_WRITE);
  if (logFile) {
    logFile.println("Timestamp,Date,Time,Lat,Lon,Alt(m),Speed(kmph),Course,Satellites");
    Serial.print("Logging to: ");
    Serial.println(filename);
  }
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  if (gps.location.isValid() && logFile) {
    // Log data to SD card
    logFile.print(millis());
    logFile.print(",");

    // Date
    logFile.print(gps.date.year());
    logFile.print("-");
    logFile.print(gps.date.month());
    logFile.print("-");
    logFile.print(gps.date.day());
    logFile.print(",");

    // Time
    logFile.print(gps.time.hour());
    logFile.print(":");
    logFile.print(gps.time.minute());
    log4File.print(":");
    logFile.print(gps.time.second());
    logFile.print(",");

    // Position
    logFile.print(gps.location.lat(), 6);
    logFile.print(",");
    logFile.print(gps.location.lng(), 6);
    logFile.print(",");

    // Altitude
    logFile.print(gps.altitude.meters());
    logFile.print(",");

    // Speed
    logFile.print(gps.speed.kmph());
    logFile.print(",");

    // Course
    logFile.print(gps.course.deg());
    logFile.print(",");

    // Satellites
    logFile.println(gps.satellites.value());

    logFile.flush();

    // Also print to serial
    Serial.print("Logged: ");
    Serial.print(gps.location.lat(), 6);
    Serial.print(",");
    Serial.println(gps.location.lng(), 6);
  }

  delay(1000);
}
```

## Distance and Bearing Calculation

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);

// Target location (example: New York City)
const double TARGET_LAT = 40.7128;
const double TARGET_LON = -74.0060;

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);
  Serial.println("GPS Distance Calculator");
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  if (gps.location.isValid()) {
    double distance = gps.distanceBetween(
      gps.location.lat(),
      gps.location.lng(),
      TARGET_LAT,
      TARGET_LON
    ) / 1000.0;  // Convert to km

    double course = gps.courseTo(
      gps.location.lat(),
      gps.location.lng(),
      TARGET_LAT,
      TARGET_LON
    );

    Serial.print("Distance to target: ");
    Serial.print(distance);
    Serial.println(" km");

    Serial.print("Course to target: ");
    Serial.print(course);
    Serial.println(" degrees");

    Serial.print("Current course: ");
    Serial.print(gps.course.deg());
    Serial.println(" degrees");

    double diff = course - gps.course.deg();
    if (diff < 0) diff += 360;
    Serial.print("Steer: ");
    if (diff < 10 || diff > 350) {
      Serial.println("Straight");
    } else if (diff < 180) {
      Serial.print("Right ");
      Serial.print(diff);
      Serial.println(" degrees");
    } else {
      Serial.print("Left ");
      Serial.print(360 - diff);
      Serial.println(" degrees");
    }
  }

  delay(2000);
}
```

## NMEA Sentence Output

```cpp
#include <HardwareSerial.h>

HardwareSerial GPSSerial(1);

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);
  Serial.println("Raw NMEA Output");
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    Serial.write(c);  // Echo NMEA to USB serial
  }
}
```

## Troubleshooting

### No GPS Fix

**Symptom:** Cannot acquire satellites

**Possible Causes:**
1. Indoor use - Must have clear sky view
2. Antenna issue - Check antenna connection
3. Cold start - First fix takes 30+ seconds
4. Poor location - Away from buildings/trees

**Solution:**
- Move outdoors with clear sky view
- Wait for cold start (30-60 seconds)
- Check antenna connection

### Garbled Data

**Symptom:** NMEA sentences corrupted

**Possible Causes:**
1. Wrong baud rate
2. TX/RX reversed
3. Wrong UART

**Solution:**
```cpp
// Try different baud rates
GPSSerial.begin(9600);   // Default
// GPSSerial.begin(115200);  // Optional high speed
```

### Slow Fix Time

**Symptom:** Takes long to acquire satellites

**Possible Causes:**
1. First use (cold start)
2. Moved far from last use
3. Poor antenna

**Solution:**
- Wait longer for cold start
- Use antenna with better view
- Aiding data can help (requires implementation)

## NMEA Sentences

The L76K outputs standard NMEA sentences:
- **GGA:** Fix data (3D location, accuracy)
- **RMC:** Recommended minimum (time, date, position, speed)
- **GSA:** Overall satellite data
- **GSV:** Detailed satellite data
- **VTG:** Track and ground speed

## Power Notes

- **Voltage:** 3.3V only (do NOT use 5V)
- **Current:** ~50mA during acquisition, ~25mA tracking
- **Antenna:** Active antenna recommended for best performance
