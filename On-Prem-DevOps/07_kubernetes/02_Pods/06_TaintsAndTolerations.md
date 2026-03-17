# Kubernetes **Taints and Tolerations**

## 1. Introduction

In Kubernetes, **Taints and Tolerations** work together to control **which pods can run on which nodes**.

Earlier scheduling methods:

* **nodeSelector**
* **NodeAffinity**
* **PodAffinity**

These methods **attract pods to nodes**.

But **Taints do the opposite**.

Taints **repel pods from nodes** unless the pod has a matching **toleration**.

Concept:

```id="9wblc3"
Node → Taint → Blocks pods
Pod → Toleration → Allowed to run
```

---

# 2. Real World Example

Suppose you have nodes dedicated for:

* GPU workloads
* Production workloads
* Database workloads

You do not want **normal applications** to run on those nodes.

So you apply a **taint on the node**.

Example:

```id="rvz4to"
gpu=true:NoSchedule
```

Now only pods with matching **tolerations** can run there.

---

# 3. What is a Taint?

A **Taint** is applied to a node to **restrict pods from scheduling on that node**.

Structure:

```id="e1o5qt"
key=value:effect
```

Example:

```id="09ppbh"
env=prod:NoSchedule
```

Meaning:

```id="5ayoj6"
Pods cannot run on this node unless they tolerate this taint
```

---

# 4. What is a Toleration?

A **Toleration** is added to a pod so it can **run on a tainted node**.

Example:

```id="g9f80u"
tolerations:
- key: "env"
  operator: "Equal"
  value: "prod"
  effect: "NoSchedule"
```

This means the pod **can run on nodes tainted with env=prod**.

---

# 5. Types of Taint Effects

Kubernetes supports **three taint effects**.

| Effect           | Meaning                                    |
| ---------------- | ------------------------------------------ |
| NoSchedule       | Pod will NOT be scheduled on the node      |
| PreferNoSchedule | Try to avoid scheduling                    |
| NoExecute        | Remove running pods if they don't tolerate |

---

# 6. NoSchedule

This prevents new pods from running.

Example taint:

```bash id="chhgpp"
kubectl taint nodes worker1 env=prod:NoSchedule
```

Meaning:

```id="iq29hx"
No pod can run on worker1 unless it tolerates env=prod
```

---

# 7. PreferNoSchedule

This is a **soft rule**.

Scheduler tries to avoid placing pods on the node.

Example:

```bash id="yrx13o"
kubectl taint nodes worker1 env=prod:PreferNoSchedule
```

Pods may still run there if no other nodes are available.

---

# 8. NoExecute

This is the most strict rule.

Example:

```bash id="c07fgi"
kubectl taint nodes worker1 env=prod:NoExecute
```

Meaning:

* New pods cannot run
* Existing pods will be **evicted**

Unless they tolerate the taint.

---

# 9. Add a Taint to Node

Example:

```bash id="60x5rc"
kubectl taint nodes worker1 env=prod:NoSchedule
```

Verify taints:

```bash id="h7gsgo"
kubectl describe node worker1
```

Look for:

```id="45okdy"
Taints:
env=prod:NoSchedule
```

---

# 10. Pod with Toleration Example

### taint-pod.yaml

```yaml id="48tr7a"
apiVersion: v1
kind: Pod
metadata:
  name: nginx-taint
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "env"
    operator: "Equal"
    value: "prod"
    effect: "NoSchedule"
```

This pod can run on nodes with taint:

```id="4hbvjy"
env=prod:NoSchedule
```

---

# 11. Create the Pod

```bash id="6b2gjq"
kubectl create -f taint-pod.yaml
```

---

# 12. Verify Pod Placement

```bash id="qpacn0"
kubectl get pods -o wide
```

Example output:

```id="df8cfq"
NAME         READY   STATUS   NODE
nginx-taint  1/1     Running  worker1
```

---

# 13. Remove Taint from Node

To remove taint:

```bash id="fnolxg"
kubectl taint nodes worker1 env=prod:NoSchedule-
```

Notice the **dash at the end**.

---

# 14. Common Real-World Use Cases

### 1. Master Node Protection

Master nodes are tainted by default.

Example:

```id="gd8h2s"
node-role.kubernetes.io/control-plane:NoSchedule
```

This prevents normal workloads from running on master nodes.

---

### 2. Dedicated Nodes

Example:

```id="sywzpd"
database nodes
gpu nodes
```

Only specific pods can run there.

---

### 3. Maintenance Mode

Administrators can temporarily taint nodes during maintenance.

---

# 15. Taints vs NodeSelector

| Feature          | NodeSelector | Taints    |
| ---------------- | ------------ | --------- |
| Scheduling style | Attraction   | Repulsion |
| Based on         | Labels       | Taints    |
| Pod control      | Simple       | Advanced  |

---

# 16. Scheduling Logic

Kubernetes scheduling flow:

```id="8ek1oc"
NodeSelector
      ↓
NodeAffinity
      ↓
Taints & Tolerations
      ↓
Pod scheduled
```

---

# 17. Summary

Taints and Tolerations allow administrators to:

* reserve nodes
* prevent unwanted workloads
* control scheduling behavior

Important concepts:

```id="6wx7zn"
Taint → applied to nodes
Toleration → applied to pods
```

Effects:

```id="9s2fcv"
NoSchedule
PreferNoSchedule
NoExecute
```

---

