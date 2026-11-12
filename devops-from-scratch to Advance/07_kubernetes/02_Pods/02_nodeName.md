Below is a **structured documentation-style explanation** you can use in your **DevOps training or lecture notes**.

---

# Assigning Pods to Worker Nodes using `nodeName` in Kubernetes

## 1. Introduction

In Kubernetes, pods are normally scheduled automatically by the **Kubernetes Scheduler**. The scheduler decides which worker node should run the pod based on:

* Resource availability (CPU, Memory)
* Node conditions
* Scheduling policies
* Labels and taints

However, sometimes we may want to **manually assign a pod to a specific node**. This can be done using the **`nodeName` field** in the pod specification.

---

# 2. What is `nodeName` in Kubernetes?

`nodeName` is a **pod specification field** that directly assigns a pod to a specific node.

When `nodeName` is specified:

* The **scheduler is bypassed**
* Kubernetes directly places the pod on the specified node

Example concept:

```
Pod
   |
   └── nodeName = worker2
```

This means the pod will run **only on the node named `worker2`**.

---

# 3. When is `nodeName` Useful?

Using `nodeName` is helpful in specific scenarios:

### 1. Testing and Debugging

Developers may want to run a pod on a **specific node for testing purposes**.

### 2. Troubleshooting

Administrators can deploy a pod to a node to debug:

* network issues
* storage problems
* node configuration

### 3. Node Maintenance

During maintenance tasks, you might temporarily run workloads on a specific node.

### 4. Lab and Learning Environments

In training environments (like DevOps labs), it helps demonstrate **pod scheduling behavior**.

⚠️ **Important:**
In production environments, **NodeSelector or NodeAffinity** is preferred over `nodeName`.

---

# 4. Can We Assign Pods to the Master Node?

Yes, technically we **can assign pods to the master node**, but normally Kubernetes prevents this.

The master node has a **taint** applied:

```
node-role.kubernetes.io/control-plane:NoSchedule
```

This prevents normal workloads from running on the master node.

To run pods on the master node you must:

1. Remove the taint

Example:

```bash
kubectl taint nodes kmaster node-role.kubernetes.io/control-plane-
```

However, this is **not recommended in production**, because the master node should only run **control plane components**.

---

# 5. Checking Node Names

Before assigning a pod, check the node names.

```bash
kubectl get nodes
```

Example output:

```
NAME      STATUS   ROLES           AGE
kmaster   Ready    control-plane
worker1   Ready    <none>
worker2   Ready    <none>
```

In this example we will assign the pod to:

```
worker2
```

---

# 6. Pod Manifest using `nodeName`

Create a YAML file.

### node-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-node-pod
  labels:
    app: nginx
spec:
  nodeName: worker2
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
```

Key line:

```
nodeName: worker2
```

This forces Kubernetes to run the pod on **worker2 node**.

---

# 7. Steps to Execute the Script

### Step 1 — Create YAML file

```
vi node-pod.yaml
```

Paste the YAML configuration.

---

### Step 2 — Create the Pod

```
kubectl create -f node-pod.yaml
```

---

### Step 3 — Verify Pod Creation

```
kubectl get pods
```

Example output:

```
NAME             READY   STATUS    AGE
nginx-node-pod   1/1     Running   5s
```

---

# 8. Verify Pod Placement

To check which node the pod is running on:

```
kubectl get pods -o wide
```

Example output:

```
NAME             READY   STATUS    AGE   IP           NODE
nginx-node-pod   1/1     Running   20s   10.244.2.5   worker2
```

Here we can see:

```
NODE = worker2
```

This confirms the pod is running on the **worker2 node**.

---

# 9. Complete Workflow

```
Check Nodes
      ↓
Create YAML file with nodeName
      ↓
kubectl create -f node-pod.yaml
      ↓
kubectl get pods
      ↓
kubectl get pods -o wide
      ↓
Verify NODE column
```

---

# 10. Best Practice

Instead of `nodeName`, production clusters usually use:

* **nodeSelector**
* **NodeAffinity**
* **Taints and Tolerations**

because they provide **more flexible scheduling**.

---

# 11. Summary

`nodeName` allows administrators to:

* Directly assign pods to specific nodes
* Bypass the Kubernetes scheduler
* Control where workloads run

But it should mainly be used for:

* Testing
* Troubleshooting
* Learning environments

