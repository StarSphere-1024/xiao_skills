# ESP32 WiFi API for MicroPython

## Overview

WiFi networking on ESP32-based XIAO boards using MicroPython network module.

## Station Mode (WiFi Client)

### Basic Connection

```python
import network

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('your-SSID', 'your-password')

# Wait for connection
while not wlan.isconnected():
    pass

print('Connected:', wlan.ifconfig())
# Output: ('192.168.1.100', '255.255.255.0', '192.168.1.1', '8.8.8.8')
```

### Connection Status

```python
import network
import time

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('SSID', 'password')

# Check status
status = wlan.status()
print(f"Status: {status}")
# 0 = Link down
# 1 = Joining
# 3 = Got IP
# -1 = Link fail
# -2 = No AP
# -3 = Auth fail

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

wlan = network.WLAN(network.STA_IF)
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

wlan = network.WLAN(network.STA_IF)
wlan.active(True)

# Static IP configuration
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

ap = network.WLAN(network.AP_IF)
ap.active(True)
ap.config(essid='XIAO-AP', password='12345678')

print('AP running:', ap.ifconfig())
# Output: ('192.168.4.1', '255.255.255.0', '192.168.4.1', '255.255.255.255')
```

### AP with DHCP

```python
import network

ap = network.WLAN(network.AP_IF)
ap.active(True)

# Configure AP
ap.config(essid='XIAO-AP',
          password='password123',
          channel=11,
          hidden=False)

# DHCP range: 192.168.4.2 - 192.168.4.254
print('AP ready, IP:', ap.ifconfig()[0])
```

### Open AP (No Password)

```python
ap = network.WLAN(network.AP_IF)
ap.active(True)
ap.config(essid='XIAO-Open', authmode=network.AUTH_OPEN)
```

## AP + STA Simultaneous

```python
import network

# Station
sta = network.WLAN(network.STA_IF)
sta.active(True)
sta.connect('HomeWiFi', 'password')

# Access Point
ap = network.WLAN(network.AP_IF)
ap.active(True)
ap.config(essid='XIAO-Hotspot')

print('STA IP:', sta.ifconfig()[0])
print('AP IP:', ap.ifconfig()[0])
```

## Power Management

### WiFi Power Save

```python
import network

wlan = network.WLAN(network.STA_IF)
wlan.active(True)

# Enable power save
wlan.config(pm=network.WIFI_PS_MIN_MODEM)  # Light sleep
# wlan.config(pm=network.WIFI_PS_MAX_MODEM)  # Deeper sleep
# wlan.config(pm=network.WIFI_PS_NONE)  # No power save

wlan.connect('SSID', 'password')
```

### Disable WiFi for Low Power

```python
import network
import machine

# Disconnect and disable
wlan = network.WLAN(network.STA_IF)
wlan.disconnect()
wlan.active(False)

# Enter deep sleep
machine.deepsleep(60000)  # 60 seconds
```

## WiFi Events

### Connection Callback

```python
import network
import time

def wifi_callback(wlan, event):
    if event == network.EVT_STA_CONNECTED:
        print("Connected to AP")
    elif event == network.EVT_STA_DISCONNECTED:
        print("Disconnected, reconnecting...")
        wlan.connect('SSID', 'password')

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.callback(wifi_callback)
wlan.connect('SSID', 'password')
```

## Network Configuration

### Get MAC Address

```python
import network
import ubinascii

wlan = network.WLAN(network.STA_IF)
wlan.active(True)

mac = ubinascii.hexlify(wlan.config('mac'), ':').decode()
print(f"MAC: {mac}")
```

### Get Signal Strength

```python
import network

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('SSID', 'password')

while not wlan.isconnected():
    pass

rssi = wlan.status('rssi')
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
import urequests

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect('SSID', 'password')

# Wait for connection
while not wlan.isconnected():
    pass

# HTTP GET
response = urequests.get('http://httpbin.org/get')
print(response.status_code)
print(response.text)
response.close()
```

### POST JSON

```python
import urequests
import ujson

data = {'sensor': 'temperature', 'value': 25.3}
headers = {'Content-Type': 'application/json'}

response = urequests.post('http://httpbin.org/post',
                          json=data,
                          headers=headers)
print(response.status_code)
print(response.text)
response.close()
```

### Download File

```python
import urequests

def download_file(url, filename):
    response = urequests.get(url)
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
import machine

wlan = network.WLAN(network.STA_IF)
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

    conn.send(response)
    conn.close()
```

### Control GPIO via Web

```python
import network
import socket
from machine import Pin

led = Pin(2, Pin.OUT)

wlan = network.WLAN(network.STA_IF)
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

    conn.send('HTTP/1.1 200 OK\n\n')
    conn.send(response)
    conn.close()
```

## MQTT

### Basic MQTT Publisher

```python
import network
import umqtt.simple
import machine
import time

wlan = network.WLAN(network.STA_IF)
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
import machine
from machine import Pin
import time

led = Pin(2, Pin.OUT)

wlan = network.WLAN(network.STA_IF)
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
import time

wlan = network.WLAN(network.STA_IF)
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
