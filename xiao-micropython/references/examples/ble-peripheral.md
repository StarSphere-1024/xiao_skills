# BLE Peripheral Example (MicroPython)

## Complete BLE Sensor with Battery

This example shows a complete BLE peripheral sensor with battery monitoring.

## Hardware

- XIAO nRF52840
- BME280 sensor (I2C)
- LED indicator

## Code

```python
import bluetooth
import struct
import time
from machine import Pin, I2C
import bme280
import gc

# BLE UUIDs
_ENV_SENSING_UUID = bluetooth.UUID(0x181A)
_TEMP_UUID = bluetooth.UUID(0x2A6E)
_HUMID_UUID = bluetooth.UUID(0x2A6F)
_PRESS_UUID = bluetooth.UUID(0x2A6D)

_BATTERY_SERVICE_UUID = bluetooth.UUID(0x180F)
_BATTERY_UUID = bluetooth.UUID(0x2A19)

# BLE Configuration
DEVICE_NAME = "XIAO-Sensor"

# Initialize sensor
i2c = I2C(0, sda=Pin(0), scl=Pin(1))
bme = bme280.BME280(i2c=i2c)

# LED
led = Pin(18, Pin.OUT)

class BLESensor:
    def __init__(self):
        self._ble = bluetooth.BLE()
        self._ble.active(True)
        self._ble.config(gap_name=DEVICE_NAME)

        # Register handlers
        self._ble.irq(self._irq)

        # Initialize services
        self._ble_services = self._setup_services()
        self._ble.gatts_register_services(self._ble_services)

        # Get characteristic handles
        self._handles = self._get_handles()

        # Set initial values
        self._set_initial_values()

        # Start advertising
        self._start_advertising()

        print(f"BLE advertising as: {DEVICE_NAME}")

    def _setup_services(self):
        """Setup BLE services and characteristics"""
        # Environment Sensing Service
        temp_char = (_TEMP_UUID, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY)
        humid_char = (_HUMID_UUID, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY)
        press_char = (_PRESS_UUID, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY)

        # Battery Service
        batt_char = (_BATTERY_UUID, bluetooth.FLAG_READ | bluetooth.FLAG_NOTIFY)

        # Create services
        env_service = (_ENV_SENSING_UUID, (temp_char, humid_char, press_char))
        batt_service = (_BATTERY_SERVICE_UUID, (batt_char,))

        return (env_service, batt_service)

    def _get_handles(self):
        """Get characteristic handles"""
        handles = self._ble.gatts_read_handles()

        return {
            'temp': handles[1],
            'humid': handles[2],
            'press': handles[3],
            'battery': handles[5]
        }

    def _set_initial_values(self):
        """Set initial characteristic values"""
        # Temperature
        self._ble.gatts_write(self._handles['temp'], struct.pack('<h', 0))

        # Humidity
        self._ble.gatts_write(self._handles['humid'], struct.pack('<H', 0))

        # Pressure
        self._ble.gatts_write(self._handles['press'], struct.pack('<I', 0))

        # Battery
        self._ble.gatts_write(self._handles['battery'], struct.pack('<B', 100))

    def _start_advertising(self):
        """Start advertising"""
        # Advertising data
        adv_data = bytearray()
        adv_data += struct.pack('<BB', 0x02, 0x01)  # Flags
        adv_data += struct.pack('<BB', 0x03, 0x03)  # Service list
        adv_data += struct.pack('<H', _ENV_SENSING_UUID)
        adv_data += struct.pack('<B', len(DEVICE_NAME))
        adv_data += DEVICE_NAME.encode()

        # Scan response
        scan_resp = bytearray()
        scan_resp += struct.pack('<BB', 0x02, 0x0A)  # TX power
        scan_resp += struct.pack('<b', 20)  # 20dBm

        self._ble.gap_advertise(100, adv_data=adv_data, scan_resp=scan_resp)

    def _irq(self, event, data):
        """BLE IRQ handler"""
        if event == 1:  # Connected
            conn_handle, addr_type, addr = data
            print(f"Connected to: {bytes(addr).hex()}")
            led.on()

        elif event == 2:  # Disconnected
            conn_handle, = data
            print("Disconnected")
            led.off()

            # Restart advertising
            self._start_advertising()

    def update_sensor_data(self):
        """Read and update sensor data"""
        # Read BME280
        temp, press, hum = bme.read_compensated_data()

        # Convert to required formats
        temp_int = int(temp / 100)  # Temperature in 0.01°C units
        hum_int = int(hum / 1024)   # Humidity in 0.01% units
        press_int = int(press)      # Pressure in Pa

        # Write to characteristics
        self._ble.gatts_write(self._handles['temp'], struct.pack('<h', temp_int))
        self._ble.gatts_write(self._handles['humid'], struct.pack('<H', hum_int))
        self._ble.gatts_write(self._handles['press'], struct.pack('<I', press_int))

        # Notify
        conn_indices = self._ble.gatts_get_connection_count()
        if conn_indices:
            self._ble.gatts_notify(0, self._handles['temp'])
            self._ble.gatts_notify(0, self._handles['humid'])
            self._ble.gatts_notify(0, self._handles['press'])

        return temp, hum, press

    def update_battery(self, percent):
        """Update battery level"""
        self._ble.gatts_write(self._handles['battery'], struct.pack('<B', percent))

        # Notify
        conn_indices = self._ble.gatts_get_connection_count()
        if conn_indices:
            self._ble.gatts_notify(0, self._handles['battery'])

    def read_battery_voltage(self):
        """Read battery voltage and estimate percentage"""
        try:
            adc = machine.ADC(0)
            adc.read()
            voltage = adc.read_u16() * 3.6 / 65535

            # Simple estimation (adjust for your battery)
            percent = int((voltage - 3.0) / (4.2 - 3.0) * 100)
            percent = max(0, min(100, percent))

            return voltage, percent
        except:
            return 3.7, 100

# Initialize BLE sensor
ble_sensor = BLESensor()

# Main loop
last_batt_read = 0
BATT_INTERVAL = 60000  # Check battery every minute

while True:
    try:
        # Update sensor data
        temp, hum, press = ble_sensor.update_sensor_data()

        # Display values
        temp_c = temp / 100
        hum_p = hum / 100
        press_hpa = press / 100

        print(f"Temp: {temp_c:.1f}C, Hum: {hum_p:.1f}%, Press: {press_hpa:.1f}hPa")

        # Update battery periodically
        now = time.ticks_ms()
        if time.ticks_diff(now, last_batt_read) > BATT_INTERVAL:
            last_batt_read = now
            voltage, percent = ble_sensor.read_battery_voltage()
            ble_sensor.update_battery(percent)
            print(f"Battery: {voltage:.2f}V ({percent}%)")

        # Free memory
        gc.collect()

        # Wait
        time.sleep(2)

    except Exception as e:
        print(f"Error: {e}")
        time.sleep(5)
```

