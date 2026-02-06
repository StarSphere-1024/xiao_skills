# ESP32 WiFi API for MicroPython

## Overview

WiFi networking on ESP32-based XIAO boards using MicroPython's `network` module.

Note: MicroPython supports multiple ESP32 variants (ESP32, ESP32-C3/C6, ESP32-S2/S3) and features can differ by SoC and by MicroPython port/build. This page documents the portable `network.WLAN` API first; ESP32-port-specific knobs are called out explicitly.

## Station Mode (WiFi Client)

### Basic Connection

```python
import network

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)
wlan.connect('your-SSID', 'your-password')

# Wait for connection
while not wlan.isconnected():
    pass

print('Connected:', wlan.ifconfig())
```

### Connection Status

```python
import network
import time

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)
wlan.connect('SSID', 'password')

# Check status
status = wlan.status()
print(f"Status: {status}")

# Portable status constants are defined in the `network` module.
# The exact numeric values are port-specific, so prefer comparing to constants:
# - network.STAT_IDLE
# - network.STAT_CONNECTING
# - network.STAT_WRONG_PASSWORD
# - network.STAT_NO_AP_FOUND
# - network.STAT_CONNECT_FAIL
# - network.STAT_GOT_IP

# Wait with timeout
timeout = 10
start = time.time()
while not wlan.isconnected() and time.time() - start < timeout:
    time.sleep(0.5)

if wlan.isconnected():
    print("Connected!")
else:
    print("Connection failed")
```

### Scan Networks

```python
import network

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)

networks = wlan.scan()
for net in networks:
    ssid = net[0].decode('utf-8')
    bssid = net[1]
    channel = net[2]
    rssi = net[3]
    authmode = net[4]
    hidden = net[5]
    print(f"{ssid}: {rssi} dBm, Ch {channel}")
```

### Static IP

```python
import network

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)

# Static IP configuration
# Note: `ifconfig()` is deprecated in MicroPython. Prefer `ipconfig()` if your port/firmware supports it.
wlan.ifconfig(('192.168.1.100', '255.255.255.0', '192.168.1.1', '8.8.8.8'))
wlan.connect('SSID', 'password')

# Verify
print(wlan.ifconfig())
```

### Disconnect

```python
wlan.disconnect()
wlan.active(False)
```

## Access Point Mode

### Basic AP

```python
import network

ap = network.WLAN(network.WLAN.IF_AP)
ap.active(True)
ap.config(ssid='XIAO-AP', password='12345678')

print('AP running:', ap.ifconfig())
```

### AP with DHCP

```python
import network

ap = network.WLAN(network.WLAN.IF_AP)
ap.active(True)

# Configure AP
ap.config(ssid='XIAO-AP',
          password='password123',
          channel=11,
          hidden=False)

print('AP ready, IP:', ap.ifconfig()[0])
```

### Open AP (No Password)

```python
import network

ap = network.WLAN(network.WLAN.IF_AP)
ap.active(True)
# ESP32 port: `network.AUTH_OPEN` is available as an authmode value.
ap.config(ssid='XIAO-Open', authmode=network.AUTH_OPEN)
```

## AP + STA Simultaneous

Note: Creating both interfaces is supported. Whether both can be active and usable at the same time may depend on the ESP32 port/build and underlying ESP-IDF configuration.

```python
import network

# Station
sta = network.WLAN(network.WLAN.IF_STA)
sta.active(True)
sta.connect('HomeWiFi', 'password')

# Access Point
ap = network.WLAN(network.WLAN.IF_AP)
ap.active(True)
ap.config(ssid='XIAO-Hotspot')

print('STA IP:', sta.ifconfig()[0])
print('AP IP:', ap.ifconfig()[0])
```

## Power Management

### WiFi Power Save

```python
import network

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)

# WiFi power management is controlled via `WLAN.config(pm=...)`.
# Allowed values are `network.WLAN.PM_PERFORMANCE`, `network.WLAN.PM_POWERSAVE`, `network.WLAN.PM_NONE`.
wlan.config(pm=network.WLAN.PM_POWERSAVE)

wlan.connect('SSID', 'password')
```

### Disable WiFi for Low Power

```python
import network
import machine

# Disconnect and disable
wlan = network.WLAN(network.WLAN.IF_STA)
wlan.disconnect()
wlan.active(False)

# Enter deep sleep
machine.deepsleep(60000)  # 60 seconds
```

## WiFi Events

MicroPython's portable `network.WLAN` API does not define a cross-port event callback mechanism. On ESP32, prefer polling `wlan.isconnected()` and `wlan.status()` (and use `wlan.config(reconnects=...)` to limit retries) rather than relying on undocumented callbacks.

## Network Configuration

### Get MAC Address

```python
import network
import binascii

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)

mac = binascii.hexlify(wlan.config('mac'), ':').decode()
print(f"MAC: {mac}")
```

### Get Signal Strength

```python
import network

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

rssi = wlan.status('rssi')
# Note: RSSI format and availability can vary across ports.
print(f"Signal: {rssi} dBm")
```

### Check Connection Quality

```python
def check_connection():
    if not wlan.isconnected():
        return False

    rssi = wlan.status('rssi')
    if rssi > -50:
        quality = "Excellent"
    elif rssi > -60:
        quality = "Good"
    elif rssi > -70:
        quality = "Fair"
    else:
        quality = "Poor"

    print(f"Quality: {quality} ({rssi} dBm)")
    return True
```

## HTTP Client

### Simple GET Request

