# ESPHome Troubleshooting

## Installation Issues

### "pip install failed"

**Python version too old**:
```bash
# Check Python version
python --version

# Must be 3.8 or higher
# Update Python on Windows/macOS from python.org
# Update Python on Linux:
sudo apt update && sudo apt install python3.11
```

**Permission denied**:
```bash
# Use --user flag
pip install --user esphome

# Or use virtual environment
python3 -m venv ~/esphome-venv
source ~/esphome-venv/bin/activate
pip install esphome
```

**SSL certificate error**:
```bash
# Disable SSL verification (not recommended for production)
pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org esphome

# Or update certificates (Linux)
sudo apt update && sudo apt install ca-certificates
```

### "esphome: command not found"

**Python script location not in PATH**:
```bash
# Run with full Python path
python -m esphome version

# Or add to PATH (Linux/macOS)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Or add to PATH (Windows PowerShell)
$env:Path += ";$env:APPDATA\Python\Python311\Scripts"
```

**Virtual environment not activated**:
```bash
# Activate virtual environment
source ~/esphome-venv/bin/activate  # Linux/macOS
~/esphome-venv/Scripts/activate     # Windows

# Verify
esphome version
```

### Docker Issues

**"Cannot connect to Docker daemon"**:
1. Start Docker Desktop application
2. Verify Docker is running: `docker ps`
3. Restart Docker Desktop if needed

**"Permission denied" on config directory** (Linux):
```bash
# Fix permissions
sudo chown -R $USER:$USER ~/esphome-config

# Or run with sudo (not recommended)
sudo docker run --rm -v ~/esphome-config:/config -it esphome/esphome version
```

**Volume mount not working** (Windows):
```powershell
# Use ${PWD} instead of %cd%
docker run --rm -v "${PWD}:/config" -it esphome/esphome version

# Or use full path
docker run --rm -v "C:\Users\YourName\esphome:/config" -it esphome/esphome version
```

## Configuration Issues

### "Invalid YAML syntax"

**Common YAML errors**:
```yaml
# ❌ Wrong: Using tabs instead of spaces
esphome:
	name: test

# ✅ Correct: Use 2 spaces for indentation
esphome:
  name: test

# ❌ Wrong: Missing colon
wifi
  ssid: "MyWiFi"

# ✅ Correct
wifi:
  ssid: "MyWiFi"

# ❌ Wrong: Quote mismatch
password: 'MyPassword"

# ✅ Correct
password: "MyPassword"
```

**Validate configuration**:
```bash
esphome config xiao-sensor.yaml
```

### "!secret not defined"

**Secrets file not found**:
1. Create `secrets.yaml` in same directory as config
2. Or specify secrets path:
   ```yaml
   esphome:
     secrets:
       - /path/to/secrets.yaml
   ```

**Secret name mismatch**:
```yaml
# secrets.yaml:
wifi_ssid: "MyWiFi"
wifi_password: "MyPassword"

# xiao-sensor.yaml:
wifi:
  ssid: !secret wifi_ssid      # ✅ Correct
  password: !secret wifi_pass   # ❌ Wrong! Should be wifi_password
```

**Secrets file location** (ESPHome add-on):
- Stored in `/config/esphome/secrets.yaml`
- Edit via Home Assistant's ESPHome web UI

### "Platform not supported"

**Wrong platform specified**:
```yaml
# ❌ Wrong: RP2040 not supported by ESPHome
esphome:
  name: xiao_rp2040
  platform: RP2040

# ✅ Correct: Use ESP32 platform for XIAO ESP32 boards
esphome:
  name: xiao_esp32c3
  platform: ESP32
  board: esp32-c3-devkitm-1
```

**Supported XIAO boards in ESPHome**:
- ESP32C3: `esp32-c3-devkitm-1`
- ESP32C6: `esp32-c6-devkitm-1`
- ESP32S3: `esp32-s3-devkitm-1`

## Compilation Issues

### "Failed to compile"

**Missing dependencies**:
```bash
# Update ESPHome
pip install --upgrade esphome

# Clear build cache
esphome clean xiao-sensor.yaml
esphome compile xiao-sensor.yaml
```

**Out of memory during compilation** (Docker):
```bash
# Increase Docker memory limit (Docker Desktop > Settings > Resources)
# Minimum 4GB recommended, 8GB for large projects
```

