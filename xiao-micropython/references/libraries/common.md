# Common MicroPython Libraries for XIAO

## urequests (HTTP Client)

```python
import urequests

# GET request
response = urequests.get('http://httpbin.org/get')
print(response.status_code)
print(response.text)
response.close()

# POST with JSON
data = {'sensor': 'temp', 'value': 25.3}
response = urequests.post('http://httpbin.org/post', json=data)
print(response.text)
response.close()

# POST with data
response = urequests.post('http://httpbin.org/post',
                          data='raw data string')
response.close()

# Headers
headers = {'User-Agent': 'XIAO'}
response = urequests.get('http://httpbin.org/headers', headers=headers)
response.close()
```

## umqtt.simple (MQTT Client)

```python
from umqtt.simple import MQTTClient
import time

# Callback for received messages
def sub_cb(topic, msg):
    print(f"Received: {topic.decode()}: {msg.decode()}")

# Connect
client = MQTTClient(
    "xiao_12345",           # Client ID (must be unique)
    "broker.hivemq.com",    # Broker
    1883                    # Port
)

client.set_callback(sub_cb)
client.connect()
client.subscribe(b"xiao/command")

# Publish
client.publish(b"xiao/sensor", b"25.5")

# Loop
while True:
    client.check_msg()  # Non-blocking
    time.sleep(1)

# Disconnect
client.disconnect()
```

## umqtt robust (with auto-reconnect)

```python
from umqtt.robust import MQTTClient

# Auto-reconnects on connection loss
client = MQTTClient("xiao", "broker.hivemq.com")

def connect():
    while not client.connect(clean_session=False):
        print("Retrying...")
        time.sleep(5)

connect()
```

## ssd1306 (OLED Display)

```python
from machine import Pin, I2C
import ssd1306

i2c = I2C(0, sda=Pin(0), scl=Pin(1))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

# Clear and set pixel
oled.fill(0)
oled.pixel(64, 32, 1)
oled.show()

# Text
oled.fill(0)
oled.text("Hello XIAO!", 0, 0)
oled.text("Line 2", 0, 16)
oled.show()

# Power control
oled.poweron()  # Wake
oled.poweroff() # Sleep (saves power)
```

## neopixel (WS2812B)

```python
from machine import Pin
from neopixel import NeoPixel
import time

# 8 LEDs on D10
np = NeoPixel(Pin(10), 8)

# Set single LED
np[0] = (255, 0, 0)  # Red
np[1] = (0, 255, 0)  # Green
np[2] = (0, 0, 255)  # Blue
np.write()

# Fill all
np.fill((255, 255, 0))
np.write()

# Color cycle
h = 0
while True:
    for i in range(len(np)):
        np[i] = hsv_to_rgb(h, 255, 255)
    np.write()
    h = (h + 1) % 256
    time.sleep(0.01)

def hsv_to_rgb(h, s, v):
    # Convert HSV to RGB
    # Implementation needed
    return (r, g, b)
```

## dht (DHT11/DHT22)

```python
from machine import Pin
import dht
import time

# DHT22 on A0
sensor = dht.DHT22(Pin(0))

while True:
    try:
        sensor.measure()
        temp = sensor.temperature()
        hum = sensor.humidity()
        print(f"Temperature: {temp}°C")
        print(f"Humidity: {hum}%")
    except OSError as e:
        print(f"Sensor error: {e}")

    time.sleep(2)  # DHT requires 2 seconds between reads
```

## bme280 (BME280 Sensor)

```python
from machine import Pin, I2C
import bme280

i2c = I2C(0, sda=Pin(0), scl=Pin(1))
bme = bme280.BME280(i2c=i2c)

while True:
    # Compensated data
    temp, pressure, hum = bme.read_compensated_data()
    print(f"Temp: {temp/100:.1f}°C")
    print(f"Pressure: {pressure/256:.1f} hPa")
    print(f"Humidity: {hum/1024:.1f}%")
    time.sleep(2)
```

## onewire (DS18B20)

```python
from machine import Pin
import onewire, ds18x20
import time

# DS18B20 on D0
ow = onewire.OneWire(Pin(0))
ds = ds18x20.DS18X20(ow)

# Scan for devices
roms = ds.scan()
print(f"Found {len(roms)} DS18B20 sensors")

# Read temperature
ds.convert_temp()
time.sleep_ms(750)  # Wait for conversion
for rom in roms:
    temp = ds.read_temp(rom)
    print(f"Temperature: {temp}°C")
```

