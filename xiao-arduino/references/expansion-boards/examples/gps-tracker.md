# GPS Tracker Examples - Arduino

## Overview

Complete Arduino examples for building a GPS tracker using the XIAO GPS/GNSS Expansion Board with L76K module. Examples demonstrate position tracking, speed monitoring, data logging to SD card, and waypoint navigation.

**Hardware Reference:** `/xiao/references/expansion_boards/gps-gnss.md`
**Arduino Implementation:** `/xiao-arduino/references/expansion-boards/gps-gnss.md`

## Required Libraries

Install via Arduino Library Manager:
- **TinyGPS++** - NMEA sentence parsing
- **SD** - For data logging (built-in)

## Example 1: Basic Position Tracker

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);  // Serial1 on D6/D7

struct Position {
  double lat;
  double lon;
  double alt;
  unsigned long age;
};

Position lastPosition;
bool hasFirstFix = false;

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);  // L76K default baud rate

  Serial.println("GPS Tracker - Waiting for fix...");
  Serial.println("Ensure outdoor location with clear sky view");
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  if (gps.location.isValid() && gps.location.age() < 2000) {
    if (!hasFirstFix) {
      hasFirstFix = true;
      Serial.println("=== GPS FIX ACQUIRED ===");
    }

    Serial.println("=== Position Update ===");
    Serial.print("Latitude:  ");
    Serial.println(gps.location.lat(), 6);
    Serial.print("Longitude: ");
    Serial.println(gps.location.lng(), 6);
    Serial.print("Altitude:  ");
    Serial.print(gps.altitude.meters());
    Serial.println(" meters");
    Serial.print("Satellites: ");
    Serial.println(gps.satellites.value());
    Serial.print("HDOP: ");
    Serial.println(gps.hdop.value());
    Serial.println();

    lastPosition.lat = gps.location.lat();
    lastPosition.lon = gps.location.lng();
    lastPosition.alt = gps.altitude.meters();
    lastPosition.age = gps.location.age();
  } else {
    if (hasFirstFix) {
      Serial.println("=== Signal Lost ===");
      hasFirstFix = false;
    } else {
      Serial.print("Waiting for fix... Satellites: ");
      Serial.println(gps.satellites.value());
    }
  }

  delay(1000);
}
```

## Example 2: Speed and Heading Display

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);

  Serial.println("Speed and Heading Monitor");
  Serial.println("------------------------");
}

void displaySpeedCourse() {
  if (gps.speed.isValid()) {
    Serial.print("Speed: ");

    // Multiple units
    Serial.print(gps.speed.kmph());
    Serial.print(" km/h (");
    Serial.print(gps.speed.mph());
    Serial.print(" mph / ");
    Serial.print(gps.speed.knots());
    Serial.println(" knots)");
  }

  if (gps.course.isValid()) {
    Serial.print("Course: ");
    Serial.print(gps.course.deg());
    Serial.println(" degrees");

    // Cardinal direction
    const char* directions[] = {
      "N", "NNE", "NE", "ENE", "E", "ESE", "SE", "SSE",
      "S", "SSW", "SW", "WSW", "W", "WNW", "NW", "NNW"
    };
    int index = (int)((gps.course.deg() + 11.25) / 22.5) % 16;
    Serial.print("Heading: ");
    Serial.println(directions[index]);
  }

  if (gps.alt.isValid()) {
    Serial.print("Altitude: ");
    Serial.print(gps.alt.meters());
    Serial.println(" meters");
  }

  Serial.println();
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  if (gps.location.isUpdated()) {
    displaySpeedCourse();
  }

  delay(1000);
}
```

## Example 3: SD Card Data Logger

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>
#include <SD.h>
#include <SPI.h>
#include <PCF8563.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);
PCF8563 pcf;

