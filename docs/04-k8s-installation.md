# Kubernetes Homelab on Raspberry Pi 4 (Ubuntu 22.04, ARM64)

This guide documents how to build a **3‑node Kubernetes cluster** using **kubeadm** on **Raspberry Pi 4** running **Ubuntu Server 22.04 LTS (ARM64)**.

✅ Tested and working  
✅ ARM64‑optimized  
✅ kubeadm‑based (CKA‑relevant)  
✅ Suitable for homelab and learning purposes  

---

## Cluster Topology

| Node Name     | Role          | IP Address   |
|--------------|---------------|--------------|
| k8s-master   | Control Plane | 10.0.0.212  |
| k8s-worker-1 | Worker Node   | 10.0.0.248  |
| k8s-worker-2 | Worker Node   | 10.0.0.116  |

---

## Prerequisites

- 3 × Raspberry Pi 4 (4GB or 8GB recommended)
- Ubuntu Server 22.04 LTS (ARM64)
- Static IPs configured
- SSH access to all nodes
- Internet connectivity
- UFW disabled during installation

---

## Components Installed

- containerd (container runtime)
- kubelet (node agent)
- kubeadm (cluster bootstrap)
- kubectl (CLI)
- Flannel (CNI)

---

# Phase 1: Operating System Preparation (ALL Nodes)

### 1. Update System

```bash
sudo apt update && sudo apt upgrade -y
```


### 2. Disable Swap (Required)

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^.*$/#&/g' /etc/fstab
free -h
```

### 3. Load Required Kernel Modules

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```
#### Persist modules:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

### 4. Configure Kernel Networking

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

### 5. Enable cgroups (Raspberry Pi Required)
#### Edit kernel command line:

```bash
sudo nano /boot/firmware/cmdline.txt
```

#### Append to the same line:

```bash
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
```

#### Reboot:

```bash
sudo reboot
```

#### Verify after reboot:

```bash
ls /sys/fs/cgroup | grep memory
```




# Phase 2: Install Container Runtime (ALL Nodes)
### 6. Install containerd

```bash
sudo apt update
sudo apt install -y containerd
```

### 7. Configure containerd for Kubernetes
#### Generate default config:

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
```

#### Edit config:

```bash
sudo nano /etc/containerd/config.toml
```

#### Ensure the following values:


```toml
SystemdCgroup = true
sandbox_image = "registry.k8s.io/pause:3.9"
```

#### Restart and enable containerd:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
systemctl status containerd
```

# Phase 3: Install Kubernetes Components (ALL Nodes)
### 8. Add Kubernetes APT Repository (v1.29)

```bash
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### 9. Install kubelet, kubeadm, kubectl

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
```

### Hold versions:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

#### Enable kubelet:

```bash
sudo systemctl enable kubelet
```

# Phase 4: Initialize Control Plane (MASTER Node Only)
#### 10. Initialize Kubernetes Cluster

```bash
sudo kubeadm init \
  --apiserver-advertise-address=10.0.0.212 \
  --pod-network-cidr=10.244.0.0/16 \
  --node-name=k8s-master \
  --cri-socket=unix:///run/containerd/containerd.sock
```
### ✅ Save the kubeadm join command output.

### 11. Configure kubectl Access (Master Node)
```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Test:

```bash
kubectl get nodes
```

# Phase 5: Install Pod Networking (CNI)
### 12. Install Flannel

```bash
kubectl apply -f \
https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

#### Wait for readiness:

```bash
kubectl wait --for=condition=ready pod \
-l app=flannel -n kube-flannel --timeout=300s
```

# Phase 6: Join Worker Nodes
### Run on each worker node:

``` bash
sudo kubeadm join 10.0.0.212:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --cri-socket=unix:///run/containerd/containerd.sock
```

# Phase 7: Label Worker Nodes (Master Node)

```bash
kubectl label node k8s-worker-1 node-role.kubernetes.io/worker=worker
kubectl label node k8s-worker-2 node-role.kubernetes.io/worker=worker
```
# Phase 8: Verify Cluster

``` bash
kubectl get nodes

kubectl get pods -A

kubectl cluster-info
```

#### Expected result:

```text
k8s-master     Ready   control-plane
k8s-worker-1   Ready   worker
k8s-worker-2   Ready   worker
```

# Phase 9: (Optional) Re‑Enable UFW
### Control Plane Node

```bash
sudo ufw allow 6443/tcp
sudo ufw allow 2379:2380/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10259/tcp
sudo ufw allow 10257/tcp
sudo ufw allow 8472/udp
sudo ufw allow from 10.0.0.0/24
sudo ufw allow from 10.244.0.0/16
```

### Worker Nodes

```bash
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
```

#### Enable UFW:

```bash
sudo ufw enable
sudo ufw status
```

## Reset Cluster (If Needed)

```bash
sudo kubeadm reset -f
sudo rm -rf /etc/kubernetes /var/lib/etcd /var/lib/kubelet ~/.kube
```

## Notes & Lessons Learned
- Raspberry Pi requires cgroups to be enabled
- containerd must use SystemdCgroup = true
- Pause image mismatch can break kubeadm on ARM
- Full reset is often faster than debugging a broken init
  

---

## License
### MIT