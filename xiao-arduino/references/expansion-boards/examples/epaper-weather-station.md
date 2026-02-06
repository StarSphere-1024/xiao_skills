# ePaper Weather Station Examples - Arduino

## Overview

Complete Arduino examples for building an ePaper weather station using the XIAO ePaper V2 Expansion Board. Examples demonstrate displaying temperature, humidity, weather icons, and battery status on ePaper displays.

**Hardware Reference:** `/xiao/references/expansion_boards/epaper-v2.md`
**Arduino Implementation:** `/xiao-arduino/references/expansion-boards/epaper-v2.md`

## Required Libraries

Install via Arduino Library Manager:
- **GxEPD2** - ePaper display driver
- **Adafruit BME280** - Temperature/humidity/pressure sensor
- **Wire** - Built-in I2C library

## Hardware Setup

**Components:**
- XIAO board (ESP32S3 recommended for more RAM)
- ePaper V2 Expansion Board
- BME280 sensor (I2C on D4/D5)
- Optional: Battery voltage divider on A0

## Example 1: Basic Weather Display (2.9" Display)

```cpp
#include <GxEPD2_BW.h>
#include <Adafruit_BME280.h>
#include <Wire.h>

// 2.9" ePaper display
GxEPD2_BW<GxEPD2_290_T94, GxEPD2_290_T94::HEIGHT>
  display(GxEPD2_290_T94(D1, D3, D0, D2, D8, D10));

Adafruit_BME280 bme;

struct WeatherData {
  float temperature;
  float humidity;
  float pressure;
};

void setup() {
  Serial.begin(115200);
  Wire.begin();

  // Initialize BME280
  if (!bme.begin(0x76)) {
    Serial.println("BME280 not found!");
    while (1);
  }

  // Initialize display
  display.init();
  display.setRotation(1);
}

WeatherData readWeather() {
  WeatherData data;
  data.temperature = bme.readTemperature();
  data.humidity = bme.readHumidity();
  data.pressure = bme.readPressure() / 100.0;  // hPa
  return data;
}

void drawWeatherDisplay() {
  WeatherData data = readWeather();

  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);

    // Title
    display.setFont(&FreeSans9pt7b);
    display.setCursor(80, 20);
    display.print("Weather Station");

    // Temperature
    display.setFont(&FreeSans18pt7b);
    display.setCursor(20, 80);
    display.print(data.temperature, 1);
    display.setFont(&FreeSans9pt7b);
    display.print(" C");

    // Humidity
    display.setFont(&FreeSans18pt7b);
    display.setCursor(20, 130);
    display.print(data.humidity, 0);
    display.setFont(&FreeSans9pt7b);
    display.print(" %");

    // Pressure
    display.setFont(&FreeSans9pt7b);
    display.setCursor(20, 170);
    display.print("Pressure: ");
    display.print(data.pressure, 0);
    display.print(" hPa");

    // Draw separator line
    display.drawLine(10, 190, 270, 190, GxEPD_BLACK);

    // Timestamp
    display.setFont(&FreeSans9pt7b);
    display.setCursor(20, 220);
    display.print("Updated: ");
    display.print(millis() / 1000);
    display.print("s");

  } while (display.nextPage());
}

void loop() {
  drawWeatherDisplay();
  delay(60000);  // Update every minute
}
```

## Example 2: Weather Icons