## axp192 (Power Management)

For XIAO with AXP192:

```python
from axp192 import AXP192

axp = AXP192()

# Read battery
battery = axp.getBatteryVoltage()
charge_current = axp.getBatteryChargeCurrent()

print(f"Battery: {battery}mV")
print(f"Charging: {charge_current}mA")

# Set power
axp.setLDO2Voltage(3300)  # 3.3V LDO
axp.setLDO3Voltage(3300)

# Enable/disable
axp.setPowerOutput("AXP192_LDO2", True)
```

## sdcard (SD Card)

```python
from machine import Pin, SPI
import sdcard
import os

# SPI pins (adjust for your board)
spi = SPI(1, baudrate=1000000, sck=Pin(14), mosi=Pin(15), miso=Pin(12))
sd = sdcard.SDCard(spi, Pin(13))  # CS

# Mount
os.mount(sd, '/sd')

# List files
print(os.listdir('/sd'))

# Write
with open('/sd/data.txt', 'w') as f:
    f.write('Hello SD!')

# Read
with open('/sd/data.txt', 'r') as f:
    print(f.read())
```

## ucryptolib (Cryptography)

```python
import ucryptolib

# AES encryption
key = b'sixteen byte key'
cipher = ucryptolib.aes(key, 1)

plaintext = b'Secret message'
encrypted = cipher.encrypt(plaintext)
decrypted = cipher.decrypt(encrypted)

print(f"Decrypted: {decrypted}")
```

## uasyncio (Async/Await)

```python
import uasyncio as asyncio

async def blink(led, period):
    while True:
        led.on()
        await asyncio.sleep(period/2)
        led.off()
        await asyncio.sleep(period/2)

async def counter():
    count = 0
    while True:
        print(f"Count: {count}")
        count += 1
        await asyncio.sleep(1)

async def main():
    led = Pin(10, Pin.OUT, value=0)
    t1 = asyncio.create_task(blink(led, 0.5))
    t2 = asyncio.create_task(counter())
    await asyncio.sleep(10)  # Run for 10 seconds
    t1.cancel()
    t2.cancel()

asyncio.run(main())
```

## ujson (JSON)

```python
import ujson

# Parse JSON
json_str = '{"temp": 25.3, "hum": 60}'
data = ujson.loads(json_str)
print(data['temp'])

# Create JSON
data = {'sensor': 'DHT22', 'temp': 25.3}
json_str = ujson.dumps(data)
print(json_str)
```

## ure (Regular Expressions)

```python
import ure

# Match pattern
pattern = ure.compile(r'temp:\s*(\d+)')
match = pattern.search('temp: 25')
if match:
    print(f"Temperature: {match.group(1)}")

# Split
text = "temp:25,hum:60"
parts = ure.split(r'[,:]', text)
print(parts)  # ['temp', '25', 'hum', '60']
```

## network (WiFi Low Power)

```python
import network
import machine

# Light sleep with WiFi
wlan = network.WLAN(network.STA_IF)
wlan.disconnect()

# Enable low power mode
wlan.active(False)  # WiFi off
machine.lightsleep(10000)  # Sleep 10 seconds
wlan.active(True)
```

## mpu6050 (IMU Sensor)

```python
from machine import Pin, I2C
import time

i2c = I2C(0, sda=Pin(0), scl=Pin(1))

# Initialize MPU6050
i2c.writeto_mem(0x68, 0x6B, b'\x00')  # Wake up
i2c.writeto_mem(0x68, 0x1B, b'\x00')  # Gyro config
i2c.writeto_mem(0x68, 0x1C, b'\x00')  # Accel config

def read_mpu6050():
    # Read 14 bytes starting from 0x3B
    data = i2c.readfrom_mem(0x68, 0x3B, 14)

    # Accelerometer (16-bit signed)
    ax = (data[0] << 8 | data[1])
    if ax > 32767: ax -= 65536
    ay = (data[2] << 8 | data[3])
    if ay > 32767: ay -= 65536
    az = (data[4] << 8 | data[5])
    if az > 32767: az -= 65536

    # Temperature
    temp = (data[6] << 8 | data[7])
    if temp > 32767: temp -= 65536
    temp_c = temp / 340 + 36.53

    # Gyroscope
    gx = (data[8] << 8 | data[9])
    if gx > 32767: gx -= 65536
    gy = (data[10] << 8 | data[11])
    if gy > 32767: gy -= 65536
    gz = (data[12] << 8 | data[13])
    if gz > 32767: gz -= 65536

    return ax, ay, az, temp_c, gx, gy, gz

while True:
    ax, ay, az, temp, gx, gy, gz = read_mpu6050()
    print(f"Accel: ({ax}, {ay}, {az})")
    print(f"Temp: {temp:.1f}C")
    print(f"Gyro: ({gx}, {gy}, {gz})")
    time.sleep(0.5)
```