```python
import network

try:
    import requests
except ImportError:
    import urequests as requests

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)
wlan.connect('SSID', 'password')

# Wait for connection
while not wlan.isconnected():
    pass

# HTTP GET
response = requests.get('http://httpbin.org/get')
print(response.status_code)
print(response.text)
response.close()
```

### POST JSON

```python
try:
    import requests
except ImportError:
    import urequests as requests
import ujson

data = {'sensor': 'temperature', 'value': 25.3}
headers = {'Content-Type': 'application/json'}

response = requests.post('http://httpbin.org/post',
                          json=data,
                          headers=headers)
print(response.status_code)
print(response.text)
response.close()
```

### Download File

```python
try:
    import requests
except ImportError:
    import urequests as requests

def download_file(url, filename):
    response = requests.get(url)
    if response.status_code == 200:
        with open(filename, 'wb') as f:
            f.write(response.content)
        print(f"Downloaded {filename}")
    response.close()

# Usage
download_file('http://example.com/data.txt', 'data.txt')
```

## HTTP Server

### Simple Web Server

```python
import network
import socket

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

print('Connected:', wlan.ifconfig()[0])

# Create socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('', 80))
s.listen(5)

while True:
    conn, addr = s.accept()
    print('Connection from', addr)

    request = conn.recv(1024)
    request = request.decode('utf-8')

    # Simple response
    response = """HTTP/1.1 200 OK

    <html>
    <body>
    <h1>Hello from XIAO ESP32!</h1>
    </body>
    </html>
    """

    conn.sendall(response.encode())
    conn.close()
```

### Control GPIO via Web

```python
import network
import socket
from machine import Pin

led = Pin(2, Pin.OUT)  # GPIO number is board-specific; adjust for your XIAO variant.

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('', 80))
s.listen(5)

while True:
    conn, addr = s.accept()
    request = conn.recv(1024).decode('utf-8')

    if 'GET /on' in request:
        led.on()
        response = 'LED ON'
    elif 'GET /off' in request:
        led.off()
        response = 'LED OFF'
    else:
        response = 'Use /on or /off'

    conn.sendall(b'HTTP/1.1 200 OK\r\n\r\n')
    conn.sendall(response.encode())
    conn.close()
```

## MQTT

The `umqtt` modules are not part of the core MicroPython firmware API. If your firmware build includes `umqtt.simple` (or you have deployed it from `micropython-lib`), you can use the following pattern.

### Basic MQTT Publisher

```python
import network
import umqtt.simple
import time

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

client = umqtt.simple.MQTTClient(
    client_id='xiao_esp32',
    server='broker.hivemq.com',
    port=1883
)

client.connect()

counter = 0
while True:
    msg = f'Count: {counter}'
    client.publish('xiao/sensor', msg)
    print(f"Published: {msg}")
    counter += 1
    time.sleep(5)
```

### MQTT Subscriber

```python
import network
import umqtt.simple
from machine import Pin
import time

led = Pin(2, Pin.OUT)

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

def on_message(topic, msg):
    print(f'Received: {topic} -> {msg}')
    if msg == b'ON':
        led.on()
    elif msg == b'OFF':
        led.off()

client = umqtt.simple.MQTTClient(
    client_id='xiao_sub',
    server='broker.hivemq.com'
)

client.set_callback(on_message)
client.connect()
client.subscribe('xiao/command')

while True:
    client.check_msg()
    time.sleep(0.1)
```

## NTP Time

```python
import network
import socket
import struct

wlan = network.WLAN(network.WLAN.IF_STA)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

def get_ntp_time():
    NTP_QUERY = b'\x1b' + 47 * b'\0'
    NTP_HOST = 'pool.ntp.org'

    addr = socket.getaddrinfo(NTP_HOST, 123)[0][-1]
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    s.sendto(NTP_QUERY, addr)

    msg, _ = s.recvfrom(256)
    s.close()

    val = struct.unpack("!I", msg[40:44])[0]
    return val - 2208988800

# Get time
epoch = get_ntp_time()
print(f"Unix timestamp: {epoch}")

# Convert to datetime
import utime
t = utime.gmtime(epoch)
print(f"Time: {t}")
```

## Troubleshooting

### Can't connect

1. Check SSID and password
2. Verify AP is within range
3. Try different security mode
4. Check for IP conflicts

### Intermittent connection

1. Check signal strength
2. Adjust power settings
3. Use static IP
4. Check for interference

### High power consumption

1. Enable power save mode
2. Disconnect when not needed
3. Use deep sleep between connections
4. Disable WiFi with wlan.active(False)

### DNS failures

1. Check DNS server in ifconfig
2. Try using IP address directly
3. Set DNS manually: `wlan.ifconfig(('192.168.1.100', '255.255.255.0', '192.168.1.1', '8.8.8.8'))`

## References

- MicroPython v1.27.0 `network.WLAN` API documentation: https://docs.micropython.org/en/v1.27.0/library/network.WLAN.html
- MicroPython v1.27.0 `network` module documentation: https://docs.micropython.org/en/v1.27.0/library/network.html
- MicroPython v1.27.0 ESP32 quick reference (Networking/WLAN notes): https://docs.micropython.org/en/v1.27.0/esp32/quickref.html#networking
- MicroPython source (tag v1.27.0), ESP32 network globals/constants: https://github.com/micropython/micropython/blob/v1.27.0/ports/esp32/modnetwork_globals.h
- MicroPython source (tag v1.27.0), ESP32 WLAN implementation (config params/status behavior): https://github.com/micropython/micropython/blob/v1.27.0/ports/esp32/network_wlan.c