```cpp
#include <GxEPD2_BW.h>
#include <Adafruit_BME280.h>

GxEPD2_BW<GxEPD2_213_BN, GxEPD2_213_BN::HEIGHT>
  display(GxEPD2_213_BN(D1, D3, D0, D2, D8, D10));

Adafruit_BME280 bme;

// Simple bitmap icons (8x8 pixels for demo)
const unsigned char icon_sun[] = {
  0x3C, 0x42, 0x99, 0xA5, 0xA5, 0x99, 0x42, 0x3C
};

const unsigned char icon_cloud[] = {
  0x3C, 0x7E, 0xFF, 0xFF, 0xFF, 0x7E, 0x3C, 0x00
};

const unsigned char icon_rain[] = {
  0x3C, 0x7E, 0xFF, 0xFF, 0x7E, 0x24, 0x24, 0x24
};

void setup() {
  display.init();
  display.setRotation(1);
  Wire.begin();
  bme.begin(0x76);
}

void drawWeatherIcon(float temp, float humidity) {
  // Select icon based on weather
  const unsigned char* icon;

  if (humidity > 70) {
    icon = icon_rain;  // Rain
  } else if (humidity > 50) {
    icon = icon_cloud;  // Cloud
  } else {
    icon = icon_sun;  // Sun
  }

  // Draw icon (scaled up)
  for (int y = 0; y < 8; y++) {
    for (int x = 0; x < 8; x++) {
      if (icon[y] & (1 << (7 - x))) {
        display.fillRect(10 + x * 4, 50 + y * 4, 4, 4, GxEPD_BLACK);
      }
    }
  }
}

void loop() {
  float temp = bme.readTemperature();
  float hum = bme.readHumidity();

  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);

    drawWeatherIcon(temp, hum);

    display.setCursor(60, 60);
    display.print(temp, 1);
    display.print("C");

    display.setCursor(60, 90);
    display.print(hum, 0);
    display.print("%");

  } while (display.nextPage());

  delay(60000);
}
```

## Example 3: Multi-Day Forecast

```cpp
#include <GxEPD2_BW.h>
#include <Adafruit_BME280.h>

GxEPD2_BW<GxEPD2_290_T94, GxEPD2_290_T94::HEIGHT>
  display(GxEPD2_290_T94(D1, D3, D0, D2, D8, D10));

Adafruit_BME280 bme;

struct DayForecast {
  float high;
  float low;
  float humidity;
  int condition;  // 0=sun, 1=cloud, 2=rain
};

// Simulated forecast data
DayForecast forecast[3];

void setup() {
  display.init();
  display.setRotation(1);
  Wire.begin();
  bme.begin(0x76);

  // Initialize with current readings + variations
  float current = bme.readTemperature();
  for (int i = 0; i < 3; i++) {
    forecast[i].high = current + random(-5, 5);
    forecast[i].low = current - random(5, 15);
    forecast[i].humidity = bme.readHumidity() + random(-10, 10);
    forecast[i].condition = random(0, 3);
  }
}

void drawForecast() {
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setFont(&FreeSans9pt7b);

    // Header
    display.setCursor(80, 20);
    display.print("3-Day Forecast");

    const char* days[] = {"Today", "Tmrw", "Day 3"};
    const char* icons[] = {"*", "o", "="};  // Sun, Cloud, Rain

    for (int i = 0; i < 3; i++) {
      int x = 10 + i * 90;
      int y = 50;

      // Day name
      display.setCursor(x, y);
      display.print(days[i]);

      // Icon
      display.setCursor(x + 30, y + 25);
      display.print(icons[forecast[i].condition]);

      // High/Low
      display.setCursor(x, y + 50);
      display.print("H:");
      display.print((int)forecast[i].high);
      display.print(" L:");
      display.print((int)forecast[i].low);

      // Humidity
      display.setCursor(x, y + 75);
      display.print((int)forecast[i].humidity);
      display.print("%");
    }

    // Current conditions
    display.setCursor(10, 170);
    display.print("Current: ");
    display.print(bme.readTemperature(), 1);
    display.print("C / ");
    display.print((int)bme.readHumidity());
    display.print("%");

  } while (display.nextPage());
}

void loop() {
  drawForecast();
  delay(3600000);  // Update every hour
}
```

## Example 4: Battery Monitor