## INA219 (Current Sensor)

```python
from machine import Pin, I2C
import time

i2c = I2C(0, sda=Pin(0), scl=Pin(1))

# Initialize INA219
def ina219_init():
    # Reset
    i2c.writeto_mem(0x40, 0x00, b'\x80')
    time.sleep_ms(10)
    # Set calibrate (for 32V, 2A range)
    i2c.writeto_mem(0x40, 0x05, b'\x00\x40')

def ina219_read():
    # Read bus voltage (register 0x02)
    vbus = i2c.readfrom_mem(0x40, 0x02, 2)
    voltage = ((vbus[0] << 8 | vbus[1]) >> 3) * 0.004

    # Read shunt voltage (register 0x01)
    vshunt = i2c.readfrom_mem(0x40, 0x01, 2)
    shunt = (vshunt[0] << 8 | vshunt[1])
    if shunt > 32767:
        shunt -= 65536
    shunt_voltage = shunt * 0.00001

    # Read current (register 0x04)
    curr = i2c.readfrom_mem(0x40, 0x04, 2)
    current = (curr[0] << 8 | curr[1])
    if current > 32767:
        current -= 65536
    current_ma = current * 0.1

    # Read power (register 0x03)
    power_bytes = i2c.readfrom_mem(0x40, 0x03, 2)
    power = power_bytes[0] << 8 | power_bytes[1]
    power_mw = power * 2

    return voltage, shunt_voltage, current_ma, power_mw

ina219_init()

while True:
    v, vs, i, p = ina219_read()
    print(f"Voltage: {v:.2f}V, Current: {i:.1f}mA, Power: {p:.1f}mW")
    time.sleep(1)
```

## ADS1115 (ADC)

```python
from machine import Pin, I2C
import time

i2c = I2C(0, sda=Pin(0), scl=Pin(1))

# Initialize ADS1115
def ads1115_read(channel=0):
    # Channel config: 0=A0, 1=A1, 2=A2, 3=A3
    config = 0x0001  # Start conversion
    config |= (channel + 0x04) << 12  # MUX
    config |= 0x0100  # 4.096V range
    config |= 0x8000  # Single shot mode

    # Write config
    i2c.writeto_mem(0x48, 0x01, bytes([config >> 8, config & 0xFF]))

    # Wait for conversion
    time.sleep_ms(10)

    # Read result
    data = i2c.readfrom_mem(0x48, 0x00, 2)
    value = (data[0] << 8 | data[1])
    if value > 32767:
        value -= 65536

    # Convert to voltage (4.096V range, +/- 32768)
    voltage = value * 4.096 / 32768
    return voltage

while True:
    for ch in range(4):
        v = ads1115_read(ch)
        print(f"A{ch}: {v:.3f}V")
    time.sleep(0.5)
```

## Stepper Motor (ULN2003)

```python
from machine import Pin
import time

# Step sequence for 28BYJ-48
STEP_SEQ = [
    [1, 0, 0, 0],
    [1, 1, 0, 0],
    [0, 1, 0, 0],
    [0, 1, 1, 0],
    [0, 0, 1, 0],
    [0, 0, 1, 1],
    [0, 0, 0, 1],
    [1, 0, 0, 1]
]

pins = [Pin(4, Pin.OUT), Pin(5, Pin.OUT), Pin(6, Pin.OUT), Pin(7, Pin.OUT)]

def step(steps, delay_ms=2):
    direction = 1 if steps >= 0 else -1
    steps = abs(steps)

    for _ in range(steps):
        for seq in (STEP_SEQ if direction == 1 else reversed(STEP_SEQ)):
            for i, pin in enumerate(pins):
                pin.value(seq[i])
            time.sleep_ms(delay_ms)

# Rotate one revolution (512 steps)
step(512)
```

