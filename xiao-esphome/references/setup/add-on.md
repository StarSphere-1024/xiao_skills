# ESPHome Home Assistant Add-on Installation

## Overview

ESPHome add-on is the easiest way to run ESPHome when using Home Assistant Supervisor (Home Assistant OS). No Python installation required.

## Prerequisites

### Home Assistant Requirements

- **Home Assistant OS**: 7.0 or later
- **Home Assistant Supervised**: Any version
- **Home Assistant Container**: ❌ Add-on not available (use CLI or Docker)

### Check Installation Type

**Home Assistant**: Settings > System > Top bar (3 dots) > About

Should show:
- "Home Assistant OS" (recommended)
- "Home Assistant Supervised" (supported)
- "Home Assistant Container" (use CLI/Docker instead)

## Installation

### Step 1: Open Add-on Store

1. Home Assistant sidebar: **Settings**
2. **Add-ons** > **Add-on Store**
3. Click search icon

### Step 2: Install ESPHome Add-on

1. Search for "ESPHome"
2. Click **ESPHome** add-on
3. Click **Install**
4. Wait for download (30-60 seconds)

### Step 3: Configure Add-on

1. Click **ESPHome** add-on
2. Click **Configuration** tab
3. Edit configuration:

```yaml
options:
  ssl: true
  certfile: fullchain.pem
  keyfile: privkey.pem
  reload: always
  lefty: false
```

4. **Valid options:**
   - `ssl`: Enable HTTPS (recommended)
   - `certfile`: SSL certificate path
   - `keyfile`: SSL private key path
   - `reload`: Auto-reload on changes (always/on_changes/on_upload)
   - `lefty`: Use Lefty editor instead of Ace (experimental)

5. Click **Save**

### Step 4: Start Add-on

1. Click **Info** tab
2. Click **Start**
3. Wait for "Add-on is running" message
4. Click **Open Web UI** (new tab opens)

## Using ESPHome Web UI

### Main Features

**Dashboard**:
- All ESPHome configurations listed
- Status indicators (online/offline)
- Quick actions (edit, compile, upload, logs)

**Editor**:
- Create and edit YAML configurations
- Built-in validation
- Syntax highlighting

**Logs**:
- Real-time device logs
- Debug ESPHome devices

**Validate**:
- Check configuration syntax
- Preview compiled firmware

### Create New Configuration

1. Click **+ New Device**
2. Enter device name: `xiao-sensor`
3. Click **Next**
4. Select device type: **ESP32-C3** (for XIAO ESP32C3)
5. Click **Next**
6. Review configuration
7. Click **Create**

### Upload Firmware

**Method 1: USB (First Time)**

1. Connect XIAO via USB
2. In ESPHome: **Edit** device
3. Click **Upload** (USB)
4. Select COM port
5. Follow on-screen prompts

**Method 2: OTA (Subsequent Updates)**

1. XIAO must be online (connected to WiFi)
2. In ESPHome: Click device
3. Click **Upload** (OTA)
4. Firmware updates wirelessly

## Advanced Configuration

### Multiple ESPHome Instances

Run multiple ESPHome instances with different configurations:

1. Settings > Add-ons > Add-on Store
2. Search "ESPHome"
3. Click three dots > **Install in...**
4. Choose different folder

### Network Access

**Local network access**:
- Default: http://homeassistant.local:6052
- Or: http://[HA-IP]:6052

**HTTPS access** (if SSL enabled):
- https://homeassistant.local:6052
- Accept self-signed certificate warning

### Backup Configuration

ESPHome configurations stored in:
```
/homeassistant/esphome/.esphome/
```

**Backup**:
1. Home Assistant > **Settings**
2. **System** > **Backups**
3. Click **Create Backup**
4. ESPHome configs included in backup

## Integration with Home Assistant

### ESPHome Integration

1. Home Assistant > **Settings**
2. **Devices & Services**
3. **Add Integration** > **Search "ESPHome"**
4. ESPHome devices auto-discovered
5. Click **Submit** for each device