const int sdCS = D2;
File logFile;
unsigned long lastLogTime = 0;
const unsigned long LOG_INTERVAL = 5000;  // Log every 5 seconds

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);
  Wire.begin();

  // Initialize RTC
  pcf.init();

  // Initialize SD card
  pinMode(sdCS, OUTPUT);
  if (!SD.begin(sdCS)) {
    Serial.println("SD Card initialization failed!");
    return;
  }

  // Create log file with timestamp
  Time now = pcf.getTime();
  char filename[32];
  sprintf(filename, "/gps_%02d%02d%02d_%02d%02d.csv",
          now.year, now.month, now.day,
          now.hour, now.minute);

  logFile = SD.open(filename, FILE_WRITE);
  if (logFile) {
    // Write CSV header
    logFile.println("Timestamp,Date,Time,Lat,Lon,Alt,Speed,Course,Satellites,HDOP");
    Serial.print("Logging to: ");
    Serial.println(filename);
  } else {
    Serial.println("Error creating log file");
  }

  Serial.println("GPS Logger Started");
}

void logGPSData() {
  if (!logFile || !gps.location.isValid()) return;

  Time now = pcf.getTime();

  // Timestamp
  logFile.print(millis());
  logFile.print(",");

  // Date
  logFile.print(now.day);
  logFile.print("/");
  logFile.print(now.month);
  logFile.print("/20");
  logFile.print(now.year);
  logFile.print(",");

  // Time
  logFile.print(now.hour);
  logFile.print(":");
  logFile.print(now.minute);
  logFile.print(":");
  logFile.print(now.second);
  logFile.print(",");

  // Position
  logFile.print(gps.location.lat(), 6);
  logFile.print(",");
  logFile.print(gps.location.lng(), 6);
  logFile.print(",");
  logFile.print(gps.altitude.meters());
  logFile.print(",");

  // Speed and course
  logFile.print(gps.speed.kmph());
  logFile.print(",");
  logFile.print(gps.course.deg());
  logFile.print(",");

  // Satellite info
  logFile.print(gps.satellites.value());
  logFile.print(",");
  logFile.println(gps.hdop.value());

  logFile.flush();

  Serial.println("Data logged");
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  // Log at interval
  if (millis() - lastLogTime >= LOG_INTERVAL) {
    lastLogTime = millis();
    logGPSData();
  }

  delay(10);
}
```

## Example 4: Distance Tracker

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);

double totalDistance = 0.0;  // in meters
double lastLat = 0.0;
double lastLon = 0.0;
bool hasLastPoint = false;

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);

  Serial.println("Distance Tracker");
  Serial.println("===============");
}

void updateDistance() {
  if (!gps.location.isUpdated()) return;

  double currentLat = gps.location.lat();
  double currentLon = gps.location.lng();

  if (hasLastPoint) {
    // Calculate distance from last point
    double segmentDistance = gps.distanceBetween(
      lastLat, lastLon,
      currentLat, currentLon
    );

    // Only update if movement (avoid GPS drift)
    if (segmentDistance > 2.0) {
      totalDistance += segmentDistance;
      Serial.print("Segment: ");
      Serial.print(segmentDistance);
      Serial.print("m | Total: ");
      Serial.print(totalDistance);
      Serial.println("m");
    }
  }

  lastLat = currentLat;
  lastLon = currentLon;
  hasLastPoint = true;
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  updateDistance();

  delay(100);
}
```

## Example 5: Waypoint Navigation

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);

// Target waypoint (example: Central Park, NY)
const double TARGET_LAT = 40.7829;
const double TARGET_LON = -73.9654;

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);

  Serial.println("Waypoint Navigation");
  Serial.println("==================");
  Serial.print("Target: ");
  Serial.print(TARGET_LAT, 6);
  Serial.print(", ");
  Serial.println(TARGET_LON, 6);
}

