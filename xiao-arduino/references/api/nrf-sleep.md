# nRF52 Sleep / Low Power API

## Overview

This page focuses on low-power behavior for **XIAO nRF52840 / XIAO nRF52840 Sense** in Arduino.

- For XIAO nRF52840, Arduino support is commonly provided via Seeed’s nRF52 board packages (for example FQBN `Seeeduino:nrf52:xiaonRF52840`).
- Depending on how you installed boards in Arduino IDE, you may also see the board listed under **Adafruit nRF52840 Boards > Seeed XIAO nRF52840**.
- Many low-level headers/APIs (such as `nrf.h` and the optional Bluefruit stack) are compatible with the **Adafruit nRF52** ecosystem.
- **Note**: nRF54L15 is **not** supported in Arduino. Use MicroPython (`/xiao-micropython`) for nRF54L15 development.

## Nordic terminology: System ON vs System OFF

Nordic uses two high-level power states:

- **System ON**: the default state after reset. The CPU/peripherals can be running or idle/sleeping.
- **System OFF**: the deepest power saving mode. Most of the system is powered down.

Important System OFF characteristics (from Nordic):

- The device can be woken from System OFF via signals such as **GPIO DETECT**, **LPCOMP ANADETECT**, **NFC field**, **VBUS detect**, or **a reset**.
- The system is **reset when it wakes up** from System OFF.
- **RAM retention in System OFF is configurable** via `RAM[n].POWER` settings (advanced; startup code may overwrite these registers).

## Practical options in Arduino sketches

### 1) Timed sleep with Watchdog (System ON sleep)

If you want to reduce CPU activity without doing a full reset on wake, a common Arduino-friendly approach is watchdog-based sleep.

```cpp
#include <Arduino.h>
#include <Adafruit_SleepyDog.h>

void setup() {
    Serial.begin(115200);
}

void loop() {
    // Sleep for (up to) 5 seconds and then continue running.
    // Returns the actual sleep time (ms).
    int slept_ms = Watchdog.sleep(5000);

    Serial.print("Slept for ");
    Serial.print(slept_ms);
    Serial.println(" ms");

    delay(1000);
}
```

Notes:

- System ON sleep keeps your program context (RAM) and returns to the next line after the sleep.
- Actual current depends heavily on what peripherals you leave enabled (UART, sensors, BLE, GPIO pullups, etc.).

### 2) Deep sleep with System OFF (wake causes reset)

If your application can tolerate a reboot on wake, System OFF is the deepest power saving mode.

The simplest Arduino-side pattern is:

```cpp
#include <Arduino.h>

#if defined(NRF52_SERIES) || defined(ARDUINO_ARCH_NRF52)
    #include <nrf.h>
#endif

void enterSystemOff() {
#if defined(NRF52_SERIES) || defined(ARDUINO_ARCH_NRF52)
    // Enter System OFF. Execution stops here.
    NRF_POWER->SYSTEMOFF = 1;

    // Recommended safety loop: CPU should not run past SYSTEMOFF write.
    while (true) {
        __WFI();
    }
#else
    // Not an nRF52 build target.
    while (true) {
        delay(1000);
    }
#endif
}

void setup() {
    Serial.begin(115200);
    Serial.println("Entering System OFF...");
    delay(100);
    enterSystemOff();
}

void loop() {
    // Not reached
}
```

Notes:

- Wake from System OFF will **reset** the chip, so `setup()` runs again.
- Wake sources (GPIO DETECT, LPCOMP, NFC, VBUS, reset) are configured at the hardware/SoC level. If you need GPIO wake on a specific pin, follow Nordic’s documentation for GPIO DETECT/SENSE configuration.

### GPIO wake from System OFF (SENSE/DETECT)

If you want to wake from **System OFF** using a GPIO pin, Nordic’s mechanism is based on **GPIO SENSE** driving a **DETECT** signal:

- A pin can be configured to **sense High or Low level** (via the `PIN_CNF[n].SENSE` field).
- When the configured level is detected, the pin’s **DETECT** signal goes high.
- This mechanism is functional in **both System ON and System OFF**, and the **POWER** peripheral uses the DETECT signal to exit System OFF.

