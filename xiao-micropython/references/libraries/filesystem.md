# Filesystem Operations for MicroPython on XIAO

## Overview

File operations on internal flash and external storage (SD cards).

## Internal Flash (os module)

### Basic File Operations

```python
import os

# List files in root
files = os.listdir('/')
print(files)

# Get current directory
print(os.getcwd())

# Create directory
os.mkdir('/data')

# Remove directory
os.rmdir('/data')

# Remove file
os.remove('test.txt')

# Rename file
os.rename('old.txt', 'new.txt')

# Get file info
stat = os.stat('file.txt')
print(f"Size: {stat[6]} bytes")
print(f"Modified: {stat[8]}")
```

### Read and Write Files

```python
# Write text file
with open('data.txt', 'w') as f:
    f.write('Hello XIAO!')
    f.write('Line 2\n')
    f.write('Line 3\n')

# Read text file
with open('data.txt', 'r') as f:
    content = f.read()
    print(content)

# Read line by line
with open('data.txt', 'r') as f:
    for line in f:
        print(f"Line: {line.strip()}")

# Append to file
with open('data.txt', 'a') as f:
    f.append('New line\n')
```

### Binary Files

```python
# Write binary data
data = bytes([0x01, 0x02, 0x03, 0x04])
with open('binary.bin', 'wb') as f:
    f.write(data)

# Read binary data
with open('binary.bin', 'rb') as f:
    content = f.read()
    print(content.hex())

# Read specific number of bytes
with open('binary.bin', 'rb') as f:
    first_byte = f.read(1)
    remaining = f.read()
```

### CSV Data Logging

```python
import time

# Create CSV header
with open('sensor.csv', 'w') as f:
    f.write('timestamp,temperature,humidity\n')

# Log data periodically
while True:
    timestamp = time.ticks_ms()
    temp = 25.3
    hum = 60.5

    with open('sensor.csv', 'a') as f:
        f.write(f'{timestamp},{temp},{hum}\n')

    print(f"Logged: {temp}C, {hum}%")
    time.sleep(60)  # Log every minute
```

### JSON Configuration

```python
import ujson
import os

# Default config
config = {
    'wifi': {
        'ssid': 'your-ssid',
        'password': 'your-password'
    },
    'mqtt': {
        'broker': 'broker.hivemq.com',
        'port': 1883
    }
}

# Save config
with open('config.json', 'w') as f:
    ujson.dump(config, f)

# Load config
if 'config.json' in os.listdir('/'):
    with open('config.json', 'r') as f:
        loaded = ujson.load(f)
        print(f"SSID: {loaded['wifi']['ssid']}")
```

### File Size and Space

```python
import os

# Get file size
stat = os.stat('data.txt')
size = stat[6]
print(f"File size: {size} bytes")

# Get filesystem info
fs_stat = os.statvfs('/')
block_size = fs_stat[0]
total_blocks = fs_stat[2]
free_blocks = fs_stat[3]

total_space = block_size * total_blocks
free_space = block_size * free_blocks

print(f"Total: {total_space} bytes")
print(f"Free: {free_space} bytes")
print(f"Used: {total_space - free_space} bytes")
```

### File Seek and Position

```python
# Write file
with open('data.txt', 'w') as f:
    f.write('0123456789')

# Read with seek
with open('data.txt', 'r') as f:
    f.seek(5)  # Go to position 5
    print(f.read())  # '56789'

# Seek from end
with open('data.txt', 'r') as f:
    f.seek(-3, 2)  # 3 bytes from end
    print(f.read())  # '789'
```

## SD Card (sdcard module)

### Initialize and Mount

```python
import os
from machine import Pin, SPI
import sdcard

# SPI for ESP32
spi = SPI(1, baudrate=1000000, sck=Pin(14), mosi=Pin(15), miso=Pin(12))
sd = sdcard.SDCard(spi, Pin(13))  # CS pin

# Mount
os.mount(sd, '/sd')

# List files
print(os.listdir('/sd'))
```

### SD Card Operations

```python
# Write to SD
with open('/sd/data.txt', 'w') as f:
    f.write('Data on SD card!')

# Read from SD
with open('/sd/data.txt', 'r') as f:
    print(f.read())

# Create directory
os.mkdir('/sd/logs')

# Write to subdirectory
with open('/sd/logs/sensor.log', 'w') as f:
    f.write('Sensor log\n')
```

