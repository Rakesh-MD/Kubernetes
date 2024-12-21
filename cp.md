
# Kubernetes Cluster Setup Steps

## 1. Disable Swap Memory

To ensure that the node does not use swap memory, run the following commands:

```bash
sudo swapoff -a && sudo sed -i '/ swap / s/^\(.*\)$/#/g' /etc/fstab
```

Set the hostname for the node:

```bash
sudo hostnamectl set-hostname controlplane
```

Switch to root:

```bash
root@ip-172-31-19-131:~# bash
root@controlplane:~#
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
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### Add Kubernetes Repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
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
kubectl version
```

## 6. Initialize the Control-Plane Node

Run the following command on the controlplane node to initialize the Kubernetes cluster:

```bash
kubeadm config images pull --kubernetes-version v1.32.0

kubeadm init
```

Set up Kubernetes config for the user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Set the KUBECONFIG environment variable:

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Get the kubeadm join command from the output and execute it on the worker nodes. Example:

```bash
kubeadm join 172.31.19.131:6443 --token ddxfi3.y56eub81uio9kvgs         --discovery-token-ca-cert-hash sha256:3e8c7cd34396ac21b7a70a2484fc2343fd4c00d60302487cc4637ebd34f698d6
```

## 7. Verify Node Status

After the join process is complete, run the following command to verify the nodes:

```bash
kubectl get nodes
```

You should see the following output:

```bash
NAME           STATUS     ROLES           AGE    VERSION
controlplane   NotReady   control-plane   12m    v1.32.0
worker1        NotReady   <none>          6m3s   v1.32.0
```

Now, your Kubernetes cluster should be initialized with both control-plane and worker nodes.

## 8. Install a Network Plugin

Install Weave Net:

```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

Wait for 30 seconds or so, then run:

```bash
kubectl get nodes
```

Nodes should now be ready:

```bash
NAME              STATUS     ROLES           AGE     VERSION
ip-172-31-11-74   Ready      control-plane    2m57s    v1.24.0
ip-172-31-9-67    Ready      <none>           1m52s    v1.24.0
```
