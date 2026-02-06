# ESP32 Sleep API for MicroPython

## Overview

Deep sleep, light sleep, and power management for ESP32-based XIAO boards.

## Deep Sleep

### Basic Deep Sleep

```python
import machine
import time

print("Going to deep sleep...")

# Sleep for 10 seconds (in milliseconds)
machine.deepsleep(10000)

# Code after deepsleep will not execute
print("This won't run")
```

### Deep Sleep with Wake Timer

```python
import machine

# Configure deep sleep for 30 seconds
machine.deepsleep(30000)
```

### RTC Memory (Preserves Data)

```python
import machine

# RTC memory variables
rtc = machine.RTC()
rtc.memory('boot_count=0')  # First time

# Read memory
mem = rtc.memory().decode('utf-8')
if 'boot_count=' in mem:
    count = int(mem.split('=')[1])
    count += 1
else:
    count = 1

print(f"Boot count: {count}")

# Save for next wake
rtc.memory(f'boot_count={count}')

# Sleep
machine.deepsleep(10000)
```

### GPIO Wakeup

```python
import machine
from machine import Pin

# Configure GPIO for wakeup
wake_pin = Pin(0, Pin.IN, Pin.PULL_UP)

# Wake on GPIO LOW
machine.wake_on_ext0(Pin(0), machine.WAKEUP_ANY_LOW)

print("Sleeping... Wake with D0 LOW")
machine.deepsleep()
```

### Multiple GPIO Wakeup

```python
import machine
from machine import Pin

# Wake on multiple pins
pins = [Pin(0), Pin(1)]
machine.wake_on_ext0(pins, machine.WAKEUP_ANY_HIGH)

print("Sleeping...")
machine.deepsleep()
```

### Touch Wakeup (ESP32S3)

```python
import machine

# Wake on touch pin (T0)
machine.wake_on_touch(True)

print("Touch pin to wake")
machine.deepsleep()
```

## Light Sleep

### Basic Light Sleep

```python
import machine
import time

# Light sleep for 5 seconds
machine.lightsleep(5000)

print("Woke from light sleep, variables preserved")
```

### Light Sleep Loop

```python
import machine
import time

while True:
    # Do work
    print("Working...")
    time.sleep(1)

    # Enter light sleep
    print("Sleeping...")
    machine.lightsleep(5000)
```

### Light Sleep with WiFi

```python
import network
import machine
import time

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.config(pm=network.WIFI_PS_MIN_MODEM)  # Enable WiFi power save
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

print("Connected, entering light sleep loop")

while True:
    # Light sleep keeps WiFi connected in low power
    machine.lightsleep(10000)
    print("Still connected:", wlan.isconnected())
```

## Power Management

### Disable WiFi/BT

```python
import network
import machine

# Disable WiFi
wlan = network.WLAN(network.STA_IF)
wlan.active(False)

# Disable Bluetooth
try:
    bluetooth = machine.Bluetooth()
    bluetooth.active(False)
except:
    pass

# Now enter deep sleep
machine.deepsleep()
```

### CPU Frequency Control

```python
import machine

# Set CPU frequency (MHz)
machine.freq(80)   # Low power
machine.freq(160)  # Medium
machine.freq(240)  # Maximum (ESP32S3)

print(f"Current frequency: {machine.freq()} MHz")
```

### Measure Current

```python
import machine
import time

# Put all pins to input mode with pull-down
from machine import Pin
pins = [Pin(i, Pin.IN, Pin.PULL_DOWN) for i in range(0, 10)]

# Disable unused peripherals
network.WLAN(network.STA_IF).active(False)

# Enter deep sleep
machine.deepsleep()
```

## Wakeup Sources

### Check Wakeup Reason

```python
import machine

def check_wakeup_reason():
    reason = machine.reset_cause()

    if reason == machine.DEEP_SLEEP_RESET:
        print("Woke from deep sleep")
    elif reason == machine.HARD_RESET:
        print("Hard reset (power-on)")
    elif reason == machine.WDT_RESET:
        print("Watchdog reset")
    elif reason == machine.SOFT_RESET:
        print("Software reset")
    else:
        print(f"Other reset: {reason}")

check_wakeup_reason()
```

### Wakeup Stub

