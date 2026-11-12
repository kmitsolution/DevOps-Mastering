
# Kubernetes **NodeAffinity**

## 1. Introduction

**NodeAffinity** is an advanced way to control **which nodes a pod can run on**.

It is similar to **nodeSelector**, but much more flexible because it allows:

* complex rules
* soft and hard constraints
* logical operators

NodeAffinity works using **node labels**.

Example label:

```
env=prod
disk=ssd
region=us-east
```

Pods can then be scheduled only on nodes that match those labels.

---

# 2. Why NodeAffinity is Needed

`nodeSelector` is very simple.

Example:

```yaml
nodeSelector:
  env: prod
```

But it has limitations:

* Only simple **key=value** matching
* No logical operators
* No preference rules

NodeAffinity solves these limitations.

---

# 3. NodeAffinity Types

NodeAffinity has **two major types**.

| Type                                            | Meaning   |
| ----------------------------------------------- | --------- |
| requiredDuringSchedulingIgnoredDuringExecution  | Hard rule |
| preferredDuringSchedulingIgnoredDuringExecution | Soft rule |

---

# 4. Hard Rule

## requiredDuringSchedulingIgnoredDuringExecution

This means:

```
Pod MUST run on nodes matching this rule
```

If no node matches:

```
Pod will remain Pending
```

Once the pod is running:

```
If node labels change, pod will NOT move
```

That is why it is called:

```
IgnoredDuringExecution
```

---

# 5. Example: Hard NodeAffinity

Example YAML (from Kubernetes documentation concept).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: In
            values:
            - prod
  containers:
  - name: nginx
    image: nginx
```

Meaning:

```
Pod will run ONLY on nodes with label env=prod
```

---

# 6. NodeAffinity Operators

NodeAffinity supports multiple operators.

| Operator     | Meaning              |
| ------------ | -------------------- |
| In           | value must match     |
| NotIn        | value must NOT match |
| Exists       | label must exist     |
| DoesNotExist | label must NOT exist |
| Gt           | greater than         |
| Lt           | less than            |

---

# 7. Example: Using **In**

```yaml
matchExpressions:
- key: env
  operator: In
  values:
  - prod
```

Meaning:

```
env must be prod
```

---

# 8. Example: Using **NotIn**

```yaml
matchExpressions:
- key: env
  operator: NotIn
  values:
  - stag
```

Meaning:

```
Node must NOT have env=stag
```

---

# 9. Example: Using **Exists**

```yaml
matchExpressions:
- key: gpu
  operator: Exists
```

Meaning:

```
Node must contain label gpu
```

Example node:

```
worker1 gpu=true
```

---

# 10. Example: Using **DoesNotExist**

```yaml
matchExpressions:
- key: testnode
  operator: DoesNotExist
```

Meaning:

```
Node must NOT contain label testnode
```

---

# 11. Example: Using **Gt**

Used with numeric labels.

Example node label:

```
cpu=8
```

Example rule:

```yaml
matchExpressions:
- key: cpu
  operator: Gt
  values:
  - "4"
```

Meaning:

```
cpu must be greater than 4
```

---

# 12. Example: Using **Lt**

Example:

```yaml
matchExpressions:
- key: cpu
  operator: Lt
  values:
  - "16"
```

Meaning:

```
cpu must be less than 16
```

---

# 13. Soft Rule

## preferredDuringSchedulingIgnoredDuringExecution

This means:

```
Pod prefers these nodes but it is NOT mandatory
```

If matching node exists:

```
Pod will be scheduled there
```

If not:

```
Pod can run on other nodes
```

---

# 14. Example: Preferred NodeAffinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-preferred
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: env
            operator: In
            values:
            - prod
  containers:
  - name: nginx
    image: nginx
```

Meaning:

```
Prefer nodes with env=prod
But if not available, schedule anywhere
```

---

# 15. Understanding **weight**

`weight` determines **priority**.

Range:

```
1 – 100
```

Example:

```
weight: 100 → highest preference
weight: 1 → lowest preference
```

---

# 16. NodeSelectorTerms

Inside NodeAffinity you will see:

```
nodeSelectorTerms
```

This represents **OR logic**.

Example:

```
env=prod OR env=stag
```

Example YAML:

```yaml
nodeSelectorTerms:
- matchExpressions:
  - key: env
    operator: In
    values:
    - prod
- matchExpressions:
  - key: env
    operator: In
    values:
    - stag
```

Meaning:

```
env=prod OR env=stag
```

---

# 17. matchExpressions

Inside `nodeSelectorTerms`, we define conditions using:

```
matchExpressions
```

Example:

```yaml
matchExpressions:
- key: env
  operator: In
  values:
  - prod
```

---

# 18. Example Full Script

### node-affinity.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: env
            operator: In
            values:
            - prod
  containers:
  - name: nginx
    image: nginx
```

---

# 19. Steps to Execute

### Step 1: Label the node

```
kubectl label nodes worker1 env=prod
```

---

### Step 2: Create the pod

```
kubectl create -f node-affinity.yaml
```

---

### Step 3: Verify pod placement

```
kubectl get pods -o wide
```

Example output:

```
NAME               READY   STATUS   NODE
node-affinity-pod  1/1     Running  worker1
```

---

# 20. NodeSelector vs NodeAffinity

| Feature             | nodeSelector | NodeAffinity |
| ------------------- | ------------ | ------------ |
| Simple matching     | Yes          | Yes          |
| Logical operators   | No           | Yes          |
| Soft rules          | No           | Yes          |
| Hard rules          | Yes          | Yes          |
| Flexible scheduling | No           | Yes          |

---

# 21. Summary

NodeAffinity provides **powerful pod scheduling control** using:

* labels
* logical operators
* priority rules

Main options:

```
requiredDuringSchedulingIgnoredDuringExecution
preferredDuringSchedulingIgnoredDuringExecution
```

Supported operators:

```
In
NotIn
Exists
DoesNotExist
Gt
Lt
```

NodeAffinity is widely used in **production Kubernetes clusters** for:

* environment separation
* hardware-specific workloads
* region-based scheduling
* GPU workloads

---

