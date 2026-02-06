# Arduino on XIAO SAMD21

## Overview

The SeeedStudio XIAO SAMD21 features the Microchip SAMD21G18A ARM Cortex-M0+ microcontroller.

## Board Manager Setup

1. Open Arduino IDE
2. File > Preferences > Additional Boards Manager URLs
3. Add: `https://raw.githubusercontent.com/Seeed-Studio/Seeed_SAMD_Boards/master/package_seeed_samd_boards_index.json`
4. Tools > Board > Boards Manager > Search "Seeed SAMD" > Install
5. Tools > Board > Seeed SAMD Boards > Select "Seeed XIAO SAMD21"

## First Sketch

```cpp
void setup() {
    Serial.begin(115200);
    pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
    Serial.println("Hello XIAO SAMD21!");
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);
    digitalWrite(LED_BUILTIN, LOW);
    delay(1000);
}
```

## Pin Mapping

| Name | GPIO | Functions | Notes |
|------|------|-----------|-------|
| D0 | PA02 | A0, ADC | - |
| D1 | PA04 | A1, ADC | - |
| D2 | PA10 | A2, ADC | - |
| D3 | PA11 | A3, ADC | - |
| D4 | PA08 | A4, ADC | - |
| D5 | PA09 | A5, ADC | - |
| D6 | PB08 | A6, ADC | - |
| D7 | PB09 | A7, ADC | - |
| D8 | PA07 | A8, ADC | - |
| D9 | PA06 | A9, ADC | - |
| D10 | PA05 | A10, DAC | DAC output |

Built-in LED: PA17 (orange)

## Special Pins

### DAC
- D10 (PA05) - 10-bit DAC output
- Voltage range: 0-3.3V

### ADC
- 12-bit ADC (0-4095) on A0-A9
- 10-bit DAC on D10
- Voltage range: 0-3.3V
- Sample rate: up to 350 ksps

### PWM
- All GPIO pins support PWM
- TCC0, TCC1, TCC2 timers
- TC3, TC4, TC5 timers

### I2C
- Default: D5 (SDA), D4 (SCL)
- Wire peripheral on SERCOM0

### SPI
- Default: D8 (MISO), D9 (SCK), D10 (MOSI)
- SPI peripheral on SERCOM1

### UART
- USB Serial (native)
- UART0: D6 (RX), D7 (TX) on SERCOM2
- SoftwareSerial on any pins

## USB Device Support

### HID (Keyboard/Mouse)

```cpp
#include <Keyboard.h>

void setup() {
    Keyboard.begin();
}

void loop() {
    Keyboard.press(KEY_LEFT_GUI);
    Keyboard.press('r');
    delay(100);
    Keyboard.releaseAll();
    delay(1000);
}
```

### Mouse

```cpp
#include <Mouse.h>

void setup() {
    Mouse.begin();
}

void loop() {
    Mouse.move(10, 0, 0);
    delay(100);
}
```

### Mass Storage

```cpp
#include <SPIFlash.h>

// Use external flash for USB mass storage
SPIFlash flash(SS);

void setup() {
    if (flash.initialize()) {
        // Present as USB drive
    }
}

void loop() {}
```

## DAC Output

```cpp
void setup() {
    analogWriteResolution(10);  // 10-bit DAC
}

void loop() {
    // Sine wave approximation
    for (int i = 0; i < 360; i += 10) {
        int value = 512 + 511 * sin(i * PI / 180);
        analogWrite(D10, value);
        delay(10);
    }
}
```

## ADC Reading

```cpp
void setup() {
    Serial.begin(115200);
    analogReadResolution(12);  // 12-bit
}

void loop() {
    int adc = analogRead(A0);
    float voltage = adc * 3.3 / 4095;
    Serial.printf("ADC: %d, Voltage: %.2fV\n", adc, voltage);
    delay(1000);
}
```

## Zero-Crossing Detection

```cpp
void setup() {
    pinMode(A0, INPUT);
    attachInterrupt(digitalPinToInterrupt(A0), zeroCross, RISING);
}

void zeroCross() {
    // Handle zero-crossing event
    Serial.println("Zero crossing");
}

void loop() {}
```

## EEPROM Emulation

```cpp
#include <FlashStorage.h>

typedef struct {
    int value;
    bool initialized;
} Config;

FlashStorage(my_config_store, Config);

Config config;

void setup() {
    Serial.begin(115200);

    // Load config
    config = my_config_store.read();

    if (!config.initialized) {
        // Initialize default values
        config.value = 100;
        config.initialized = true;
        my_config_store.write(config);
    }

    Serial.println(config.value);
}

void loop() {}
```

## Low Power

### Standby Mode

```cpp
#include <RTCZero.h>

RTCZero rtc;

void setup() {
    Serial.begin(115200);
    rtc.begin();

    // Set alarm for 10 seconds
    rtc.setAlarmEpoch(rtc.getEpoch() + 10);
    rtc.enableAlarm(rtc.MATCH_YYMMDDHHMMSS);
    rtc.attachInterrupt(alarmMatch);

    // Enter standby
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    __WFI();
}

void alarmMatch() {
    // Woke up
}

void loop() {}
```