```python
import machine

def wake_stub():
    # Runs immediately on wake
    # Set LED to indicate wake
    from machine import Pin
    led = Pin(2, Pin.OUT)
    led.on()

# Note: MicroPython doesn't support wake stub directly
# Use this pattern instead:
import machine
rtc = machine.RTC()

# Check if just woke from deep sleep
if machine.reset_cause() == machine.DEEP_SLEEP_RESET:
    # Do wake-up tasks
    print("Just woke from deep sleep")
    # Then continue normal operation
```

## Battery Management

### Read Battery Voltage

```python
from machine import ADC, Pin
import time

adc = ADC(Pin(1))
adc.atten(ADC.ATTN_11DB)

def read_battery():
    # Read ADC and convert to voltage
    value = adc.read()
    voltage = value / 4095 * 3.9  # Adjust for your divider
    return voltage

while True:
    bat = read_battery()
    print(f"Battery: {bat:.2f}V")
    time.sleep(5)
```

### Low Power Shutdown

```python
from machine import ADC, Pin
import machine

adc = ADC(Pin(1))
adc.atten(ADC.ATTN_11DB)

def check_battery_and_sleep():
    voltage = adc.read() / 4095 * 3.9

    if voltage < 3.0:  # Low battery threshold
        print("Low battery, entering deep sleep")
        machine.deepsleep(60000)  # Sleep 1 minute
    else:
        print(f"Battery OK: {voltage:.2f}V")

check_battery_and_sleep()
```

## Timed Tasks with Sleep

### Periodic Sensor Reading

```python
import machine
import time
from machine import Pin, ADC

rtc = machine.RTC()

# Initialize counter in RTC memory
if rtc.memory() == b'':
    rtc.memory('0')

# Read counter
count = int(rtc.memory().decode())
count += 1
rtc.memory(str(count).encode())

print(f"Reading #{count}")

# Read sensor
sensor = ADC(Pin(1))
print(f"Sensor: {sensor.read()}")

# Sleep for 30 seconds
machine.deepsleep(30000)
```

### Data Logging with Sleep

```python
import machine
import time
import uos
from machine import Pin, ADC

# Check if just woke from deep sleep
if machine.reset_cause() == machine.DEEP_SLEEP_RESET:
    # Read RTC memory
    rtc = machine.RTC()
    count = int(rtc.memory().decode())
    count += 1
    rtc.memory(str(count).encode())

    # Read sensor
    sensor = ADC(Pin(1))
    value = sensor.read()

    # Log to file
    try:
        with open('log.txt', 'a') as f:
            f.write(f"{count},{value}\n")
        print(f"Logged: {count}, {value}")
    except:
        print("Failed to log")

    # Sleep again
    machine.deepsleep(30000)
else:
    # First boot - initialize
    print("First boot - initializing")
    rtc = machine.RTC()
    rtc.memory('0')
    machine.deepsleep(1000)
```

## ULP Coprocessor (ESP32 Only)

### Basic ULP Wake

```python
import machine
import ulp

# Note: ULP requires assembly code
# This is a simplified example

# Define ULP code (simplified)
ulp_code = """
// ULP assembly to wake every 30 seconds
set_wakeup 30
"""

# Load and run ULP
# Note: Full ULP support requires detailed assembly
# See MicroPython docs for complete examples

machine.deepsleep()
```

## Troubleshooting

### Won't wake from deep sleep

1. Verify wakeup source is configured
2. Check GPIO pull-up/down
3. Ensure timer value is correct (milliseconds)
4. Try hard reset

### Loses data on wake

1. Use RTC.memory() for preservation
2. Keep data minimal (< 2KB)
3. Store to flash for larger data
4. Check reset_cause() to detect wake

### WiFi won't reconnect

1. Always reinitialize WiFi after deep sleep
2. Add delay before connection attempt
3. Use wlan.disconnect() before reconnect

### High current in sleep

1. Ensure WiFi/BT are off
2. Set all unused pins to INPUT_PULLDOWN
3. Disconnect external peripherals
4. Check for short circuits

### Light sleep not working

1. Check MicroPython version support
2. Verify no blocking code
3. Use lightsleep() instead of deepsleep()
4. Keep wake periods short

### RTC memory corrupted

1. Keep writes minimal
2. Use simple string encoding
3. Verify memory size limits
4. Check for buffer overflows
