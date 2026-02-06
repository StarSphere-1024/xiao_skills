# nRF52 BLE API (MicroPython)

## Overview

XIAO nRF52840 and nRF52840-Sense feature the Nordic nRF52840 chip with Bluetooth Low Energy (BLE) support. In MicroPython, BLE support depends on the firmware build.

## Key Features

- **BLE**: Feature set depends on SoC + firmware build
- **Multi-role**: Simultaneous peripheral, central, broadcaster, observer
- **Ultra-low power**: Designed for battery-powered applications
- **NFC**: nRF52840 includes NFC-A tag hardware (MicroPython support varies by firmware)
- **USB**: Built-in USB for debugging and programming

## Quick Start - Peripheral (Beacon)

```python
import bluetooth
import time

# Initialize BLE (nRF uses 'bluetooth' not 'ubluetooth')
ble = bluetooth.BLE()
ble.active(True)

# Simple advertisement (MicroPython interval is in microseconds)
ble.gap_advertise(100000)  # 100ms interval

print("Advertising...")
print("MAC:", ble.config('mac')[1])

while True:
    time.sleep(1)
```

## Quick Start - Central (Scanner)

```python
import bluetooth
import time
from micropython import const

ble = bluetooth.BLE()
ble.active(True)

_IRQ_SCAN_RESULT = const(5)

def irq_handler(event, data):
    if event == _IRQ_SCAN_RESULT:
        addr_type, addr, adv_type, rssi, adv_data = data
        print(f"Found: {bytes(addr).hex()}, RSSI: {rssi}")

ble.irq(irq_handler)

# Scan
ble.gap_scan(10000)

time.sleep(11)  # Wait for scan
```

## nRF52 vs ESP32 BLE Differences

| Feature | ESP32 | nRF52 |
|---------|-------|-------|
| Module | `bluetooth` | `bluetooth` |
| BLE Version | SoC/firmware dependent | SoC/firmware dependent |
| Power | Higher | Ultra-low |
| Advertising interval | Configured via `gap_advertise(interval_us, ...)` | Configured via `gap_advertise(interval_us, ...)` |
| Max connections | Firmware/stack dependent | Firmware/stack dependent |
| throughput | Application dependent | Application dependent |

## GATT Server with Environmental Sensing

```python
import bluetooth
import struct
import time
from machine import Pin
from micropython import const

# Initialize
ble = bluetooth.BLE()
ble.active(True)

_IRQ_CENTRAL_CONNECT = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)

# LED indicator
led = Pin(13, Pin.OUT)

# UUIDs
ENV_SENSING = bluetooth.UUID(0x181A)
TEMP_CHAR = bluetooth.UUID(0x2A6E)
HUMIDITY_CHAR = bluetooth.UUID(0x2A6F)

# Service
SERVICE = (
    ENV_SENSING,
    (
        (TEMP_CHAR, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
        (HUMIDITY_CHAR, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
    ),
)

# Register services
handles = ble.gatts_register_services((SERVICE,))
temp_handle = handles[0][0]
humidity_handle = handles[0][1]

# Set initial values
ble.gatts_write(temp_handle, struct.pack("<h", 2200))  # 22.00°C
ble.gatts_write(humidity_handle, struct.pack("<H", 5500))  # 55.00%

# Connection tracking
connected = False
conn_handle = None

def irq(event, data):
    global connected, conn_handle

    if event == _IRQ_CENTRAL_CONNECT:
        conn_handle, addr_type, addr = data
        connected = True
        led.value(1)
        print(f"Connected: {bytes(addr).hex()}")

    elif event == _IRQ_CENTRAL_DISCONNECT:
        connected = False
        conn_handle = None
        led.value(0)
        print("Disconnected")
        # Restart advertising
        ble.gap_advertise(100000, adv_data)

ble.irq(irq)

# Advertise
ble.config('gap_name', 'XIAO-nRF')
adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags
adv_data += b'\x03\x03\x1A\x18'  # Environmental Sensing
ble.gap_advertise(100000, adv_data)

print("Environmental sensor ready")

# Update loop
counter = 0
while True:
    if connected:
        # Simulate sensor data
        temp = 2200 + (counter % 50) - 25  # 21.75 - 22.25°C
        hum = 5500 + (counter % 200) - 100  # 54.00 - 56.00%

        # Update characteristics
        ble.gatts_write(temp_handle, struct.pack("<h", temp))
        ble.gatts_write(humidity_handle, struct.pack("<H", hum))

        # Notify
        ble.gatts_notify(conn_handle, temp_handle)
        ble.gatts_notify(conn_handle, humidity_handle)

        print(f"Temp: {temp/100:.2f}°C, Hum: {hum/100:.2f}%")

    counter += 1
    time.sleep(5)
```