**Toolchain error**:
```bash
# Remove platformio cache
rm -rf ~/.platformio/.cache
esphome clean xiao-sensor.yaml
esphome compile xiao-sensor.yaml
```

### "Sketch too large"

**Firmware exceeds flash size**:
1. Check board flash size:
   ```yaml
   esphome:
     platform: ESP32
     board: esp32-c3-devkitm-1
     flash_size: 4MB
   ```

2. Reduce features:
   - Disable logger: `logger:`
   - Disable web server: Remove `web_server:`
   - Reduce components: Remove unused sensors

3. Optimize code:
   ```yaml
   esphome:
     platformio_options:
       build_flags:
         - -O3  # Maximum optimization
   ```

## Upload Issues

### "Failed to connect to device"

**USB device not found**:
```bash
# List available ports
# Windows:
Get-PnpDevice -Class Ports | Select-Object Status, FriendlyName

# macOS:
ls -l /dev/cu.*

# Linux:
ls -l /dev/ttyUSB* /dev/ttyACM*
```

**Wrong COM port selected**:
1. Unplug XIAO
2. Note available ports
3. Plug XIAO back in
4. Note new port that appears
5. Use that port:
   ```bash
   esphome run xiao-sensor.yaml --device /dev/ttyUSB0  # Linux
   esphome run xiao-sensor.yaml --device COM3          # Windows
   ```

**Wrong baud rate**:
```yaml
esphome:
  platformio_options:
   upload_speed: 115200  # Try 921600, 460800, 115200
```

### "Permission denied" (USB)

**Linux permissions**:
```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Log out and log back in

# Or configure udev rules
sudo nano /etc/udev/rules.d/99-xiao.rules
# Add:
SUBSYSTEM=="tty", ATTRS{idVendor}=="303a", ATTRS{idProduct}=="1001", MODE="0666"
sudo udevadm control --reload-rules
```

**Windows drivers**:
- ESP32 series: No drivers needed (built-in USB CDC)
- Install Arduino IDE for CH340 drivers (if using older boards)

### Board Not Entering Bootloader

**ESP32C3/C6/S3 not entering upload mode**:
1. Hold BOOT button
2. Press RESET button (or plug USB)
3. Release both
4. Run upload within 5 seconds

**Auto-upload not working**:
```yaml
esphome:
  platformio_options:
    upload_flags:
      - --port="/dev/ttyUSB0"  # Force specific port
```

**Manual bootloader mode**:
1. Hold BOOT button
2. Plug USB cable
3. Release BOOT button
4. Run: `esphome run xiao-sensor.yaml`

## WiFi Issues

### "WiFi not connecting"

**Common causes**:

1. **5GHz network**: ESP32 only supports 2.4GHz
   ```yaml
   wifi:
     ssid: "MyWiFi_2G"  # Use 2.4GHz network
     password: !secret wifi_password
   ```

2. **Wrong password**: Check `secrets.yaml`

3. **SSID not found**:
   ```yaml
   wifi:
     ssid: "Exact SSID Name"
     password: !secret wifi_password
     # Enable logging to debug
   logger:
     level: DEBUG
   ```

4. **WiFi power too low**:
   ```yaml
   esp32:
     board: esp32-c3-devkitm-1
     framework:
       type: arduino
   wifi:
     power_save_mode: NONE
     output_power: 20dB  # Maximum power
   ```

### WiFi connects but no internet

**DNS issues**:
```yaml
wifi:
  ssid: "MyWiFi"
  password: !secret wifi_password
  manual_ip:
    static_ip: 192.168.1.100
    gateway: 192.168.1.1
    subnet: 255.255.255.0
    dns1: 8.8.8.8
    dns2: 8.8.4.4
```

**Firewall blocking**: Check router firewall settings

## OTA Issues

### "OTA upload failed"

**First upload must be via USB**:
```bash
# First upload (USB)
esphome run xiao-sensor.yaml

# Subsequent uploads (OTA)
esphome upload xiao-sensor.yaml
```

**Device offline**:
1. Check device is connected to WiFi
2. Ping device: `ping xiao-sensor.local` (or use IP)
3. Check device logs via USB:
   ```bash
   esphome logs xiao-sensor.yaml
   ```

