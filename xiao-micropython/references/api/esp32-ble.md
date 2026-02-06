# ESP32 BLE API (MicroPython)

## Overview

ESP32 series (C3, C6, S3) support Bluetooth Low Energy (BLE) with MicroPython's `bluetooth` module.

## Basic Concepts

BLE has two main roles:
- **Peripheral**: Advertises data, accepts connections (e.g., sensor, beacon)
- **Central**: Scans for devices, initiates connections (e.g., phone, gateway)

MicroPython's BLE API supports operating in multiple roles concurrently, but practical limits (connections, throughput, memory) depend on your firmware build and BLE stack.

## Quick Start - Peripheral

```python
import bluetooth
import time

# Initialize BLE
ble = bluetooth.BLE()
ble.active(True)

# Advertise
ble.gap_advertise(100000, b'\x02\x01\x06\x02\x0A\x00')  # 100ms interval (interval_us)

# Set device name
ble.config('gap_name', 'XIAO-ESP32')

print("BLE active, advertising...")
print("MAC:", ble.config('mac')[1])

while True:
    time.sleep(1)
```

## Quick Start - Central

```python
import bluetooth
import time

# Initialize BLE
from micropython import const

_IRQ_SCAN_RESULT = const(5)

ble = bluetooth.BLE()
ble.active(True)

def irq_handler(event, data):
    if event == _IRQ_SCAN_RESULT:  # Scan result
        addr_type, addr, adv_type, rssi, adv_data = data
        print(f"Found: {bytes(addr).hex()}, RSSI: {rssi}")

ble.irq(irq_handler)

# Scan for 10 seconds
ble.gap_scan(10000)
```

## Advertising

### Simple Advertisement

```python
import bluetooth

ble = bluetooth.BLE()
ble.active(True)

# Advertise with 100ms interval
ble.gap_advertise(100000)

# Stop advertising
ble.gap_advertise(None)
```

### Custom Advertisement Data

```python
import bluetooth
import struct

ble = bluetooth.BLE()
ble.active(True)

# Build advertising data
# Flags: discoverable, BLE only
# UUID: 0x181A - Environmental Sensing
adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags
adv_data += b'\x03\x03\x1A\x18'  # UUID 0x181A

ble.gap_advertise(100, adv_data)
```

### Advertisement with Name

```python
import bluetooth
import struct

ble = bluetooth.BLE()
ble.active(True)

# Set device name
ble.config('gap_name', 'XIAO-Sensor')

# Advertising data with complete name (AD type 0x09)
name = b'XIAO-Sensor'
adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags
adv_data += struct.pack('BB', len(name) + 1, 0x09) + name

ble.gap_advertise(100, adv_data)
```

### Advertisement with Service Data

```python
import bluetooth
import struct

ble = bluetooth.BLE()
ble.active(True)

# Environmental sensing service data (temperature)
temp = 2500  # 25.00°C in 0.01°C units

adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags
adv_data += b'\x03\x03\x1A\x18'  # Environmental Sensing UUID
adv_data += struct.pack("<BBH", 5, 0x16, 0x181A)  # Service data
adv_data += struct.pack("<h", temp)  # Temperature value

ble.gap_advertise(100, adv_data)
```

## GAP Events (IRQ)

```python
import bluetooth

from micropython import const

_IRQ_CENTRAL_CONNECT = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)
_IRQ_GATTS_WRITE = const(3)

ble = bluetooth.BLE()
ble.active(True)

def irq(event, data):
    if event == _IRQ_CENTRAL_CONNECT:
        conn_handle, addr_type, addr = data
        print(f"Connected: {bytes(addr).hex()}")

    elif event == _IRQ_CENTRAL_DISCONNECT:
        conn_handle, addr_type, addr = data
        print(f"Disconnected: {bytes(addr).hex()}")
        # Restart advertising
        ble.gap_advertise(100000)

    elif event == _IRQ_GATTS_WRITE:
        conn_handle, value_handle = data
        print(f"Written to handle: {value_handle}")

ble.irq(irq)
```

