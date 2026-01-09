# Network Configuration

## Network Architecture

Our Kubernetes cluster will use a dedicated subnet for optimal performance and isolation.

```
Internet Router (10.0.0.1)
         |
    Cudy Switch
         |
    ┌────┼────┐
    │    │    │
k8s-master  worker-1  worker-2
10.0.0.212  .248    .116
```

## Static IP Configuration

### Method 1: Using dhcpcd (Recommended)

Edit `/etc/dhcpcd.conf` on each node:

**Master Node (10.0.0.212):**

```bash
sudo nano /etc/dhcpcd.conf

# Add at the end:
interface eth0
static ip_address=10.0.0.212/24
static routers=10.0.0.1
static domain_name_servers=10.0.0.1 8.8.8.8 1.1.1.1
```

**Worker Node 1 (10.0.0.248):**

```bash
interface eth0
static ip_address=10.0.0.248/24
static routers=10.0.0.1
static domain_name_servers=10.0.0.1 8.8.8.8 1.1.1.1
```

**Worker Node 2 (10.0.0.116):**

```bash
interface eth0
static ip_address=10.0.0.116/24
static routers=10.0.0.1
static domain_name_servers=10.0.0.1 8.8.8.8 1.1.1.1
```

## Hostname Configuration

### Set Hostnames

On each node, set the hostname:

```bash
# Master node
sudo hostnamectl set-hostname k8s-master

# Worker node 1
sudo hostnamectl set-hostname k8s-worker-1

# Worker node 2
sudo hostnamectl set-hostname k8s-worker-2
```

### Update Hosts File

Add entries to `/etc/hosts` on all nodes:

```bash
sudo nano /etc/hosts

# Add these lines:
10.0.0.212   k8s-master
10.0.0.248   k8s-worker-1
10.0.0.116   k8s-worker-2
```

## Network Verification

### Test Connectivity

Run these tests from each node:

```bash
# Test internet connectivity
ping -c 4 google.com

# Test DNS resolution
nslookup kubernetes.io

# Test inter-node connectivity
ping -c 4 k8s-master
ping -c 4 k8s-worker-1
ping -c 4 k8s-worker-2

# Test specific ports (after K8s installation)
nc -zv k8s-master 6443  # API server
nc -zv k8s-master 2379  # etcd
```

### Network Performance Testing

```bash
# Install iperf3 for bandwidth testing
sudo apt install -y iperf3

# On one node (server)
iperf3 -s

# On another node (client)
iperf3 -c k8s-master -t 30
```

Expected results: ~900 Mbps on Gigabit network

## Firewall Configuration

### UFW Rules for Kubernetes

**Master Node:**

```bash
# API Server
sudo ufw allow 6443/tcp

# etcd
sudo ufw allow 2379:2380/tcp

# Kubelet API
sudo ufw allow 10250/tcp

# kube-scheduler
sudo ufw allow 10259/tcp

# kube-controller-manager
sudo ufw allow 10257/tcp
```

**Worker Nodes:**

```bash
# Kubelet API
sudo ufw allow 10250/tcp

# NodePort Services
sudo ufw allow 30000:32767/tcp
```

**All Nodes:**

```bash
# Flannel VXLAN (if using Flannel CNI)
sudo ufw allow 8472/udp

# Allow cluster communication
sudo ufw allow from 10.0.0.0/24
sudo ufw allow from 10.244.0.0/16  # Pod network (Flannel default)
```

### Alternative: Disable UFW (Development Only)

For initial setup and testing:

```bash
sudo ufw disable
```

**Note**: Re-enable firewall for production use.

## Container Network Interface (CNI) Planning

### Pod Network CIDR

We'll use Flannel with default settings:

- **Pod Network**: 10.244.0.0/16
- **Service Network**: 10.96.0.0/12

### Network Policies

Plan for network segmentation:

- Default deny-all policies
- Allow specific inter-service communication
- Isolate namespaces

## SSH Key Distribution

### Generate SSH Keys

On your local machine:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/k8s-homelab -C "k8s-homelab"
```

### Distribute Keys

```bash
# Copy to all nodes
for host in k8s-master k8s-worker-1 k8s-worker-2; do
    ssh-copy-id -i ~/.ssh/k8s-homelab.pub ubuntu@$host
done
```

### SSH Config

Add to `~/.ssh/config`:

```bash
Host k8s-*
    User ubuntu
    IdentityFile ~/.ssh/k8s-homelab
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null

Host k8s-master
    HostName 10.0.0.212

Host k8s-worker-1
    HostName 10.0.0.248

Host k8s-worker-2
    HostName 10.0.0.116
```

## Network Monitoring Setup

### Install Network Tools

```bash
# On all nodes
sudo apt install -y net-tools iftop nethogs nload

# Monitor network usage
sudo iftop -i eth0
sudo nethogs eth0
nload eth0
```

### Network Diagnostics Script

Create `scripts/network-check.sh`:

```bash
#!/bin/bash

echo "=== Network Connectivity Check ==="

nodes=("k8s-master" "k8s-worker-1" "k8s-worker-2")

for node in "${nodes[@]}"; do
    echo "Testing connectivity to $node..."
    if ping -c 1 -W 2 $node > /dev/null 2>&1; then
        echo "✓ $node is reachable"
    else
        echo "✗ $node is NOT reachable"
    fi
done

echo ""
echo "=== DNS Resolution Check ==="
if nslookup kubernetes.io > /dev/null 2>&1; then
    echo "✓ DNS resolution working"
else
    echo "✗ DNS resolution failed"
fi

echo ""
echo "=== Network Interface Status ==="
ip addr show eth0 | grep "inet "
```

## Troubleshooting

### Common Network Issues

**Static IP not working:**

```bash
# Check dhcpcd status
sudo systemctl status dhcpcd

# Restart networking
sudo systemctl restart dhcpcd
sudo systemctl restart networking
```

**DNS resolution issues:**

```bash
# Check resolv.conf
cat /etc/resolv.conf

# Test DNS servers
nslookup google.com 8.8.8.8
```

**Inter-node connectivity:**

```bash
# Check routing table
ip route show

# Check ARP table
arp -a

# Test with traceroute
traceroute k8s-worker-1
```

### Network Performance Issues

```bash
# Check interface statistics
cat /proc/net/dev

# Check for errors
ethtool eth0

# Monitor real-time traffic
sudo tcpdump -i eth0 host k8s-worker-1
```

## Next Steps

With networking configured:

1. Install container runtime (containerd)
2. Install Kubernetes components (kubeadm, kubelet, kubectl)
3. Initialize the cluster
4. Install CNI plugin (Flannel)

## Security Considerations

- Use VPN for remote access to cluster
- Implement network policies for pod-to-pod communication
- Regular security updates for network components
- Monitor network traffic for anomalies
