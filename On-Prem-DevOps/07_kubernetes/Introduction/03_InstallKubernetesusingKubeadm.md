Below is a **clean documentation-style guide** based on your exact lab setup using **VirtualBox + Ubuntu 24.04 + kubeadm + Kubernetes v1.35.0**. This format is suitable for **training notes, lab manuals, or DevOps course documentation**.

---

# Kubernetes Cluster Installation using kubeadm (VirtualBox Lab)

## 1. Introduction

This document explains how to install a **Kubernetes v1.35.0 cluster using kubeadm** on **Ubuntu 24.04 virtual machines** created in VirtualBox.

The cluster consists of:

* **1 Control Plane (Master Node)**
* **2 Worker Nodes**

The Kubernetes cluster is created using **kubeadm** and **containerd runtime**, and **Flannel** is used as the networking plugin.

---

# 2. Lab Environment

## Virtual Machines

Three Ubuntu 24.04 VMs were created in **VirtualBox**.

| Node Name | Role        | IP Address  |
| --------- | ----------- | ----------- |
| kmaster   | Master Node | 192.168.0.5 |
| worker1   | Worker Node | 192.168.0.6 |
| worker2   | Worker Node | 192.168.0.7 |

---

## Network Configuration

All VMs are connected using **Adapter 2 – NAT Network**.

Network Details:

```
Network Name: inputnetwork
CIDR: 192.168.0.0/24
```

Each VM has a **static IP address** configured.

This allows communication between the master and worker nodes.

---

# 3. Kubernetes Version

```
Kubernetes Version: 1.35.0
Runtime: containerd
Bootstrap Tool: kubeadm
Network Plugin: Flannel
OS: Ubuntu 24.04
```

---

# 4. Execute on ALL Nodes (Master & Worker)

The following steps must be executed on:

```
kmaster
worker1
worker2
```

---

# Step 1 — Update System Packages

Update the system and install required utilities.

```bash
sudo apt-get update
sudo apt install apt-transport-https curl -y
```

---

# Step 2 — Install Container Runtime (containerd)

Add Docker repository key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Add Docker repository:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update package list:

```bash
sudo apt-get update
```

Install containerd:

```bash
sudo apt-get install containerd.io -y
```

---

# Step 3 — Configure containerd

Create containerd configuration:

```bash
sudo mkdir -p /etc/containerd
```

Generate default config:

```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Enable **systemd cgroup driver**:

```bash
sudo sed -i -e 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```

Restart container runtime:

```bash
sudo systemctl restart containerd
```

---

# Step 4 — Install Kubernetes Packages

Add Kubernetes repository key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Set permissions:

```bash
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add Kubernetes repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Set permissions:

```bash
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
```

Update packages:

```bash
sudo apt-get update
```

Install Kubernetes components:

```bash
sudo apt-get install -y kubelet kubeadm kubectl
```

Prevent automatic upgrades:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Enable kubelet:

```bash
sudo systemctl enable --now kubelet
```

---

# Step 5 — Disable Swap

Kubernetes requires swap to be disabled.

```bash
sudo swapoff -a
```

Load networking module:

```bash
sudo modprobe br_netfilter
```

Enable IP forwarding:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

---

# 5. Initialize Kubernetes Cluster (Master Node Only)

Run the following command **only on kmaster**.

```bash
sudo kubeadm init --apiserver-advertise-address=192.168.0.5 --pod-network-cidr=10.244.0.0/16
```

This command performs:

* Initializes Kubernetes Control Plane
* Generates cluster certificates
* Starts cluster components
* Generates worker node join token

---

# Step 6 — Configure kubectl (Master Node)

After initialization, configure kubectl.

```bash
mkdir -p $HOME/.kube
```

```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

# Step 7 — Install Network Plugin

Kubernetes requires a CNI plugin for pod networking.

Install **Flannel Network Plugin**.

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Flannel creates the pod network using:

```
10.244.0.0/16
```

---

# 6. Join Worker Nodes

During `kubeadm init`, Kubernetes generates a **join command**.

Example:

```bash
kubeadm join 192.168.0.5:6443 --token <token> \
--discovery-token-ca-cert-hash sha256:<hash>
```

Run this command on:

```
worker1
worker2
```

This will connect the workers to the Kubernetes cluster.

---

# 7. Verify Cluster

Run the following command on the **master node**.

```bash
kubectl get nodes
```

Expected output:

```
NAME      STATUS   ROLES           AGE
kmaster   Ready    control-plane
worker1   Ready    <none>
worker2   Ready    <none>
```

All nodes should be in **Ready** state.

---

# 8. Cluster Architecture

The Kubernetes cluster now consists of:

```
Control Plane (kmaster)
       |
       |
 -------------------------
 |                       |
worker1               worker2
```

Master Node manages:

* API Server
* Scheduler
* Controller Manager
* etcd

Worker Nodes run:

* kubelet
* kube-proxy
* Pods

---

# 9. Test the Cluster

Deploy a test application.

```bash
kubectl create deployment nginx --image=nginx
```

Check pods:

```bash
kubectl get pods
```

Check nodes:

```bash
kubectl get nodes
```

---

# 10. Summary

In this lab we successfully:

* Created **3 Ubuntu 24.04 VMs in VirtualBox**
* Configured **NAT Network**
* Installed **containerd runtime**
* Installed **Kubernetes v1.35.0**
* Initialized the **Kubernetes control plane**
* Joined **worker nodes**
* Installed **Flannel networking**
* Verified the cluster using `kubectl get nodes`

The Kubernetes cluster is now ready to deploy containerized applications.

