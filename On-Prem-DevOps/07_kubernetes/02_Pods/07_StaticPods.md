Below is a **clear DevOps training–style explanation of Static Pods in Kubernetes** suitable for **lecture notes or documentation**.

---

# Kubernetes **Static Pods**

## 1. Introduction

A **Static Pod** is a pod that is managed **directly by the kubelet**, not by the Kubernetes API Server.

Normally pods are created using:

```
kubectl commands
YAML manifests
Deployments
```

These pods are scheduled by the **Kubernetes scheduler**.

However, **Static Pods are different**.

They are:

* created directly on a node
* managed by the **kubelet**
* not scheduled by Kubernetes

---

# 2. Key Characteristics of Static Pods

| Feature       | Static Pods          |
| ------------- | -------------------- |
| Managed by    | kubelet              |
| Created using | local manifest files |
| Scheduled by  | NOT scheduler        |
| Location      | specific node        |
| API Server    | only observes them   |

---

# 3. Static Pod Architecture

Normal Pod:

```
User
 ↓
API Server
 ↓
Scheduler
 ↓
Worker Node
```

Static Pod:

```
Manifest File
      ↓
Kubelet
      ↓
Pod Created
```

The **scheduler is bypassed**.

---

# 4. Where Static Pod Manifests Are Stored

Static pods are created from a directory configured in kubelet.

Default location in most clusters:

```
/etc/kubernetes/manifests
```

This directory is monitored by the kubelet.

Whenever a YAML file appears in this directory:

```
kubelet automatically creates the pod
```

---

# 5. Example Static Pod Manifest

Example file:

```
nginx-static.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-nginx
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

---

# 6. Create Static Pod

Copy the manifest file to:

```
/etc/kubernetes/manifests
```

Example command:

```bash
sudo cp nginx-static.yaml /etc/kubernetes/manifests/
```

Immediately kubelet will create the pod.

---

# 7. Verify Static Pod

Run:

```bash
kubectl get pods -o wide
```

Example output:

```
NAME                READY   STATUS   NODE
static-nginx        1/1     Running  worker1
```

You will notice the pod name includes the node name.

Example:

```
static-nginx-worker1
```

---

# 8. Static Pod Behavior

Static pods behave differently from normal pods.

| Action                 | Result               |
| ---------------------- | -------------------- |
| Delete Pod via kubectl | kubelet recreates it |
| Delete manifest file   | pod removed          |
| Scheduler              | NOT used             |

Example:

```bash
kubectl delete pod static-nginx
```

The pod will **reappear automatically** because kubelet still sees the manifest file.

---

# 9. Delete Static Pod

To delete the static pod:

Remove the manifest file.

Example:

```bash
sudo rm /etc/kubernetes/manifests/nginx-static.yaml
```

The pod will stop running.

---

# 10. Why Static Pods Are Important

Static pods are mainly used for **Kubernetes control plane components**.

Example components:

```
kube-apiserver
kube-scheduler
kube-controller-manager
etcd
```

These components run as **static pods on the master node**.

Example:

```
/etc/kubernetes/manifests/kube-apiserver.yaml
```

---

# 11. Check Control Plane Static Pods

Run:

```bash
kubectl get pods -n kube-system
```

Example output:

```
kube-apiserver-kmaster
kube-scheduler-kmaster
kube-controller-manager-kmaster
etcd-kmaster
```

These are **static pods created by kubeadm**.

---

# 12. Static Pods vs Normal Pods

| Feature        | Static Pod        | Normal Pod   |
| -------------- | ----------------- | ------------ |
| Created by     | kubelet           | API Server   |
| Scheduler used | No                | Yes          |
| Managed by     | local file        | Kubernetes   |
| Used for       | system components | applications |

---

# 13. Real World Use Cases

Static Pods are used for:

### Control Plane Components

```
API Server
Scheduler
Controller Manager
etcd
```

### Node-Level Monitoring

Example:

```
node monitoring agent
log collector
```

---

# 14. Static Pod Workflow

```
Create YAML Manifest
       ↓
Copy to /etc/kubernetes/manifests
       ↓
Kubelet detects file
       ↓
Pod created automatically
```

---

# 15. Summary

Static Pods are:

* managed by **kubelet**
* created from **local manifest files**
* not scheduled by Kubernetes
* mainly used for **control plane components**

Important directory:

```
/etc/kubernetes/manifests
```

---

