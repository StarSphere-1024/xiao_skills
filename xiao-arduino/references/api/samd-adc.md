# SAMD21 ADC APIs for XIAO

## Overview

XIAO SAMD21 (Seeed XIAO M0) uses the Atmel SAMD21G18 microcontroller with a 12-bit ADC (0-4095) supporting up to 350ksps.

## ADC Characteristics

### Specifications

```cpp
// SAMD21 ADC specifications
// - Resolution: 12-bit (0-4095)
// - Max sample rate: 350ksps
// - Input voltage: 0-3.3V
// - Reference options: Internal 1.0V, 1.65V, 2.23V, or external AREF
// - Analog pins: A0-A5 (D0-D5)
```

### Pin Mapping

| Analog Pin | Digital Pin | Description |
|------------|-------------|-------------|
| A0 | D0 | ADC0/AIN0 |
| A1 | D1 | ADC1/AIN1 |
| A2 | D2 | ADC2/AIN2 |
| A3 | D3 | ADC3/AIN3 |
| A4 | D4 | ADC4/AIN4 |
| A5 | D5 | ADC5/AIN5 |

## Basic Analog Read

```cpp
const int ADC_PIN = A0;  // D0

void setup() {
    Serial.begin(115200);
    analogReference(AR_DEFAULT);  // 3.3V default
}

void loop() {
    int rawValue = analogRead(ADC_PIN);  // 0-4095
    float voltage = rawValue * 3.3 / 4095.0;

    Serial.printf("Raw: %d, Voltage: %.2fV\n", rawValue, voltage);
    delay(1000);
}
```

## ADC Resolution

### Change Resolution

```cpp
void setup() {
    // Default is 12-bit (0-4095)
    // Can be set to 10-bit (0-1023) for faster reads
    analogReadResolution(12);  // 12-bit (default)

    // Other options: 8, 10, or 12 bits
}

void loop() {
    int value = analogRead(A0);
    // value range depends on resolution setting
}
```

## ADC Reference Voltage

### Reference Options

```cpp
void setup() {
    Serial.begin(115200);

    // Reference voltage options:
    // AR_DEFAULT: 3.3V (default)
    // AR_INTERNAL1V0: Internal 1.0V reference
    // AR_INTERNAL1V65: Internal 1.65V reference
    // AR_INTERNAL2V23: Internal 2.23V reference
    // AR_EXTERNAL: AREF pin (external reference)
}

void readWithInternal1V() {
    analogReference(AR_INTERNAL1V0);
    delay(10);  // Allow reference to stabilize

    int value = analogRead(A0);
    float voltage = value * 1.0 / 4095.0;
    Serial.printf("Voltage: %.3fV\n", voltage);
}

void readWithExternalRef() {
    analogReference(AR_EXTERNAL);
    delay(10);

    // External reference voltage on AREF pin
    // Assuming 2.5V external reference
    float external_ref = 2.5;
    int value = analogRead(A0);
    float voltage = value * external_ref / 4095.0;
}
```

## ADC Averaging

### Hardware Averaging

```cpp
// SAMD21 supports hardware averaging
// Requires Zero DMA library or direct register access

void setup() {
    Serial.begin(115200);

    // Enable 16-sample hardware averaging
    ADC->CTRLB.bit.RESSEL = ADC_CTRLB_RESSEL_12BIT_Val;
    ADC->AVGCTRL.reg = ADC_AVGCTRL_SAMPLENUM_16;
}

void loop() {
    // Read with averaging enabled
    int sum = 0;
    for (int i = 0; i < 16; i++) {
        sum += analogRead(A0);
    }
    int average = sum / 16;

    Serial.printf("Averaged: %d\n", average);
    delay(1000);
}
```

## Differential ADC

### Differential Measurements

```cpp
// SAMD21 supports differential ADC inputs
// Measures difference between two pins

void setup() {
    Serial.begin(115200);

    // Configure for differential input
    // Positive input: A0, Negative input: A1
    ADC->INPUTCTRL.bit.MUXNEG = ADC_INPUTCTRL_MUXNEG_AIN1_Val;
}

void loop() {
    // Differential read (A0 - A1)
    int diffValue = analogRead(A0);
    float diffVoltage = diffValue * 3.3 / 4095.0;

    Serial.printf("Differential: %.2fV\n", diffVoltage);
    delay(1000);
}
```

## Temperature Sensor

### Internal Temperature

```cpp
// SAMD21 has internal temperature sensor
// Not directly exposed in Arduino API
// Requires register access

float readInternalTemperature() {
    // Enable temperature sensor
    ADC->INPUTCTRL.bit.MUXPOS = 0x18;  // Temp sensor

    // Start conversion
    ADC->SWTRIG.bit.START = 1;
    while (!ADC->INTFLAG.bit.RESRDY);

    // Read result
    int32_t raw = ADC->RESULT.reg;

    // Convert to temperature (simplified)
    // See SAMD21 datasheet for exact calibration
    float temp = (raw - 2000) / 10.0;  // Approximate
    return temp;
}

void setup() {
    Serial.begin(115200);
}

void loop() {
    float temp = readInternalTemperature();
    Serial.printf("Die temp: %.2f°C\n", temp);
    delay(1000);
}
```

## Low Power ADC

### Reduced Power Mode

