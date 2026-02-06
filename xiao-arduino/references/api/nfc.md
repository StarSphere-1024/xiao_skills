# NFC API (Near Field Communication)

## Overview

XIAO nRF52840 and nRF52840 Sense boards feature built-in NFC (Near Field Communication) capability.

**Compatible Boards:**
- XIAO nRF52840
- XIAO nRF52840 Sense

**NFC Hardware:**
- Frequency: 13.56 MHz
- Interface: P0.09 (NFC1), P0.10 (NFC2)
- Mode: NFC Forum Type 2 Tag / NDEF format
- Requires: External NFC antenna (sold separately)

## Hardware Setup

### NFC Antenna Connection

The NFC antenna must be soldered to the back of the XIAO nRF52840:

| NFC Pad | nRF Pin | Description |
|---------|---------|-------------|
| NFC1 | P0.09 | Differential pair A |
| NFC2 | P0.10 | Differential pair B |

**Important Notes:**
- Both pads must be connected (cannot use just one)
- Differential interface (phase matters)
- Bare pins - wear anti-static wrist strap when soldering
- Antenna impedance should be 100 Ω differential at 13.56 MHz

### Antenna Specifications

| Parameter | Value |
|-----------|-------|
| Frequency | 13.56 MHz (fixed) |
| Target Load | 100 Ω differential |
| Tuning Capacitors | C1 = C2 = C/2 (typically 39-68 pF) |

**Resonance Formula:**
```
C = 1 / (4 * π² * f² * L)
C1 = C2 = C / 2
```

Where:
- f = 13.56 MHz (fixed)
- L = coil inductance (measure with LCR meter)

## Basic Usage

### Simple Text Message

```cpp
#include <NFCT.h>

void setup() {
    // Set NFC message
    NFC.setTXTmessage("Hello World!", "en");
    // Start NFC module
    NFC.start();
    Serial.println("NFC active - bring phone close");
}

void loop() {
    // NFC runs continuously
}
```

### URI/URL Message

```cpp
#include <NFCT.h>

void setup() {
    // Set NFC URL message
    NFC.setURImessage("https://www.seeedstudio.com");
    NFC.start();
    Serial.println("NFC URL ready");
}

void loop() {
}
```

## API Reference

### Text Message

```cpp
void setTXTmessage(const char *message, const char *lang)
```

Set a plain text NDEF message.

**Parameters:**
- `message`: Text string to store
- `lang`: Language code (e.g., "en" for English)

**Example:**
```cpp
NFC.setTXTmessage("Hello from XIAO!", "en");
```

### URI Message

```cpp
void setURImessage(const char *uri)
```

Set a URL/URI NDEF message.

**Parameters:**
- `uri`: URL string (e.g., "https://example.com")

**Example:**
```cpp
NFC.setURImessage("https://wiki.seeedstudio.com");
```

### Start NFC

```cpp
void start(void)
```

Enable NFC and start transmitting NDEF message.

**Example:**
```cpp
NFC.setTXTmessage("Hello!", "en");
NFC.start();
```

### Stop NFC

```cpp
void stop(void)
```

Disable NFC module.

**Example:**
```cpp
NFC.stop();
```

### Clear Message

```cpp
void clear(void)
```

Clear the current NDEF message.

**Example:**
```cpp
NFC.clear();
```

## Advanced Usage

### Dynamic Message Update

```cpp
#include <NFCT.h>

int counter = 0;

void setup() {
    NFC.start();
    Serial.begin(115200);
}

void loop() {
    // Update NFC message every 5 seconds
    static unsigned long lastUpdate = 0;
    if (millis() - lastUpdate > 5000) {
        lastUpdate = millis();

        String message = "Count: " + String(counter++);
        NFC.setTXTmessage(message.c_str(), "en");

        Serial.print("NFC updated: ");
        Serial.println(message);
    }
}
```

### Sensor Data in NFC

```cpp
#include <NFCT.h>
#include <Adafruit_BME280.h>

Adafruit_BME280 bme;

void setup() {
    Wire.begin();
    bme.begin(0x76);
    NFC.start();
}

void loop() {
    // Read sensor
    float temp = bme.readTemperature();
    float humi = bme.readHumidity();

    // Create message
    String data = String(temp, 1) + "C, " + String(humi, 0) + "%";
    NFC.setTXTmessage(data.c_str(), "en");

    delay(5000);
}
```

### JSON Data in NFC

```cpp
#include <NFCT.h>
#include <ArduinoJson.h>

void setup() {
    NFC.start();
}

void loop() {
    StaticJsonDocument<128> doc;
    doc["device"] = "XIAO";
    doc["status"] = "online";
    doc["battery"] = 85;

    String output;
    serializeJson(doc, output);

    NFC.setTXTmessage(output.c_str(), "en");
    delay(10000);
}
```

## Antenna Tuning

### Pre-Check: Is Coil Size OK?

Calculate expected impedance:

```
Z = 2 * π * f * L
```

