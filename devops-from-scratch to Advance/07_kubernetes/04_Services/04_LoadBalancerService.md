## LoadBalancer Service in Kubernetes (EKS Example)

### What is LoadBalancer Service?

A **LoadBalancer Service** exposes your application **to the internet** using a **cloud provider’s load balancer**.

👉 In your case (EKS – Amazon EKS):

* Kubernetes automatically provisions an **AWS Elastic Load Balancer**
* You get a **public IP / DNS**
* Traffic is routed to your Pods

---

## Scenario (Same Example)

* Deployment with **3 replicas**
* Running inside an **EKS cluster**
* Need **external/public access**

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

* 3 Pods created
* Each Pod has its own internal IP

---

### 2. Expose Deployment as LoadBalancer Service

Now we expose it using **LoadBalancer**

Conceptual YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - port: 80          # Service port
    targetPort: 80    # Pod port
```

👉 Important:

* No need to specify `nodePort` (Kubernetes assigns it automatically)
* AWS will create a **Load Balancer**

---

### 3. Using `--dry-run=client -o yaml`

Generate YAML first:

```bash
kubectl expose deployment my-app \
  --port=80 --target-port=80 \
  --type=LoadBalancer \
  --dry-run=client -o yaml > service.yaml
```

👉 This:

* Generates YAML
* Saves it to `service.yaml`
* Does NOT create the Service yet

---

### 4. Apply the YAML

```bash
kubectl apply -f service.yaml
```

👉 Now Kubernetes:

* Creates the Service
* Talks to AWS
* AWS provisions a **Load Balancer**

---

## What Happens in EKS (Behind the Scenes)

When you create a LoadBalancer Service:

1. Kubernetes calls AWS APIs
2. AWS creates an **Elastic Load Balancer (ELB)**
3. ELB gets:

   * Public IP or DNS name
4. ELB forwards traffic to:

   * NodePort → Pods

---

## Traffic Flow

### External Access (Internet)

```
User → AWS Load Balancer → NodePort → Service → Pod
```

Example:

```
http://a1b2c3d4e5.elb.amazonaws.com
```

---

### Internal Access

```
Pod → ClusterIP → Pod
```

👉 Same internal behavior as ClusterIP

---

## Layers in LoadBalancer Service

A LoadBalancer Service automatically includes:

1. **ClusterIP** (internal communication)
2. **NodePort** (node-level access)
3. **External Load Balancer** (AWS ELB)

---

## Key Characteristics (EKS)

* Fully managed by AWS
* Provides **public access**
* Automatically provisions ELB
* Supports scaling and high availability
* Works with **security groups** and AWS networking

---

## Summary (All 3 Together)

| Feature     | ClusterIP     | NodePort         | LoadBalancer (EKS)       |
| ----------- | ------------- | ---------------- | ------------------------ |
| Access      | Internal      | Node IP          | Public Internet          |
| External IP | ❌ No          | ❌ (uses Node IP) | ✅ Yes (ELB DNS/IP)       |
| Complexity  | Low           | Medium           | High (cloud integration) |
| Use case    | Internal apps | Testing          | Production apps          |

---

## Simple Understanding

* Deployment → 3 Pods
* LoadBalancer Service → creates AWS ELB
* ELB → exposes app to internet
* Traffic → ELB → Nodes → Pods

