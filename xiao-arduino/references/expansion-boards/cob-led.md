# COB LED Driver Board for XIAO - Arduino

## Overview

Arduino implementation for the XIAO COB LED Driver Board. Controls high-brightness COB (Chip-on-Board) LED modules with PWM-based dimming and color mixing.

## Hardware Reference

For hardware specifications, pin connections, and compatibility, see `/xiao/references/expansion_boards/cob-led.md`.

## Pin Connections

| Function | XIAO Pin | Notes |
|----------|----------|-------|
| PWM Red | D0 or D1 | PWM output |
| PWM Green | D2 or D3 | PWM output |
| PWM Blue | D6 or D7 | PWM output |
| Enable | D8 or similar | On/off control |

**Important:** COB LEDs require significant current (1A+). Always use external power supply with adequate capacity.

## Power Requirements

**Critical:** COB LEDs require significant power:

| COB Size | Current | Voltage |
|----------|---------|---------|
| Small | ~0.5A | 12V |
| Medium | ~1A | 12V or 24V |
| Large | ~2A+ | 24V or 36V |

**Power Supply:**
- Use external power supply adequate for COB module
- Connect power directly to COB LED
- DO NOT power through XIAO

## Basic Single-Color COB Control

```cpp
#define COB_PIN D0  // PWM pin

void setup() {
  pinMode(COB_PIN, OUTPUT);
}

void loop() {
  // Fade in
  for (int i = 0; i <= 255; i++) {
    analogWrite(COB_PIN, i);
    delay(10);
  }

  // Fade out
  for (int i = 255; i >= 0; i--) {
    analogWrite(COB_PIN, i);
    delay(10);
  }
}
```

## RGB COB Color Mixing

```cpp
#define RED_PIN   D0
#define GREEN_PIN D2
#define BLUE_PIN  D6

void setup() {
  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);
}

void setCOBColor(int red, int green, int blue) {
  analogWrite(RED_PIN, red);
  analogWrite(GREEN_PIN, green);
  analogWrite(BLUE_PIN, blue);
}

void loop() {
  // Red
  setCOBColor(255, 0, 0);
  delay(1000);

  // Green
  setCOBColor(0, 255, 0);
  delay(1000);

  // Blue
  setCOBColor(0, 0, 255);
  delay(1000);

  // White
  setCOBColor(255, 255, 255);
  delay(1000);

  // Yellow
  setCOBColor(255, 255, 0);
  delay(1000);

  // Cyan
  setCOBColor(0, 255, 255);
  delay(1000);

  // Magenta
  setCOBColor(255, 0, 255);
  delay(1000);
}
```

## Color Fading

```cpp
#define RED_PIN   D0
#define GREEN_PIN D2
#define BLUE_PIN  D6

void setup() {
  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);
}

void loop() {
  // Fade through rainbow colors
  for (int hue = 0; hue < 360; hue++) {
    int r, g, b;

    // Convert HSV to RGB
    if (hue < 60) {
      r = 255;
      g = hue * 4.25;
      b = 0;
    } else if (hue < 120) {
      r = (120 - hue) * 4.25;
      g = 255;
      b = 0;
    } else if (hue < 180) {
      r = 0;
      g = 255;
      b = (hue - 120) * 4.25;
    } else if (hue < 240) {
      r = 0;
      g = (240 - hue) * 4.25;
      b = 255;
    } else if (hue < 300) {
      r = (hue - 240) * 4.25;
      g = 0;
      b = 255;
    } else {
      r = 255;
      g = 0;
      b = (360 - hue) * 4.25;
    }

    analogWrite(RED_PIN, r);
    analogWrite(GREEN_PIN, g);
    analogWrite(BLUE_PIN, b);

    delay(20);
  }
}
```

## Warm to Cool White Adjustment

```cpp
#define RED_PIN   D0
#define GREEN_PIN D2
#define BLUE_PIN  D6

void setup() {
  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);
}

void setWhiteTemperature(int kelvin) {
  int r, g, b;

  // Convert Kelvin to RGB (simplified)
  if (kelvin <= 6600) {
    r = 255;
    g = 99.47 * log(kelvin / 100.0) - 161.12;
    b = 0;
  } else {
    r = 329.7 * pow(kelvin / 100.0 - 60, -0.133);
    g = 288.12 * pow(kelvin / 100.0 - 60, -0.0755);
    b = 255;
  }

  // Clamp values
  r = constrain(r, 0, 255);
  g = constrain(g, 0, 255);
  b = constrain(b, 0, 255);

  analogWrite(RED_PIN, r);
  analogWrite(GREEN_PIN, g);
  analogWrite(BLUE_PIN, b);
}

void loop() {
  // Fade from warm (2700K) to cool (6500K)
  for (int k = 2700; k <= 6500; k += 100) {
    setWhiteTemperature(k);
    delay(100);
  }

  delay(1000);

  // Fade back
  for (int k = 6500; k >= 2700; k -= 100) {
    setWhiteTemperature(k);
    delay(100);
  }

  delay(1000);
}
```

