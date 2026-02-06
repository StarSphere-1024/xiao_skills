# RP2040/RP2350 Sleep API for MicroPython

## Overview

RP2040 has multiple sleep modes though less sophisticated than ESP32.

## Sleep Modes

| Mode | Current | Wake Time | RAM Retained |
|------|---------|-----------|--------------|
| Active | 25mA | - | Yes |
| Lightsleep | 180µA | ~100µs | Yes |
| Deepsleep | ~1µA | Reset | No* |

*Deep sleep requires special firmware configuration

## Light Sleep

```python
import machine
import time

# Light sleep - CPU stops, RAM kept
machine.lightsleep(5000)  # 5 seconds
print("Woke from light sleep")

# Continuous light sleep loop
while True:
    print("Working...")
    time.sleep(1)
    machine.lightsleep(5000)
```

## Delay Sleep

```python
import time

# Simple delay sleep
while True:
    print("Active...")
    time.sleep(5)  # 5 second delay
```

## Deep Sleep (if available)

```python
import machine

print("Going to deep sleep...")

# Note: RP2040 MicroPython may not support deepsleep without custom firmware
machine.deepsleep(10000)  # 10 seconds

# Code after may not execute
print("This might not run")
```

## GPIO Wake (if supported)

```python
import machine
from machine import Pin

# Configure wake pin
wake_pin = Pin(6, Pin.IN, Pin.PULL_UP)

# Wake on LOW (if supported)
try:
    machine.wake_on_ext0(Pin(6), machine.WAKEUP_ANY_LOW)
    print("Sleeping... Connect D6 to GND to wake")
    machine.deepsleep()
except:
    print("Deep sleep wake not supported on this firmware")
```

## RTC Memory

```python
import machine

# Check wake reason
if machine.reset_cause() == machine.DEEPSLEEP_RESET:
    print("Woke from deep sleep")
else:
    print("Normal boot")

# Use RTC for data preservation
rtc = machine.RTC()
rtc.memory('boot_count=0')

# Read on next boot
mem = rtc.memory()
print(f"RTC memory: {mem}")
```

## Timer Wake Loop

```python
import machine
import time

# Periodic wake with work
while True:
    print("Working...")
    # Do tasks
    time.sleep(1)

    # Sleep to save power
    machine.lightsleep(10000)  # 10 seconds
```

## Disable USB

```python
# Don't initialize USB to save power
# Comment out or remove:
# import machine
# machine.USB()

# Or disable after init
# (firmware dependent)
```

## Lower CPU Frequency

```python
import machine

# Set CPU frequency
machine.freq(50000000)  # 50MHz
print(f"CPU: {machine.freq()} Hz")

# Can be adjusted dynamically
machine.freq(125000000)  # Back to 125MHz
```

## Battery Measurement

```python
from machine import ADC, Pin
import time

# ADC on RP2040
adc = ADC(26)  # GPIO26

def read_battery():
    value = adc.read_u16()
    voltage = value * (3.3 / 65535)
    # Adjust for voltage divider
    return voltage * 2.0

while True:
    bat = read_battery()
    print(f"Battery: {bat:.2f}V")
    time.sleep(5)
    machine.lightsleep(5000)
```

## Dual Core Sleep

```python
import _thread
import machine
import time

# Core 1 function
def core1_task():
    while True:
        print("Core 1 running")
        time.sleep(2)
        machine.lightsleep(1000)

# Launch core 1
_thread.start_new_thread(core1_task, ())

# Core 0 main loop
while True:
    print("Core 0 running")
    time.sleep(2)
    machine.lightsleep(1000)
```

## PIO Monitoring

```python
# Use PIO to monitor while CPU sleeps
# (Requires PIO programming)
from machine import Pin
import rp2

# PIO state machine can watch GPIO
@rp2.asm_pio()
def gpio_monitor:
    wrap_target()
    in pins, 1 [31]
    jmp pin, 0  # Wait for GPIO change
    irq 0       # Trigger interrupt
    wrap()

# Start PIO
sm = rp2.StateMachine(0, gpio_monitor, in_base=Pin(6))
sm.active(1)

# CPU can sleep while PIO monitors
machine.lightsleep(10000)
```

## RP2350 Specific (if available)

```python
# RP2350 Cortex-M33 features
import machine

# Check if RP2350
try:
    machine.freq(250000000)  # 250MHz if supported
    print("RP2350 high frequency")
except:
    print("RP2040 - max 125MHz")
```

## Flash Storage

```python
# Persistent storage through sleep
import uos

# Write to file
with open('data.txt', 'w') as f:
    f.write('boot_count=1')

# Read on wake
try:
    with open('data.txt', 'r') as f:
        data = f.read()
        print(f"Saved: {data}")
except:
    print("No saved data")
```

## Best Practices

1. **Use lightsleep()** for periodic wake-ups
2. **Lower CPU frequency** when possible
3. **Disable USB** for battery operation
4. **Store data in flash** for persistence
5. **Use PIO** for monitoring while CPU sleeps

## Typical Current Consumption

| Activity | Current |
|----------|---------|
| Active (125MHz) | 25mA |
| Active (50MHz) | 10mA |
| Lightsleep | 180µA |
| Deepsleep | ~1µA |
| USB active | +20mA |

## Troubleshooting

### Deep sleep not working

1. Check firmware supports deepsleep
2. May need custom firmware build
3. Use lightsleep as alternative

### High current in sleep

1. Disable USB (don't call print/serial)
2. Set unused GPIOs to input/pull-down
3. Lower CPU frequency
4. Disconnect external peripherals

### Loses data on wake

1. RAM lost in deep sleep
2. Use RTC.memory() for small data
3. Store to flash for larger data
4. Reinitialize peripherals on wake