## BLE Client (Testing)

### Testing with nRF Connect

1. Install nRF Connect app
2. Scan for "XIAO-Sensor"
3. View services:
   - Environmental Sensing (181A)
   - Battery (180F)
4. Subscribe to notifications
5. View live data

## Python BLE Scanner

### Scan and Connect

```python
import bluetooth

def scan_devices(duration=4):
    """Scan for BLE devices"""
    ble = bluetooth.BLE()
    ble.active(True)

    def bt_irq(event, data):
        if event == 5:  # Scan result
            addr_type, addr, adv_type, rssi, adv_data = data
            name = None
            # Parse adv_data for name
            for i in range(len(adv_data) - 1):
                if adv_data[i] == 0x08 and adv_data[i + 1] == 0x02:
                    name = adv_data[i + 2:i + 2 + adv_data[i - 1]]
                    break
            if name:
                print(f"Found: {name} - {addr.hex()}")

    ble.irq(bt_irq)
    ble.gap_scan(duration)
    time.sleep(duration + 1)
    ble.active(False)

# Scan
scan_devices()
```

## Custom Service

### Add Custom Characteristic

```python
# Custom UUIDs
_CUSTOM_SERVICE_UUID = bluetooth.UUID(0x180F)
_CUSTOM_CHAR_UUID = bluetooth.UUID(0x2A19)

# In setup_services()
custom_char = (_CUSTOM_CHAR_UUID, bluetooth.FLAG_READ | bluetooth.FLAG_WRITE)
custom_service = (_CUSTOM_SERVICE_UUID, (custom_char,))

# Handle write events
def _irq(self, event, data):
    if event == 3:  # Write
        conn_handle, value_handle = data
        # Handle write
```

## Connection Parameters

### Optimize for Battery

```python
def update_connection_params(self):
    """Update connection parameters for battery optimization"""
    # Min interval: 500ms (0x0004 * 1.25ms)
    # Max interval: 1000ms (0x0008 * 1.25ms)
    # Slave latency: 4
    # Supervision timeout: 20s (0x0BB0 * 10ms)

    try:
        self._ble.gap_conn_parameter_update(
            conn_handle=0,
            conn_interval_min=0x0004,
            conn_interval_max=0x0008,
            conn_latency=4,
            supervision_timeout=0x0BB0
        )
    except:
        pass
```

## Bonding (Pairing)

### Enable Secure Connections

```python
def enable_bonding(self):
    """Enable BLE bonding"""
    # Set IO capabilities
    self._ble.config(gap_security=True)

    # Set passkey (for display)
    self._ble.config(passkey=123456)

    # Enable pairing
    # Note: Full bonding requires more setup
```

## RSSI Monitoring

### Monitor Signal Strength

```python
def get_rssi(self):
    """Get connection RSSI"""
    try:
        rssi = self._ble.gap_get_rssi()
        return rssi
    except:
        return None

# In main loop
rssi = ble_sensor.get_rssi()
if rssi is not None:
    print(f"RSSI: {rssi} dBm")
```

## Troubleshooting

### Advertising not visible

1. Check advertising interval (not too short)
2. Verify device name
3. Check for BLE interference
4. Restart scanning device

### Can't connect

1. Check connection parameters
2. Verify service UUIDs
3. Monitor heap memory
4. Check for errors in IRQ handler

### Characteristics not updating

1. Verify notify flag is set
2. Check client is subscribed
3. Ensure valid data format
4. Monitor connection state

### Memory errors

1. Free unused objects
2. Use smaller buffers
3. Call gc.collect()
4. Monitor free memory

## Best Practices

1. **Advertising**: Use appropriate interval
2. **Notifications**: Don't spam, use appropriate rate
3. **Memory**: Free objects, use gc.collect()
4. **Connections**: Handle disconnect gracefully
5. **Data**: Use correct format for characteristics

## Data Formats

### Temperature (0x2A6E)
- sint16: Temperature in 0.01°C units
- Example: 23.50°C = 2350

### Humidity (0x2A7D or 0x2A6F)
- uint16: Humidity in 0.01% units
- Example: 45.20% = 4520

### Pressure (0x2A6D)
- uint32: Pressure in Pascal
- Example: 101325 Pa = 1013.25 hPa

### Battery (0x2A19)
- uint8: Percentage (0-100)
- Example: 85% = 85
