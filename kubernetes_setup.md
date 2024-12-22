# Kubernetes Setup with Containerd

This script automates the setup of Kubernetes with `containerd` as the container runtime.

## Script Content

```bash
#!/bin/bash

# Update and upgrade the system
echo "Updating and upgrading the system..."
sudo apt update && sudo apt upgrade -y

# Install necessary packages
echo "Installing apt-transport-https and curl..."
sudo apt install apt-transport-https curl -y

# Install containerd
echo "Installing containerd..."
sudo apt install containerd -y

# Configure containerd
echo "Configuring containerd..."
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd

# Add Kubernetes repository and key
echo "Adding Kubernetes repository..."
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update package list and install Kubernetes components
echo "Installing kubelet, kubeadm, and kubectl..."
sudo apt update
sudo apt install -y kubelet kubeadm kubectl

# Mark Kubernetes packages to hold to avoid updates
echo "Marking Kubernetes packages to hold..."
sudo apt-mark hold kubelet kubeadm kubectl

# Disable swap
echo "Disabling swap..."
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#/g' /etc/fstab

# Enable necessary kernel modules
echo "Enabling necessary kernel modules..."
sudo modprobe overlay
sudo modprobe br_netfilter

# Configure sysctl settings for Kubernetes
echo "Configuring sysctl settings for Kubernetes..."
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl settings
echo "Applying sysctl settings..."
sudo sysctl --system

echo "Kubernetes setup completed successfully!"
```

## Usage Instructions

1. Save the script into a file:
   ```bash
   vi setup_k8s.sh
   ```
   Paste the above script, save, and quit.

2. Make the script executable:
   ```bash
   chmod +x setup_k8s.sh
   ```

3. Run the script:
   ```bash
   ./setup_k8s.sh
   ```

This will automate the setup process and configure Kubernetes on your system.
