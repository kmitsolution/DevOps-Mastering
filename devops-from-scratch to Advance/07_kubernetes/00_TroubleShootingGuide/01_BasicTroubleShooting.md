---

# 🔧 Kubernetes Troubleshooting Guide (Beginner Level)

---

## 1. Pod is in Pending State

### ❓ Problem

Pod is stuck in **Pending**

### 🔍 Possible Reasons

* No resources (CPU/Memory)
* No node available
* PVC not bound

### ✅ Solution

```bash
kubectl describe pod <pod-name>
```

👉 Look for:

* `Insufficient CPU`
* `node selector mismatch`
* `PVC not found`

---

## 2. Pod is in CrashLoopBackOff

### ❓ Problem

Pod keeps restarting

### 🔍 Reasons

* Application crash
* Wrong command
* Missing dependency

### ✅ Solution

```bash
kubectl logs <pod-name>
```

👉 Check error logs

```bash
kubectl describe pod <pod-name>
```

👉 Look for:

* Exit code
* Events

---

## 3. Pod is Running but Not Accessible

### ❓ Problem

Application is running but cannot access

### 🔍 Reasons

* Service not created
* Wrong port
* Firewall issue

### ✅ Solution

```bash
kubectl get svc
```

👉 Verify:

* Port
* TargetPort

```bash
kubectl describe svc <service-name>
```

---

## 4. Pod IP Not Reachable Across Nodes

### ❓ Problem

Pod works only on same node

### 🔍 Reason

* CNI issue (Flannel/Calico)

### ✅ Solution

```bash
kubectl get pods -n kube-system
```

👉 Check CNI pods

Fix:

* Configure correct interface (`--iface`)
* Check routes

---

## 5. ImagePullBackOff Error

### ❓ Problem

Pod cannot pull image

### 🔍 Reasons

* Wrong image name
* Private registry
* No internet

### ✅ Solution

```bash
kubectl describe pod <pod-name>
```

👉 Look for:

* `ErrImagePull`

Fix:

* Correct image name
* Add imagePullSecrets

---

## 6. Service Not Working

### ❓ Problem

Service created but no response

### 🔍 Reasons

* Selector mismatch
* Pod labels incorrect

### ✅ Solution

```bash
kubectl get pods --show-labels
kubectl get svc
```

👉 Match:

* Service selector = Pod labels

---

## 7. Node Not Ready

### ❓ Problem

Node shows **NotReady**

### 🔍 Reasons

* kubelet down
* network issue
* disk pressure

### ✅ Solution

```bash
kubectl get nodes
```

```bash
kubectl describe node <node-name>
```

👉 Check conditions

---

## 8. DNS Not Working

### ❓ Problem

Service name not resolving

### 🔍 Reasons

* CoreDNS issue

### ✅ Solution

```bash
kubectl get pods -n kube-system
```

👉 Check:

* coredns pods running

Test:

```bash
nslookup kubernetes.default
```

---

## 9. PVC in Pending State

### ❓ Problem

PersistentVolumeClaim not bound

### 🔍 Reasons

* No matching PV
* StorageClass issue

### ✅ Solution

```bash
kubectl get pvc
kubectl describe pvc <pvc-name>
```

---

## 10. Ingress Not Working

### ❓ Problem

Ingress created but not accessible

### 🔍 Reasons

* No Ingress Controller
* Wrong rules

### ✅ Solution

```bash
kubectl get pods -n ingress-nginx
```

👉 Ensure controller is running

---

## 🧠 Golden Debug Commands (Must Know)

```bash
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get svc
kubectl get nodes
kubectl get events
```

---

# 🎯 Final Tip (Very Important)

👉 Always follow this order:

```text
Pod → Service → Network → Node → Logs
```
