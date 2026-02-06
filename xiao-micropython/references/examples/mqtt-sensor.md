# MQTT Temperature & Humidity Sensor (MicroPython)

## Complete MQTT sensor with BME280 and Home Assistant integration.

## Hardware

- XIAO ESP32C3/C6 or ESP32S3
- BME280 sensor (I2C)
- LED indicator (optional)

## Wiring

| XIAO Pin | BME280 |
|----------|--------|
| D4 | SDA |
| D5 | SCL |
| 3.3V | VCC |
| GND | GND |

## Code

```python
from machine import Pin, I2C
import network
import time
import ubinascii
import urequests
import ujson
from umqtt.simple import MQTTClient
import gc

# --- Configuration ---
WIFI_SSID = "YourSSID"
WIFI_PASSWORD = "YourPassword"

MQTT_BROKER = "broker.hivemq.com"
MQTT_PORT = 1883
MQTT_CLIENT_ID = f"xiao-{ubinascii.hexlify(machine.unique_id())}"
MQTT_USER = ""
MQTT_PASSWORD = ""

# Topics
TOPIC_TEMP = "home/xiao/sensor/temperature"
TOPIC_HUM = "home/xiao/sensor/humidity"
TOPIC_PRESS = "home/xiao/sensor/pressure"
TOPIC_BATT = "home/xiao/sensor/battery"
TOPIP_STATUS = "home/xiao/sensor/status"

# Update interval
UPDATE_INTERVAL = 60  # seconds

# --- BME280 Sensor ---
class BME280:
    def __init__(self, i2c, address=0x76):
        self.i2c = i2c
        self.address = address
        self.dig_T1 = self._read_u16(0x88)
        self.dig_T2 = self._read_s16(0x8A)
        self.dig_T3 = self._read_s16(0x8C)
        self.dig_P1 = self._read_u16(0x8E)
        self.dig_P2 = self._read_s16(0x90)
        self.dig_P3 = self._read_s16(0x92)
        self.dig_P4 = self._read_s16(0x94)
        self.dig_P5 = self._read_s16(0x96)
        self.dig_P6 = self._read_s16(0x98)
        self.dig_P7 = self._read_s16(0x9A)
        self.dig_P8 = self._read_s16(0x9C)
        self.dig_P9 = self._read_s16(0x9E)
        self.dig_H1 = self._read_u8(0xA1)
        self.dig_H2 = self._read_s16(0xE1)
        self.dig_H3 = self._read_s8(0xE3)
        self.dig_H4 = self._read_s8(0xE4)
        self.dig_H5 = self._read_s8(0xE5)
        self.dig_H6 = self._read_s8(0xE6)
        self.dig_H7 = self._read_s8(0xE7)
        self.dig_H8 = self._read_s8(0xE8)
        self.dig_H9 = self._read_s8(0xE9)
        self.dig_H10 = self._read_u8(0xED)

        self.T_fine = 0

        # Configure
        self.i2c.writto(self.address, b'\xF2\x01')  # Humidity oversampling x1
        self.i2c.writeto(self.address, b'\xF4\x00')  # Pressure oversampling x1
        self.i2c.writeto(self.address, b'\xF5\x00')  # Normal mode

    def _read_u8(self, reg):
        return self.i2c.readfrom_mem(self.address, reg, 1)[0]

    def _read_s8(self, reg):
        return self._read_u8(reg) - 256 if self._read_u8(reg) > 127 else self._read_u8(reg)

    def _read_u16(self, reg):
        return self.i2c.readfrom_mem(self.address, reg, 2)[0] | (self.i2c.readfrom_mem(self.address, reg, 2)[1] << 8)

    def _read_s16(self, reg):
        val = self._read_u16(reg)
        return val - 65536 if val > 32767 else val

    def read_compensated_data(self):
        # Read raw data
        data = self.i2c.readfrom_mem(self.address, 0xF7, 8)

        adc_P = (data[0] << 12) | (data[1] << 4) | (data[2] >> 4)
        adc_T = (data[3] << 12) | (data[4] << 4) | (data[5] >> 4)
        adc_H = (data[6] << 8) | data[7]

        # Temperature compensation
        var1 = ((adc_T >> 3) - (self.dig_T1 << 1)) * (self.dig_T2 >> 11)
        var2 = (((((adc_T >> 4) - self.dig_T1) * ((adc_T >> 4) - self.dig_T1)) >> 12) * (self.dig_T3 >> 14))
        self.T_fine = var1 + var2
        temp = (self.T_fine * 5 + 128) >> 8

        # Pressure compensation
        var1 = self.T_fine - 12800
        var2 = var1 * var1 * self.dig_P6
        var2 = var2 + ((var1 * self.dig_P5) << 17)
        var2 = var2 + (self.dig_P4 << 35)
        var1 = ((var1 * var1 * self.dig_P3) >> 8) + ((var1 * self.dig_P2) << 12)
        var1 = (((1 << 47) + var1) * self.dig_P1) >> 33

        if var1 == 0:
            pressure = 0
        else:
            p = 1048576 - adc_P
            p = (((p << 31) - var2) * 3125) // var1
            var1 = (self.dig_P9 * (p >> 13) * (p >> 13)) >> 25
            var2 = (self.dig_P8 * p) >> 19
            pressure = ((p + var1 + var2) >> 15) * ((p >> 31) >> 1) >> 12

        # Humidity compensation
        h = self.T_fine - 76800
        h = (((((adc_H << 14) - (self.dig_H4 << 20) - (self.dig_H5 * h)) + (16384) >> 15) *
             (((((((h * self.dig_H6) >> 10) * (((h * self.dig_H3) >> 11) + 32768)) >> 10) +
              (2097152)) * self.dig_H2 + 8192) * h + self.dig_H1) >> 15)
        h = (h * ((h >> 3) * (h >> 3) >> 13) >> 12) - 76800
        humidity = h if h > 0 else 0
        humidity = humidity if humidity < 102400 else 102400

        return temp / 5120.0, pressure / 25600.0, humidity / 1024.0

# --- WiFi Connection ---
def connect_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)

    if not wlan.isconnected():
        print("Connecting to WiFi...")
        wlan.connect(WIFI_SSID, WIFI_PASSWORD)

        while not wlan.isconnected():
            time.sleep(1)

    print(f"WiFi connected: {wlan.ifconfig()[0]}")
    return wlan

# --- MQTT Callbacks ---
def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print("Connected to MQTT broker")
        client.subscribe(TOPIC_STATUS)
    else:
        print(f"MQTT connection failed: {rc}")

def on_message(client, userdata, msg):
    print(f"Received: {msg.topic} = {msg.payload}")

# --- Main ---
def main():
    # Initialize LED
    led = Pin(13, Pin.OUT)

    # Initialize I2C
    i2c = I2C(0, sda=Pin(4), scl=Pin(5), freq=100000)

    # Initialize sensor
    try:
        bme = BME280(i2c)
        print("BME280 initialized")
    except:
        print("BME280 not found!")
        return

    # Connect WiFi
    wlan = connect_wifi()

    # Initialize MQTT
    client = MQTTClient(MQTT_CLIENT_ID, MQTT_BROKER, MQTT_PORT,
                        MQTT_USER, MQTT_PASSWORD)
    client.set_callback(on_message)
    client.on_connect = on_connect

    try:
        client.connect()
    except:
        print("MQTT connection failed")
        return

    # Publish online status
    client.publish(TOPIC_STATUS, "online")

    # Main loop
    counter = 0

    while True:
        try:
            # Read sensor
            temp, press, hum = bme.read_compensated_data()

            print(f"Temp: {temp:.2f}C, Hum: {hum:.1f}%, Press: {press:.1f}hPa")

            # Publish to MQTT
            client.publish(TOPIC_TEMP, f"{temp:.2f}")
            client.publish(TOPIC_HUM, f"{hum:.1f}")
            client.publish(TOPIC_PRESS, f"{press:.1f}")

            # Blink LED
            led.value(1)
            time.sleep(0.1)
            led.value(0)

            counter += 1

            # Check WiFi
            if not wlan.isconnected():
                print("WiFi lost, reconnecting...")
                wlan = connect_wifi()

            # Check MQTT
            client.check_msg()

            # Wait
            time.sleep(UPDATE_INTERVAL)

        except Exception as e:
            print(f"Error: {e}")
            time.sleep(5)

            # Reconnect
            if not wlan.isconnected():
                wlan = connect_wifi()

if __name__ == "__main__":
    main()
```

