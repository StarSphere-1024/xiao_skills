# Arduino on XIAO RA4M1

## Overview

The SeeedStudio XIAO RA4M1 features the Renesas RA4M1 ARM Cortex-M33 microcontroller - the same chip as Arduino UNO R4.

## Board Manager Setup

### Using Arduino IDE (Renesas Core)

1. Open Arduino IDE
2. File > Preferences > Additional Boards Manager URLs
3. Add: `https://github.com/renesas/Arduino/archive/refs/heads/main.zip`
   Or use Boards Manager directly for "Renesas RA"
4. Tools > Board > Boards Manager > Search "Renesas" > Install
5. Tools > Board > Renesas RA > Select "XIAO RA4M1"

### Alternative: FSP (Flexible Software Package)

For advanced features, use Renesas e2 studio with FSP.

## First Sketch

```cpp
void setup() {
    Serial.begin(115200);
    pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
    Serial.println("Hello XIAO RA4M1!");
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);
    digitalWrite(LED_BUILTIN, LOW);
    delay(1000);
}
```

## Pin Mapping

| Name | Pin | Function | Arduino |
|------|-----|----------|---------|
| D0 | P014 | ADC09, GPIO | Analog |
| D1 | P000 | ADC00, GPIO | Analog |
| D2 | P001 | ADC01, GPIO | Analog |
| D3 | P002 | ADC02, GPIO | Analog |
| D4 | P206 | SDA1, ADC | I2C1 SDA |
| D5 | P100 | SCL1, ADC | I2C1 SCL |
| D6 | P302 | TXD2, SDA2 | UART2 TX |
| D7 | P301 | RXD2, SCL2 | UART2 RX |
| D8 | P111 | SPI1_SCK | SPI SCK |
| D9 | P110 | SPI1_MISO, CRX0 | SPI MISO |
| D10 | P109 | SPI1_MOSI, CTX0 | SPI MOSI |
| D11 | P408 | RX9 | UART RX |
| D12 | P409 | TX9 | UART TX |
| D13 | P013 | GPIO | - |
| D14 | P012 | GPIO | - |
| D15 | P101 | TXD0, SDA0, ADC21, SPI0_MOSI | UART0 TX |
| D16 | P104 | RXD0, SCL0, SPI0_MISO | UART0 RX |
| D17 | P102 | CRX0, ADC20, SPI0_SCK | UART0 RX |
| D18 | P103 | CTX0, ADC19 | UART0 TX |

Other key pins (from Seeed Ref pin map):

- **ADC_BAT**: P400
- **RESET**: RES
- **BOOT**: P201
- **RGB LED**: P112
- **USER_LED / LED_BUILTIN**: P011

## Special Features

### ARM Cortex-M33

```cpp
void setup() {
    Serial.begin(115200);

    // CPU information
    uint32_t cpu_id = SCB->CPUID;
    Serial.printf("CPU ID: 0x%08lX\n", cpu_id);
}

void loop() {}
```

### 12-bit ADC

```cpp
void setup() {
    Serial.begin(115200);
    analogReadResolution(12);
}

void loop() {
    int adc = analogRead(A0);
    float voltage = adc * 3.3 / 4095;
    Serial.printf("ADC: %d, Voltage: %.2fV\n", adc, voltage);
    delay(1000);
}
```

### DAC (8-bit)

```cpp
void setup() {
    analogWriteResolution(8);
}

void loop() {
    // Sine wave
    for (int i = 0; i < 360; i += 10) {
        int value = 128 + 127 * sin(i * PI / 180);
        analogWrite(DAC, value);
        delay(10);
    }
}
```

### UART (Multiple)

```cpp
// UART0 (USB Serial)
Serial.begin(115200);

// UART2 (Hardware)
Serial2.begin(9600);

void setup() {
    Serial.begin(115200);
    Serial2.begin(9600);
}

void loop() {
    Serial2.println("Hardware UART");
    delay(1000);
}
```

### I2C (Multiple)

```cpp
#include <Wire.h>

// I2C0 on D15/D16
// I2C1 on D4/D5

void setup() {
    Wire.begin();  // I2C0
    Wire1.begin(); // I2C1
}

void loop() {}
```

### SPI (Multiple)

```cpp
#include <SPI.h>

// SPI0 on D15/D16/D17/D18
// SPI1 on D8/D9/D10

#define SPI0_CS D13
#define SPI1_CS D14

void setup() {
    SPI.begin();   // SPI0
    SPI1.begin();  // SPI1
    pinMode(SPI0_CS, OUTPUT);
    pinMode(SPI1_CS, OUTPUT);
}

void loop() {
    digitalWrite(SPI0_CS, LOW);
    SPI.transfer(0x00);
    digitalWrite(SPI0_CS, HIGH);
}
```

## PWM

```cpp
void setup() {
    // PWM available on most pins
    pinMode(D6, OUTPUT);
    analogWrite(D6, 128);  // 50% duty
}

void loop() {}
```

