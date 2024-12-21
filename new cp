root@ip-172-31-16-150:~# history
    1  sudo apt update && sudo apt upgrade -y
    2  sudo apt install apt-transport-https curl -y
    3  sudo apt install containerd -y
    4  sudo mkdir -p /etc/containerd
    5  containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
    6  sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
    7  sudo systemctl restart containerd
    8  curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    9  echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
   10  sudo apt update
   11  sudo apt install -y kubelet kubeadm kubectl
   12  sudo apt-mark hold kubelet kubeadm kubectl
   13  sudo swapoff -a
   14  sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
   15  sudo modprobe overlay
   16  sudo modprobe br_netfilter
   17  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

   18  sudo sysctl --system
   19  sudo kubeadm init --pod-network-cidr=10.244.0.0/16
   20  export KUBECONFIG=/etc/kubernetes/admin.conf
   21  kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
   22  kubectl get nodes
   23  kubectl get pods --all-namespaces
   24  kubectl create deployment nginx --image=nginx
   25  kubectl expose deployment nginx --port=80 --type=NodePort
   26  kubectl get svc
  
   29  kubectl get pods
   30  curl http://54.196.220.129:31677
   31  history