```cpp
#include <GxEPD2_BW.h>

GxEPD2_BW<GxEPD2_213_BN, GxEPD2_213_BN::HEIGHT>
  display(GxEPD2_213_BN(D1, D3, D0, D2, D8, D10));

const int batteryPin = A0;  // Voltage divider on A0

// Calibration values
const float R1 = 100000.0;  // 100k
const float R2 = 100000.0;  // 100k
const float VREF = 3.3;

float readBattery() {
  int adc = analogRead(batteryPin);
  float voltage = (adc * VREF) / 4095.0;
  // Reverse voltage divider
  float batteryVoltage = voltage * ((R1 + R2) / R2);
  return batteryVoltage;
}

int getBatteryPercent(float voltage) {
  // Approximate LiPo discharge curve
  if (voltage >= 4.1) return 100;
  if (voltage >= 3.9) return 80 + (voltage - 3.9) * 100;
  if (voltage >= 3.7) return 60 + (voltage - 3.7) * 100;
  if (voltage >= 3.5) return 30 + (voltage - 3.5) * 150;
  if (voltage >= 3.3) return 10 + (voltage - 3.3) * 100;
  return 0;
}

void drawBatteryIcon(int x, int y, int percent) {
  // Battery outline
  display.drawRect(x, y, 30, 15, GxEPD_BLACK);
  display.fillRect(x + 30, y + 4, 3, 7, GxEPD_BLACK);

  // Fill level
  int fillWidth = (percent * 28) / 100;
  if (fillWidth > 0) {
    display.fillRect(x + 1, y + 1, fillWidth, 13, GxEPD_BLACK);
  }

  // Percentage text
  display.setCursor(x + 40, y + 12);
  display.print(percent);
  display.print("%");
}

void loop() {
  float batteryVoltage = readBattery();
  int batteryPercent = getBatteryPercent(batteryVoltage);

  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setFont(&FreeSans9pt7b);

    display.setCursor(50, 30);
    display.print("Battery Monitor");

    drawBatteryIcon(60, 60, batteryPercent);

    display.setCursor(20, 100);
    display.print("Voltage: ");
    display.print(batteryVoltage, 2);
    display.print("V");

    // Status message
    display.setCursor(20, 130);
    if (batteryPercent > 50) {
      display.print("Status: Good");
    } else if (batteryPercent > 20) {
      display.print("Status: OK");
    } else {
      display.print("Status: LOW!");
    }

  } while (display.nextPage());

  delay(60000);
}
```

## Example 5: Partial Update for Fast Refresh

```cpp
#include <GxEPD2_BW.h>

GxEPD2_BW<GxEPD2_213_BN, GxEPD2_213_BN::HEIGHT>
  display(GxEPD2_213_BN(D1, D3, D0, D2, D8, D10));

unsigned long lastFullRefresh = 0;

void setup() {
  display.init();
  display.setRotation(1);
}

void loop() {
  // Full refresh every 5 minutes
  if (millis() - lastFullRefresh > 300000) {
    display.firstPage();
    do {
      display.fillScreen(GxEPD_WHITE);
      display.setTextColor(GxEPD_BLACK);
      display.setFont(&FreeSans9pt7b);
      display.setCursor(20, 50);
      display.print("Full Refresh");
    } while (display.nextPage());
    lastFullRefresh = millis();
  }

  // Partial update for counter
  static int counter = 0;
  counter++;

  display.setPartialWindow(0, 0, display.width(), 30);
  display.firstPage();
  do {
    display.fillRect(0, 0, display.width(), 30, GxEPD_WHITE);
    display.setCursor(20, 20);
    display.print("Count: ");
    display.print(counter);
  } while (display.nextPage());

  delay(1000);
}
```

## Low Power Deep Sleep (ESP32 Only)

```cpp
#include <GxEPD2_BW.h>
#include <esp_sleep.h>

GxEPD2_BW<GxEPD2_213_BN, GxEPD2_213_BN::HEIGHT>
  display(GxEPD2_213_BN(D1, D3, D0, D2, D8, D10));

void setup() {
  display.init();

  // Update display once
  display.firstPage();
  do {
    display.fillScreen(GxEPD_WHITE);
    display.setTextColor(GxEPD_BLACK);
    display.setCursor(20, 50);
    display.print("Sleeping...");
    display.setCursor(20, 80);
    display.print("Wake in 5 min");
  } while (display.nextPage());

  // Enter deep sleep (ePaper retains image)
  esp_sleep_enable_timer_wakeup(300 * 1000000);  // 5 minutes
  esp_deep_sleep_start();
}

void loop() {}
```

## Troubleshooting

### Display Not Updating

**Symptom:** Display shows old content

**Solution:** Wait for full refresh cycle (2-15 seconds depending on size)

### RAM Overflow on SAMD21

**Symptom:** Compilation fails or sketch too large

**Solution:** Use ESP32S3 or RP2040 for larger displays (4.26", 5.65", 7.5")

### Ghosting After Multiple Updates

**Symptom:** Previous image faintly visible

**Solution:** Run a full refresh cycle to clear ghosting

## See Also

- **ePaper Basics:** `/xiao-arduino/references/expansion-boards/epaper-v2.md`
- **Hardware Docs:** `/xiao/references/expansion_boards/epaper-v2.md`
