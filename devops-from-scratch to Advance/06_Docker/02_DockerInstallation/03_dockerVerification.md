# 🐳 How to Verify Docker Installation Deeply (Production-Level Checks)

Most people stop at:

```bash
docker run hello-world
```

But in **real DevOps / Production**, we verify multiple layers of Docker Engine:

* CLI
* Daemon (`dockerd`)
* containerd
* runc
* Kernel features (cgroups, namespaces)
* Storage driver
* Networking
* Logs

Let’s verify step-by-step 🔍

---

# ✅ 1️⃣ Verify Docker CLI

```bash
docker --version
```

Example output:

```text
Docker version 25.x.x, build xxxx
```

Also check detailed client/server info:

```bash
docker version
```

This shows:

```text
Client:
 Version:

Server:
 Engine:
 containerd:
 runc:
```

👉 Confirms CLI ↔ Daemon communication.

---

# ✅ 2️⃣ Verify Docker Daemon (dockerd)

Check service status:

```bash
sudo systemctl status docker
```

Expected:

```text
Active: active (running)
```

Check if daemon is listening:

```bash
ps -ef | grep dockerd
```

Or:

```bash
sudo ss -lntp | grep dockerd
```

---

# ✅ 3️⃣ Verify containerd Runtime

Docker internally uses:

containerd

Check:

```bash
containerd --version
```

Check service:

```bash
sudo systemctl status containerd
```

---

# ✅ 4️⃣ Verify runc (Low-Level Runtime)

Docker uses:

runc

Check:

```bash
runc --version
```

This confirms OCI runtime exists.

---

# ✅ 5️⃣ Deep System Information Check

Run:

```bash
docker info
```

This is VERY important.

It shows:

```text
Storage Driver:
Logging Driver:
Cgroup Driver:
Kernel Version:
Operating System:
CPUs:
Total Memory:
Docker Root Dir:
```

---

### 🔍 What to Verify Inside `docker info`

### ✔ Storage Driver

Example:

```text
Storage Driver: overlay2
```

Recommended:

* overlay2 (Linux)

---

### ✔ Cgroup Driver

```text
Cgroup Driver: systemd
```

Important for Kubernetes compatibility.

---

### ✔ Security Options

```text
Security Options:
 seccomp
```

Shows container security support.

---

# ✅ 6️⃣ Verify Namespaces Support (Kernel Level)

Check:

```bash
ls /proc/self/ns
```

Expected:

```text
mnt  net  pid  user  uts  ipc
```

This confirms namespace support.

---

# ✅ 7️⃣ Verify cgroups Support

Check:

```bash
ls /sys/fs/cgroup/
```

If directory exists → cgroups enabled.

Also check:

```bash
docker info | grep -i cgroup
```

---

# ✅ 8️⃣ Verify Image Pulling

Test Docker Hub connectivity:

```bash
docker pull nginx
```

(Images pulled from Docker Hub)

---

# ✅ 9️⃣ Verify Container Lifecycle

Run container:

```bash
docker run -d --name test nginx
```

Check:

```bash
docker ps
```

Stop:

```bash
docker stop test
```

Remove:

```bash
docker rm test
```

---

# ✅ 🔟 Verify Networking

List networks:

```bash
docker network ls
```

Inspect default bridge:

```bash
docker network inspect bridge
```

Check IP assignment:

```bash
docker inspect <container_id> | grep IPAddress
```

---

# ✅ 1️⃣1️⃣ Verify Volume Functionality

Create volume:

```bash
docker volume create testvol
```

List volumes:

```bash
docker volume ls
```

---

# ✅ 1️⃣2️⃣ Verify Logs

Check Docker daemon logs:

```bash
sudo journalctl -u docker
```

Or:

```bash
sudo journalctl -xe
```

---

# ✅ 1️⃣3️⃣ Verify Docker Socket

Docker communicates via UNIX socket:

```bash
ls -l /var/run/docker.sock
```

Should show:

```text
srw-rw---- 1 root docker
```

Check permissions.

---

# 🧪 Advanced Deep Validation

### Check Kernel Version

```bash
uname -r
```

Docker requires modern Linux kernel (>= 3.10 recommended).

---

### Check OverlayFS Support

```bash
lsmod | grep overlay
```

---

# 🏗️ Internal Architecture Verification

When you run:

```bash
docker run nginx
```

Internally:

```text
Docker CLI
   ↓
dockerd
   ↓
containerd
   ↓
runc
   ↓
Linux Kernel
```

You verified:

* CLI ✓
* Daemon ✓
* containerd ✓
* runc ✓
* Namespaces ✓
* cgroups ✓
* Networking ✓
* Storage ✓

Now Docker is fully validated.

---

# 🎯 Production-Ready Checklist

| Component  | Command                     | Status         |
| ---------- | --------------------------- | -------------- |
| CLI        | docker --version            | ✅              |
| Daemon     | systemctl status docker     | ✅              |
| containerd | systemctl status containerd | ✅              |
| Runtime    | runc --version              | ✅              |
| Storage    | docker info                 | overlay2       |
| Network    | docker network ls           | bridge present |
| Logs       | journalctl -u docker        | No errors      |

---

# 🧠 Interview-Level Answer

> To deeply verify Docker installation, I check Docker CLI, daemon status, containerd, runc, docker info output (storage driver, cgroup driver), kernel namespace and cgroup support, networking, volumes, and daemon logs. This ensures Docker is correctly installed and production-ready.

---
