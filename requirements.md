# WinBoat Installation and Troubleshooting Guide

## System Information
- **OS**: Ubuntu 24.04 (Noble)
- **Architecture**: amd64
- **User**: mmkhathinidev

## Installation Summary

### 1. WinBoat
- **Version**: 0.9.0
- **Installation Method**: .deb package from GitHub releases
- **Download URL**: https://github.com/TibixDev/winboat/releases/download/v0.9.0/winboat-0.9.0-amd64.deb
- **Installation Location**: `/opt/winboat`
- **Binary**: `/usr/bin/winboat`
- **Config Directory**: `~/.config/winboat/`
- **Data Directory**: `~/.local/share/winboat/`
- **Cache Directory**: `~/.cache/winboat/`

### 2. Container Runtimes

#### Docker
- **Version**: 29.1.1, build 0aedba5
- **Status**: Installed and running
- **Service**: Active
- **Socket**: `/var/run/docker.sock`
- **User Group**: User added to `docker` group

#### Podman
- **Version**: 4.9.3
- **Status**: Installed and working
- **Podman Compose**: 1.0.6
- **Socket**: Enabled for rootless operation
- **Additional Tools**:
  - buildah (container image builder)
  - netavark (network stack)
  - aardvark-dns (DNS for containers)
  - passt (network connectivity)

### 3. Windows ISO Files
- **Location**: `~/Downloads/`
- **Files Available**:
  - Windows 11 ISO files (multiple versions)

## Common Issues and Solutions

### Issue 1: Docker Socket Permission Error
**Error**: `Error: connect ENOENT /var/run/docker.sock`

**Cause**: User not in docker group or session not refreshed after group addition

**Solutions**:
1. Add user to docker group:
   ```bash
   sudo usermod -aG docker $USER
   ```

2. Launch winboat with docker group permissions:
   ```bash
   sg docker -c "winboat"
   ```

3. **Permanent fix**: Log out and log back in to refresh group memberships

### Issue 2: WinBoat Container Port Mapping
**Problem**: WinBoat GUI shows wrong port for web interface

**Details**:
- Container internal port: 8006
- Actual mapped port: 47279 (or similar in range 47270-47370)
- RDP port mapping: 47309 (maps to 3389)

**Solution**: Check actual port mappings:
```bash
docker port WinBoat
```

### Issue 3: Guest API Offline During Installation
**Status**: Normal behavior

**Explanation**: The "Guest API - Offline" message is expected during Windows installation. The API comes online after:
1. Windows installation completes
2. Windows boots successfully
3. Guest tools are loaded

### Issue 4: Incomplete Windows Download/Installation
**Solution**: Remove container and start fresh:
```bash
# Stop and remove container
docker stop WinBoat
docker kill WinBoat  # If stop hangs
docker rm WinBoat

# Check for volumes
docker volume ls | grep -i winboat
docker volume rm <volume_name>  # If any exist
```

## Complete Removal Process

### Remove WinBoat
```bash
# Stop all processes
pkill -9 winboat

# Remove package
sudo dpkg --remove winboat

# Remove system files
sudo rm -rf /opt/winboat
sudo rm -f /usr/bin/winboat
sudo rm -f /usr/share/applications/winboat.desktop
sudo rm -rf /usr/share/icons/hicolor/*/apps/winboat.png

# Remove user files
rm -rf ~/.config/winboat
rm -rf ~/.local/share/winboat
rm -rf ~/.cache/winboat
rm -rf ~/.local/share/applications/*winboat*
```

### Remove Docker Containers
```bash
# List all containers
docker ps -a | grep -i winboat

# Remove specific container
docker rm -f WinBoat

# Remove all stopped containers
docker container prune
```

## Installation Commands

### Fresh WinBoat Installation
```bash
cd ~/Downloads

# Download latest version
wget https://github.com/TibixDev/winboat/releases/download/v0.9.0/winboat-0.9.0-amd64.deb

# Install
sudo dpkg -i winboat-0.9.0-amd64.deb

# Launch with docker permissions
sg docker -c "winboat"
```