Practical notes for Arduino sketches:

- You still need to ensure the wake pin has a **defined level** during sleep (external resistor, or internal pullup/pulldown if your core keeps it active).
- Avoid enabling SENSE while the pin is already at the trigger level, otherwise it may wake immediately.
- Because waking from System OFF causes a **reset**, your sketch should treat it like a fresh boot. A common pattern is to read `RESETREAS` early in `setup()`.

Example: read (and clear) the reset reason after boot:

```cpp
#include <Arduino.h>

#if defined(NRF52_SERIES) || defined(ARDUINO_ARCH_NRF52)
    #include <nrf.h>
#endif

void setup() {
    Serial.begin(115200);

#if defined(NRF52_SERIES) || defined(ARDUINO_ARCH_NRF52)
    uint32_t reset_reason = NRF_POWER->RESETREAS;
    NRF_POWER->RESETREAS = reset_reason; // write 1s to clear the flags we read

    Serial.print("RESETREAS=0x");
    Serial.println(reset_reason, HEX);
#endif
}

void loop() {
    delay(1000);
}
```

If you need a register-level implementation (mapping Arduino pin numbers to P0/P1 and `PIN_CNF[n]`), follow Nordic’s GPIO documentation and your Arduino core’s variant/pin mapping.

### 3) BLE: reduce advertising duty cycle (Bluefruit)

If you are using the Adafruit Bluefruit stack, you can usually reduce average power by increasing advertising interval and limiting fast advertising time.

```cpp
#include <Arduino.h>

#if defined(NRF52_SERIES) || defined(ARDUINO_ARCH_NRF52)
    #include <bluefruit.h>
#endif

void startAdvertisingLowPower() {
#if defined(NRF52_SERIES) || defined(ARDUINO_ARCH_NRF52)
    Bluefruit.Advertising.restartOnDisconnect(true);

    // Intervals are in units of 0.625 ms.
    // Example: fast = 20 ms (32 * 0.625), slow = 152.5 ms (244 * 0.625)
    Bluefruit.Advertising.setInterval(32, 244);
    Bluefruit.Advertising.setFastTimeout(30);  // seconds in fast mode

    // start(0) advertises forever until connected.
    Bluefruit.Advertising.start(0);
#endif
}

void setup() {
#if defined(NRF52_SERIES) || defined(ARDUINO_ARCH_NRF52)
    Bluefruit.begin();
    Bluefruit.setName("XIAO-nRF52");
    startAdvertisingLowPower();
#endif
}

void loop() {
    delay(1000);
}
```

## Best practices checklist

- Prefer **System OFF** for long battery life if a reset-on-wake is acceptable.
- Prefer **System ON sleep** if you must keep RAM state and continue after wake.
- Disable unused peripherals before sleeping; floating GPIOs and always-on sensors often dominate sleep current.
- For BLE peripherals, tune advertising interval/timeout and only advertise when needed.
- Always measure on your actual board (peripherals, pullups, external sensors change everything).

## References

- Nordic Semiconductor — nRF52840 POWER (System OFF mode, wake sources, reset behavior, RAM retention):
    https://docs.nordicsemi.com/bundle/ps_nrf52840/page/power.html
- Nordic Semiconductor — nRF52840 GPIO (SENSE/DETECT mechanism works in System ON and System OFF):
    https://docs.nordicsemi.com/bundle/ps_nrf52840/page/gpio.html
- Adafruit — Bluefruit nRF52 BSP docs: BLEAdvertising API (`setInterval`, `setFastTimeout`, `start`):
    https://learn.adafruit.com/bluefruit-nrf52-feather-learning-guide/bleadvertising
- Adafruit — nRF52 Arduino core (source, examples, platform behavior):
    https://github.com/adafruit/Adafruit_nRF52_Arduino
- Adafruit — low power & battery life guide (practical tips):
    https://learn.adafruit.com/bluefruit-nrf52-feather-learning-guide/low-power-and-battery-life
