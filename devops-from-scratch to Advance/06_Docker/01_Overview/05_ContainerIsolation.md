# 🐳 How Container Isolation Works in Docker

## (Linux Namespaces & cgroups – Deep Explanation with Examples)

When you run a container using Docker Engine, Docker does **NOT** create a virtual machine.

Instead, it uses **Linux Kernel features**:

* **Namespaces** → Isolation
* **cgroups (Control Groups)** → Resource control

---

# 🧠 Big Picture

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
      ↙           ↘
  Namespaces     cgroups
 (Isolation)   (Resource Limits)
```

---

# 🔐 Part 1: Namespaces (Isolation)

Namespaces make a container think:

> "I am the only system running here."

They isolate system resources.

---

## 🔹 Types of Linux Namespaces Used by Docker

| Namespace | What It Isolates            |
| --------- | --------------------------- |
| PID       | Process IDs                 |
| NET       | Network interfaces          |
| MNT       | Mount points                |
| UTS       | Hostname                    |
| IPC       | Inter-process communication |
| USER      | User IDs                    |

---

## 1️⃣ PID Namespace (Process Isolation)

### Without Container

Run:

```bash
ps -ef
```

You see **all system processes**.

---

### Inside Docker Container

```bash
docker run -it ubuntu bash
ps -ef
```

Output:

```text
PID   CMD
1     bash
7     ps -ef
```

Inside container:

* Bash appears as PID 1
* Container cannot see host processes

👉 Each container has its own **PID namespace**.

---

## 2️⃣ Network Namespace (Network Isolation)

Each container gets:

* Its own IP
* Its own network interface
* Its own routing table

Example:

```bash
docker run -d nginx
docker inspect <container_id>
```

You’ll see:

```text
"IPAddress": "172.17.0.2"
```

Docker creates a virtual bridge:

```text
Host
  ↓
docker0 bridge
  ↓
Container1 (172.17.0.2)
Container2 (172.17.0.3)
```

Containers cannot directly see host network unless you use:

```bash
docker run --network host nginx
```

---

## 3️⃣ Mount Namespace (Filesystem Isolation)

Each container has its own filesystem layer.

Example:

```bash
docker run -it ubuntu bash
touch testfile.txt
```

Exit container.

Check on host:

```bash
ls
```

You will NOT see `testfile.txt`.

Because container has its own mount namespace.

---

## 4️⃣ UTS Namespace (Hostname Isolation)

Inside container:

```bash
hostname
```

Each container has different hostname.

---

## 5️⃣ IPC Namespace

Prevents containers from sharing:

* Shared memory
* Semaphores
* Message queues

---

# 📊 Summary of Namespaces

```text
Container A:
   PID namespace
   NET namespace
   MNT namespace

Container B:
   Completely separate namespaces
```

Isolation is achieved.

---

# 💻 Part 2: cgroups (Control Groups)

Namespaces isolate visibility.
cgroups limit resource usage.

Without cgroups:

A container could use 100% CPU and crash system.

---

## 🔹 What cgroups Control

* CPU usage
* Memory usage
* Disk I/O
* Network bandwidth

---

## 🧪 Example 1: Memory Limit

Run container with memory limit:

```bash
docker run -it --memory="200m" ubuntu
```

Now container can only use 200MB RAM.

If it tries more:

```text
Killed
```

Linux OOM Killer terminates it.

---

## 🧪 Example 2: CPU Limit

```bash
docker run -it --cpus="1.0" ubuntu
```

Container can only use 1 CPU core.

Even if host has 8 cores.

---

# 🏗️ How Docker Uses Both Together

Example:

```bash
docker run -d --memory=500m --cpus=1 nginx
```

What happens internally:

1. Docker CLI sends request
2. dockerd creates container spec
3. containerd prepares environment
4. runc:

   * Creates namespaces
   * Applies cgroup limits
5. Kernel runs isolated process

---

# 🔬 Deep Internal View

Linux stores cgroups here:

```bash
/sys/fs/cgroup/
```

Namespaces are managed by:

```bash
/proc/<pid>/ns/
```

Check namespace of process:

```bash
ls -l /proc/$$/ns
```

---

# 🆚 Namespaces vs cgroups

| Feature  | Namespaces             | cgroups                    |
| -------- | ---------------------- | -------------------------- |
| Purpose  | Isolation              | Resource control           |
| Controls | What container can see | How much container can use |
| Example  | Separate PID list      | Limit RAM to 200MB         |

---

# 🚀 Why This Is Powerful

Containers are:

* Lightweight (no guest OS)
* Fast startup
* Highly scalable
* Efficient

Because:

They reuse **same Linux kernel**
And isolate via namespaces + cgroups.

---

# 🧠 Real DevOps Interview Explanation

> Docker container isolation works using Linux namespaces and cgroups.
> Namespaces provide process, network, and filesystem isolation.
> cgroups provide resource limitation like CPU and memory control.
> Docker uses runc to create namespaces and apply cgroup limits via Linux kernel.

---

# 🎯 Final Visualization

```text
Linux Kernel
   ├── Namespaces (Isolation)
   │       ├── PID
   │       ├── NET
   │       └── MNT
   │
   └── cgroups (Resource Limits)
           ├── CPU
           ├── Memory
           └── IO
```

That is the core reason why containers are lightweight compared to Virtual Machines.

---