## Interrupts

```cpp
volatile int counter = 0;

void ISR() {
    counter++;
}

void setup() {
    pinMode(D13, INPUT_PULLUP);
    attachInterrupt(digitalPinToInterrupt(D13), ISR, RISING);
}

void loop() {
    Serial.printf("Count: %d\n", counter);
    delay(1000);
}
```

## EEPROM (Flash)

```cpp
#include <Flash.h>

#define FLASH_PAGE_SIZE 256

void readFlash(uint8_t* buffer, size_t length) {
    // Read from flash
    memcpy(buffer, (void*)0x40100000, length);
}

void writeFlash(const uint8_t* data, size_t length) {
    // Write to flash (requires erasing first)
    // Note: Limited write cycles
}
```

## RTC (Real Time Clock)

```cpp
#include <RTC.h>

RTC rtc;

void setup() {
    Serial.begin(115200);
    rtc.begin();

    // Set time: 2024-01-15 12:30:00
    rtc.setDateTime(2024, 1, 15, 12, 30, 0);
}

void loop() {
    RTC_DateTime dt = rtc.now();
    Serial.printf("%04d-%02d-%02d %02d:%02d:%02d\n",
                  dt.year, dt.month, dt.day,
                  dt.hour, dt.minute, dt.second);
    delay(1000);
}
```

## Watchdog Timer

```cpp
#include <wdt.h>

void setup() {
    Serial.begin(115200);

    // Enable watchdog (2 seconds)
    wdt_init(wdt_2seconds);
    wdt_enable();
}

void loop() {
    wdt_reset();  // Feed the watchdog
    Serial.println("Watchdog reset");
    delay(1000);
}
```

## Low Power

### Sleep Mode

```cpp
#include <LowPower.h>

void setup() {
    Serial.begin(115200);
}

void loop() {
    Serial.println("Going to sleep...");
    delay(100);

    // Enter sleep mode
    LowPower.sleep();

    // Wake up continues here
    Serial.println("Woke up!");
}
```

### Deep Sleep

```cpp
#include <LowPower.h>

void setup() {
    Serial.begin(115200);
}

void loop() {
    Serial.println("Going to deep sleep...");
    delay(100);

    // Enter deep sleep (reset on wake)
    LowPower.deepSleep();
}
```

## Key Differences from UNO R4

| Feature | UNO R4 | XIAO RA4M1 |
|---------|--------|------------|
| Form Factor | Standard Arduino | XIAO compact |
| USB | USB-C | USB-C |
| Pins | 20 | 18 (smaller) |
| LED | Orange | Orange |
| Compatible | UNO R4 shields | XIAO ecosystem |

## Key Specifications

| Feature | Value |
|---------|-------|
| MCU | RA4M1 |
| Core | ARM Cortex-M33 @ 48MHz |
| Flash | 256 KB |
| RAM | 32 KB |
| GPIO | 18 pins |
| ADC | 12-bit, 14 channels |
| DAC | 8-bit, 2 channels |
| PWM | All pins |
| I2C | 2 |
| SPI | 2 |
| UART | 3 |
| USB | Native USB |
| CAN | Yes (unpopulated) |

## Arduino UNO R4 Compatibility

XIAO RA4M1 uses the same RA4M1 chip as Arduino UNO R4, so:

✅ Same core libraries
✅ Same peripheral support
✅ Same programming model
❌ Different pinout
❌ Different form factor

## Known Issues

1. **New platform**: Renesas core is newer
2. **Pin differences**: From UNO R4
3. **Documentation**: Less mature
4. **Community**: Smaller than AVR

## Troubleshooting

### Can't upload

1. Check board selection
2. Install Renesas core properly
3. Try different USB cable
4. Check port selection

### Serial monitor issues

1. Close other serial programs
2. Check baud rate
3. Reset board after opening monitor

### ADC readings wrong

1. Set resolution with `analogReadResolution(12)`
2. Check reference voltage
3. Verify analog pin number

### Compile errors

1. Update Renesas core
2. Check library compatibility
3. Verify correct board selected

## Best Practices

1. **Power**: Use stable 3.3V supply
2. **Decoupling**: Add 100nF capacitors
3. **USB**: Use USB-C data cable
4. **Interrupts**: Keep ISRs short
5. **Flash**: Minimize flash writes

## Performance Tips

1. **Clock**: Run at 48MHz
2. **Interrupts**: Use for time-critical tasks
3. **DMA**: Use for data transfer
4. **Optimization**: Use -O3 flag
5. **Cache**: Enable cache if available

## Migration from UNO R4

```cpp
// UNO R4 code typically works with minor changes:

// Change pin numbers
// UNO R4: 2, 3, 4...
// XIAO: D0, D1, D2...

// Add proper includes
#include <Adafruit_TinyUSB.h>

// Adjust serial
Serial.begin(115200);  // Same
Serial2.begin(9600);   // Hardware UART2
```