## BLE Heart Rate Sensor

```python
import bluetooth
import struct
import time
from micropython import const

ble = bluetooth.BLE()
ble.active(True)

_IRQ_CENTRAL_CONNECT = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)

# Heart Rate Service
HR_SERVICE = bluetooth.UUID(0x180D)
HR_MEASUREMENT = bluetooth.UUID(0x2A37)
BODY_SENSOR_LOC = bluetooth.UUID(0x2A38)

SERVICE = (
    HR_SERVICE,
    (
        (HR_MEASUREMENT, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
        (BODY_SENSOR_LOC, bluetooth.FLAG_READ),
    ),
)

handles = ble.gatts_register_services((SERVICE,))
hr_handle = handles[0][0]
loc_handle = handles[0][1]

# Set sensor location (0x02: Wrist)
ble.gatts_write(loc_handle, b'\x02')

connected = False
conn_handle = None

def irq(event, data):
    global connected, conn_handle

    if event == _IRQ_CENTRAL_CONNECT:
        conn_handle, addr_type, addr = data
        connected = True
        print("Connected")

    elif event == _IRQ_CENTRAL_DISCONNECT:
        connected = False
        conn_handle = None
        ble.gap_advertise(100000, adv_data)

ble.irq(irq)

# Advertise heart rate
ble.config('gap_name', 'XIAO-HR')
adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags
adv_data += b'\x03\x03\x0D\x18'  # Heart Rate Service
ble.gap_advertise(100000, adv_data)

# Simulate heart rate
import random
while True:
    if connected:
        bpm = 70 + random.randint(-5, 10)
        # Heart Rate Measurement format (flags=00, BPM value)
        ble.gatts_write(hr_handle, struct.pack("<BB", 0x00, bpm))
        ble.gatts_notify(conn_handle, hr_handle)
        print(f"Heart rate: {bpm} BPM")

    time.sleep(1)
```

## GATT Client - Connect to Sensor

