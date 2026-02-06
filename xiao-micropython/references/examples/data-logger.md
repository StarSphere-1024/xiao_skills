# Data Logger with SD Card (MicroPython)

## Complete data logging system with SD card storage and CSV export.

## Hardware

- XIAO ESP32C3/C6/S3 or RP2040
- SD card module (SPI)
- BME280 sensor (I2C)
- Optional: OLED display

## Wiring

### SD Card (SPI)
| XIAO Pin | SD Card |
|----------|---------|
| D8 | SCK |
| D9 | MISO |
| D10 | MOSI |
| D2 | CS |
| 3.3V | VCC |
| GND | GND |

### BME280 (I2C)
| XIAO Pin | BME280 |
|----------|--------|
| D4 | SDA |
| D5 | SCL |
| 3.3V | VCC |
| GND | GND |

## Code

```python
from machine import Pin, SPI, I2C
import sdcard
import uos
import time
import gc

# --- Configuration ---
SD_CS = Pin(2)
SD_SPI = SPI(1, sck=Pin(8), miso=Pin(9), mosi=Pin(10), baudrate=1000000)

I2C_SDA = Pin(4)
I2C_SCL = Pin(5)

LOG_INTERVAL = 60  # seconds
CSV_FILENAME = "datalog.csv"

# --- SD Card Initialization ---
def init_sdcard():
    try:
        sd = sdcard.SDCard(SD_SPI, SD_CS)
        uos.mount(sd, '/sd')
        print("SD card mounted")

        # List files
        print("Files on SD:", uos.listdir('/sd'))
        return True
    except Exception as e:
        print(f"SD card error: {e}")
        return False

# --- BME280 Sensor (Simplified) ---
class BME280:
    def __init__(self, i2c, address=0x76):
        self.i2c = i2c
        self.address = address
        # Configure sensor
        self.i2c.writeto(self.address, b'\xF2\x01')  # Humidity oversampling
        self.i2c.writeto(self.address, b'\xF4\x00')  # Pressure oversampling
        self.i2c.writeto(self.address, b'\xF5\x00')  # Normal mode

    def read_raw(self):
        # Read all data at once
        data = self.i2c.readfrom_mem(self.address, 0xF7, 8)

        # Parse data
        press = (data[0] << 12) | (data[1] << 4) | (data[2] >> 4)
        temp = (data[3] << 12) | (data[4] << 4) | (data[5] >> 4)
        hum = (data[6] << 8) | data[7]

        return press, temp, hum

    def read_compensated(self):
        press, temp, hum = self.read_raw()
        # Simplified compensation (use full library for accuracy)
        temp_c = (temp / 5120.0) - 25.0
        press_hpa = press / 25600.0
        hum_pct = hum / 1024.0
        return temp_c, press_hpa, hum_pct

# --- CSV Logging ---
def init_csv(filename):
    """Initialize CSV file with headers"""
    try:
        # Check if file exists
        try:
            with open(f'/sd/{filename}', 'r') as f:
                pass  # File exists
        except:
            # Create new file with headers
            with open(f'/sd/{filename}', 'w') as f:
                f.write('timestamp,temperature,humidity,pressure\n')
            print(f"Created {filename}")
    except Exception as e:
        print(f"CSV init error: {e}")

def log_data(filename, data):
    """Append data row to CSV"""
    try:
        with open(f'/sd/{filename}', 'a') as f:
            timestamp = time.ticks_ms()
            temp, hum, press = data
            line = f"{timestamp},{temp:.2f},{hum:.1f},{press:.1f}\n"
            f.write(line)
        print(f"Logged: {temp:.1f}°C, {hum:.0f}%, {press:.0f}hPa")
        return True
    except Exception as e:
        print(f"Log error: {e}")
        return False

# --- SD Card Info ---
def get_sd_info():
    """Get SD card information"""
    try:
        stat = uos.statvfs('/sd')
        total = stat[0] * stat[2] / 1024  # KB
        used = stat[0] * (stat[2] - stat[3]) / 1024
        free = total - used

        print(f"SD Card: {free:.0f}KB free / {total:.0f}KB total")
        return total, free
    except:
        return 0, 0

# --- File Management ---
def list_log_files():
    """List all CSV files"""
    try:
        files = uos.listdir('/sd')
        csv_files = [f for f in files if f.endswith('.csv')]
        print("Log files:")
        for f in csv_files:
            stat = uos.stat(f'/sd/{f}')
            size = stat[6]
            print(f"  {f}: {size} bytes")
        return csv_files
    except Exception as e:
        print(f"List error: {e}")
        return []

def read_last_lines(filename, lines=5):
    """Read last N lines from CSV"""
    try:
        with open(f'/sd/{filename}', 'r') as f:
            all_lines = f.readlines()
            last_lines = all_lines[-lines:]
            for line in last_lines:
                print(line.strip())
    except Exception as e:
        print(f"Read error: {e}")

# --- Data Export ---
def export_to_json(filename):
    """Export CSV data to JSON"""
    import ujson

    try:
        data = []
        with open(f'/sd/{filename}', 'r') as f:
            headers = f.readline().strip().split(',')

            for line in f:
                values = line.strip().split(',')
                entry = dict(zip(headers, values))
                data.append(entry)

        # Write JSON
        json_name = filename.replace('.csv', '.json')
        with open(f'/sd/{json_name}', 'w') as f:
            ujson.dump(data, f)

        print(f"Exported to {json_name}")
    except Exception as e:
        print(f"Export error: {e}")

# --- Main Loop ---
def main():
    print("=== XIAO Data Logger ===")

    # Initialize SD card
    if not init_sdcard():
        print("SD card init failed - using flash storage")
        csv_path = "datalog.csv"  # Flash
    else:
        csv_path = "/sd/datalog.csv"

    # Show SD info
    get_sd_info()

    # Initialize I2C
    i2c = I2C(0, sda=I2C_SDA, scl=I2C_SCL, freq=100000)

    # Initialize sensor
    try:
        bme = BME280(i2c)
        print("BME280 initialized")
    except Exception as e:
        print(f"Sensor error: {e}")
        print("Logging simulated data")
        bme = None

    # Initialize CSV
    init_csv(csv_path.split('/')[-1])

    # LED indicator
    led = Pin(13, Pin.OUT)

    # Logging loop
    counter = 0

    while True:
        try:
            # Blink LED
            led.value(1)
            time.sleep(0.1)
            led.value(0)

            # Read sensor
            if bme:
                temp, hum, press = bme.read_compensated()
            else:
                # Simulated data
                temp = 25.0 + (counter % 5)
                hum = 50 + (counter % 20)
                press = 1013 + (counter % 10)

            # Log data
            if csv_path.startswith('/sd/'):
                log_data(csv_path.split('/')[-1], (temp, hum, press))
            else:
                log_data(csv_path, (temp, hum, press))

            # Show SD info every 10 readings
            if counter % 10 == 0:
                get_sd_info()

            counter += 1

            # Wait
            print(f"Waiting {LOG_INTERVAL}s...")
            time.sleep(LOG_INTERVAL)

            # Cleanup
            gc.collect()

        except KeyboardInterrupt:
            print("\nStopped by user")
            break

        except Exception as e:
            print(f"Error: {e}")
            time.sleep(5)

    # Final status
    print("\n=== Session Summary ===")
    get_sd_info()
    list_log_files()

if __name__ == "__main__":
    main()
```

