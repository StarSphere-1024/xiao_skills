# nRF52/nRF54 Sleep API for MicroPython

## Overview

nRF52840 has excellent low power capabilities. MicroPython provides several sleep options.

## Sleep Modes

| Mode | Current | Wake Time | RAM Retained |
|------|---------|-----------|--------------|
| Active | ~5mA | - | Yes |
| Lightsleep | ~30µA | ~100µs | Yes |
| Deep sleep | ~2µA | Reset | No |

## Light Sleep

```python
import machine
import time

# Light sleep - stops CPU, keeps RAM
machine.lightsleep(5000)  # 5 seconds
print("Woke from light sleep")

# Continuous light sleep loop
while True:
    # Do work
    print("Active...")
    time.sleep(1)

    # Sleep to save power
    machine.lightsleep(5000)
```

## Deep Sleep

```python
import machine
import time

print("Going to deep sleep...")

# Sleep for 10 seconds
machine.deepsleep(10000)

# Code after deepsleep never executes
print("This won't run")
```

## GPIO Wakeup

```python
import machine
from machine import Pin

# Configure wake pin
wake_pin = Pin(6, Pin.IN, Pin.PULL_UP)

# Wake on LOW level
machine.wake_on_ext0(Pin(6), machine.WAKEUP_ANY_LOW)

print("Sleeping... Connect D6 to GND to wake")
machine.deepsleep()
```

## RTC Memory (Preserve Data)

```python
import machine

# Check if woke from deep sleep
if machine.reset_cause() == machine.DEEPSLEEP_RESET:
    print("Woke from deep sleep")

    # Restore data from RTC memory
    # (MicroPython nRF port may have limited RTC memory)
else:
    print("First boot")

# Sleep
machine.deepsleep(10000)
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

    print("Going back to sleep")
    machine.lightsleep(10000)  # 10 seconds
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
    ble.gap_advertise(500)  # 500ms interval

ble_adv()
machine.lightsleep(10000)
```

## Disable Unused Peripherals

```python
import machine

# Disable UART to save power
# (don't call uart = machine.UART(...))

# Disable SPI/I2C if not used
# (don't initialize them)

# Enter sleep
machine.lightsleep(10000)
```

## Battery Measurement

```python
from machine import ADC
import time

# ADC for battery (if available)
adc = ADC(0)

def read_battery():
    # Read and convert
    value = adc.read_u16()
    voltage = value * (3.6 / 65535)
    return voltage

while True:
    bat = read_battery()
    print(f"Battery: {bat:.2f}V")
    time.sleep(5)
    machine.lightsleep(5000)
```

## Low Power Shutdown on Low Battery

```python
from machine import ADC, Pin
import machine

adc = ADC(0)

def check_battery():
    voltage = adc.read_u16() * 3.6 / 65535

    if voltage < 3.0:
        print("Low battery - shutting down")
        machine.deepsleep(60000)  # Sleep 1 minute
    else:
        print(f"Battery OK: {voltage:.2f}V")

check_battery()
```

## Typical Current Consumption

| Activity | Current |
|----------|---------|
| Active | 5mA |
| BLE advertising | 5mA |
| BLE connected | 3-5mA |
| Lightsleep | 30µA |
| Deep sleep | 2µA |

## Best Practices

1. **Use lightsleep** for periodic wake-ups
2. **Longer BLE advertising** intervals save power
3. **Disable unused peripherals**
4. **Check reset_cause()** after deep sleep
5. **Minimize wake time** to save battery

## Troubleshooting

### Won't wake from deep sleep

1. Verify wake source configured
2. Check GPIO pull-up/down
3. Ensure timer is correct (milliseconds)

### High current in sleep

1. Disable BLE before sleep
2. Set unused pins to input with pull-down
3. Disconnect external peripherals

### Loses data on wake

1. RTC memory limited on nRF
2. Store to flash for persistence
3. Reinitialize after wake