## PWM Frequency Control

For ESP32 series, you can control PWM frequency:

```cpp
// ESP32 only
#define RED_PIN   D0
#define GREEN_PIN D2
#define BLUE_PIN  D6
#define PWM_FREQ  5000  // 5kHz PWM
#define PWM_RES   8     // 8-bit resolution

void setup() {
  // Configure LEDC for each pin (channel is selected automatically)
  ledcAttach(RED_PIN, PWM_FREQ, PWM_RES);
  ledcAttach(GREEN_PIN, PWM_FREQ, PWM_RES);
  ledcAttach(BLUE_PIN, PWM_FREQ, PWM_RES);
}

void setCOBColor(int red, int green, int blue) {
  ledcWrite(RED_PIN, red);
  ledcWrite(GREEN_PIN, green);
  ledcWrite(BLUE_PIN, blue);
}

void loop() {
  setCOBColor(255, 255, 255);
  delay(1000);
}
```

## Thermal Management

```cpp
#define RED_PIN   D0
#define GREEN_PIN D2
#define BLUE_PIN  D6
#define THERM_PIN A0  // Thermistor on A0

void setup() {
  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);
  Serial.begin(115200);
}

float getTemperature() {
  int raw = analogRead(THERM_PIN);
  // Convert to Celsius (example for 10K NTC)
  float resistance = 10000.0 * ((4095.0 / raw) - 1);
  float temp = 1.0 / (log(resistance / 10000.0) / 3950.0 + 1.0 / 298.15) - 273.15;
  return temp;
}

void loop() {
  float temp = getTemperature();

  Serial.print("Temperature: ");
  Serial.print(temp);
  Serial.println(" C");

  if (temp > 60.0) {
    // Reduce brightness if too hot
    analogWrite(RED_PIN, 127);
    analogWrite(GREEN_PIN, 127);
    analogWrite(BLUE_PIN, 127);
    Serial.println("Thermal throttling active");
  } else {
    // Normal brightness
    analogWrite(RED_PIN, 255);
    analogWrite(GREEN_PIN, 255);
    analogWrite(BLUE_PIN, 255);
  }

  delay(1000);
}
```

## Troubleshooting

### COB Not Lighting

**Symptom:** COB LED remains dark

**Possible Causes:**
1. No external power
2. Wrong PWM pins
3. Enable not set
4. Driver not configured

**Solution:**
- Verify external power supply connected
- Check pin connections
- Ensure PWM initialized correctly

### Poor Color Mixing

**Symptom:** Uneven color distribution

**Possible Causes:**
1. Wrong PWM values
2. Low PWM frequency
3. COB quality

**Solution:**
- Adjust RGB values for calibration
- Increase PWM frequency to 500Hz+
- Verify COB module quality

### COB Overheating

**Symptom:** COB gets very hot

**Possible Causes:**
1. No heat sink
2. Overdriven
3. Poor ventilation

**Solution:**
- Add adequate heat sinking
- Reduce current/brightness
- Improve cooling/airflow

### Flickering

**Symptom:** Noticeable flicker

**Possible Causes:**
1. PWM frequency too low
2. Duty cycle issues
3. Power supply issue

**Solution:**
- Increase PWM frequency to 500Hz+
- Check PWM configuration
- Verify stable power supply

## Safety Considerations

1. **Heat:** COB LEDs get HOT - provide adequate heatsinking
2. **Current:** High currents - use appropriate wire gauge
3. **Voltage:** Match power supply to COB requirements
4. **Eye Safety:** Bright COB can damage eyes - avoid direct viewing
5. **Power Off:** Allow cooling before handling

## COB LED Types

**Single Color:**
- White (various color temperatures)
- Warm White (2700-3000K)
- Cool White (4000-6500K)

**RGB COB:**
- Full color mixing
- Adjustable white balance
- Wide color gamut
