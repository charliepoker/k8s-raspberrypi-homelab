# Hardware Assembly and Setup

## Components Overview

### Raspberry Pi 4 Specifications

- **Model**: Raspberry Pi 4 Model B 8GB RAM
- **Storage**: 128GB microSD card (included in GeeekPi kit)
- **Power**: 18W 5V 3.6A power supply with ON/OFF switch
- **Cooling**: PWM fan in case for temperature management

### Network Switch

- **Model**: Cudy GS108 8-Port Gigabit Ethernet Switch
- **Type**: Unmanaged, plug-and-play
- **Ports**: 8x Gigabit Ethernet ports
- **Design**: Fanless, desktop metal case

### Cables

- **Type**: Cat6 Ethernet cables
- **Length**: 10ft (snagless connectors)
- **Quantity**: 4 cables (3 for Pi nodes + 1 for uplink)

## Assembly Steps

### 1. Prepare Raspberry Pi Units

For each of the 3 Raspberry Pi devices:

1. **Install heatsinks** (if included in kit)
2. **Mount Pi in case** with proper orientation
3. **Connect PWM fan** to GPIO pins 
4. **Insert microSD card**
5. **Connect power supply**

### 2. Network Switch Setup

1. **Place switch** in central location near your router
2. **Connect power adapter** to switch
3. **Verify all 8 ports** are accessible

### 3. Cable Management

- 3 cables from Pi nodes to switch
- 1 cable from switch to your main router/network
- Keep 1 cable as spare

## Node Naming Convention

For organization, label your nodes:

- **pi-master** 
- **pi-worker-1**
- **pi-worker-2**

## Power Considerations

- Each Pi draws ~15W under load
- Total power consumption: ~45W for all 3 nodes
- Switch power consumption: ~8W


## Temperature Management

- PWM fans should keep temps under 70°C


## Network Topology

```
Internet
    |
[Router] ---- [Cudy GS108 Switch]
                    |    |    |
              [Pi-1] [Pi-2] [Pi-3]
```

## Next Steps

Once hardware is assembled:

1. Flash Raspberry Pi OS to SD cards
2. Configure SSH and network settings
3. Set up static IP addresses
4. Install Kubernetes components

## Troubleshooting

### Common Issues

- **No boot**: Check SD card insertion and power supply
- **Overheating**: Verify fan connection and case ventilation
- **Network issues**: Test cables and switch port LEDs
- **Power instability**: Ensure adequate power supply rating

### Useful Commands (for later)

```bash
# Check temperature
vcgencmd measure_temp

# Check power supply
vcgencmd get_throttled

# Network interface status
ip addr show
```