```python
import bluetooth
import time
from micropython import const

ble = bluetooth.BLE()
ble.active(True)

_IRQ_SCAN_RESULT = const(5)
_IRQ_PERIPHERAL_CONNECT = const(7)
_IRQ_PERIPHERAL_DISCONNECT = const(8)
_IRQ_GATTC_SERVICE_RESULT = const(9)
_IRQ_GATTC_CHARACTERISTIC_RESULT = const(11)
_IRQ_GATTC_READ_RESULT = const(15)
_IRQ_GATTC_NOTIFY = const(18)

# Target device address (discover with scan first)
TARGET_ADDR = None

def irq(event, data):
    if event == _IRQ_PERIPHERAL_CONNECT:
        conn_handle, addr_type, addr = data
        print(f"Connected to {bytes(addr).hex()}")
        # Discover services
        ble.gattc_discover_services(conn_handle)

    elif event == _IRQ_GATTC_SERVICE_RESULT:
        conn_handle, start_handle, end_handle, uuid = data
        print(f"Service {uuid}: {start_handle}-{end_handle}")
        # Discover characteristics
        ble.gattc_discover_characteristics(conn_handle, start_handle, end_handle)

    elif event == _IRQ_GATTC_CHARACTERISTIC_RESULT:
        conn_handle, def_handle, val_handle, properties, uuid = data
        print(f"  Characteristic {uuid}: handle={val_handle}")

        # Read if readable
        if properties & 0x02:  # FLAG_READ
            ble.gattc_read(conn_handle, val_handle)

    elif event == _IRQ_GATTC_READ_RESULT:
        conn_handle, value_handle, char_data = data
        print(f"    Read: {char_data}")

    elif event == _IRQ_GATTC_NOTIFY:
        conn_handle, value_handle, notify_data = data
        print(f"    Notify: {notify_data}")

    elif event == _IRQ_SCAN_RESULT:
        addr_type, addr, adv_type, rssi, adv_data = data
        addr_bytes = bytes(addr)
        addr_str = addr_bytes.hex()
        print(f"Found {addr_str}, RSSI: {rssi}")
        # Auto-connect to specific device
        if TARGET_ADDR and addr_bytes == TARGET_ADDR:
            ble.gap_connect(addr_type, addr_bytes)

ble.irq(irq)

# Scan first
print("Scanning...")
ble.gap_scan(5000)

time.sleep(6)
```

## Beacon (iBeacon Format)

```python
import bluetooth
import struct

ble = bluetooth.BLE()
ble.active(True)

# iBeacon data
# UUID: 0x01122334-4556-6778-899A-ABBCCDDEEFF0
# Major: 1
# Minor: 1
# TX Power: -59 dBm

UUID_BYTES = bytes.fromhex('0112233445566778899AABBCCDDEEFF0')
MAJOR = 1
MINOR = 1
TX_POWER = 0xC5  # -59 dBm

# Build manufacturer data (Apple 0x004C)
mfg_data = bytearray()
mfg_data += b'\x4C\x00'  # Apple Company ID
mfg_data += b'\x02'  # iBeacon type
mfg_data += b'\x15'  # Data length
mfg_data += UUID_BYTES  # UUID (big endian)
mfg_data += struct.pack(">H", MAJOR)  # Major
mfg_data += struct.pack(">H", MINOR)  # Minor
mfg_data += bytes([TX_POWER])  # TX power

# Advertisement data
adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags

# Manufacturer specific data
adv_data += bytes([len(mfg_data) + 1, 0xFF])  # Length, Type
adv_data += mfg_data

# Start advertising (non-connectable)
ble.gap_advertise(100000, adv_data, connectable=False)

print("iBeacon advertising")
```

## Eddystone URL Beacon

```python
import bluetooth

ble = bluetooth.BLE()
ble.active(True)

# Eddystone-URL
# URL: https://seeed.io
url = b'https://seeed.io'

# Build Eddystone URL frame
frame = bytearray()
frame += b'\x00'  # Frame type (URL)
frame += b'\x00'  # TX power (-0dBm = 0x00)
frame += b'\x03'  # URL scheme: https://
frame += url[8:]  # URL after scheme (or use encoding)

# Service Data AD structure uses a 16-bit UUID followed by service data.
# For Eddystone-URL the UUID is 0xFEAA (little-endian in the payload).
service_data = b'\xAA\xFE' + frame

# Advertisement
adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags
adv_data += b'\x03\x03\xAA\xFE'  # Eddystone UUID

# Add service data
adv_data += bytes([len(service_data) + 1, 0x16])
adv_data += service_data

# Non-connectable
ble.gap_advertise(100000, adv_data, connectable=False)

print("Eddystone URL beacon")
```

## Battery Service