### View ESPHome Devices

1. **Settings** > **Devices & Services**
2. **ESPHome** integration
3. All XIAO devices listed

### Automations with ESPHome

ESPHome devices appear as native HA entities:

**Example automation**:
```yaml
alias: "XIAO Motion Light"
description: "Turn on light when XIAO detects motion"
trigger:
  - platform: state
    entity_id: binary_sensor.xiao_esp32c3_pir
    to: "on"
action:
  - service: light.turn_on
    target:
      entity_id: light.living_room
```

## Troubleshooting

### Add-on Won't Start

**Check logs**:
1. Add-on > **Info** tab
2. **Check Logs**
3. Look for error messages

**Common errors**:

*"Address already in use"*:
- Port 6052 already in use
- Stop other ESPHome instances
- Or change port in configuration

*"Permission denied"*:
- Check add-on has proper permissions
- Restart Home Assistant

### Web UI Not Accessible

**Cannot access http://homeassistant.local:6052**:

1. Check add-on is running
2. Try IP address: http://[HA-IP]:6052
3. Check firewall settings
4. Clear browser cache

### Device Not Discovered

**ESPHome integration doesn't find devices**:

1. Check device is online (ping IP)
2. Check device logs in ESPHome
3. Verify API password matches
4. Manual configuration:
   - Settings > Devices & Services > Add Integration
   - Enter device IP: http://[XIAO-IP]
   - Enter API password

### USB Upload Fails

**"Failed to connect to device"**:

1. Check XIAO is connected via USB
2. Check COM port in ESPHome
3. Hold BOOT button while uploading
4. Try different USB cable (data + power)
5. Check USB drivers installed

### OTA Upload Fails

**"Connection refused"**:

1. XIAO must be on same network as HA
2. Check XIAO is online (ping test)
3. Check firewall not blocking port 6053
4. Verify API password correct

## Updating ESPHome Add-on

1. Settings > Add-ons > Add-on Store
2. Click ESPHome add-on
3. If update available: **Update**
4. Restart add-on after update

## Uninstallation

### Remove Add-on

1. Settings > Add-ons > Add-on Store
2. Click ESPHome add-on
3. Three dots > **Uninstall**
4. Confirm uninstall

### Backup Before Uninstall

1. **Settings** > **System** > **Backups**
2. **Create Backup**
3. Download backup file

## Comparison: Add-on vs CLI vs Docker

| Feature | Add-on | CLI | Docker |
|---------|--------|-----|--------|
| Installation | Easiest | Easy | Medium |
| Updates | ✅ Auto | Manual | Manual |
| HA Integration | ✅ Native | Manual | Manual |
| USB Access | ⚠️ Complex | Simple | Complex |
| Customization | Limited | Full | Full |
| Resource Usage | Shared | Low | Isolated |
| Dependencies | ✅ Bundled | Python | Docker |
| CI/CD | ❌ | ✅ | ✅ |

## When to Use Add-on

**Use ESPHome add-on when:**
- Running Home Assistant OS/Supervised
- Want easiest setup experience
- Don't need advanced customization
- Prefer web UI over command-line
- Don't use CI/CD pipelines

**Use CLI/Docker when:**
- Running Home Assistant Container
- Need command-line automation
- Using CI/CD pipelines
- Need custom ESPHome versions
- Want resource isolation
- Developing ESPHome custom components

## Links

- ESPHome Add-on: https://github.com/esphome/home-assistant-addon
- ESPHome Docs: https://esphome.io/
- Home Assistant Add-ons: https://www.home-assistant.io/add-ons/

## Next Steps

After ESPHome add-on installation:

1. [Create first device](#create-new-configuration)
2. [Configure XIAO board](#configure-xiao-boards)
3. [Upload firmware](#upload-firmware)
4. [Integrate with Home Assistant](#integration-with-home-assistant)
5. [Explore example configurations](../examples/weather-station.md)
