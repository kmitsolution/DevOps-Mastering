

# Kubernetes Flannel Network Error – Pod Stuck in `ContainerCreating`

## Problem

When deploying pods in Kubernetes, pods remain stuck in **`ContainerCreating`** state.

Running:

```bash
kubectl get pods
```

Output:

```
NAME   READY   STATUS              RESTARTS   AGE
pod1   0/1     ContainerCreating   0          52s
```

Checking events using:

```bash
kubectl describe pod <pod-name>
```

Error message appears:

```
Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox
plugin type="flannel" failed (add):
loadFlannelSubnetEnv failed:
open /run/flannel/subnet.env: no such file or directory
```

This indicates **Flannel CNI network is not initialized properly**.

---

# Root Causes

This issue commonly occurs due to:

1. **Swap enabled on the node**
2. **IP forwarding disabled**
3. **br_netfilter kernel module not loaded**
4. **Required Kubernetes sysctl parameters missing**

These settings prevent the Flannel network plugin from creating network interfaces.

---

# Solution

## 1. Disable Swap

Check swap:

```bash
free -h
```

Disable swap temporarily:

```bash
sudo swapoff -a
```

Make it permanent by editing `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Comment the swap entry:

```
#/swap.img none swap sw 0 0
```

Save and exit.

Verify:

```bash
free -h
```

Swap should show **0B**.

---

# 2. Enable Required Kubernetes Networking Parameters

Create the Kubernetes sysctl configuration file:

```bash
sudo nano /etc/sysctl.d/k8s.conf
```

Add:

```
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
```

Apply the configuration:

```bash
sudo sysctl --system
```

---

# 3. Load Required Kernel Module

Create module configuration:

```bash
sudo nano /etc/modules-load.d/k8s.conf
```

Add:

```
br_netfilter
```

Save and exit.

Load module immediately:

```bash
sudo modprobe br_netfilter
```

Verify:

```bash
lsmod | grep br_netfilter
```

---

# 4. Restart Kubernetes Components

Restart kubelet:

```bash
sudo systemctl restart kubelet
```

Or reboot the node:

```bash
sudo reboot
```

---

# 5. Verify Flannel Pods

Check flannel pods:

```bash
kubectl get pods -n kube-flannel
```

Expected output:

```
NAME                    READY   STATUS    RESTARTS   AGE
kube-flannel-ds-xxxxx   1/1     Running   0          1m
```

---

# 6. Verify Application Pods

Check pods again:

```bash
kubectl get pods
```

Expected result:

```
NAME   READY   STATUS    RESTARTS   AGE
pod1   1/1     Running   0          2m
```

---

# Key Files Used

| File                           | Purpose                           |
| ------------------------------ | --------------------------------- |
| `/etc/fstab`                   | Disable swap permanently          |
| `/etc/sysctl.d/k8s.conf`       | Kubernetes networking parameters  |
| `/etc/modules-load.d/k8s.conf` | Load kernel modules automatically |

---

# Summary

To fix **Flannel CNI network errors in Kubernetes**:

1. Disable swap
2. Enable IP forwarding
3. Enable bridge network iptables
4. Load `br_netfilter` module
5. Restart kubelet

These steps ensure the **Flannel network plugin initializes correctly** and pods can start.

---