### Install Podman (Alternative to Docker)
```bash
# Update package lists
sudo apt update

# Install Podman and Podman Compose
sudo apt install -y podman podman-compose

# Enable Podman socket
systemctl --user enable --now podman.socket

# Verify installation
podman --version
podman-compose --version
podman ps
```

## Verification Commands

### Check Docker Status
```bash
# Check if Docker is running
systemctl is-active docker

# Check Docker version
docker --version

# Check user groups
groups | grep docker

# Test Docker access
docker ps

# Check socket permissions
ls -la /var/run/docker.sock
```

### Check WinBoat Status
```bash
# Check if winboat is installed
which winboat

# Check running processes
ps aux | grep winboat | grep -v grep

# Check container status
docker ps -a | grep -i winboat

# Check container logs
docker logs WinBoat --tail 50

# Check port mappings
docker port WinBoat
```

### Check Podman Status
```bash
# Check Podman version
podman --version

# Check running containers
podman ps

# Check Podman socket
systemctl --user status podman.socket
```

## WinBoat Configuration

### Recommended VM Settings
- **RAM**: 4-6 GB (not 8GB - too close to system limit)
- **CPU Cores**: 2-4 cores
- **Disk Size**: 64 GB or more
- **Windows Version**: Windows 11 Pro

### Port Ranges
- **WinBoat Port Range**: 47270-47370
- **Web Interface**: Port 8006 (mapped to 472xx)
- **RDP**: Port 3389 (mapped to 473xx)
- **Guest API**: Port 7148, 7149

### Default Paths
- **Install Path**: `~/winboat` (default in v0.9.0)
- **ISO Storage**: User-specified during VM creation
- **VM Data**: Stored in Docker volumes

## New Features in WinBoat 0.9.0
- Podman support (in addition to Docker)
- UWP app support
- Custom app creation from existing apps
- App filtering
- Application scaling adjustment
- Custom FreeRDP arguments
- Better port management (47270-47370 range)
- Default install path set to `~/winboat`
- Animation disable option

## Troubleshooting Checklist

1. **Docker/Podman Running?**
   ```bash
   systemctl is-active docker
   # or
   systemctl --user is-active podman.socket
   ```

2. **User in Docker Group?**
   ```bash
   groups | grep docker
   ```

3. **WinBoat Process Running?**
   ```bash
   ps aux | grep winboat | grep -v grep
   ```

4. **Container Running?**
   ```bash
   docker ps | grep -i winboat
   ```

5. **Check Container Logs**
   ```bash
   docker logs WinBoat --tail 50
   ```

6. **Check Port Mappings**
   ```bash
   docker port WinBoat
   ```

7. **Test Web Interface**
   ```bash
   curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:47279/
   ```

## Known Issues

### PPA Error During apt update
**Error**: `The repository 'https://ppa.launchpadcontent.net/gezakovacs/ppa/ubuntu noble Release' does not have a Release file.`

**Impact**: Does not affect Podman/WinBoat installation

**Solution**: Can be ignored or remove the PPA:
```bash
sudo add-apt-repository --remove ppa:gezakovacs/ppa
```

### CNI Plugin Warning During Podman Installation
**Warning**: `Error validating CNI config file /etc/cni/net.d/87-podman-bridge.conflist`

**Impact**: Does not affect Podman functionality (uses netavark instead)

**Solution**: No action needed - Podman uses netavark for networking

## Additional Resources
- **WinBoat GitHub**: https://github.com/TibixDev/winboat
- **WinBoat Releases**: https://github.com/TibixDev/winboat/releases
- **Docker Documentation**: https://docs.docker.com/
- **Podman Documentation**: https://docs.podman.io/

## Session Notes
- Multiple installation/removal cycles performed
- Docker socket permission issues resolved
- Container port mapping confusion addressed
- Both Docker and Podman successfully installed
- WinBoat 0.9.0 successfully installed and launched
- Windows 11 installation attempted but incomplete (container removed for fresh start)
