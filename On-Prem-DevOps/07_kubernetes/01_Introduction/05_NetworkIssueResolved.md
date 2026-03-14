# Kubernetes `kubectl exec` Failure Due to Duplicate Node Internal IPs (VirtualBox Setup)
<img src=https://github.com/kmitsolution/DevOps-Mastering/blob/main/On-Prem-DevOps/07_kubernetes/ss1.jpg height=900 width=900 />

## 1. Problem Statement

In a Kubernetes cluster created using **three VirtualBox Ubuntu 24.04 machines (1 control plane + 2 workers)**, the following error occurred when attempting to access a running pod:

```
kubectl exec -it new-pod -- bash
error: Internal error occurred: unable to upgrade connection: pod does not exist
```

Even though the pod was **running successfully**, `kubectl exec` failed.

Example:

```
kubectl get pods

NAME      READY   STATUS    RESTARTS   AGE
new-pod   1/1     Running   1          5h33m
```

However, executing:

```
kubectl exec -it new-pod -- bash
```

resulted in the error shown above.

---

# 2. Environment Setup

### Virtualization

* Oracle VirtualBox

### Operating System

* Ubuntu 24.04.4 LTS

### Kubernetes Version

* v1.35.2

### Container Runtime

* containerd 2.2.2

### Cluster Nodes

| Node    | Role          |
| ------- | ------------- |
| kmaster | Control Plane |
| worker1 | Worker        |
| worker2 | Worker        |

---

# 3. VirtualBox Network Configuration

Each VM had **two adapters configured**.

| Adapter   | Type        | Purpose                        |
| --------- | ----------- | ------------------------------ |
| Adapter 1 | NAT         | Internet access                |
| Adapter 2 | NAT Network | Internal cluster communication |

NAT Network range:

```
192.168.0.0/24
```

Example assigned addresses:

| Node    | NAT Network IP |
| ------- | -------------- |
| kmaster | 192.168.0.5    |
| worker1 | 192.168.0.6    |
| worker2 | 192.168.0.7    |

---

# 4. Kubernetes Initialization

The control plane was initialized using:

```
sudo kubeadm init \
--apiserver-advertise-address=192.168.0.5 \
--pod-network-cidr=10.244.0.0/16
```

The cluster was then joined by the worker nodes using the `kubeadm join` command.

---

# 5. Observed Issue

Running:

```
kubectl get nodes -o wide
```

returned:

```
NAME      STATUS   ROLES           VERSION   INTERNAL-IP
kmaster   Ready    control-plane   v1.35.2   10.0.2.15
worker1   Ready    <none>          v1.35.2   10.0.2.15
worker2   Ready    <none>          v1.35.2   10.0.2.15
```

### Problem

All nodes reported the **same Internal IP address**:

```
10.0.2.15
```

This IP comes from the **VirtualBox NAT adapter**, which assigns the same address to each VM internally.

---

# 6. Why This Causes `kubectl exec` Failure

When a user runs:

```
kubectl exec
```

the communication path is:

```
kubectl → API Server → Node kubelet → Pod container
```

The API server connects to the node's **kubelet using the node InternalIP**.

Since **all nodes had the same InternalIP**, the API server could not reach the correct node hosting the pod.

Example scenario:

```
Pod running on worker1
worker1 IP (reported): 10.0.2.15
worker2 IP (reported): 10.0.2.15
master IP (reported): 10.0.2.15
```

The API server attempts to contact:

```
10.0.2.15
```

but it may reach the wrong node, causing:

```
unable to upgrade connection: pod does not exist
```

This affects:

* `kubectl exec`
* `kubectl logs`
* `kubectl attach`
* `kubectl port-forward`

---

# 7. Root Cause

Kubernetes **automatically selected the first network interface** (NAT adapter) instead of the **cluster network adapter**.

Therefore, nodes registered themselves using:

```
10.0.2.15
```

instead of the correct **192.168.0.x** addresses.

---

# 8. Solution

Force Kubernetes **kubelet to use the correct node IP**.

---

# Step 1: Identify Correct Node IP

On each node run:

```
ip a
```

Find the **192.168.0.x** address from the NAT Network adapter.

Example:

| Node    | Correct IP  |
| ------- | ----------- |
| kmaster | 192.168.0.5 |
| worker1 | 192.168.0.6 |
| worker2 | 192.168.0.7 |

---

# Step 2: Configure Kubelet Node IP

Edit the kubelet configuration file.

```
sudo nano /etc/default/kubelet
```

Add the following:

### Master

```
KUBELET_EXTRA_ARGS=--node-ip=192.168.0.5
```

### Worker1

```
KUBELET_EXTRA_ARGS=--node-ip=192.168.0.6
```

### Worker2

```
KUBELET_EXTRA_ARGS=--node-ip=192.168.0.7
```

---

# Step 3: Restart Kubelet

Run on each node:

```
sudo systemctl daemon-reexec
sudo systemctl restart kubelet
```

---

# Step 4: Verify Node IPs

Run:

```
kubectl get nodes -o wide
```

Expected result:

```
NAME      STATUS   ROLES           VERSION   INTERNAL-IP
kmaster   Ready    control-plane   v1.35.2   192.168.0.5
worker1   Ready    <none>          v1.35.2   192.168.0.6
worker2   Ready    <none>          v1.35.2   192.168.0.7
```

Each node now has a **unique InternalIP**.

---

# Step 5: Test `kubectl exec`

Run:

```
kubectl exec -it new-pod -- sh
```

Note: Many containers (like nginx or busybox) **do not include bash**, so `sh` is safer.

Example result:

```
/ #
```

Pod access should now work correctly.

---

# 9. Final Result

After correcting the node IP configuration:

* Nodes register with correct IP addresses
* API server communicates correctly with kubelets
* `kubectl exec` works successfully
* Cluster networking behaves correctly

---

# 10. Key Lessons Learned

⚠️ When running Kubernetes in **VirtualBox**, avoid using the NAT adapter IP as the node IP.

Best practices:

* Always configure kubelet with `--node-ip`
* Use **Host-Only Adapter or NAT Network for cluster communication**
* Ensure each node has a **unique InternalIP**

---

# 11. Recommended VirtualBox Network Layout

| Adapter   | Type                    | Purpose                          |
| --------- | ----------------------- | -------------------------------- |
| Adapter 1 | NAT                     | Internet access                  |
| Adapter 2 | Host-only / NAT Network | Kubernetes cluster communication |

Example network:

```
192.168.56.0/24
```

---

# 12. Conclusion

The error:

```
unable to upgrade connection: pod does not exist
```

can occur even when pods are running if **node InternalIP addresses are incorrect or duplicated**.

Correctly configuring the kubelet node IP resolves the issue and ensures stable Kubernetes cluster communication.

---

✅ Issue successfully resolved.


