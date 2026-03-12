
<img width="318" height="159" alt="image" src="https://github.com/user-attachments/assets/55bc7f83-f611-4fc7-bfa8-e3e7af44fc20" />

<img width="800" height="450" alt="image" src="https://github.com/user-attachments/assets/070f227a-3264-4c49-9a68-5952b7537a45" />

## kubernetes Cluster (Container Orchestration )

![ss1](https://github.com/user-attachments/assets/67e308f2-abd8-439b-9179-fd4a85fa6633)


The diagram above illustrates a **basic Kubernetes cluster workflow**.

From the image you can observe:

* A **Kubernetes cluster** consists of a **Master node** and multiple **Worker nodes**.
* The **Master node** manages the cluster using components like authentication tokens, certificates, and cluster management logic.
* **Worker nodes** run the actual workloads.

### Pods

* Kubernetes does not run containers directly.
* Containers run inside an object called a **Pod**.

In the diagram:

**Pod1**

* Pod IP: `192.168.0.12`
* Contains one container (**nginx**)
* Container port: `80`

**Pod2**

* Pod IP: `192.168.0.13`
* Contains **multiple containers** (example: tomcat and myapp)
* Ports: `90` and `100`
* Containers inside the same pod communicate using **localhost**

### Pod Networking

Each pod receives a **unique IP address**.

Example:

* `192.168.0.12:80`
* `192.168.0.13:90`
* `192.168.0.13:100`

Kubernetes uses **Container Network Interface (CNI)** plugins such as:

* Calico
* Flannel
* Weave

Example network range in the diagram:

```
192.168.0.0/16
```

---

# Why Docker is NOT the Default Runtime in Kubernetes

Earlier versions of Kubernetes used **Docker as the container runtime**.

However, Kubernetes later introduced **CRI (Container Runtime Interface)** to support multiple runtimes.

Reasons Docker was removed as default:

1. Docker was not built specifically for Kubernetes.
2. Kubernetes only needs a **container runtime**, not the full Docker engine.
3. Docker added an extra layer called **Dockershim** which increased complexity.
4. Kubernetes removed Dockershim starting from **v1.24**.

Now Kubernetes directly supports runtimes like:

* **containerd**
* **CRI-O**

These are lighter and designed for Kubernetes.

---

# Can Kubernetes Use Other Container Runtimes?

Yes.

Kubernetes supports multiple container runtimes via **CRI**.

Examples:

* containerd (most common)
* CRI-O
* Docker (indirectly through containerd)
* Kata Containers
* gVisor

So Kubernetes is **runtime-agnostic**.

---

# Who Created Kubernetes?

Kubernetes was originally created by:

🏢 **Google**

It was inspired by Google's internal cluster management system called **Borg**.

In **2014**, Google open-sourced Kubernetes.

---

# Who Maintains Kubernetes Today?

Kubernetes is now maintained by:

🏛 **Cloud Native Computing Foundation (CNCF)**

CNCF is part of the **Linux Foundation**.

Major contributors include:

* Google
* Red Hat
* Microsoft
* Amazon
* IBM
* VMware
* Intel

---

# Kubernetes Certifications

Kubernetes certifications are also managed by **CNCF + Linux Foundation**.

### 1️⃣ CKA (Certified Kubernetes Administrator)

Focus:

* Cluster administration
* Installation
* Networking
* Storage
* Troubleshooting

Exam details:

* Questions: **15–20 performance-based tasks**
* Duration: **2 hours**
* Passing Score: **66%**
* Mode: **Hands-on lab**

---

### 2️⃣ CKAD (Certified Kubernetes Application Developer)

Focus:

* Application deployment
* Pods
* ConfigMaps
* Secrets
* Troubleshooting applications

Exam details:

* Questions: **15–20**
* Duration: **2 hours**
* Passing Score: **66%**
* Hands-on exam

---

### 3️⃣ CKS (Certified Kubernetes Security Specialist)

Focus:

* Kubernetes security
* Runtime security
* Cluster hardening
* Supply chain security

Requirements:

* Must pass **CKA first**

Exam details:

* Questions: **15–20**
* Duration: **2 hours**
* Passing Score: **67%**

---

# Managed Kubernetes Services (AKS, EKS, GKE)

Cloud providers offer **managed Kubernetes services** so you don't have to manage control planes manually.

### AKS

**Azure Kubernetes Service**

Provider: Microsoft Azure

Features:

* Managed control plane
* Integrated with Azure networking and security
* Auto scaling

---

### EKS

**Elastic Kubernetes Service**

Provider: AWS

Features:

* Fully managed Kubernetes
* Deep integration with AWS services
* IAM security integration

---

### GKE

**Google Kubernetes Engine**

Provider: Google Cloud

Features:

* Created by the Kubernetes creators
* Highly optimized Kubernetes service
* Autopilot mode available

---

# Simple Summary

| Topic             | Description       |
| ----------------- | ----------------- |
| Creator           | Google            |
| Maintained by     | CNCF              |
| Container Runtime | containerd, CRI-O |
| Default Runtime   | containerd        |
| Certifications    | CKA, CKAD, CKS    |
| Cloud Kubernetes  | AKS, EKS, GKE     |

---

If you want, I can also help you create a **very clean YouTube lecture script + slide explanation for this diagram (perfect for a DevOps course lecture)**.
