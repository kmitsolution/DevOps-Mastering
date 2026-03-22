### Why do we need Services in Kubernetes?

In Kubernetes, you typically run your application using a **Deployment**, which manages a set of Pods.

However, Pods have two important characteristics:

* Their **IP addresses are dynamic** (they can change if a Pod is recreated)
* They are **ephemeral** (created and destroyed frequently)

👉 So, if one Pod wants to talk to another, or if an external user wants to access your app, relying on Pod IPs is unreliable.

### Solution: Kubernetes Service

A **Service** in Kubernetes provides a **stable network endpoint** to access a group of Pods.

Think of it like:

> A **permanent address (IP + DNS name)** that forwards traffic to the right Pods behind the scenes.

---

## Accessing Applications

### 1. Inside the Cluster

If one application (Pod) needs to talk to another:

* It uses the **Service name**
* Kubernetes DNS resolves it to the Service IP

✔ Example: `my-service.default.svc.cluster.local`

---

### 2. Outside the Cluster

If users or external systems need access:

* A Service can expose the app **outside the cluster**
* This depends on the **type of Service** you choose

---

## How Services Get IP Addresses

Kubernetes assigns Services an IP from a special range called:

👉 **Service CIDR range**

* Defined when the cluster is created (e.g., `10.96.0.0/12`)
* Managed by the cluster (not your cloud provider or local network)
* These IPs are **virtual IPs (Cluster IPs)**

Important:

* These IPs are **not assigned to a physical network interface**
* They are implemented using **kube-proxy / networking rules**
* Traffic sent to the Service IP is **load-balanced to Pods**

---

## Types of Kubernetes Services

### 1. ClusterIP (Default)

**What it does:**

* Exposes the Service **only inside the cluster**

**Key points:**

* Gets a **Cluster IP** from Service CIDR
* Not accessible from outside
* Used for **internal communication between services**

**Use case:**

* Backend services (e.g., database, internal APIs)

---

### 2. NodePort

**What it does:**

* Exposes the Service on each Node’s IP at a static port

**Key points:**

* Kubernetes allocates a port (default range: **30000–32767**)
* Accessible via:

  ```
  NodeIP:NodePort
  ```
* Still has a ClusterIP internally

**Flow:**
External request → Node IP → NodePort → Service → Pod

**Use case:**

* Simple external access (testing, small setups)

---

### 3. LoadBalancer

**What it does:**

* Exposes the Service externally using a **cloud provider load balancer**

**Key points:**

* Creates:

  * ClusterIP
  * NodePort
  * External Load Balancer (from cloud like AWS, Azure, GCP)
* Gets an **external IP**

**Flow:**
User → Load Balancer → NodePort → Service → Pods

**Use case:**

* Production apps needing public access

---

## Summary

* **Deployment** → manages Pods
* **Pods** → dynamic and unstable IPs
* **Service** → stable access point

| Service Type | Scope            | External Access | IP Type                  |
| ------------ | ---------------- | --------------- | ------------------------ |
| ClusterIP    | Internal only    | ❌ No            | Cluster IP               |
| NodePort     | Node level       | ✅ Yes           | Cluster IP + NodePort    |
| LoadBalancer | External (cloud) | ✅ Yes           | Cluster IP + External IP |

---


