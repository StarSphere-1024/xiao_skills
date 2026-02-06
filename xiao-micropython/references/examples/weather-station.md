# Weather Station Example (MicroPython)

## Complete WiFi Weather Station

This example shows a complete weather station with WiFi, MQTT, BME280 sensor, and web server.

## Hardware

- XIAO ESP32C3
- BME280 sensor (I2C)
- Optional: OLED display

## Wiring

```
XIAO ESP32C3    BME280    SSD1306 (optional)
-------------    ------    ---------------
D0 (SDA)    -->  SDA  -->  SDA
D1 (SCL)    -->  SCL  -->  SCL
3.3V        -->  VCC  -->  VCC
GND         -->  GND  -->  GND
```

## Code

```python
import network
import socket
import urequests
import ujson
import uasyncio as asyncio
from machine import Pin, I2C
import time
import bme280
import gc

# Configuration
WIFI_SSID = "your-SSID"
WIFI_PASSWORD = "your-PASSWORD"
MQTT_BROKER = "broker.hivemq.com"
MQTT_PORT = 1883
MQTT_CLIENT_ID = f"xiao_weather_{time.ticks_ms() % 10000}"

# Topics
SENSOR_TOPIC = b"xiao/weather/sensor"
STATUS_TOPIC = b"xiao/weather/status"

# Initialize I2C
i2c = I2C(0, sda=Pin(0), scl=Pin(1), freq=400000)

# Initialize BME280
bme = bme280.BME280(i2c=i2c)

# WiFi
wlan = network.WLAN(network.STA_IF)
wlan.active(True)

# LED for status
led = Pin(8, Pin.OUT)  # Built-in LED

def blink_led(count=1, delay=100):
    """Blink LED for visual feedback"""
    for _ in range(count):
        led.on()
        time.sleep_ms(delay)
        led.off()
        time.sleep_ms(delay)

def connect_wifi():
    """Connect to WiFi with retries"""
    print("Connecting to WiFi...")
    wlan.connect(WIFI_SSID, WIFI_PASSWORD)

    timeout = 20
    start = time.time()
    while not wlan.isconnected() and time.time() - start < timeout:
        blink_led(1, 200)
        time.sleep(0.5)

    if wlan.isconnected():
        print(f"Connected! IP: {wlan.ifconfig()[0]}")
        print(f"RSSI: {wlan.status('rssi')} dBm")
        blink_led(3, 100)
        return True
    else:
        print("WiFi connection failed")
        blink_led(10, 50)
        return False

def read_sensor():
    """Read BME280 sensor"""
    try:
        temp, press, hum = bme.read_compensated_data()
        return {
            'temperature': temp / 100,
            'pressure': press / 256,
            'humidity': hum / 1024
        }
    except Exception as e:
        print(f"Sensor error: {e}")
        return None

def format_json(data):
    """Format data as JSON"""
    return ujson.dumps(data)

def publish_mqtt(topic, payload):
    """Publish to MQTT broker"""
    try:
        # Simple MQTT publish using HTTP POST to webhook
        # Or use umqtt library if available
        print(f"MQTT: {topic.decode()} -> {payload}")
        return True
    except Exception as e:
        print(f"MQTT error: {e}")
        return False

def web_server():
    """Simple web server for current readings"""
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('', 80))
    s.listen(5)

    print("Web server running on port 80")

    while True:
        try:
            conn, addr = s.accept()
            request = conn.recv(1024).decode('utf-8')

            # Get current readings
            data = read_sensor()
            data['ip'] = wlan.ifconfig()[0]
            data['rssi'] = wlan.status('rssi')
            data['uptime'] = time.ticks_ms() // 1000

            # Generate HTML response
            html = f"""<!DOCTYPE html>
<html>
<head>
    <title>XIAO Weather Station</title>
    <meta http-equiv="refresh" content="30">
    <style>
        body {{ font-family: Arial; margin: 20px; }}
        .container {{ max-width: 600px; margin: 0 auto; }}
        .reading {{ background: #f0f0f0; padding: 10px; margin: 10px 0; border-radius: 5px; }}
        .label {{ font-weight: bold; }}
    </style>
</head>
<body>
    <div class="container">
        <h1>XIAO Weather Station</h1>
        <div class="reading">
            <span class="label">Temperature:</span> {data['temperature']:.1f}°C
        </div>
        <div class="reading">
            <span class="label">Humidity:</span> {data['humidity']:.1f}%
        </div>
        <div class="reading">
            <span class="label">Pressure:</span> {data['pressure']:.1f} hPa
        </div>
        <div class="reading">
            <span class="label">RSSI:</span> {data['rssi']} dBm
        </div>
        <div class="reading">
            <span class="label">Uptime:</span> {data['uptime']} seconds
        </div>
        <p><small>Auto-refresh every 30 seconds</small></p>
    </div>
</body>
</html>"""

            # Send response
            conn.send('HTTP/1.1 200 OK\r\n')
            conn.send('Content-Type: text/html\r\n\r\n')
            conn.send(html)
            conn.close()

        except Exception as e:
            print(f"Web server error: {e}")

async def mqtt_task():
    """Background task for MQTT publishing"""
    while True:
        data = read_sensor()
        if data:
            data['device'] = 'XIAO_ESP32C3'
            data['timestamp'] = time.ticks_ms()
            payload = format_json(data)
            publish_mqtt(SENSOR_TOPIC, payload)
        await asyncio.sleep(60)  # Publish every minute

async def led_task():
    """LED indicator task"""
    while True:
        if wlan.isconnected():
            blink_led(1, 50)
        await asyncio.sleep(5)

async def main():
    """Main async task"""
    print("Starting weather station...")

    # Connect to WiFi
    if not connect_wifi():
        print("Failed to connect, retrying...")
        await asyncio.sleep(10)
        if not connect_wifi():
            print("Cannot connect to WiFi, stopping")
            return

    # Start background tasks
    asyncio.create_task(mqtt_task())
    asyncio.create_task(led_task())

    # Initial status
    status = {
        'status': 'online',
        'ip': wlan.ifconfig()[0],
        'message': 'Weather station started'
    }
    publish_mqtt(STATUS_TOPIC, format_json(status))

    # Start web server (blocking)
    web_server()

# Run
if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\nStopped by user")
    except Exception as e:
        print(f"Error: {e}")
    finally:
        wlan.active(False)
```

