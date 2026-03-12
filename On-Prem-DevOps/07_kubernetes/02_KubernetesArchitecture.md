## Kubernetes Architecture

<img width="800" height="450" alt="image" src="https://github.com/user-attachments/assets/b613e60f-6814-4ddf-a4b3-e6f1aea016df" />


The image above represents the **Kubernetes Architecture**, which is composed of two major parts:

1️⃣ **Control Plane (Master Node)**
2️⃣ **Worker Nodes**

A Kubernetes cluster works using a **central control plane that manages multiple worker nodes**, where the actual applications (containers inside pods) run.

---

# 1️⃣ Control Plane (Master Node)

The **Control Plane** is responsible for **managing the entire Kubernetes cluster**.
It makes decisions about scheduling, scaling, networking, and maintaining the desired state of applications.

Main components inside the Control Plane:

* API Server
* ETCD
* Scheduler
* Controller Manager

---

## API Server

The **API Server** is the **entry point of the Kubernetes cluster**.

All communication in Kubernetes happens through the API Server.

### Responsibilities

* Receives commands from users
* Validates requests
* Updates cluster state
* Communicates with other components

### Who communicates with API Server?

* kubectl
* CI/CD pipelines
* Kubernetes dashboard
* Controller Manager
* Scheduler
* Worker nodes

Example command:

```bash
kubectl get pods
```

This request goes to:

```
kubectl → API Server → etcd / Scheduler / Nodes
```

The API server acts as the **central communication hub**.

---

## ETCD

**ETCD** is a **distributed key-value database**.

It stores the **entire cluster state**.

### What is stored in ETCD?

* Pods
* Nodes
* ConfigMaps
* Secrets
* Services
* Deployments
* Cluster configuration

Example record stored:

```
Pod name
Namespace
IP address
Replica count
```

### Key characteristics

* Highly available
* Consistent storage
* Distributed
* Fast key-value lookup

If **ETCD is lost, the cluster state is lost**.

That is why **ETCD backups are very important**.

---

## Scheduler

The **Scheduler** decides **which worker node will run a pod**.

When a new pod is created:

```
User → API Server → Scheduler
```

The Scheduler checks:

* CPU availability
* Memory availability
* Node labels
* Node taints
* Resource requests

Then it selects the **best worker node**.

Example decision:

```
Pod A → Worker Node 2
```

The scheduler **only assigns nodes**, it does not start containers.

---

## Controller Manager

The **Controller Manager** ensures that the cluster always matches the **desired state**.

Example desired state:

```
3 replicas of nginx pod
```

If one pod crashes:

```
Desired = 3
Current = 2
```

Controller Manager will **create another pod automatically**.

### Controllers inside Controller Manager

Examples:

**Node Controller**

* Detects node failures

**Replication Controller**

* Ensures required number of pods are running

**Endpoint Controller**

* Updates service endpoints

**Service Account Controller**

* Manages service accounts

Controllers constantly monitor cluster state.

---

# 2️⃣ Worker Nodes

Worker nodes are the machines where **applications actually run**.

Each worker node contains:

* Kubelet
* Kube Proxy
* Container Runtime
* Pods

---

## Kubelet

The **Kubelet** is an **agent running on every worker node**.

It communicates with the **API Server**.

### Responsibilities

* Receives pod specifications
* Pulls container images
* Starts containers
* Monitors container health
* Reports node status

Workflow:

```
API Server → Kubelet → Container Runtime → Pod
```

Kubelet ensures the containers defined in **PodSpecs** are running.

---

## Container Runtime

The **Container Runtime** is responsible for **running containers**.

Examples:

* containerd
* CRI-O
* Docker (older versions)

### Responsibilities

* Pull container images
* Start containers
* Stop containers
* Manage container lifecycle

Example:

```
docker run nginx
```

In Kubernetes, the container runtime runs containers **inside pods**.

---

## Kube Proxy

**Kube Proxy** manages **network communication inside the cluster**.

It handles **service networking**.

### Responsibilities

* Maintains network rules
* Load balances traffic to pods
* Handles service IP routing

Example:

```
Service IP → Multiple Pods
```

Kube Proxy distributes traffic across pods.

Example:

```
Client Request
      ↓
Service IP
      ↓
Pod1 / Pod2 / Pod3
```

This provides **internal load balancing**.

---

## Pods

A **Pod** is the **smallest deployable unit in Kubernetes**.

Pods contain:

* One container
* OR multiple containers

Containers inside the same pod share:

* Network
* Storage
* localhost communication

Example pod:

```
Pod
 ├── Container 1 (nginx)
 └── Container 2 (log collector)
```

Both containers communicate using:

```
localhost
```

---

# Kubernetes Communication Flow

Basic request flow:

```
User
  ↓
kubectl
  ↓
API Server
  ↓
Scheduler selects node
  ↓
Kubelet receives instruction
  ↓
Container Runtime runs container
  ↓
Pod created
```

---

# Simple Cluster Workflow

1️⃣ User runs kubectl command
2️⃣ API Server receives request
3️⃣ Scheduler selects worker node
4️⃣ Kubelet creates the pod
5️⃣ Container runtime runs containers
6️⃣ Kube Proxy enables networking

---

# Quick Summary

| Component          | Role                        |
| ------------------ | --------------------------- |
| API Server         | Entry point of cluster      |
| ETCD               | Cluster state database      |
| Scheduler          | Assigns pods to nodes       |
| Controller Manager | Maintains desired state     |
| Kubelet            | Node agent                  |
| Container Runtime  | Runs containers             |
| Kube Proxy         | Networking & load balancing |
| Pod                | Smallest deployable unit    |

---

✅ This architecture enables Kubernetes to provide:

* High availability
* Auto healing
* Auto scaling
* Container orchestration
* Efficient resource utilization

---