```python
import bluetooth
import struct
from micropython import const

ble = bluetooth.BLE()
ble.active(True)

_IRQ_CENTRAL_CONNECT = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)

# Battery Service
BATT_SERVICE = bluetooth.UUID(0x180F)
BATT_LEVEL = bluetooth.UUID(0x2A19)

SERVICE = (
    BATT_SERVICE,
    (
        (BATT_LEVEL, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
    ),
)

handles = ble.gatts_register_services((SERVICE,))
batt_handle = handles[0][0]

# Simulate battery level
def read_battery():
    # Read actual battery voltage if available
    # For now, simulate 85%
    return 85

connected = False
conn_handle = None

def irq(event, data):
    global connected, conn_handle

    if event == _IRQ_CENTRAL_CONNECT:
        conn_handle, addr_type, addr = data
        connected = True

    elif event == _IRQ_CENTRAL_DISCONNECT:
        connected = False
        conn_handle = None
        ble.gap_advertise(100000, adv_data)

ble.irq(irq)

# Advertise
ble.config('gap_name', 'XIAO-Batt')
adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags
adv_data += b'\x03\x03\x0F\x18'  # Battery Service
ble.gap_advertise(100000, adv_data)

# Update battery
import time
while True:
    level = read_battery()

    # Update characteristic
    ble.gatts_write(batt_handle, bytes([level]))

    # Notify if connected
    if connected:
        ble.gatts_notify(conn_handle, batt_handle)

    print(f"Battery: {level}%")
    time.sleep(60)
```

## Custom Service with Multiple Characteristics

```python
import bluetooth
import struct
from micropython import const

ble = bluetooth.BLE()
ble.active(True)

_IRQ_GATTS_WRITE = const(3)

# Custom 128-bit UUID
CUSTOM_SERVICE_UUID = bluetooth.UUID('00001101-0000-1000-8000-00805F9B34FB')

# Characteristics
RX_CHAR = bluetooth.UUID('00001142-0000-1000-8000-00805F9B34FB')
TX_CHAR = bluetooth.UUID('00001143-0000-1000-8000-00805F9B34FB')

SERVICE = (
    CUSTOM_SERVICE_UUID,
    (
        (RX_CHAR, bluetooth.FLAG_WRITE | bluetooth.FLAG_WRITE_NO_RESPONSE),
        (TX_CHAR, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
    ),
)

handles = ble.gatts_register_services((SERVICE,))
rx_handle = handles[0][0]
tx_handle = handles[0][1]

def irq(event, data):
    if event == _IRQ_GATTS_WRITE:
        conn_handle, value_handle = data
        if value_handle == rx_handle:
            received = ble.gatts_read(rx_handle)
            print(f"Received: {received}")

            # Echo back
            ble.gatts_write(tx_handle, received)
            ble.gatts_notify(conn_handle, tx_handle)

ble.irq(irq)

# Advertise
ble.config('gap_name', 'XIAO-Custom')
ble.gap_advertise(100000)

print("Custom service ready")
```

## Low Power Beacon

```python
import bluetooth
import machine
import time

ble = bluetooth.BLE()
ble.active(True)

# Simple beacon data
adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags
adv_data += b'\x05\x09\x58\x49\x41\x4F'  # "XIAO"

# Non-connectable advertising
ble.gap_advertise(1000000, adv_data, connectable=False)

print("Low power beacon")

# Sleep between advertising
while True:
    print("Advertising for 10 seconds...")
    time.sleep(10)

    print("Sleeping...")
    # Turn off radio
    ble.gap_advertise(None)

    # Sleep for 50 seconds
    machine.lightsleep(50000)

    # Wake and resume
    ble.gap_advertise(1000000, adv_data, connectable=False)
```

## RSSI Distance Estimation