## GATT Server

### Creating a Service

```python
import bluetooth
import struct

ble = bluetooth.BLE()
ble.active(True)

# UUIDs
ENV_SENSING_SERVICE = bluetooth.UUID(0x181A)
TEMP_CHAR = bluetooth.UUID(0x2A6E)
HUM_CHAR = bluetooth.UUID(0x2A6F)

# Create service
SERVICE = (
    ENV_SENSING_SERVICE,
    (
        (TEMP_CHAR, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
        (HUM_CHAR, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
    ),
)

# Register services
services = (SERVICE,)
handles = ble.gatts_register_services(services)

temp_handle = handles[0][0]
hum_handle = handles[0][1]

# Set initial values
ble.gatts_write(temp_handle, struct.pack("<h", 2500))  # 25.00°C
ble.gatts_write(hum_handle, struct.pack("<H", 5000))  # 50.00%

# Start advertising
ble.gap_advertise(100)

print("GATT server ready")
```

### Notifying Characteristic Changes

```python
import bluetooth
import struct
import time

from micropython import const

_IRQ_CENTRAL_CONNECT = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)

ble = bluetooth.BLE()
ble.active(True)

# Setup GATT server (see above)
# ... register service ...

# Connection tracking
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
        print("Disconnected")

ble.irq(irq)

# Advertise
ble.gap_advertise(100000)

# Update loop
counter = 0
while True:
    if connected:
        # Update temperature
        temp = 2500 + (counter % 100)  # 25.00°C +/- 0.5°C
        ble.gatts_write(temp_handle, struct.pack("<h", temp))

        # Notify subscribers
        ble.gatts_notify(conn_handle, temp_handle)

        counter += 1

    time.sleep(1)
```

### Readable Characteristics

```python
import bluetooth

ble = bluetooth.BLE()
ble.active(True)

# Register service with readable characteristics
SERVICE = (
    bluetooth.UUID(0x1800),  # Generic Access
    (
        (bluetooth.UUID(0x2A00), bluetooth.FLAG_READ),  # Device Name
    ),
)

handles = ble.gatts_register_services((SERVICE,))
name_handle = handles[0][0]

# Set device name
ble.gatts_write(name_handle, b'XIAO-ESP32')
```

### Writable Characteristics

```python
import bluetooth

from micropython import const

_IRQ_GATTS_WRITE = const(3)

ble = bluetooth.BLE()
ble.active(True)

UART_SERVICE = bluetooth.UUID('6E400001-B5A3-F393-E0A9-E50E24DCCA9E')
UART_RX = (bluetooth.UUID('6E400002-B5A3-F393-E0A9-E50E24DCCA9E'), bluetooth.FLAG_WRITE)

SERVICE = (UART_SERVICE, (UART_RX,))

handles = ble.gatts_register_services((SERVICE,))
rx_handle = handles[0][0]

def irq(event, data):
    if event == _IRQ_GATTS_WRITE:
        conn_handle, value_handle = data
        if value_handle == rx_handle:
            value = ble.gatts_read(rx_handle)
            print(f"RX wrote: {value}")

ble.irq(irq)
```

## GATT Client

### Connecting to Peripheral

```python
import bluetooth
import time

from micropython import const

_IRQ_PERIPHERAL_CONNECT = const(7)
_IRQ_GATTC_SERVICE_RESULT = const(9)
_IRQ_GATTC_CHARACTERISTIC_RESULT = const(11)

ble = bluetooth.BLE()
ble.active(True)

# Target device address (example)
TARGET_ADDR = b'\x11\x22\x33\x44\x55\x66'
TARGET_ADDR_TYPE = 0  # 0x00 public, 0x01 random

def irq(event, data):
    if event == _IRQ_PERIPHERAL_CONNECT:
        conn_handle, addr_type, addr = data
        print(f"Connected to {bytes(addr).hex()}")

        # Discover services
        ble.gattc_discover_services(conn_handle)

    elif event == _IRQ_GATTC_SERVICE_RESULT:
        conn_handle, start_handle, end_handle, uuid = data
        print(f"Service: {uuid}")

        # Discover characteristics
        ble.gattc_discover_characteristics(conn_handle, start_handle, end_handle)

    elif event == _IRQ_GATTC_CHARACTERISTIC_RESULT:
        conn_handle, end_handle, value_handle, properties, uuid = data
        print(f"Characteristic: {uuid}, handle: {value_handle}")

ble.irq(irq)

# Scan first
ble.gap_scan(2000)

# Then connect
ble.gap_connect(TARGET_ADDR_TYPE, TARGET_ADDR)
```