## Rotating Log Files

```python
import uos
import time

def rotate_logs(max_size=1024*1024):  # 1MB
    """Rotate log files when they get too big"""
    csv_path = "/sd/datalog.csv"

    try:
        stat = uos.stat(csv_path)
        if stat[6] > max_size:
            # Archive current file
            timestamp = time.ticks_ms()
            archive_name = f"/sd/datalog_{timestamp}.csv"
            uos.rename(csv_path, archive_name)
            print(f"Archived to {archive_name}")

            # Create new file
            init_csv("datalog.csv")
    except:
        pass  # File doesn't exist yet
```

## Data Analysis (On-Device)

```python
def analyze_data(filename, last_n=100):
    """Simple data analysis"""
    temps = []

    try:
        with open(f'/sd/{filename}', 'r') as f:
            # Skip header
            f.readline()

            # Read last N lines
            lines = f.readlines()[-last_n:]

            for line in lines:
                parts = line.strip().split(',')
                if len(parts) >= 2:
                    temps.append(float(parts[1]))

        if temps:
            avg = sum(temps) / len(temps)
            min_t = min(temps)
            max_t = max(temps)

            print(f"Temperature (last {len(temps)} readings):")
            print(f"  Average: {avg:.2f}°C")
            print(f"  Min: {min_t:.2f}°C")
            print(f"  Max: {max_t:.2f}°C")

    except Exception as e:
        print(f"Analysis error: {e}")
```