```cpp
void setup() {
    Serial.begin(115200);

    // Reduce ADC clock for lower power
    // GCLK_GENDIV_DIV = 8 (slower ADC clock)
    GCLK->GENDIV.reg = GCLK_GENDIV_ID(2) | GCLK_GENDIV_DIV(8);
}

void loop() {
    // ADC will consume less power
    // Trade-off: slower conversion
    int value = analogRead(A0);
    Serial.println(value);
    delay(1000);
}
```

## Oversampling

### Software Oversampling

```cpp
// Increase effective resolution with oversampling
uint16_t oversampledRead(uint8_t pin, uint8_t bits) {
    // Oversample to achieve higher effective resolution
    // Each additional bit requires 4^n samples

    uint8_t extraBits = bits - 12;  // Target bits - 12-bit ADC
    uint32_t samples = 1 << (2 * extraBits);
    uint32_t sum = 0;

    for (uint32_t i = 0; i < samples; i++) {
        sum += analogRead(pin);
    }

    return sum >> extraBits;  // Right shift to average
}

void setup() {
    Serial.begin(115200);
    analogReadResolution(16);  // Set to 16-bit mode
}

void loop() {
    // 16-bit result requires 4^(16-12) = 256 samples
    uint16_t highRes = oversampledRead(A0, 16);
    float voltage = highRes * 3.3 / 65535.0;

    Serial.printf("High-res: %.4fV\n", voltage);
    delay(1000);
}
```

## Error Correction

### Offset and Gain Calibration

```cpp
// Calibrate ADC for offset and gain errors
// Requires reference voltage

int32_t readCalibratedADC(uint8_t pin) {
    // Calibration constants (determined empirically)
    const float OFFSET = 5.0;     // Offset correction
    const float GAIN = 1.02;      // Gain correction

    int32_t raw = analogRead(pin);
    float corrected = (raw + OFFSET) * GAIN;

    return (int32_t)constrain(corrected, 0, 4095);
}

void setup() {
    Serial.begin(115200);
}

void loop() {
    int32_t value = readCalibratedADC(A0);
    float voltage = value * 3.3 / 4095.0;

    Serial.printf("Calibrated: %.2fV\n", voltage);
    delay(1000);
}
```

## DMA ADC

### Automatic Sampling with DMA

```cpp
// Use DMA for high-speed ADC sampling
#include <Adafruit_ZeroDMA.h>

#define SAMPLE_COUNT 1000
uint16_t adc_buffer[SAMPLE_COUNT];

Adafruit_ZeroDMA dma;
ZeroDMAstatus dma_status;

void dma_callback(transfer_descriptor_t *desc) {
    // DMA transfer complete
    Serial.println("DMA transfer complete");
}

void setup() {
    Serial.begin(115200);

    // Configure DMA for ADC
    dma.setTrigger(ADC_DMAC_ID_RX);
    dma.setAction(DMA_TRIGGER_ACTON_BEAT);

    dma_status = dma.allocate();
    if (dma_status != DMA_STATUS_OK) {
        Serial.println("DMA allocation failed");
        while(1);
    }

    // Add transfer descriptor
    dma.addDescriptor(
        (void*)&ADC->RESULT.reg,  // Source: ADC result register
        adc_buffer,                // Destination: buffer
        SAMPLE_COUNT,              // Data count
        DMA_BEAT_SIZE_HWORD,       // 16-bit transfers
        true,                      // Increment destination
        false                      // Don't increment source
    );

    dma.setCallback(dma_callback);

    // Start continuous ADC conversion
    ADC->CTRLA.bit.ENABLE = 1;
    ADC->SWTRIG.bit.START = 1;

    // Start DMA
    dma.startJob();
}

void loop() {
    // Process adc_buffer when ready
    delay(1000);
}
```

## Best Practices

### ADC Accuracy Tips

```cpp
void optimizeADCReadings() {
    // 1. Use appropriate reference voltage
    analogReference(AR_DEFAULT);

    // 2. Allow settling time after changing reference
    delay(10);

    // 3. Discard first reading after reference change
    analogRead(A0);

    // 4. Use averaging for noise reduction
    int sum = 0;
    for (int i = 0; i < 4; i++) {
        sum += analogRead(A0);
    }
    int average = sum / 4;

    // 5. Use appropriate resolution
    analogReadResolution(12);  // Full 12-bit for maximum accuracy

    // 6. Avoid reading during high-frequency PWM
    // Disable interrupts if necessary
}
```

## Power Considerations

```cpp
void setup() {
    // ADC is enabled by default
    // Disable ADC when not in use to save power

    pinMode(A0, INPUT_DISABLE);  // Disable input buffer
}

void disableADC() {
    ADC->CTRLA.bit.ENABLE = 0;  // Disable ADC
}

void enableADC() {
    ADC->CTRLA.bit.ENABLE = 1;  // Enable ADC
}

void loop() {
    enableADC();
    int value = analogRead(A0);
    disableADC();

    // Process reading...
    delay(1000);
}
```

## References

- [SAMD21 Datasheet](http://www.atmel.com/images/atmel-42181-samd21-datasheet.pdf)
- [Arduino Zero ADC](https://www.arduino.cc/reference/en/language/functions/analog-io/analogread/)
- [Adafruit ZeroDMA Library](https://github.com/adafruit/Adafruit_ZeroDMA)