### Data Logging to SD

```python
import time
import os
from machine import Pin, SPI
import sdcard

# Initialize SD
spi = SPI(1, baudrate=1000000, sck=Pin(14), mosi=Pin(15), miso=Pin(12))
sd = sdcard.SDCard(spi, Pin(13))
os.mount(sd, '/sd')

# Create log file with timestamp
def log_data(temp, hum):
    timestamp = time.ticks_ms()
    with open('/sd/sensor.log', 'a') as f:
        f.write(f'{timestamp},{temp},{hum}\n')
    print(f"Logged: {temp}C, {hum}%")

# Main loop
while True:
    # Read sensor
    temp = 25.0
    hum = 60.0

    # Log to SD
    log_data(temp, hum)

    time.sleep(60)
```

### Large File Operations

```python
# Write large file in chunks
def write_large_file(filename, data, chunk_size=1024):
    with open(filename, 'wb') as f:
        for i in range(0, len(data), chunk_size):
            chunk = data[i:i+chunk_size]
            f.write(chunk)

# Read large file in chunks
def read_large_file(filename, chunk_size=1024):
    with open(filename, 'rb') as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            # Process chunk
            print(f"Read {len(chunk)} bytes")
```

## LittleFS (Alternative Filesystem)

### Format to LittleFS

```python
import os
import vfs

# Unmount if mounted
try:
    os.umount('/')
except:
    pass

# Format flash as LittleFS (more efficient than FAT)
vfs.VfsLfs2.mkfs(bdev)

# Mount LittleFS
fs = vfs.VfsLfs2(bdev)
os.mount(fs, '/')
```

### LittleFS Operations

```python
# LittleFS works like regular filesystem
with open('/data.txt', 'w') as f:
    f.write('LittleFS data')

with open('/data.txt', 'r') as f:
    print(f.read())
```

## Flash Storage (esp32 module)

### Flash Partition

```python
import esp32
import os

# Get flash size
flash_size = esp32.flash_size()
print(f"Flash size: {flash_size} bytes")

# Create file in flash
with open('/flash/data.txt', 'w') as f:
    f.write('Stored in flash')
```

## File Utilities

### Check File Exists

```python
import os

def file_exists(filename):
    try:
        os.stat(filename)
        return True
    except OSError:
        return False

if file_exists('data.txt'):
    print("File exists!")
else:
    print("File not found")
```

### Copy File

```python
def copy_file(src, dst):
    with open(src, 'rb') as f_src:
        with open(dst, 'wb') as f_dst:
            while True:
                chunk = f_src.read(1024)
                if not chunk:
                    break
                f_dst.write(chunk)

copy_file('source.txt', 'destination.txt')
```

### Get File Extension

```python
def get_extension(filename):
    parts = filename.rsplit('.', 1)
    return parts[1] if len(parts) > 1 else ''

print(get_extension('data.txt'))  # 'txt'
print(get_extension('image.png'))  # 'png'
```

### Clean Old Files

```python
import os
import time

def clean_old_files(directory, max_age_hours=24):
    now = time.time()
    max_age = max_age_hours * 3600

    for filename in os.listdir(directory):
        filepath = f"{directory}/{filename}"
        try:
            stat = os.stat(filepath)
            mtime = stat[8]
            age = now - mtime

            if age > max_age:
                os.remove(filepath)
                print(f"Removed old file: {filename}")
        except:
            pass

clean_old_files('/sd/logs', 48)  # Remove files older than 48 hours
```

## Troubleshooting

### Out of memory

1. Process files in smaller chunks
2. Use `del` to free large variables
3. Close files when done
4. Check free space with `os.statvfs()`

### File corrupted

1. Unmount before power off
2. Use `flush()` to ensure writes complete
3. Consider LittleFS for better reliability
4. Add checksums for critical data

### SD card fails

1. Check SPI wiring
2. Verify format (FAT32)
3. Try lower baudrate
4. Use shorter cables
5. Check card size (< 32GB recommended)

### Slow writes

1. Write in larger chunks
2. Reduce write frequency
3. Consider LittleFS for faster operations
4. Check SD card class (Class 10 recommended)