### Sleep Modes

```cpp
void setup() {
    Serial.begin(115200);

    // Idle mode (CPU sleep, peripherals active)
    SCB->SCR &= ~SCB_SCR_SLEEPDEEP_Msk;
    __WFI();

    // Standby mode (deep sleep)
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    __WFI();
}

void loop() {}
```

## Watchdog Timer

```cpp
#include <wdt.h>

void setup() {
    Serial.begin(115200);

    // Enable watchdog (4 seconds)
    wdt_enable(WDT_PERIOD_4X);
}

void loop() {
    // Pet the watchdog
    wdt_reset();

    Serial.println("Watchdog reset");
    delay(1000);
}
```

## RTC (Real Time Clock)

```cpp
#include <RTCZero.h>

RTCZero rtc;

void setup() {
    Serial.begin(115200);
    rtc.begin();

    // Set time: 2024-01-15 12:30:00
    rtc.setDate(15, 1, 2024);
    rtc.setTime(12, 30, 0);
}

void loop() {
    Serial.print(rtc.getDay());
    Serial.print("/");
    Serial.print(rtc.getMonth());
    Serial.print("/");
    Serial.print(rtc.getYear());
    Serial.print(" ");
    Serial.print(rtc.getHours());
    Serial.print(":");
    Serial.print(rtc.getMinutes());
    Serial.print(":");
    Serial.println(rtc.getSeconds());
    delay(1000);
}
```

## External Interrupts

```cpp
volatile int counter = 0;

void setup() {
    Serial.begin(115200);
    pinMode(D5, INPUT_PULLUP);

    attachInterrupt(digitalPinToInterrupt(D5), countPulse, RISING);
}

void countPulse() {
    counter++;
}

void loop() {
    Serial.printf("Count: %d\n", counter);
    delay(1000);
}
```

## Timer Interrupts

```cpp
volatile bool timerFlag = false;

void TC5_Handler() {
    if (TC5->COUNT16.INTFLAG.bit.OVF == 1) {
        TC5->COUNT16.INTFLAG.bit.OVF = 1;
        timerFlag = true;
    }
}

void setupTimer() {
    // Configure TC5 for 1 Hz interrupt
    REG_GCLK_CLKCTRL = GCLK_CLKCTRL_ID_TC5_GCLK_ID |
                      GCLK_CLKCTRL_GEN_GCLK0 |
                      GCLK_CLKCTRL_CLKEN;
    while (GCLK->STATUS.bit.SYNCBUSY);

    TC5->COUNT16.CTRLA.reg = TC_CTRLA_MODE_COUNT16 |
                             TC_CTRLA_WAVEGEN_MFRQ |
                             TC_CTRLA_PRESCALER_DIV1024;
    TC5->COUNT16.CC[0].reg = 48000;  // 1 Hz at 48MHz/1024
    TC5->COUNT16.INTENSET.bit.OVF = 1;
    TC5->COUNT16.CTRLA.bit.ENABLE = 1;
    NVIC_EnableIRQ(TC5_IRQn);
}

void setup() {
    Serial.begin(115200);
    setupTimer();
}

void loop() {
    if (timerFlag) {
        timerFlag = false;
        Serial.println("Timer tick");
    }
}
```

## Key Specifications

| Feature | Value |
|---------|-------|
| MCU | SAMD21G18A |
| Core | ARM Cortex-M0+ @ 48MHz |
| Flash | 256 KB |
| RAM | 32 KB |
| GPIO | 15 pins |
| ADC | 12-bit, 10 channels |
| DAC | 10-bit, 1 channel |
| PWM | All pins |
| USB | Native USB |
| I2C | 1 |
| SPI | 1 |
| UART | 1 + USB |

## Known Issues

1. **USB**: Disconnects during deep sleep
2. **DAC**: Only available on D10
3. **ADC2**: Limited to 12-bit resolution
4. **Power**: Needs careful management for battery

## Troubleshooting

### Can't upload
- Double-click RESET button twice quickly
- Look for "XIAO" drive to appear
- Copy .uf2 file if using bootloader

### USB not detected
- Install Seeed SAMD board package
- Check USB cable (data cable required)
- Try different USB port

### ADC readings noisy
- Add 100nF capacitor at analog input
- Use averaging: multiple reads
- Keep traces short

### DAC output wrong
- Use `analogWriteResolution(10)`
- Check DAC is on D10 only
- Verify 3.3V reference

## Best Practices

1. **Power**: Use 3.3V regulated supply
2. **Decoupling**: Add 100nF + 10uF capacitors
3. **USB**: Close Serial monitor before sleep
4. **RTC**: Set time after power-on
5. **Flash**: Use FlashStorage for persistent data

## Performance Tips

1. **Clock**: Run at 48MHz for best performance
2. **Interrupts**: Use ISR for time-critical code
3. **ADC**: Average multiple readings
4. **PWM**: Use TCC timers for smooth PWM
5. **Sleep**: Use standby mode for battery
