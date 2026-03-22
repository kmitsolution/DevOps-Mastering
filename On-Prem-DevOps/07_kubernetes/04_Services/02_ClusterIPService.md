## ClusterIP Service in Kubernetes (Concept + Example)

### What is ClusterIP?

A **ClusterIP Service** is the **default Service type** in Kubernetes.

👉 It provides:

* A **stable internal IP address**
* Access to your application **only within the cluster**

> It is mainly used for **communication between Pods** (internal traffic).

---

## Scenario

* You have a **Deployment** with **3 replicas (Pods)**
* Each Pod runs your application
* You want a **single stable endpoint** to access all Pods

👉 That’s where a **ClusterIP Service** comes in.

---

## Step-by-Step Conceptual Flow

### 1. Create a Deployment (3 replicas)

* Deployment creates:

  * 3 Pods
  * Each Pod gets its own IP (dynamic)

Example (conceptual YAML):

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

👉 Now:

* 3 Pods are running
* But accessing them directly is unreliable (IPs can change)

---

### 2. Expose Deployment as a ClusterIP Service

You use the **expose command** to create a Service from the Deployment.

Conceptually, this generates a Service like:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: ClusterIP   # default
  selector:
    app: my-app
  ports:
  - port: 80        # Service port
    targetPort: 80  # Pod container port
```

👉 Important points:

* `selector` matches Pods with label `app: my-app`
* Service automatically connects to those Pods
* A **ClusterIP is assigned** (e.g., `10.96.x.x`)

---

### 3. Using `--dry-run=client -o yaml`

Instead of directly creating the Service, you can:

* Generate YAML first
* Review/edit it
* Then apply it

Concept:

```bash
kubectl expose deployment my-app \
  --port=80 --target-port=80 \
  --type=ClusterIP \
  --dry-run=client -o yaml > service.yaml
```

👉 What this does:

* Does NOT create the Service
* Outputs YAML definition
* Saves it into `service.yaml`

---

### 4. Apply the YAML

Then you create the Service using the YAML file:

```bash
kubectl apply -f service.yaml
```

👉 Now the ClusterIP Service is created.

---

## How ClusterIP Works Internally

* Kubernetes assigns a **virtual IP** (ClusterIP)
* Example: `10.96.45.12`
* This IP:

  * Is **internal only**
  * Is handled by **kube-proxy**

### Traffic Flow

```
Pod → Service (ClusterIP) → One of the 3 Pods
```

👉 Load balancing:

* Kubernetes distributes traffic across all 3 Pods
* Typically round-robin

---

## How to Access It

Inside the cluster, you can use:

* Service name: `my-app`
* Full DNS: `my-app.default.svc.cluster.local`

Example:

```
http://my-app
```

---

## Key Characteristics of ClusterIP

* Default Service type
* Internal access only (not reachable from outside)
* Stable IP even if Pods restart
* Automatically load balances traffic
* Uses label selectors to find Pods

---

## Simple Summary

* Deployment → creates 3 Pods
* Pods → dynamic IPs
* ClusterIP Service → gives **one stable IP + DNS name**
* All internal traffic goes through the Service