```python
import bluetooth
from micropython import const

ble = bluetooth.BLE()
ble.active(True)

_IRQ_SCAN_RESULT = const(5)

devices = {}

def irq(event, data):
    if event == _IRQ_SCAN_RESULT:
        addr_type, addr, adv_type, rssi, adv_data = data
        addr_str = bytes(addr).hex()

        # Estimate distance (very rough heuristic; environment dependent)
        tx_power = -59  # Typical BLE TX power
        if rssi != 0:
            distance = 10 ** ((tx_power - rssi) / 20)
        else:
            distance = 0

        if addr_str not in devices or abs(devices[addr_str]['distance'] - distance) > 0.5:
            devices[addr_str] = {
                'rssi': rssi,
                'distance': distance
            }
            print(f"{addr_str}: {distance:.1f}m (RSSI: {rssi})")

ble.irq(irq)

# Scan
ble.gap_scan(10000)

import time
time.sleep(11)

print("\nDevice summary:")
for addr, info in devices.items():
    print(f"  {addr}: {info['distance']:.1f}m")
```

## Connection Interval Optimization

```python
import bluetooth
from micropython import const

ble = bluetooth.BLE()
ble.active(True)

_IRQ_CENTRAL_CONNECT = const(1)
_IRQ_CONNECTION_UPDATE = const(27)

def irq(event, data):
    if event == _IRQ_CENTRAL_CONNECT:
        conn_handle, addr_type, addr = data
        print(f"Connected: {conn_handle}")

    elif event == _IRQ_CONNECTION_UPDATE:
        conn_handle, conn_interval, conn_latency, supervision_timeout = data
        print(f"Parameters updated: interval={conn_interval*1.25}ms, latency={conn_latency}")

ble.irq(irq)

ble.gap_advertise(100000)
```

## NFC Tag (nRF52840 Only)

```python
import machine

# NFC support varies by MicroPython firmware build.
try:
    nfc = machine.NFC()
    print("NFC available")
except:
    print("NFC not available in this firmware")
```

## Troubleshooting

### Module import error

```
ImportError: no module named 'bluetooth'
```

- Ensure nRF52840 MicroPython firmware is flashed
- Some builds may not include BLE (use official firmware)

### High power consumption

1. Reduce advertising interval
2. Use longer connection intervals
3. Turn off radio when not needed
4. Use lightsleep between advertising bursts
5. Minimize notification frequency

### Can't connect

1. Ensure device is advertising
2. Check advertising interval (e.g. 100ms = 100000us)
3. Verify device name matches
4. Check connection parameters
5. Ensure you are not at the firmware/stack connection limit

### Notifications not received

1. Ensure CCC is written by client
2. Verify notify flag is set
3. Check connection is still active
4. Verify handle is correct

## Best Practices

1. **Use long advertising intervals** for battery devices (1s-10s)
2. **Minimize notification data** - send only what changed
3. **Optimize connection parameters** - balance latency vs power
4. **Implement reconnection logic** - handle disconnections gracefully
5. **Use appropriate TX power** - lower for battery, higher for range
6. **Cache handles** - store discovered handles
7. **Handle all IRQ events** - don't ignore events

## Power Tips

nRF52 is designed for ultra-low power:

```python
# Long advertising interval (10 seconds)
ble.gap_advertise(10_000_000)  # Battery friendly (10s)

# Sleep between advertising
while True:
    ble.gap_advertise(100000)
    time.sleep(5)  # Advertise 5 seconds
    ble.gap_advertise(None)
    machine.lightsleep(55000)  # Sleep 55 seconds
```

## Platform Support

| Board | BLE | Notes |
|-------|-----|-------|
| nRF52840 | ✅ | BLE support (verify firmware build) |
| nRF52840-Sense | ✅ | + IMU, MIC |
| nRF54L15-Sense | ✅ | BLE support (verify firmware build) |
| ESP32C3/C6/S3 | ✅ | Use `bluetooth` module |
| RP2040 | ❌ | No radio |

## Related Resources

- `esp32-ble.md` - ESP32 BLE API
- `nrf-sleep.md` - nRF52 sleep modes
- `../examples/ble-peripheral.md` - BLE sensor example