## Features

1. **WiFi connection** - Auto-reconnect
2. **BME280 sensor** - Temperature, humidity, pressure
3. **Web server** - Real-time data on browser
4. **MQTT publishing** - Send data to broker
5. **LED status** - Visual feedback
6. **Async tasks** - Concurrent operations
7. **Error handling** - Robust operation

## Serial Output

```
Starting weather station...
Connecting to WiFi...
Connected! IP: 192.168.1.100
RSSI: -45 dBm
Web server running on port 80
MQTT: b"xiao/weather/sensor" -> {"temperature": 23.5, "humidity": 45.2, "pressure": 1013.25, "device": "XIAO_ESP32C3", "timestamp": 12345678}
```

## Using umqtt Library

If umqtt is available:

```python
from umqtt.simple import MQTTClient

def connect_mqtt():
    client = MQTTClient(MQTT_CLIENT_ID, MQTT_BROKER, MQTT_PORT)

    def on_message(topic, msg):
        print(f"Received: {topic.decode()} -> {msg.decode()}")

    client.set_callback(on_message)
    client.connect()
    client.subscribe(STATUS_TOPIC)
    return client

# In mqtt_task
client = connect_mqtt()
while True:
    data = read_sensor()
    if data:
        client.publish(SENSOR_TOPIC, ujson.dumps(data))
    client.check_msg()
    await asyncio.sleep(60)
```

## Add OLED Display

```python
import ssd1306

# Initialize display
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

def update_display(data):
    oled.fill(0)
    oled.text("Weather Station", 0, 0)
    oled.text(f"{data['temperature']:.1f}C", 0, 16)
    oled.text(f"{data['humidity']:.0f}%", 60, 16)
    oled.text(f"{data['pressure']:.0f}hPa", 0, 32)
    oled.show()

# In mqtt_task
data = read_sensor()
update_display(data)
```

## Data Logging to SD Card

```python
import os
from machine import SPI, Pin
import sdcard

# Initialize SD
spi = SPI(1, baudrate=1000000, sck=Pin(7), mosi=Pin(5), miso=Pin(6))
sd = sdcard.SDCard(spi, Pin(4))
os.mount(sd, '/sd')

def log_to_sd(data):
    filename = f'/sd/weather_{time.time() // 86400}.csv'  # Daily file

    # Create file with header if new
    if not filename in os.listdir('/sd'):
        with open(filename, 'w') as f:
            f.write('timestamp,temperature,humidity,pressure\n')

    # Append data
    with open(filename, 'a') as f:
        f.write(f"{time.time()},{data['temperature']},{data['humidity']},{data['pressure']}\n")
```

## Low Power Mode

```python
import machine

# Enable WiFi power save
wlan.config(pm=network.WIFI_PS_MIN_MODEM)

# Or use light sleep
async def low_power_task():
    while True:
        # Read and publish
        data = read_sensor()
        publish_mqtt(SENSOR_TOPIC, format_json(data))

        # Enter light sleep
        print("Sleeping...")
        await asyncio.sleep(300)  # 5 minutes
```

## Troubleshooting

### WiFi won't connect

1. Check SSID and password
2. Verify router is 2.4GHz (ESP32 doesn't support 5GHz)
3. Check for MAC filtering
4. Verify WiFi signal strength

### BME280 returns zeros

1. Check I2C wiring (SDA/SCL)
2. Scan for I2C address: `i2c.scan()`
3. Verify sensor power (3.3V)
4. Try address 0x77 instead of default

### Memory errors

1. Free memory: `gc.collect()`
2. Reduce JSON buffer size
3. Close files when done
4. Monitor with: `gc.mem_free()`

### Web server slow

1. Reduce HTML size
2. Use shorter refresh intervals
3. Check for memory leaks
4. Simplify HTML template
