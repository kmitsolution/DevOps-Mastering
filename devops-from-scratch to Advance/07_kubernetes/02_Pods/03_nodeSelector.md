Below is a **structured documentation-style explanation** for your **Kubernetes NodeSelector lecture**, organized step-by-step for training or lecture notes.

---

# Assigning Pods to Worker Nodes using **nodeSelector** in Kubernetes

## 1. Introduction

By default, Kubernetes automatically schedules pods to any available worker node based on available resources.

However, in many real-world environments we want workloads to run only on specific nodes.
For example:

* Production workloads → Production nodes
* Staging workloads → Staging nodes
* GPU workloads → GPU nodes

To achieve this, Kubernetes uses **Node Labels** and **nodeSelector**.

---

# 2. What are Node Labels?

Labels are **key-value pairs attached to Kubernetes objects**, including nodes.

Example:

```id="x0r2d0"
env=prod
env=stag
disk=ssd
region=us-east
```

These labels help Kubernetes:

* Organize resources
* Select nodes for scheduling
* Control where pods run

---

# 3. Show Node Labels

To view all node labels:

```bash id="i6ypi4"
kubectl get nodes --show-labels
```

Example output:

```id="v8n0nl"
NAME      STATUS   ROLES           LABELS
kmaster   Ready    control-plane   kubernetes.io/os=linux
worker1   Ready    <none>          kubernetes.io/os=linux
worker2   Ready    <none>          kubernetes.io/os=linux
```

---

# 4. View or Edit Node Labels

To edit node configuration and labels:

```bash id="2ypr8d"
kubectl edit node worker1
```

Inside the configuration you will see:

```id="ysl7o3"
metadata:
  labels:
```

You can view or modify labels here.

---

# 5. Why Labels are Important

Labels are critical in Kubernetes because they help:

* Identify resources
* Select nodes for scheduling
* Implement environment-based deployments
* Organize workloads
* Apply policies and automation

For example:

```id="mq0c7k"
Production workloads → Production nodes
Staging workloads → Staging nodes
```

---

# 6. Scenario Example

Suppose we have two environments:

| Node    | Environment |
| ------- | ----------- |
| worker1 | Production  |
| worker2 | Staging     |

We assign labels as follows:

```
worker1 → env=prod
worker2 → env=stag
```

---

# 7. Add Labels to Nodes

Assign labels using the following commands.

### Label worker1 as production

```bash id="msmq7c"
kubectl label nodes worker1 env=prod
```

### Label worker2 as staging

```bash id="d0h25g"
kubectl label nodes worker2 env=stag
```

Verify labels:

```bash id="l4uh4x"
kubectl get nodes --show-labels
```

---

# 8. Create Pod Using nodeSelector

Now we will create a pod that runs only on **production nodes**.

### Pod YAML File

Create file:

```
prod-pod.yaml
```

```id="ljlp85"
apiVersion: v1
kind: Pod
metadata:
  name: prod-pod
spec:
  nodeSelector:
    env: prod
  containers:
  - name: nginx
    image: nginx
```

Important line:

```id="axtyk3"
nodeSelector:
  env: prod
```

This tells Kubernetes:

```
Run this pod only on nodes where label env=prod
```

---

# 9. Create the Pod

Run:

```bash id="92pxnj"
kubectl create -f prod-pod.yaml
```

Check pod status:

```bash id="3gtcai"
kubectl get pods
```

---

# 10. Verify Pod Placement

Check which node the pod is running on.

```bash id="y2n1w0"
kubectl get pods -o wide
```

Example output:

```id="93nsb9"
NAME       READY   STATUS    AGE   IP           NODE
prod-pod   1/1     Running   10s   10.244.1.15  worker1
```

Here we can see:

```id="lb03ec"
NODE = worker1
```

This confirms the pod was scheduled on the **production node (worker1)**.

---

# 11. Change Node Label

Now change the label of **worker1**.

From:

```
env=prod
```

to

```
env=preprod
```

Run:

```bash id="p5h7j1"
kubectl label nodes worker1 env=preprod --overwrite
```

Verify:

```bash id="rhlw1p"
kubectl get nodes --show-labels
```

---

# 12. What Happens to the Pod?

Now check the pod again.

```bash id="4h2rye"
kubectl get pods -o wide
```

You will notice:

```
prod-pod is still running on worker1
```

The pod **will NOT be removed**.

---

# 13. Why the Pod is Not Removed

Once a pod is scheduled to a node:

* Kubernetes **does not re-evaluate nodeSelector rules**
* The pod becomes independent of label changes

So even though the label changed from:

```
env=prod → env=preprod
```

the existing pod **continues running**.

NodeSelector only affects **pod scheduling**, not **existing running pods**.

---

# 14. Important Concept

NodeSelector works only during:

```
Pod Scheduling Time
```

It does **not affect running pods**.

---

# 15. Example Workflow

```
Label Node
     ↓
Create Pod with nodeSelector
     ↓
Scheduler selects node
     ↓
Pod starts running
     ↓
Changing node label does NOT move the pod
```

---

# 16. Summary

In this lecture we learned:

* What **Node Labels** are
* How to **view labels**
* How to **assign labels to nodes**
* Why labels are important
* How to schedule pods using **nodeSelector**
* How to verify pod placement
* Why label changes do not affect running pods

NodeSelector is useful for **simple node selection**, but for advanced scheduling Kubernetes provides:

* **NodeAffinity**
* **Taints and Tolerations**
