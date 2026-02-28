# 🐳 What is Docker?
## 🔹 Definition

Docker is a containerization platform that allows you to:

* Build container images
* Run containers
* Ship applications consistently across environments

It made container technology easy for developers and DevOps engineers.

---

# ⚙️ What is Docker Engine?

Docker Engine is the **core runtime component** of Docker.

It is responsible for:

* Creating containers
* Running containers
* Managing images
* Managing networks
* Managing volumes

You install Docker → You are installing **Docker Engine**.

---

# 🔧 What is dockerd?

`dockerd` is the **Docker Daemon**.

It is a background service that:

* Listens to Docker API requests
* Manages images
* Manages containers
* Talks to container runtimes (containerd → runc)

When you run:

```bash
docker run nginx
```

Flow is:

```text
Docker CLI → dockerd → containerd → runc → Linux Kernel
```

---

# 🏗️ Docker Architecture

Docker follows a **Client–Server Architecture**.

---

## 🔹 1️⃣ Docker Client

The command line tool:

```bash
docker build
docker run
docker pull
docker ps
```

It sends commands to `dockerd` via:

* REST API
* UNIX socket
* TCP socket

---

## 🔹 2️⃣ Docker Daemon (dockerd)

* Runs in background
* Manages containers
* Handles networking
* Manages volumes
* Talks to container runtime

---

## 🔹 3️⃣ containerd

containerd

* Manages container lifecycle
* Pulls images
* Handles storage

---

## 🔹 4️⃣ runc

runc

* Low-level runtime
* Uses Linux namespaces & cgroups
* Actually creates the container

---

## 🔹 5️⃣ Docker Registry

Example:

Docker Hub

Stores container images.

When you run:

```bash
docker pull nginx
```

Image is pulled from Docker Hub.

---

# 🧱 Full Docker Architecture Flow

```text
User
  ↓
Docker CLI
  ↓
Docker Daemon (dockerd)
  ↓
containerd
  ↓
runc
  ↓
Linux Kernel (Namespaces + cgroups)
```

---

# 📦 Important Docker Components

### 1️⃣ Images

* Read-only templates
* Built using Dockerfile
* Stored in Registry

### 2️⃣ Containers

* Running instance of image
* Lightweight
* Isolated

### 3️⃣ Volumes

* Persistent storage

### 4️⃣ Networks

* Bridge network
* Host network
* Overlay network

---

# 🧠 Internally What Happens When You Run a Container?

Example:

```bash
docker run -d nginx
```

Step-by-step:

1. CLI sends request to dockerd
2. dockerd checks image locally
3. If not found → pulls from Docker Hub
4. containerd prepares container
5. runc creates namespaces & cgroups
6. Kernel runs container process

Container starts in seconds 🚀

---

# 🎯 Why Docker Became Popular?

* Simple CLI
* Dockerfile concept
* Docker Hub ecosystem
* Works perfectly with microservices
* Integrates with Kubernetes

---

# 🔥 Quick Interview Answer

> Docker is a containerization platform. Docker Engine consists of Docker CLI, Docker Daemon (dockerd), containerd, and runc. It follows client-server architecture where the CLI communicates with dockerd, which manages container lifecycle using containerd and runc.

---