## Ultrasonic Distance (HC-SR04)

```python
from machine import Pin, time_pulse_us
import time

trig = Pin(4, Pin.OUT)
echo = Pin(5, Pin.IN)

def measure_distance():
    # Send trigger
    trig.low()
    time.sleep_us(2)
    trig.high()
    time.sleep_us(10)
    trig.low()

    # Measure echo
    duration = time_pulse_us(echo, 1, 30000)  # 30ms timeout

    if duration > 0:
        # Distance = duration * speed of sound / 2
        # speed of sound = 343 m/s = 0.0343 cm/us
        distance = duration * 0.0343 / 2
        return distance
    return -1

while True:
    dist = measure_distance()
    if dist >= 0:
        print(f"Distance: {dist:.1f} cm")
    else:
        print("No echo")
    time.sleep(0.5)
```

## DS3231 RTC

```python
from machine import Pin, I2C
import time

i2c = I2C(0, sda=Pin(0), scl=Pin(1))

def bcd_to_dec(bcd):
    return (bcd & 0x0F) + ((bcd >> 4) * 10)

def dec_to_bcd(dec):
    return (dec % 10) | ((dec // 10) << 4)

def set_rtc(year, month, day, hour, minute, second):
    data = bytes([
        dec_to_bcd(second),
        dec_to_bcd(minute),
        dec_to_bcd(hour),
        dec_to_bcd(1),  # weekday (not used)
        dec_to_bcd(day),
        dec_to_bcd(month),
        dec_to_bcd(year - 2000)
    ])
    i2c.writeto_mem(0x68, 0x00, data)

def read_rtc():
    data = i2c.readfrom_mem(0x68, 0x00, 7)
    second = bcd_to_dec(data[0])
    minute = bcd_to_dec(data[1])
    hour = bcd_to_dec(data[2])
    day = bcd_to_dec(data[4])
    month = bcd_to_dec(data[5])
    year = 2000 + bcd_to_dec(data[6])
    return (year, month, day, hour, minute, second)

# Set time: 2024-01-15 12:30:00
set_rtc(2024, 1, 15, 12, 30, 0)

# Read time
while True:
    y, m, d, h, mn, s = read_rtc()
    print(f"{y:04d}-{m:02d}-{d:02d} {h:02d}:{mn:02d}:{s:02d}")
    time.sleep(1)
```

## Troubleshooting

### I2C device not found

1. Check wiring (SDA/SCL swapped?)
2. Verify pull-up resistors (4.7k typical)
3. Scan with I2C scanner
4. Try lower I2C frequency (100kHz)

### SD card initialization fails

1. Check SPI wiring (MOSI/MISO/SCK/CS)
2. Verify format (FAT32 recommended)
3. Try lower SPI frequency (1MHz)
4. Check card size (< 32GB)

### NeoPixel flickering

1. Add 300-500 ohm resistor on data line
2. Add 1000-4700uF capacitor across power
3. Shorten data wire (< 30cm)
4. Check power supply (3.3V-5V)

### DHT sensor errors

1. Add 10k pull-up on data line
2. Check 3.3V vs 5V (DHT11 needs 5V)
3. Wait 2 seconds between reads
4. Try external power instead of parasitic

### Memory errors

1. Reduce buffer sizes
2. Free objects with `del`
3. Use `ujson` instead of `json`
4. Check free memory with `gc.mem_free()`

## Library Availability

| Library | ESP32 | RP2040 | nRF52 | Notes |
|---------|-------|--------|-------|-------|
| urequests | ✅ | ✅ | ✅ | HTTP client |
| umqtt | ✅ | ✅ | ✅ | MQTT client |
| ssd1306 | ✅ | ✅ | ✅ | OLED display |
| neopixel | ✅ | ✅ | ✅ | WS2812B LEDs |
| dht | ✅ | ✅ | ✅ | DHT11/22 |
| bme280 | ✅ | ✅ | ✅ | BME280 sensor |
| onewire | ✅ | ✅ | ✅ | DS18B20 |
| sdcard | ✅ | ✅ | ❌ | SD card |
| uasyncio | ✅ | ✅ | ✅ | Async/await |
| ujson | ✅ | ✅ | ✅ | JSON |
| ure | ✅ | ✅ | ✅ | Regex |
| axp192 | ✅ | ❌ | ❌ | Power management |
| ucryptolib | ✅ | ✅ | ✅ | Encryption |