Where:
- f = 13.56 MHz
- L = coil inductance in µH

| Result | Meaning | Action |
|--------|---------|--------|
| < 60 Ω | Antenna too small | Add turns or enlarge area |
| 90-120 Ω | **PASS** | Proceed to calculate capacitors |
| > 150 Ω | Antenna too big | Remove turns or shrink area |

### Calculate Resonant Capacitance

For f = 13.56 MHz:

```
C = 1 / (4 * π² * f² * L)
C1 = C2 = C / 2
```

**Example:**
```
If L = 2.2 µH:
C = 1 / (4 * π² * (13.56×10⁶)² * 2.2×10⁻⁶)
C ≈ 62.5 pF
C1 = C2 = 31.25 pF → Use 33 pF (closest E12 value)
```

### Recommended Capacitor Values

| Inductance | Total C | C1, C2 (use) |
|------------|---------|--------------|
| 1.8 µH | 77 pF | 39 pF |
| 2.0 µH | 69 pF | 33 pF or 39 pF |
| 2.2 µH | 63 pF | 33 pF |
| 2.5 µH | 55 pF | 27 pF |
| 3.0 µH | 46 pF | 22 pF or 27 pF |

## Testing NFC

### Using NFC TagInfo App

1. Download NFC TagInfo app:
   - Android: [NXP NFC TagInfo](https://play.google.com/store/apps/details?id=com.nxp.taginfolite)
   - iOS: [NFC TagInfo](https://apps.apple.com/us/app/nfc-taginfo-by-nxp/id1246143596)

2. Upload NFC sketch to XIAO

3. Open app and tap "Scan & Launch"

4. Bring phone close to NFC antenna

5. Should display NDEF message

### Expected Output

With text message "Hello World!":
```
NFC Forum Type 2 Tag
NDEF Record:
- Type: Text (UTF-8)
- Language: en
- Payload: Hello World!
```

## Troubleshooting

### NFC Not Detected

1. **Check antenna connection**: Both NFC1 and NFC2 must be soldered
2. **Check antenna impedance**: Should be ~100 Ω at 13.56 MHz
3. **Check tuning capacitors**: Verify C1 and C2 values
4. **Test with different phone**: Some phones have weaker NFC

### Wrong UID After Swapping Pads

If you swap NFC1 and NFC2, the NFC will work but the UID will change.

**Solution**: Resolder antenna with correct polarity

### Library Not Found

Ensure you're using the recommended library:
- **Recommended**: "Seeed nRF52 mbed-enabled Boards" version 2.9.2 or higher
- **Also works**: "Seeed nRF52 Boards" version 1.1.3

Install via:
`Sketch > Include Library > Manage Libraries > Search "NFCT"`

### Message Not Updating

If dynamic updates aren't working:
1. Ensure `NFC.start()` is called in setup()
2. Move phone away and back between reads
3. Add small delay after `setTXTmessage()`

## Examples

### Device Identification Tag

```cpp
#include <NFCT.h>

void setup() {
    // Set device info
    NFC.setTXTmessage(
        "XIAO nRF52840\n"
        "FW: 1.0.2\n"
        "ID: XIAO-001"
    );
    NFC.start();
}

void loop() {
}
```

### WiFi Credentials Sharing

```cpp
#include <NFCT.h>

void setup() {
    // Share WiFi credentials (read-only)
    NFC.setTXTmessage(
        "WiFi: MyNetwork\n"
        "Pass: password123"
    );
    NFC.start();
    Serial.println("Scan to get WiFi credentials");
}

void loop() {
}
```

### Dynamic Status Display

```cpp
#include <NFCT.h>

enum Status { OK, WARNING, ERROR };
Status currentStatus = OK;

void setup() {
    NFC.start();
    Serial.begin(115200);
}

void loop() {
    // Update status based on some condition
    if (someErrorCondition()) {
        currentStatus = ERROR;
    } else if (someWarningCondition()) {
        currentStatus = WARNING;
    } else {
        currentStatus = OK;
    }

    // Update NFC message
    switch (currentStatus) {
        case OK:
            NFC.setTXTmessage("Status: OK", "en");
            break;
        case WARNING:
            NFC.setTXTmessage("Status: WARNING", "en");
            break;
        case ERROR:
            NFC.setTXTmessage("Status: ERROR", "en");
            break;
    }

    delay(1000);
}

bool someErrorCondition() {
    // Your error detection logic
    return false;
}

bool someWarningCondition() {
    // Your warning detection logic
    return false;
}
```

## Resources

- [Nordic NFC Antenna Design (nWP_026)](https://docs.nordicsemi.com/bundle/nwp_026/page/WP/nwp_026/nWP_026_intro.html)
- [NFC Forum Type 2 Tag Specification](https://nfc-forum.org/specifications/)
- [NXP NFC TagInfo Apps](https://www.nxp.com/design/software/design-tools:NCIP-TRANSPORTATION-SOFTWARE)
