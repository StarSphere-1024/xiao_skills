# ESP32 Sleep API for MicroPython

## Overview

Deep sleep, light sleep, and power management for ESP32-family XIAO boards running the MicroPython ESP32 port.

Note: ESP32 variants (ESP32 / ESP32-S2 / ESP32-S3 / ESP32-C3 / ESP32-C6, etc.) differ in available wake sources and peripherals. The examples below use only APIs documented by MicroPython; where a feature is SoC-dependent, it is called out explicitly.

## Deep Sleep

### Basic Deep Sleep

```python
import machine

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
rtc.memory(b'boot_count=0')  # First time

# Read memory
mem = rtc.memory().decode('utf-8')
if 'boot_count=' in mem:
    count = int(mem.split('=')[1])
    count += 1
else:
    count = 1

print(f"Boot count: {count}")

# Save for next wake
rtc.memory(f'boot_count={count}'.encode())

# Sleep
machine.deepsleep(10000)
```

### GPIO Wakeup

```python
import machine
from machine import Pin
import esp32

# Configure GPIO for wakeup
wake_pin = Pin(0, Pin.IN, Pin.PULL_UP)

# EXT0 wake: wake when the selected RTC-capable pin is LOW.
# Note: EXT0/EXT1 wake configuration is in the `esp32` module (SoC/board support varies).
esp32.wake_on_ext0(wake_pin, esp32.WAKEUP_ALL_LOW)

print("Sleeping... wake by pulling GPIO0 LOW")
machine.deepsleep()
```

### Multiple GPIO Wakeup

```python
import machine
from machine import Pin
import esp32

# EXT1 wake: wake when ANY selected pin is HIGH.
pins = (Pin(0, Pin.IN), Pin(1, Pin.IN))
esp32.wake_on_ext1(pins, esp32.WAKEUP_ANY_HIGH)

print("Sleeping...")
machine.deepsleep()
```

### Touch Wakeup (ESP32S3)

```python
import machine
from machine import Pin, TouchPad
import esp32

# Capacitive touch is supported on ESP32 / ESP32-S2 / ESP32-S3 (not ESP32-C3/C6).
# Configure a TouchPad threshold and enable touch wake.
t = TouchPad(Pin(14))
t.config(500)
esp32.wake_on_touch(True)

print("Touch pin to wake")
machine.lightsleep()
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
try:
    wlan.config(pm=network.WLAN.PM_POWERSAVE)
except Exception:
    # Power-management constants/config vary by port/version.
    pass
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    machine.idle()

print("Connected, entering light sleep loop")

while True:
    # Light sleep retains RAM/state, but WiFi connectivity across sleep is port/firmware dependent.
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
    import bluetooth
    ble = bluetooth.BLE()
    ble.active(False)
except Exception:
    pass

# Now enter deep sleep
machine.deepsleep()
```

### CPU Frequency Control

```python
import machine

# machine.freq() is in Hz (and supported settable values vary by SoC/port build).
machine.freq(80_000_000)   # 80 MHz
# machine.freq(160_000_000)  # 160 MHz
# machine.freq(240_000_000)  # 240 MHz (original ESP32; may not apply to all variants)

print(f"Current frequency: {machine.freq()} Hz")
```

### Measure Current

```python
import machine
import network

# Put all pins to input mode with pull-down
from machine import Pin
# Note: pull resistors can increase leakage during sleep on some boards.
pins = [Pin(i, Pin.IN) for i in range(0, 10)]
for p in pins:
    try:
        p.init(pull=None)
    except Exception:
        pass

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

    if reason == machine.DEEPSLEEP_RESET:
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

# Check if just woke from deep sleep
if machine.reset_cause() == machine.DEEPSLEEP_RESET:
    # Do wake-up tasks
    print("Just woke from deep sleep")
    # Then continue normal operation
```

## Battery Management

### Read Battery Voltage

```python
from machine import ADC, Pin
import time

# Board-specific note: choose an ADC-capable GPIO and use a safe voltage divider.
# The ADC input must not exceed the chip's absolute maximum rating.
ADC_GPIO = 1
adc = ADC(Pin(ADC_GPIO))
adc.atten(ADC.ATTN_11DB)

# If your board uses a resistor divider, set this to V_battery / V_adc.
DIVIDER_RATIO = 1.0

def read_battery():
    # Prefer calibrated voltage if available on your firmware.
    if not hasattr(adc, 'read_uv'):
        raise RuntimeError("ADC.read_uv() is not available on this firmware; use a calibrated method or scale raw ADC readings with board-specific calibration.")
    v_adc = adc.read_uv() / 1_000_000
    return v_adc * DIVIDER_RATIO

while True:
    bat = read_battery()
    print(f"Battery: {bat:.2f}V")
    time.sleep(5)
```

### Low Power Shutdown

```python
from machine import ADC, Pin
import machine

ADC_GPIO = 1
adc = ADC(Pin(ADC_GPIO))
adc.atten(ADC.ATTN_11DB)

DIVIDER_RATIO = 1.0
LOW_BATTERY_V = 3.0

def check_battery_and_sleep():
    if not hasattr(adc, 'read_uv'):
        print("ADC.read_uv() is not available on this firmware; cannot compare against a voltage threshold without calibration.")
        return
    voltage = (adc.read_uv() / 1_000_000) * DIVIDER_RATIO

    if voltage < LOW_BATTERY_V:
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
    rtc.memory(b'0')

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
if machine.reset_cause() == machine.DEEPSLEEP_RESET:
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
    rtc.memory(b'0')
    machine.deepsleep(1000)
```

## ULP Coprocessor (ESP32 Only)

### Basic ULP Wake

```python
import machine
import esp32

# ULP support is available on ESP32 / ESP32-S2 / ESP32-S3 (not ESP32-C3/C6).
# Using the ULP requires loading a pre-built ULP binary program.
ulp = esp32.ULP()

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
2. Keep data minimal (RTC user memory is limited; esp32 default is 2048 bytes)
3. Store to flash for larger data
4. Check reset_cause() to detect wake

### WiFi won't reconnect

1. Always reinitialize WiFi after deep sleep
2. Add delay before connection attempt
3. Use wlan.disconnect() before reconnect

### High current in sleep

1. Ensure WiFi/BT are off
2. Disable pull-ups/pull-downs on RTC pins if they can cause leakage
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

## References

- MicroPython `machine` module (sleep, reset cause, freq): https://docs.micropython.org/en/latest/library/machine.html
- MicroPython `machine.RTC` (RTC user memory persistence/size): https://docs.micropython.org/en/latest/library/machine.RTC.html
- MicroPython ESP32 quick reference (deep-sleep notes, pin behavior): https://docs.micropython.org/en/latest/esp32/quickref.html
- MicroPython `esp32` module (EXT0/EXT1/Touch/ULP wake configuration): https://docs.micropython.org/en/latest/library/esp32.html
- MicroPython `network.WLAN` (WiFi power management `pm`): https://docs.micropython.org/en/latest/library/network.WLAN.html
- MicroPython `bluetooth` module (`bluetooth.BLE().active(False)`): https://docs.micropython.org/en/latest/library/bluetooth.html