### Reading Characteristic

```python
import bluetooth

from micropython import const

_IRQ_GATTC_READ_RESULT = const(15)

ble = bluetooth.BLE()
ble.active(True)

value_handle = None  # From discovery

def irq(event, data):
    global value_handle

    if event == _IRQ_GATTC_READ_RESULT:
        conn_handle, value_handle, char_data = data
        print(f"Read: {char_data}")

# Read characteristic
ble.gattc_read(conn_handle, value_handle)
```

### Writing Characteristic

```python
import bluetooth
import struct

ble = bluetooth.BLE()
ble.active(True)

value_handle = None  # From discovery

# mode=0 (default): write-without-response
ble.gattc_write(conn_handle, value_handle, b'\x01', 0)

# mode=1: write-with-response
ble.gattc_write(conn_handle, value_handle, struct.pack("<h", 2500), 1)
```

### Subscribing to Notifications

```python
import bluetooth

from micropython import const

_IRQ_GATTC_NOTIFY = const(18)

ble = bluetooth.BLE()
ble.active(True)

def irq(event, data):
    if event == _IRQ_GATTC_NOTIFY:
        conn_handle, value_handle, notify_data = data
        print(f"Notified: {notify_data}")

# Subscribe to notifications.
# Note: CCCD is often at value_handle+1 but this is not guaranteed. Prefer to
# discover descriptors and find UUID 0x2902.
ble.gattc_write(conn_handle, value_handle + 1, b'\x01\x00', 1)
```

## Complete Example - BLE Environmental Sensor

```python
import bluetooth
import struct
import time
import machine
from machine import Pin

from micropython import const

_IRQ_CENTRAL_CONNECT = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)

# Hardware
led = Pin(10, Pin.OUT)

# BLE setup
ble = bluetooth.BLE()
ble.active(True)

# UUIDs
ENV_SERVICE = bluetooth.UUID(0x181A)
TEMP_CHAR = bluetooth.UUID(0x2A6E)
HUM_CHAR = bluetooth.UUID(0x2A6F)
PRESS_CHAR = bluetooth.UUID(0x2A6D)

# Service definition
SERVICE = (
    ENV_SERVICE,
    (
        (TEMP_CHAR, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
        (HUM_CHAR, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
        (PRESS_CHAR, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY),
    ),
)

# Register
handles = ble.gatts_register_services((SERVICE,))
temp_h, hum_h, press_h = handles[0]

# Connection state
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
        ble.gap_advertise(100000)

ble.irq(irq)

# Advertise
ble.config('gap_name', 'XIAO-Weather')
adv_data = bytearray()
adv_data += b'\x02\x01\x06'  # Flags
adv_data += b'\x03\x03\x1A\x18'  # Environmental Sensing UUID
ble.gap_advertise(100000, adv_data)

# Simulate sensor data
counter = 0
while True:
    # Simulated values
    temp = 2500 + (counter % 100) - 50  # 20.00 - 30.00°C
    hum = 5000 + (counter % 200) - 100  # 40.00 - 60.00%
    press = 100000 + (counter % 500) - 250  # ~1000 hPa

    # Update characteristics
    ble.gatts_write(temp_h, struct.pack("<h", temp))
    ble.gatts_write(hum_h, struct.pack("<H", hum))
    ble.gatts_write(press_h, struct.pack("<i", press))

    # Notify if connected
    if connected:
        ble.gatts_notify(conn_handle, temp_h)
        ble.gatts_notify(conn_handle, hum_h)
        ble.gatts_notify(conn_handle, press_h)

    print(f"Temp: {temp/100:.1f}°C, Hum: {hum/100:.1f}%, Press: {press/100:.1f}hPa")

    counter += 1
    time.sleep(5)
```

