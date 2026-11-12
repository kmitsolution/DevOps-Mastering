## NodePort Service in Kubernetes 

### What is NodePort?

A **NodePort Service** is used to expose your application **outside the cluster**.

👉 It opens a specific port on **every Node (VM/Server)** in the cluster.

> So anyone can access your app using:
> **`NodeIP : NodePort`**

---

## Scenario (Same as before)

* Deployment with **3 replicas**
* Each Pod runs your application
* Now we want to access it **from outside the cluster**

---

## Step-by-Step Conceptual Flow

### 1. Deployment (3 replicas)

Same as before:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

👉 Result:

* 3 Pods running
* Each has its own dynamic IP

---

### 2. Expose Deployment as NodePort Service

Instead of ClusterIP, we now use **NodePort**

Conceptually, this creates:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - port: 80          # Service port (inside cluster)
    targetPort: 80    # Pod port
    nodePort: 30007   # External port (on each Node)
```

👉 Important:

* `type: NodePort` enables external access
* `nodePort` is usually auto-assigned (30000–32767)
* You can also manually specify it (like `30007`)

---

### 3. Using `--dry-run=client -o yaml`

Generate YAML without creating the Service:

```bash
kubectl expose deployment my-app \
  --port=80 --target-port=80 \
  --type=NodePort \
  --dry-run=client -o yaml > service.yaml
```

👉 This:

* Creates YAML definition
* Saves it to `service.yaml`
* Does NOT create the Service yet

---

### 4. Apply the YAML

```bash
kubectl apply -f service.yaml
```

👉 Now NodePort Service is created.

---

## How NodePort Works Internally

NodePort actually includes **two layers**:

1. **ClusterIP (internal)**
2. **NodePort (external)**

👉 So every NodePort Service has:

* A **ClusterIP** (for internal access)
* A **NodePort** (for external access)

---

## Traffic Flow

### From Outside the Cluster:

```
User → NodeIP:NodePort → Service → One of the Pods
```

Example:

```
http://192.168.1.10:30007
```

👉 You can use **any node's IP** in the cluster.

---

### From Inside the Cluster:

```
Pod → ClusterIP → Pod
```

👉 Same as ClusterIP behavior internally.

---

## Key Characteristics of NodePort

* Exposes app on **every Node**
* Accessible from outside the cluster
* Uses port range: **30000–32767**
* Includes ClusterIP internally
* Basic load balancing across Pods

---

## Summary (ClusterIP vs NodePort)

| Feature       | ClusterIP     | NodePort                  |
| ------------- | ------------- | ------------------------- |
| Access scope  | Internal only | Internal + External       |
| External URL  | ❌ No          | ✅ Yes (NodeIP:Port)       |
| Port exposure | No            | Yes (30000–32767)         |
| Use case      | Internal apps | Testing / simple exposure |

---

## Simple Understanding

* Deployment → 3 Pods
* NodePort Service → opens a port on all Nodes
* External users → hit Node IP + port
* Kubernetes → routes traffic to Pods

