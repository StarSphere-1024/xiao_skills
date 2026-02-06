# RP2040 / RP2350 Sleep / Low Power API

## Overview

Seeed Studio XIAO RP2040 / RP2350 typically uses the **Arduino-Pico** core (Earle Philhower) in the Arduino ecosystem.

- RP2040 FQBN: `rp2040:rp2040:seeed_xiao_rp2040`
- RP2350 FQBN: `rp2040:rp2040:seeed_xiao_rp2350`

The Arduino-Pico core **bundles the Raspberry Pi Pico SDK (PICO-SDK)**, so you can include Pico SDK headers and call Pico SDK APIs directly from an Arduino sketch.

Note: once you start using SDK-level low-level calls (changing clocks, disabling peripherals, entering deeper power states, etc.), Arduino core/libraries may not be aware of those changes. It becomes your responsibility to keep things compatible and restore the system to a state the Arduino runtime expects.

## What “sleep” usually means here

On RP2040/RP2350, many Arduino projects use “sleep” to mean:

- **Avoid busy-waiting** (e.g. no `while (1) {}` spinning) so the CPU can idle while waiting.
- Use `delay()` or Pico SDK `sleep_ms()` / `sleep_us()` to wait using timers.

This kind of waiting is not the same as a “deep sleep” on some other SoCs (which may automatically shut down USB, debug links, and peripherals). For deeper power optimization you typically need to manage clocks, peripherals, power domains, and board-level components.

## The simplest win: avoid busy-wait loops

```cpp
void setup() {
  Serial.begin(115200);
}

void loop() {
  Serial.println("Work...");
  delay(100);

  // Don't spin in for/while loops; use delay() so the core can idle
  Serial.println("Idle for 5s");
  delay(5000);
}
```

## Calling Pico SDK sleep_ms/sleep_us directly

Arduino-Pico documents that you can include Pico SDK headers and call `sleep_ms()` directly from an Arduino sketch.

```cpp
#include "pico/stdlib.h"

void setup() {
  Serial.begin(115200);
}

void loop() {
  Serial.println("sleep_ms(1000)");
  sleep_ms(1000);

  Serial.println("sleep_us(200000)");
  sleep_us(200000);
}
```

## Reduce dynamic power: lower the system clock (advanced)

Lowering CPU frequency can reduce dynamic power, but it can also affect UART baud rates, USB behavior, timer accuracy, and library assumptions.
For battery-powered projects, start with a minimal sketch and measure on real hardware first, then add features incrementally.

```cpp
#include "hardware/clocks.h"

void setup() {
  Serial.begin(115200);
  delay(100);

  // Example: set system clock to 50 MHz
  // Whether this works depends on your core/board configuration; returns false on failure.
  bool ok = set_sys_clock_khz(50000, true);

  Serial.print("set_sys_clock_khz ok = ");
  Serial.println(ok ? 1 : 0);
}

void loop() {
  delay(1000);
}
```

## Waiting for external events: interrupt + low-frequency polling

A common Arduino pattern is to capture an event with `attachInterrupt()`, then reduce idle spinning by calling a short `sleep_ms()` in the main loop.

```cpp
#include "pico/stdlib.h"

static volatile bool g_triggered = false;

static void onWake() {
  g_triggered = true;
}

void setup() {
  Serial.begin(115200);
  pinMode(6, INPUT_PULLUP);

  attachInterrupt(digitalPinToInterrupt(6), onWake, FALLING);
  Serial.println("Waiting for D6 falling...");
}

void loop() {
  if (g_triggered) {
    g_triggered = false;
    Serial.println("Event!");
  }

  // sleep_ms() reduces idle power (still wakes periodically to check the flag)
  sleep_ms(10);
}
```

## Battery project notes (board-level)

- **USB often increases power draw**: if you don’t need USB CDC, consider using UART (e.g. `Serial1`) and avoiding USB-related features.
- **Peripherals and GPIO states matter**: `end()`/power down what you don’t use, avoid floating pins, and account for on-board parts like LDOs, sensors, and LEDs.
- **Measure before concluding**: current draw can vary a lot across boards and wiring.

## References

- Arduino-Pico docs (Pico SDK calls from Arduino): https://arduino-pico.readthedocs.io/en/latest/sdk.html
- Arduino-Pico docs (RP2040 helper, includes frequency notes like `rp2040.f_cpu()`): https://arduino-pico.readthedocs.io/en/latest/rp2040.html
- Arduino-Pico source (`delay()` uses `sleep_ms()`): https://github.com/earlephilhower/arduino-pico/blob/master/cores/rp2040/delay.cpp
- Raspberry Pi Pico SDK (official PDF): https://datasheets.raspberrypi.org/pico/raspberry-pi-pico-c-sdk.pdf
- This repo's Arduino CLI/FQBN table: `xiao-arduino/references/setup/arduino-cli.md`