void navigateToWaypoint() {
  if (!gps.location.isValid()) return;

  // Calculate distance to target
  double distance = gps.distanceBetween(
    gps.location.lat(),
    gps.location.lng(),
    TARGET_LAT,
    TARGET_LON
  );

  // Calculate course to target
  double courseTo = gps.courseTo(
    gps.location.lat(),
    gps.location.lng(),
    TARGET_LAT,
    TARGET_LON
  );

  // Get current course
  double currentCourse = gps.course.deg();

  // Calculate heading difference
  double headingDiff = courseTo - currentCourse;
  if (headingDiff < 0) headingDiff += 360;
  if (headingDiff > 360) headingDiff -= 360;

  Serial.println("=== Navigation Update ===");
  Serial.print("Distance to target: ");
  Serial.print(distance);
  Serial.println(" meters");

  Serial.print("Target course: ");
  Serial.print(courseTo);
  Serial.println(" degrees");

  Serial.print("Your course: ");
  Serial.print(currentCourse);
  Serial.println(" degrees");

  // Steering direction
  Serial.print("Steer: ");
  if (headingDiff < 10 || headingDiff > 350) {
    Serial.println("Straight");
  } else if (headingDiff < 180) {
    Serial.print("RIGHT ");
    Serial.print((int)headingDiff);
    Serial.println(" degrees");
  } else {
    Serial.print("LEFT ");
    Serial.print((int)(360 - headingDiff));
    Serial.println(" degrees");
  }

  // Arrival check
  if (distance < 10) {
    Serial.println("=== ARRIVED AT WAYPOINT ===");
  }

  Serial.println();
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  if (gps.location.isUpdated()) {
    navigateToWaypoint();
  }

  delay(1000);
}
```

## Example 6: Geofence Alert

```cpp
#include <TinyGPS++.h>
#include <HardwareSerial.h>

TinyGPSPlus gps;
HardwareSerial GPSSerial(1);

// Geofence parameters
const double FENCE_LAT = 40.7580;  // Times Square, NY
const double FENCE_LON = -73.9855;
const double FENCE_RADIUS = 100.0;  // 100 meters
bool insideFence = false;

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);

  Serial.println("Geofence Monitor");
  Serial.print("Fence center: ");
  Serial.print(FENCE_LAT, 6);
  Serial.print(", ");
  Serial.println(FENCE_LON, 6);
  Serial.print("Fence radius: ");
  Serial.print(FENCE_RADIUS);
  Serial.println(" meters");
}

void checkGeofence() {
  if (!gps.location.isValid()) return;

  double distance = gps.distanceBetween(
    gps.location.lat(),
    gps.location.lng(),
    FENCE_LAT,
    FENCE_LON
  );

  bool currentlyInside = (distance <= FENCE_RADIUS);

  // Check for state change
  if (currentlyInside != insideFence) {
    insideFence = currentlyInside;

    if (insideFence) {
      Serial.println("=== ENTERED GEOFENCE ===");
    } else {
      Serial.println("=== EXITED GEOFENCE ===");
    }
  }

  // Display status
  Serial.print("Distance from fence: ");
  Serial.print(distance);
  Serial.print("m | Status: ");
  Serial.println(insideFence ? "INSIDE" : "OUTSIDE");
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    gps.encode(c);
  }

  checkGeofence();
  delay(1000);
}
```

## Example 7: NMEA Sentence Viewer

```cpp
#include <HardwareSerial.h>

HardwareSerial GPSSerial(1);

void setup() {
  Serial.begin(115200);
  GPSSerial.begin(9600);

  Serial.println("Raw NMEA Output");
  Serial.println("=================");
}

void loop() {
  while (GPSSerial.available() > 0) {
    char c = GPSSerial.read();
    Serial.write(c);  // Echo to USB serial
  }
}
```

## GPS Antenna Tips

1. **Outdoor Use Only:** GPS requires clear sky view
2. **Away from Buildings:** Stay away from tall structures
3. **Open Area:** Best reception in open fields
4. **Antenna Orientation:** Point antenna upward
5. **Cold Start:** First fix may take 30-60 seconds

## Troubleshooting

### No GPS Fix

**Symptom:** Cannot acquire satellites

**Possible Causes:**
1. Indoor use - Must have clear sky view
2. Antenna issue - Check antenna connection
3. Cold start - First fix takes 30+ seconds

**Solution:** Move outdoors, wait for cold start

### Inaccurate Position

**Symptom:** Position jumps around

**Possible Causes:**
1. Poor satellite geometry (low HDOP)
2. Multipath interference
3. Low satellite count

**Solution:** Check HDOP value (>5 is poor), move to open area

### Slow Update Rate

**Symptom:** Position updates slowly

**Solution:** Default is 1Hz, can configure up to 10Hz (requires specific commands)

## See Also

- **GPS Basics:** `/xiao-arduino/references/expansion-boards/gps-gnss.md`
- **Hardware Docs:** `/xiao/references/expansion_boards/gps-gnss.md`
