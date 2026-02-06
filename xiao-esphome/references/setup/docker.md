# ESPHome Docker Installation

## Overview

Run ESPHome in Docker container for isolated, reproducible builds. Ideal for CI/CD or avoiding Python dependency issues.

## Prerequisites

### Install Docker

**Windows:**
1. Download Docker Desktop: https://www.docker.com/products/docker-desktop/
2. Run installer
3. Restart computer
4. Verify: `docker --version`

**macOS:**
1. Download Docker Desktop for Mac
2. Install and start Docker
3. Verify: `docker --version`

**Linux:**
```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker
```

## Running ESPHome in Docker

### Basic Usage

```bash
# Pull ESPHome image
docker pull esphome/esphome

# Run ESPHome command
docker run --rm -v %cd%:/config -it esphome/esphome version
```

### Linux/macOS

```bash
# Run from current directory
docker run --rm -v "$(pwd)":/config -it esphome/esphome compile

# Or use absolute path
docker run --rm -v ~/esphome:/config -it esphome/esphome run xiao-sensor.yaml
```

### Windows (PowerShell)

```powershell
# From current directory
docker run --rm -v "${PWD}:/config" -it esphome/esphome compile

# Or use full path
docker run --rm -v "C:\Users\YourName\esphome:/config" -it esphome/esphome run
```

## Docker Compose Setup

### Create docker-compose.yml

```yaml
version: '3'
services:
  esphome:
    image: esphome/esphome
    volumes:
      - ./config:/config
      - /dev/ttyUSB0:/dev/ttyUSB0
    devices:
      - /dev/ttyUSB0:/dev/ttyUSB0
    network_mode: host
```

### Use Docker Compose

```bash
# Run ESPHome commands
docker-compose run esphome version
docker-compose run esphome config xiao-sensor.yaml
docker-compose run esphome compile xiao-sensor.yaml
docker-compose run esphome run xiao-sensor.yaml
```

## USB Device Access

### Linux

For USB device access in Docker:

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Verify device access
ls -l /dev/ttyUSB0
```

Docker Compose with device permissions:
```yaml
version: '3'
services:
  esphome:
    image: esphome/esphome
    volumes:
      - ./config:/config
    group_add:
      - " dialout"  # Add to dialout group
    devices:
      - /dev/ttyUSB0:/dev/ttyUSB0
    privileged: true  # May need for USB access
```

### Windows

Docker Desktop automatically shares drives. No additional setup needed.

### macOS

Docker Desktop shares `/Users` directory by default.

For USB access (macOS doesn't expose USB devices directly):
1. Use network upload (OTA) after initial upload
2. Or use ESPHome add-on in Home Assistant

## Persistent Configuration

### Mount Config Directory

```bash
# Create config directory
mkdir -p ~/esphome-config
cd ~/esphome-config
```

### Create Docker Alias (Linux/macOS)

Add to `~/.bashrc` or `~/.zshrc`:

```bash
alias esphome='docker run --rm -v ~/esphome-config:/config -it esphome/esphome'
```

Use:
```bash
esphome version
esphome config xiao-sensor.yaml
```

### Windows PowerShell Function

Add to `$PROFILE`:

```powershell
function esphome {
    docker run --rm -v "${PWD}:/config" -it esphome/esphome $args
}
```

Use:
```powershell
esphome version
esphome config xiao-sensor.yaml
```

## Docker Image Tags

### Official Images

- `esphome/esphome:latest` - Latest release
- `esphome/esphome:2023.12.0` - Specific version
- `esphome/esphome:beta` - Beta release
- `esphome/esphome:dev` - Development build

### Pull Specific Version

```bash
docker pull esphome/esphome:2023.12.0
docker run --rm -v %cd%:/config -it esphome/esphome:2023.12.0 version
```

## Common Docker Commands

### Update ESPHome Image

```bash
docker pull esphome/esphome:latest
```

### Clean Up Docker Resources

```bash
# Remove dangling images
docker image prune

# Remove unused images
docker image prune -a

# Remove all ESPHome containers
docker ps -a | grep esphome | awk '{print $1}' | xargs docker rm
```

### View Docker Logs

```bash
# View container logs
docker logs <container_id>

# Follow logs
docker logs -f <container_id>
```

## Troubleshooting

### "Permission Denied" on Config Directory

```bash
# Fix permissions
sudo chown -R $USER:$USER ~/esphome-config
```

### "Cannot Connect to Docker Daemon"

1. Start Docker Desktop application
2. Verify Docker is running: `docker ps`
3. Restart Docker Desktop if needed

### USB Device Not Accessible (Linux)

1. Add user to dialout group
2. Add device to docker-compose.yml:
   ```yaml
   devices:
     - /dev/ttyUSB0:/dev/ttyUSB0
   ```
3. Use privileged mode:
   ```yaml
   privileged: true
   ```

### "No Space Left on Device"

```bash
# Clean Docker cache
docker system prune -a

# Remove old images
docker image prune -a
```

### Slow Compilation

```bash
# Increase Docker resources (Docker Desktop settings)
# - More CPUs
# - More memory
# - More disk space
```

## Docker vs CLI Installation

| Feature | Docker | CLI |
|---------|--------|-----|
| Isolation | ✅ Full | ❌ Shared |
| Dependencies | ✅ Bundled | ❌ Requires Python |
| Setup Time | Medium | Fast |
| Disk Space | High | Low |
| USB Access | Complex | Simple |
| CI/CD | ✅ Excellent | ⚠️ Possible |
| Updates | ✅ Easy | ⚠️ Manual |

## Alternative: Podman (Linux)

For rootless containers:

```bash
# Install podman
sudo apt install podman

# Run ESPHome
podman run --rm -v ~/esphome-config:/config -it esphome/esphome version

# Create alias
alias esphome='podman run --rm -v ~/esphome-config:/config -it esphome/esphome'
```

## Next Steps

After Docker ESPHome setup:

1. [Create first configuration](cli.md#first-run-configuration)
2. [Compile firmware](cli.md#compile-firmware)
3. [Upload to XIAO](cli.md#uploading-to-xiao)
4. [OTA updates](cli.md#ota-upload-subsequent)

## Links

- Docker Hub: https://hub.docker.com/r/esphome/esphome
- Docker Desktop: https://www.docker.com/products/docker-desktop/
- ESPHome Docker Guide: https://esphome.io/guides/docker/
