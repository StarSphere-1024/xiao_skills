# nRF52/nRF54 Sleep API for MicroPython

## Overview

MicroPython provides generic sleep APIs via the `machine` module, but the actual behavior
is **port- and firmware-dependent**.

This page focuses on what can be stated with evidence from upstream MicroPython
documentation and the upstream `nrf` port implementation (MicroPython v1.27.0).

## Light Sleep

```python
import machine
import time

# lightsleep(): attempts to enter a low power state.
# Note: on the upstream MicroPython `nrf` port (v1.27.0), the optional
# time_ms argument is not used by the port implementation.
machine.lightsleep()
print("Woke from lightsleep")

# Continuous light sleep loop
while True:
    # Do work
    print("Active...")
    time.sleep(1)

    # Sleep to save power
    machine.lightsleep()
```

## Deep Sleep

```python
import machine
import time

print("Going to deepsleep...")

# Generic MicroPython semantics: deepsleep resumes from the main script.
# Upstream `nrf` port note (MicroPython v1.27.0): deepsleep() is implemented
# as a system reset.
machine.deepsleep()
```

## Wake Sources (GPIO)

MicroPython's core documentation describes configuring wake sources such as
`Pin` change before calling `machine.lightsleep()` / `machine.deepsleep()`.

Important portability note:

- `wake_on_ext0` / `wake_on_ext1` and `WAKEUP_*` constants are **ESP32-specific** APIs
    in the `esp32` module, not generic `machine` APIs.

For portable GPIO wake behavior, use a `Pin.irq()` handler and then call
`machine.lightsleep()`.

```python
import machine
from machine import Pin

woke = False

def on_wake(pin):
    global woke
    woke = True

# Example pin name; choose a pin available on your board.
pin = Pin("P1.11", Pin.IN, Pin.PULL_UP)
pin.irq(trigger=Pin.IRQ_FALLING, handler=on_wake)

print("Entering lightsleep; toggle the pin to wake")
machine.lightsleep()

print("Woke:", woke)
```

## RTC Memory (Preserve Data)

```python
import machine

# reset_cause() is the portable way to inspect reset reasons.
# The available constants vary by port.
print("reset_cause:", machine.reset_cause())

# Persist any state you need across resets (e.g. filesystem, external storage).
machine.deepsleep()
```

## Timer Wake with Work

```python
import machine
import time

# Wake and do work
while True:
    print("Working...")
    # Do your tasks here
    time.sleep(1)

    print("Going back to lightsleep")
    machine.lightsleep()
```

## BLE Low Power

```python
import bluetooth
import machine

ble = bluetooth.BLE()
ble.active(True)

# Set longer advertising interval for lower power
# Note: MicroPython BLE API may vary
def ble_adv():
    # Configure low power advertising
    ble.gap_advertise(500)  # 500ms interval (API availability depends on firmware)

ble_adv()
machine.lightsleep()
```

## Disable Unused Peripherals

```python
import machine

# Disable UART to save power
# (don't call uart = machine.UART(...))

# Disable SPI/I2C if not used
# (don't initialize them)

# Enter sleep
machine.lightsleep()
```

## Battery Measurement

```python
from machine import ADC, Pin
import time

# ADC input varies by board. Prefer a Pin-based ADC constructor when available.
adc = ADC(Pin("P0.14"))

def read_battery():
    # Read raw value (0-65535). Voltage conversion is board-specific.
    return adc.read_u16()

while True:
    raw = read_battery()
    print("Battery ADC raw:", raw)
    time.sleep(5)
    machine.lightsleep()
```

## Low Power Shutdown on Low Battery

```python
from machine import ADC, Pin
import machine

adc = ADC(Pin("P0.14"))

def check_battery():
    raw = adc.read_u16()

    # Thresholding requires a board-specific conversion.
    print("Battery ADC raw:", raw)

check_battery()
```

## Best Practices

1. **Use lightsleep** for periodic wake-ups
2. **Longer BLE advertising** intervals save power
3. **Disable unused peripherals**
4. **Use reset_cause()** to understand resets
5. **Minimize wake time** to save battery

## Troubleshooting

### Won't wake from deep sleep

Deep sleep wake configuration is port-specific. If your firmware does not expose
a dedicated wake configuration API, prefer `machine.lightsleep()` with a `Pin.irq()`
handler.

### High current in sleep

1. Ensure external peripherals are in a low-power state
2. Set unused pins to a defined state (avoid floating inputs)

### Loses data on wake

Deep sleep may restart the program. Persist required state to storage and
reinitialize peripherals after wake.

## References

- MicroPython `machine` module documentation: https://docs.micropython.org/en/latest/library/machine.html
- Upstream MicroPython source (v1.27.0) `machine` docs: https://github.com/micropython/micropython/blob/v1.27.0/docs/library/machine.rst
- Upstream MicroPython source (v1.27.0) ESP32 wake APIs (`esp32.wake_on_ext0/1`): https://github.com/micropython/micropython/blob/v1.27.0/docs/library/esp32.rst
- Upstream MicroPython source (v1.27.0) `nrf` port `machine` implementation (lightsleep/deepsleep/reset_cause): https://github.com/micropython/micropython/blob/v1.27.0/ports/nrf/modules/machine/modmachine.c