## HTTP Data Export

```python
import network
import socket

def start_web_server():
    """Simple web server to download logs"""
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect('YourSSID', 'YourPassword')

    while not wlan.isconnected():
        time.sleep(1)

    print(f"Server: http://{wlan.ifconfig()[0]}")

    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('', 80))
    s.listen(5)

    while True:
        conn, addr = s.accept()
        request = conn.recv(1024).decode()

        if 'GET /download' in request:
            # Send CSV file
            with open('/sd/datalog.csv', 'rb') as f:
                data = f.read()
                conn.send('HTTP/1.1 200 OK\r\n')
                conn.send('Content-Type: text/csv\r\n')
                conn.send(f'Content-Length: {len(data)}\r\n')
                conn.send('\r\n')
                conn.send(data)

        conn.close()
```

## Flash Storage Fallback

```python
# If SD card not available, use flash
try:
    init_sdcard()
    use_sd = True
except:
    print("SD card not available, using flash")
    use_sd = False
    csv_path = "datalog.csv"  # Flash
```

## Data Compression

```python
import uzlib

def compress_data(filename):
    """Compress CSV data"""
    try:
        with open(f'/sd/{filename}', 'rb') as f:
            data = f.read()

        compressed = uzlib.compress(data)

        # Save compressed
        comp_name = filename.replace('.csv', '.csv.gz')
        with open(f'/sd/{comp_name}', 'wb') as f:
            f.write(compressed)

        print(f"Compressed: {len(data)} -> {len(compressed)} bytes")
        return len(compressed)
    except Exception as e:
        print(f"Compression error: {e}")
        return 0
```

## Complete Example: Battery Powered Logger

```python
from machine import Pin, SPI, I2C
import sdcard
import uos
import time
import gc
import machine

# Low power configuration
LOG_INTERVAL = 300  # 5 minutes

def init_logger():
    # Initialize SD card
    sd = sdcard.SDCard(SPI(1, sck=Pin(8), miso=Pin(9), mosi=Pin(10)), Pin(2))
    uos.mount(sd, '/sd')

    # Initialize sensor
    i2c = I2C(0, sda=Pin(4), scl=Pin(5))
    bme = BME280(i2c)

    return bme

def log_and_sleep():
    # Quick wake
    bme = init_logger()

    # Read sensor
    temp, hum, press = bme.read_compensated()

    # Log to SD
    with open('/sd/log.csv', 'a') as f:
        f.write(f"{time.ticks_ms()},{temp:.2f},{hum:.1f},{press:.1f}\n")

    print(f"Logged: {temp:.1f}°C")

    # Cleanup
    bme = None
    uos.umount('/sd')
    gc.collect()

    # Deep sleep
    print(f"Sleeping for {LOG_INTERVAL}s...")
    machine.deepsleep(LOG_INTERVAL * 1000)

# Main
if __name__ == "__main__":
    log_and_sleep()
```

## Troubleshooting

### SD card not detected

1. Check wiring (especially CS pin)
2. Verify SPI pin mappings
3. Try lower baudrate: `baudrate=400000`
4. Check SD card format (FAT32)

### File corruption

1. Always close files properly
2. Use `with open()` statement
3. Disable caching if needed
4. Check SD card health

### Out of memory

1. Read file in chunks
2. Process line by line
3. Call `gc.collect()` frequently
4. Reduce buffer sizes

### Slow logging

1. Reduce log frequency
2. Optimize sensor reads
3. Use binary format instead of CSV
4. Buffer writes

## Best Practices

1. **Use `with open()`** - Ensures files are closed
2. **Flush periodically** - `f.flush()` to ensure data written
3. **Handle SD errors** - SD cards can fail
4. **Rotate logs** - Prevent files getting too large
5. **Backup data** - Copy important logs off SD card
6. **Monitor free space** - Check SD card capacity
7. **Use timestamps** - Make data searchable
