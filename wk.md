
# Kubernetes Cluster Setup Steps for Worker Node and Kubeadm Join Issue Resolution

## 1. Disable Swap Memory

To ensure that the node does not use swap memory, run the following commands:

```bash
sudo swapoff -a && sudo sed -i '/ swap / s/^\(.*\)$/#/g' /etc/fstab
```

Set the hostname for the node:

```bash
sudo hostnamectl set-hostname worker1
```

Switch to root:

```bash
root@ip-172-31-89-136:~# bash
root@worker1:~#
```

## 2. Install Docker Engine on Both Nodes

### Update and Install Dependencies:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl -y 
sudo install -m 0755 -d /etc/apt/keyrings
```

Download Docker GPG key:

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Add Docker Repository:

```bash
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu   $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update package index:

```bash
sudo apt-get update
```

Install Docker packages:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### Start and Enable Docker Service:

```bash
sudo systemctl start docker && sudo systemctl enable docker
```

### Verify Docker Installation:

```bash
sudo systemctl status docker
docker ps
```

## 3. Configure Cgroup Driver

To ensure the kubelet process works correctly, the cgroup driver must match the one used by Docker. Adjust the Docker configuration:

```bash
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

Restart Docker service:

```bash
sudo systemctl daemon-reload && sudo systemctl restart docker
```

### Configure and Restart containerd:

```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo systemctl restart containerd
sudo systemctl status containerd
```

Verify containerd socket:

```bash
ls -l /var/run/containerd/containerd.sock
```

## 4. Install kubeadm, kubelet, and kubectl on Each Node

### Install Dependencies:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

Download the Kubernetes GPG key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### Add Kubernetes Repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update package index and install Kubernetes components:

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Optionally, enable the kubelet service:

```bash
sudo systemctl enable --now kubelet
```

## 5. Check the Versions of Installed Kubernetes Components

```bash
kubeadm version
kubelet --version
kubectl version --short
```

## 6. Join the Worker Node to the Cluster

Run the following command on the worker node to join it to the Kubernetes cluster:

```bash
kubeadm join 172.31.19.131:6443 --token ddxfi3.y56eub81uio9kvgs         --discovery-token-ca-cert-hash sha256:3e8c7cd34396ac21b7a70a2484fc2343fd4c00d60302487cc4637ebd34f698d6
```

### Issue: `/etc/kubernetes/pki/ca.crt already exists`

If you encounter the error `/etc/kubernetes/pki/ca.crt already exists`, it indicates that the node has already been partially configured for Kubernetes, or it has been part of a previous attempt to join the cluster.

### Resolution Steps:

1. **Remove existing Kubernetes files on the worker node:**

```bash
sudo rm -rf /etc/kubernetes/pki
sudo rm -rf /etc/kubernetes/manifests
sudo rm -rf /etc/kubernetes/kubelet.conf
sudo rm -rf /var/lib/kubelet/*
```

2. **Re-run the `kubeadm join` command:**

```bash
kubeadm join 172.31.19.131:6443 --token ddxfi3.y56eub81uio9kvgs --discovery-token-ca-cert-hash sha256:3e8c7cd34396ac21b7a70a2484fc2343fd4c00d60302487cc4637ebd34f698d6
```

3. **Check the status of the kubelet service on the worker node:**

```bash
sudo systemctl status kubelet
```

Once the worker node has joined successfully, the kubelet status should show that it's running.
