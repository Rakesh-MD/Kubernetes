# Kubernetes Cluster Setup and Verification

This documentation covers essential commands for setting up, verifying, and testing a Kubernetes cluster.

## 1. Initialize the Kubernetes Control Plane

Run the following command on the **Master Node** to initialize the cluster. Replace the `--pod-network-cidr` value if you're using a different pod network add-on.

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

## 2. Configure kubectl

Set up the `kubeconfig` file to use `kubectl`:

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## 3. Deploy a Pod Network Add-On

Install Flannel as the pod network add-on:

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

## 4. Verify Cluster Status

Check the status of the nodes to ensure they are `Ready`:

```bash
kubectl get nodes
```

List all pods in all namespaces:

```bash
kubectl get pods --all-namespaces
```

## 5. Deploy a Sample Application

Deploy an NGINX application:

```bash
kubectl create deployment nginx --image=nginx
```

Expose the NGINX deployment as a service:

```bash
kubectl expose deployment nginx --port=80 --type=NodePort
```

List the services:

```bash
kubectl get svc
```

## 6. Generate the Node Join Command

To add Worker nodes to the cluster, use the following command on each Worker node:

```bash
kubeadm join 172.31.16.150:6443 --token mlq023.yo11ym9abta6dqh4     --discovery-token-ca-cert-hash sha256:5908ca97579a4bf7c15f8ea55807e7d83282346af84e8935415d6d5c0db522d6
```

## 7. Verify Pods

Check the pods again to see if the application is running:

```bash
kubectl get pods
```

## 8. Test the Application

Use the external IP and port from the `kubectl get svc` output to test the application. Replace `<NODE-IP>` and `<NODE-PORT>` with the appropriate values.

```bash
curl http://<NODE-IP>:<NODE-PORT>
```
---

## Additional Notes

- Replace `<NODE-IP>` and `<NODE-PORT>` with the actual values for testing.
- Ensure you have proper network configuration and firewall rules to allow traffic to the NodePort.