## BLE Scanner

```python
import bluetooth
import time

from micropython import const

_IRQ_SCAN_RESULT = const(5)

ble = bluetooth.BLE()
ble.active(True)

devices = {}

def irq(event, data):
    if event == _IRQ_SCAN_RESULT:
        addr_type, addr, adv_type, rssi, adv_data = data
        addr_str = bytes(addr).hex()

        if addr_str not in devices:
            devices[addr_str] = {
                'addr_type': addr_type,
                'rssi': rssi,
                'adv_data': adv_data
            }
            print(f"New device: {addr_str}, RSSI: {rssi}")
        else:
            devices[addr_str]['rssi'] = rssi

ble.irq(irq)

# Start scan
print("Scanning...")
ble.gap_scan(5000)  # 5 seconds

# Wait for scan to complete
time.sleep(6)

# Show all devices
print(f"\nFound {len(devices)} devices:")
for addr, info in devices.items():
    print(f"  {addr}: RSSI {info['rssi']}")
```

## Advertising Types

```python
import bluetooth

ble = bluetooth.BLE()
ble.active(True)

# Connectable undirected (default)
ble.gap_advertise(100000)

# Non-connectable (beacon)
ble.gap_advertise(100000, connectable=False)

# Faster interval (higher power)
ble.gap_advertise(20000)  # 20ms (interval_us)
```

## Connection Parameters

```python
import bluetooth

ble = bluetooth.BLE()
ble.active(True)

# Set preferred ATT MTU. MTU exchange is not automatic and is typically
# initiated by the central (use ble.gattc_exchange_mtu(conn_handle)).
ble.config(mtu=512)
```

## Power Management

```python
import bluetooth
import machine

ble = bluetooth.BLE()
ble.active(True)

# Low power mode
ble.config(rxbuf=512)  # Smaller event ring buffer
ble.config(gap_name='XIAO')  # Shorter name = less data

# Sleep between advertising
ble.gap_advertise(100)
machine.lightsleep(5000)  # Sleep 5 seconds
```

## Troubleshooting

### BLE not starting

1. Check if another BLE device is nearby interfering
2. Reset board and try again
3. Check firmware includes `ubluetooth` module

### Can't discover services

1. Ensure connected first
2. Use correct handle range
3. Check UUIDs are correct format

### Notifications not working

1. Ensure CCC (Client Characteristic Configuration) is written
2. Check notify flag is set on characteristic
3. Verify connection is still active

### Connection drops

1. Adjust connection parameters
2. Check power supply stability
3. Reduce MTU if needed
4. Add reconnect logic

## Best Practices

1. **Advertise sparingly** - Use longer intervals for battery devices
2. **Minimize notification data** - Send only changed values
3. **Handle disconnections** - Always implement reconnect logic
4. **Use appropriate MTU** - Larger MTU for data transfer
5. **Set device name** - Helps identification
6. **Cache handles** - Store discovered handles for reuse
7. **Use GAP events** - Don't poll, use IRQ handler

## Platform Support

| Board | BLE Support | Notes |
|-------|-------------|-------|
| ESP32C3/C6/S3 | ✅ | Requires MicroPython firmware built with BLE support |
| RP2040 | ❌ | No radio |
| nRF52840 | ✅ | MicroPython supports BLE (module name differs by port) |

## Related Resources

- `nrf-ble.md` - nRF52 BLE API
- `../examples/ble-peripheral.md` - BLE sensor example

## References

- MicroPython official BLE API docs (latest): https://docs.micropython.org/en/latest/library/bluetooth.html
- MicroPython source (function signatures and config keys): https://github.com/micropython/micropython/blob/master/extmod/modbluetooth.c