**OTA password mismatch**:
```yaml
ota:
  password: !secret ota_password  # Must match secrets.yaml
```

### "Connection refused" (OTA)

**Wrong port**:
```yaml
ota:
  port: 3232  # Default ESPHome OTA port
```

**Firewall blocking OTA**:
- Allow port 3232 on firewall
- Check router firewall settings

## API Issues

### "API connection failed"

**Password mismatch**:
```yaml
api:
  password: !secret api_password  # Must match HA configuration
```

**Home Assistant can't connect**:
1. Check device IP is correct
2. Ping device from HA server
3. Check firewall not blocking port 6053
4. Verify API password in HA integration

**API not enabled**:
```yaml
api:  # Required for Home Assistant integration
  password: !secret api_password
  reboot_timeout: 0s
```

## Device Reboots

### Constant restart loop

**Power supply issues**:
- Check USB cable quality (data + power)
- Use powered USB hub
- Check for short circuit

**Watchdog timer reset**:
```yaml
# Disable watchdog for debugging
esphome:
  on_boot:
    - priority: -100
      then:
        - logger.log: "ESPHome started"
```

**Exception in code**:
```bash
# View logs via USB
esphome logs xiao-sensor.yaml

# Look for "Guru Meditation Error" or "E (1234)"
```

**WiFi watchdog**:
```yaml
wifi:
  reboot_timeout: 5min  # Time before reboot on WiFi disconnect
```

## Home Assistant Integration Issues

### Device not discovered

**Manual integration**:
1. Home Assistant > Settings > Devices & Services
2. Add Integration > ESPHome
3. Enter device address: `http://xiao-sensor.local` or IP
4. Enter API password
5. Submit

### Device shows as "Unavailable"

**Check device is online**:
```bash
# Ping device
ping xiao-sensor.local

# Or use IP
ping 192.168.1.100
```

**Check API is enabled**:
```yaml
api:
  password: !secret api_password
```

**Reconnect device**:
1. Settings > Devices & Services > ESPHome
2. Click device > Configure
3. Re-enter password if needed

## Performance Issues

### Slow compilation

**Reduce build time**:
```yaml
esphome:
  platformio_options:
    build_flags:
      - -O3  # Maximum optimization
    board_build.f_cpu: 160000000L  # Overclock (if supported)
```

**Use cache**:
```bash
# First build: download dependencies
esphome compile xiao-sensor.yaml

# Subsequent builds use cache
esphome compile xiao-sensor.yaml
```

### High memory usage

**Reduce logger buffer**:
```yaml
logger:
  level: INFO
  baud_rate: 0  # Disable serial logging
```

**Disable web server**:
```yaml
# Remove web_server: section entirely
```

## Getting Help

### Debug Information to Collect

1. **ESPHome version**: `esphome version`
2. **Config file**: Attach YAML (remove secrets)
3. **Error message**: Full error text
4. **Logs**: `esphome logs xiao-sensor.yaml` output
5. **Board model**: XIAO ESP32C3 / ESP32C6 / ESP32S3
6. **Upload method**: USB or OTA

### Useful Resources

- **ESPHome Discord**: https://discord.gg/KhAMKrd
- **ESPHome Forum**: https://community.esphome.io/
- **ESPHome GitHub**: https://github.com/esphome/esphome/issues
- **Home Assistant Forum**: https://community.home-assistant.io/
- **XIAO Wiki**: https://wiki.seeedstudio/XIAO

### Enable Verbose Logging

```yaml
logger:
  level: VERY_VERBOSE
  logs:
    component: DEBUG
    wifi: DEBUG
```

```bash
# View logs
esphome logs xiao-sensor.yaml

# Compile with verbose output
esphome compile xiao-sensor.yaml --verbose
```

## Recovery

### Restore Corrupted Firmware

1. Hold BOOT button
2. Plug USB
3. Release BOOT
4. Flash new firmware:
   ```bash
   esphome run xiao-sensor.yaml --device COM3
   ```

### Factory Reset XIAO

**Clear WiFi credentials**:
```bash
esphome clear-wifi xiao-sensor.yaml
```

**Full firmware erase** (esptool):
```bash
esptool.py --chip esp32c3 --port COM3 erase_flash
```

Then reflash:
```bash
esphome run xiao-sensor.yaml
```
