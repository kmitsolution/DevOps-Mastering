
# Kubernetes **Pod Affinity**

## 1. Introduction

**Pod Affinity** is used to control **which pods should run close to other pods**.

Instead of selecting **nodes**, Pod Affinity selects **other pods**.

Meaning:

```
Run this pod on a node where another specific pod is already running
```

Example scenario:

```
App Pod → run close to Database Pod
```

This helps reduce:

* network latency
* communication delays

---

# 2. Why Pod Affinity is Useful

Pod Affinity is beneficial in scenarios like:

### 1. Microservices Communication

Example:

```
Frontend Pod → Backend Pod
```

To reduce network calls, both pods can run on the **same node**.

---

### 2. Database Applications

Example:

```
Application Pod
Database Pod
```

Keeping them together improves performance.

---

### 3. High Performance Workloads

When pods need **fast communication**, they should run on the **same node or zone**.

---

# 3. Pod Affinity Types

Like NodeAffinity, PodAffinity has **two types**.

| Type                                            | Meaning   |
| ----------------------------------------------- | --------- |
| requiredDuringSchedulingIgnoredDuringExecution  | Hard rule |
| preferredDuringSchedulingIgnoredDuringExecution | Soft rule |

---

# 4. Hard Pod Affinity

## requiredDuringSchedulingIgnoredDuringExecution

This means:

```
Pod MUST run on nodes where another specific pod exists
```

If no matching pod exists:

```
Pod will remain Pending
```

---

# 5. Example: Pod Affinity

Suppose we have:

```
backend-pod
```

Now we want:

```
frontend-pod
```

to run **on the same node as backend-pod**.

---

### Backend Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  labels:
    app: backend
spec:
  containers:
  - name: backend
    image: nginx
```

---

### Frontend Pod using Pod Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - backend
        topologyKey: kubernetes.io/hostname
  containers:
  - name: frontend
    image: nginx
```

---

# 6. Understanding the Important Fields

## labelSelector

This identifies **which pods we want to match**.

Example:

```yaml
labelSelector:
  matchExpressions:
  - key: app
    operator: In
    values:
    - backend
```

Meaning:

```
Look for pods with label app=backend
```

---

## topologyKey

This defines **where pods should be placed together**.

Example:

```yaml
topologyKey: kubernetes.io/hostname
```

Meaning:

```
Run pods on the same node
```

Other examples:

```
topology.kubernetes.io/zone
topology.kubernetes.io/region
```

---

# 7. Soft Pod Affinity

## preferredDuringSchedulingIgnoredDuringExecution

This means:

```
Prefer to run near other pods, but not mandatory
```

If matching pods exist:

```
Scheduler places pod nearby
```

If not:

```
Scheduler runs pod anywhere
```

---

# 8. Example: Preferred Pod Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - backend
          topologyKey: kubernetes.io/hostname
  containers:
  - name: nginx
    image: nginx
```

Meaning:

```
Prefer node where backend pod exists
```

But if none exists:

```
Pod still runs
```

---

# 9. Example Workflow

```
Create Backend Pod
       ↓
Create Frontend Pod with Pod Affinity
       ↓
Scheduler finds node with backend pod
       ↓
Frontend Pod scheduled on same node
```

---

# 10. Verify Pod Placement

Check where pods are running:

```bash
kubectl get pods -o wide
```

Example output:

```
NAME           NODE
backend-pod    worker1
frontend-pod   worker1
```

Both pods are on the **same node**.

---

# 11. Pod Affinity vs Node Affinity

| Feature           | Node Affinity          | Pod Affinity          |
| ----------------- | ---------------------- | --------------------- |
| Based on          | Node labels            | Pod labels            |
| Scheduling target | Node                   | Pod location          |
| Use case          | Hardware / environment | Application proximity |

---

# 12. Pod Anti-Affinity

Pod Anti-Affinity is the opposite.

Example:

```
Do NOT schedule pods together
```

Useful for:

* high availability
* spreading pods across nodes

Example:

```
Two replicas should NOT run on the same node
```

---

# 13. Summary

Pod Affinity allows Kubernetes to:

* schedule pods **close to other pods**
* improve **performance**
* reduce **network latency**

Two main types:

```
requiredDuringSchedulingIgnoredDuringExecution
preferredDuringSchedulingIgnoredDuringExecution
```

Key components:

```
labelSelector
matchExpressions
topologyKey
```

---

