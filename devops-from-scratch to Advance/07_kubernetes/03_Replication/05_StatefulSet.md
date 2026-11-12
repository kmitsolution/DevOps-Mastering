Below is a **structured DevOps training–style explanation of StatefulSet in Kubernetes**, suitable for **lecture notes, documentation, or DevOps course material**.

---

# Kubernetes **StatefulSet**

## 1. Introduction

A **StatefulSet** is a Kubernetes workload used to manage **stateful applications**.

Unlike Deployments (which manage **stateless applications**), StatefulSets maintain:

* **stable network identity**
* **persistent storage**
* **ordered deployment**
* **ordered scaling**

Examples of stateful applications:

* Databases (MySQL, PostgreSQL)
* Distributed systems (Kafka, Zookeeper)
* Message queues (RabbitMQ)
* File systems

---

# 2. Stateless vs Stateful Applications

### Stateless Application

Example:

```text
Web Servers
Nginx
Frontend Applications
```

Characteristics:

* No stored session data
* Any pod can handle requests
* Pods are interchangeable

Managed by:

```text
Deployment
ReplicaSet
```

---

### Stateful Application

Example:

```text
Databases
Kafka
Redis
```

Characteristics:

* Stores data
* Pod identity must be preserved
* Storage must persist even if pod restarts

Managed by:

```text
StatefulSet
```

---

# 3. StatefulSet Architecture

Example with 3 replicas:

```text
StatefulSet
   |
   |---- pod-0
   |---- pod-1
   |---- pod-2
```

Each pod has:

* unique name
* unique storage
* stable network identity

Example pod names:

```text
mysql-0
mysql-1
mysql-2
```

---

# 4. Key Features of StatefulSet

| Feature            | Description                 |
| ------------------ | --------------------------- |
| Stable Pod Name    | Pod names remain constant   |
| Persistent Storage | Each pod has its own volume |
| Ordered Deployment | Pods created sequentially   |
| Ordered Scaling    | Pods scaled in order        |

---

# 5. Example StatefulSet YAML

Create file:

```text
statefulset-nginx.yaml
```

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-stateful
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
  volumeClaimTemplates:
  - metadata:
      name: nginx-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

---

# 6. Headless Service Requirement

StatefulSets require a **Headless Service**.

Example service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - port: 80
```

Important line:

```text
clusterIP: None
```

This creates a **headless service**.

---

# 7. Create StatefulSet

Apply configuration:

```bash
kubectl apply -f statefulset-nginx.yaml
```

Verify:

```bash
kubectl get statefulset
```

---

# 8. Verify Pods

```bash
kubectl get pods
```

Example output:

```text
nginx-stateful-0
nginx-stateful-1
nginx-stateful-2
```

Pods are created **in sequence**.

---

# 9. Verify Pod Placement

```bash
kubectl get pods -o wide
```

Example output:

```text
NAME             NODE
nginx-stateful-0 worker1
nginx-stateful-1 worker2
nginx-stateful-2 worker1
```

---

# 10. Persistent Storage

Each pod gets its **own persistent volume claim (PVC)**.

Example:

```bash
kubectl get pvc
```

Output:

```text
nginx-storage-nginx-stateful-0
nginx-storage-nginx-stateful-1
nginx-storage-nginx-stateful-2
```

Each pod has its **own storage**.

---

# 11. Ordered Pod Creation

Pods are created sequentially:

```text
nginx-stateful-0 → first
nginx-stateful-1 → second
nginx-stateful-2 → third
```

Next pod starts only after previous pod is **Running**.

---

# 12. Ordered Pod Deletion

When scaling down:

```bash
kubectl scale statefulset nginx-stateful --replicas=2
```

Pods are deleted in reverse order:

```text
nginx-stateful-2 → deleted
nginx-stateful-1 → running
nginx-stateful-0 → running
```

---

# 13. Stable Network Identity

Each pod gets a **stable DNS name**.

Example format:

```text
podname.servicename.namespace.svc.cluster.local
```

Example:

```text
nginx-stateful-0.nginx.default.svc.cluster.local
```

This allows applications to **communicate reliably**.

---

# 14. StatefulSet vs Deployment

| Feature          | Deployment | StatefulSet       |
| ---------------- | ---------- | ----------------- |
| Application type | Stateless  | Stateful          |
| Pod identity     | Random     | Stable            |
| Storage          | Shared     | Dedicated per pod |
| Pod creation     | Parallel   | Ordered           |

---

# 15. Real World Example

Example: **MySQL cluster**

```text
mysql-0 → primary
mysql-1 → replica
mysql-2 → replica
```

Each pod has:

* stable hostname
* dedicated storage

---

# 16. StatefulSet Workflow

```text
Create Headless Service
        ↓
Create StatefulSet
        ↓
Pod-0 created
        ↓
Pod-1 created
        ↓
Pod-2 created
```

---

# 17. Summary

StatefulSets are used for **stateful applications**.

Key features:

```text
Stable Pod Identity
Persistent Storage
Ordered Deployment
Ordered Scaling
```

Common examples:

```text
MySQL
Kafka
Zookeeper
MongoDB
Redis
```

---

If you'd like, I can also create a **very useful Kubernetes workload comparison chart** for your course showing:

```text
Pod vs Deployment vs ReplicaSet vs DaemonSet vs StatefulSet vs Job vs CronJob
```

This makes it **very easy for students to remember when to use each Kubernetes resource**.
