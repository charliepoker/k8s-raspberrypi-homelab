# Operating System Installation

## Overview

We'll install Raspberry Pi OS Lite (64-bit) on all three nodes for optimal Kubernetes performance.

## Prerequisites

- Raspberry Pi Imager (download from rpi.org)
- 3x 128GB microSD cards
- SD card reader
- Computer for flashing

## OS Selection

**Recommended**: Ubuntu 22.04 Server (64-bit)

- Minimal installation without desktop
- Better performance for server workloads
- Full 64-bit support for Kubernetes

## Flashing Process

### 1. Download Raspberry Pi Imager

```bash
# macOS
brew install --cask raspberry-pi-imager

# Or download from: https://rpi.org/imager
```

### 2. Flash Each SD Card

For each of the 3 SD cards:

1. **Insert SD card** into reader
2. **Open Raspberry Pi Imager**
3. **Choose OS**: Raspberry Pi OS Lite (64-bit)
4. **Click gear icon** for advanced options
5. **Configure settings**:
   - Enable SSH with password authentication
   - Set username: `ubuntu`
   - Set password: `[your-secure-password]`
   - Configure WiFi (optional, we'll use ethernet)
   - Set locale settings

### 3. Pre-boot Configuration

Before first boot, modify files on the SD card:

#### Enable SSH (if not done in imager)

```bash
# Mount SD card and create empty ssh file
touch /Volumes/boot/ssh
```

#### Set Static Hostnames

Edit `/Volumes/rootfs/etc/hostname` on each card:

- Card 1: `pi-master`
- Card 2: `pi-worker-1`
- Card 3: `pi-worker-2`



## First Boot Setup

### 1. Insert SD Cards and Power On

1. Insert flashed SD cards into respective Pi units
2. Connect ethernet cables to switch
3. Power on all three units
4. Wait 5-10 minutes for initial boot

### 2. Find IP Addresses

```bash
# Scan network for Pi devices
nmap -sn 10.0.0.0/24 | grep -B2 -A2 "Raspberry Pi"

# Or check router admin panel for connected devices
```

### 3. SSH into Each Node

```bash
# Connect to master node
ssh ubuntu@ip-address

# Connect to worker nodes
ssh ubuntu@ip-address
ssh ubuntu@ip-address
```

### 4. Initial System Updates

Run on each node:

```bash
# Update package lists
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y curl wget git vim htop

# Enable container features
echo 'cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1' | sudo tee -a /boot/cmdline.txt

# Reboot to apply changes
sudo reboot
```

## Post-Installation Verification

### System Information

```bash
# Check OS version
cat /etc/os-release

# Check architecture
uname -a

# Check memory
free -h

# Check storage
df -h
```

### Network Connectivity

```bash
# Test internet connectivity
ping -c 4 google.com

# Test inter-node connectivity
ping -c 4 <pi-worker-1 IP-ADDRESS>  # from master to worker-1
```


## Security Hardening

### 1. SSH Key Authentication

Generate SSH keys on your local machine:

```bash
ssh-keygen -t ed25519 -C "k8s-homelab"
```

Copy to each node:

```bash
ssh-copy-id ubuntu@10.0.0.212
ssh-copy-id ubuntu@10.0.0.248
ssh-copy-id ubuntu@10.0.0.116
```

### 2. Disable Password Authentication

Edit `/etc/ssh/sshd_config` on each node:

```bash
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

### 3. Configure Firewall

```bash
# Install and configure UFW
sudo apt install -y ufw

# Allow SSH
sudo ufw allow ssh

# Allow Kubernetes ports (we'll configure these later)
sudo ufw --force enable
```

## Next Steps

With OS installed and configured:

1. Set up network configuration and DNS
2. Install container runtime (containerd)
3. Install Kubernetes components
4. Initialize the cluster

## Troubleshooting

### Boot Issues

- Check SD card integrity
- Verify power supply stability
- Monitor boot logs via HDMI if needed

### Network Issues

- Verify ethernet cable connections
- Check switch port LEDs
- Test with DHCP before static IPs

### SSH Issues

- Verify SSH service is running: `sudo systemctl status ssh`
- Check firewall rules: `sudo ufw status`
- Test local connectivity first