## Home Assistant Auto-Discovery

```python
# Add Home Assistant auto-discovery
def publish_discovery(client):
    # Temperature sensor
    temp_config = {
        "device_class": "temperature",
        "name": "XIAO Temperature",
        "state_topic": TOPIC_TEMP,
        "unit_of_measurement": "°C",
        "value_template": "{{ value | float }}",
        "unique_id": "xiao_temp_01",
        "device": {
            "identifiers": ["xiao_sensor_01"],
            "name": "XIAO Sensor",
            "model": "XIAO ESP32C3",
            "manufacturer": "SeeedStudio"
        }
    }

    client.publish(
        "homeassistant/sensor/xiao_temp/config",
        ujson.dumps(temp_config),
        retain=True
    )
```

## Battery Monitoring

```python
from machine import ADC

# Read battery voltage
def read_battery():
    adc = ADC(Pin(1))
    adc.atten(ADC.ATTN_11DB)
    value = adc.read_u16()
    voltage = value * 3.9 / 65535

    # Convert to percentage
    percent = (voltage - 3.0) / (4.2 - 3.0) * 100
    percent = max(0, min(100, percent))

    return voltage, percent

# In main loop
voltage, percent = read_battery()
client.publish(TOPIC_BATT, f"{percent:.0f}")
```

## Configuration File

```python
# config.py
WIFI_SSID = "YourSSID"
WIFI_PASSWORD = "YourPassword"
MQTT_BROKER = "broker.hivemq.com"
UPDATE_INTERVAL = 60

# In main.py
from config import *
```

## Troubleshooting

### MQTT connection fails

1. Check broker address and port
2. Verify network connectivity
3. Check firewall settings
4. Try public broker like HiveMQ

### Sensor not reading

1. Verify I2C connections
2. Check I2C address (0x76 or 0x77)
3. Scan I2C bus with `i2c.scan()`
4. Check sensor power supply

### WiFi drops frequently

1. Check signal strength
2. Ensure 2.4GHz network
3. Reconnect in main loop
4. Add error handling

## Best Practices

1. **Use error handling** - WiFi/MQTT can fail
2. **Auto-reconnect** - Handle disconnects gracefully
3. **Retain config** - Use `retain=True` for discovery
4. **Low QoS** - Use QoS 0 for sensors
5. **Batch messages** - Publish all readings at once
6. **Add LWT** - Last Will Testament for offline status
