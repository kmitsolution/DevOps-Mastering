## 🔹 What is a Monolithic Application?

A **Monolithic application** is a single, tightly coupled software unit where:

* All functionalities are built
* Compiled
* Deployed
* Scaled

as **one single binary/package**.

---

## 🏦 Banking Application Example (Monolithic)

Imagine a traditional banking system with:

* 💰 Withdraw
* 💳 Deposit
* 📞 Customer Care
* 🌐 Internet Banking
* 👤 Account Management

All these modules are inside:

```
BankingApplication.jar (Single Binary)
```

### Internal Structure (Layered Architecture)

```
Presentation Layer (PL)
        ↓
Business Logic Layer (BAL)
        ↓
Data Access Layer (DAO)
        ↓
Database
```

Even though layers exist internally, the deployment is still **one single application**.

---

# 📈 Scaling in Monolithic Applications

During early **Waterfall era (1990s–2005)**:

* Applications were deployed on **Physical Servers**
* Hardware was expensive
* Cloud did not exist

So companies used **Vertical Scaling**

---

# 🔼 Vertical Scaling (Scale Up / Scale Down)

## 🔹 What is Vertical Scaling?

Increasing or decreasing the **hardware capacity of a single server**.

### Example:

* Increase RAM from 16GB → 64GB
* Increase CPU cores from 4 → 16
* Add more storage

---

## 🔺 Scale Up

Adding more resources to the same machine.

```
Before:
CPU: 4 cores
RAM: 8GB

After Scale Up:
CPU: 16 cores
RAM: 64GB
```

---

## 🔻 Scale Down

Reducing hardware resources of the same machine.

---

## ❌ Problems with Vertical Scaling

1. 🛑 **Application Shutdown Required**

   * Need downtime to upgrade hardware.

2. 💥 **Single Point of Failure**

   * If server crashes → entire banking system down.

3. 💰 Expensive Hardware

4. 🚫 Limited Maximum Capacity

5. ❌ Not elastic

---

# 🖥️ What is Virtualization?

![Image](https://www.researchgate.net/publication/280095977/figure/fig3/AS%3A667718931984388%401536208007132/rtualization-architecture.png)

![Image](https://www.researchgate.net/publication/335866538/figure/fig2/AS%3A882394324287494%401587390609903/Type-1-and-type-2-hypervisors.png)

![Image](https://cdn.diskinternals.com/media/en/products/raid-recovery/722-min.png)

![Image](https://cdn.diskinternals.com/media/thumbs/400w_media_en_products_raid-recovery_722-min.png.webp)

## 🔹 Definition

Virtualization is the process of creating **multiple virtual machines (VMs)** on a single physical server using a software layer called a **Hypervisor**.

---

## 🏗️ Basic Virtualization Architecture

```
Physical Server (CPU, RAM, Disk)
        ↓
Hypervisor
        ↓
---------------------------------
VM1   VM2   VM3   VM4
---------------------------------
Each VM has:
  - Guest OS
  - Kernel
  - Libraries
  - Application
```

---

## 🔹 Important Terms

### Host OS

Operating system installed on physical machine (in Type 2).

### Guest OS

OS installed inside a Virtual Machine.

In VM:

```
Step 1: OS Image Loaded
Step 2: Binaries Installed
Step 3: Application Runs
```

Each VM has its own:

* Kernel
* OS
* Libraries

---

# 🧱 Types of Virtualization

---

## 1️⃣ Type 1 Hypervisor (Bare Metal)

VMware ESXi
Microsoft Hyper-V

Installed directly on hardware.

```
Hardware
   ↓
Hypervisor (ESXi / Hyper-V)
   ↓
VM1  VM2  VM3
```

✅ High performance
✅ Used in data centers
✅ Enterprise production systems

---

## 2️⃣ Type 2 Hypervisor (Hosted)

Oracle VM VirtualBox
VMware Workstation

Runs on top of Host OS.

```
Hardware
   ↓
Host OS (Windows/Linux)
   ↓
Hypervisor (VirtualBox)
   ↓
VM1  VM2
```

✅ Used for development/testing
❌ Slightly slower than Type 1

---

# 🏗️ What Type of Applications Were Supported?

Mostly **Layered Monolithic Applications**

```
Presentation Layer (PL)
Business Logic Layer (BAL)
Data Access Layer (DAO)
```

Example:

* Java EE apps
* .NET apps
* Enterprise banking systems

Virtualization allowed:

* Running multiple applications
* Running multiple environments (Dev/Test/Prod)
* Better hardware utilization

---

# ➡️ Horizontal Scaling (Scale Out / Scale In)

Now instead of increasing one server’s power, we increase the number of servers.

---

## 🔹 Scale Out

Adding more VM instances.

```
Before:
1 VM running Banking App

After Scale Out:
VM1 → Banking App
VM2 → Banking App
VM3 → Banking App
```

Load balancer distributes traffic.

---

## 🔹 Scale In

Reducing number of VMs.

```
From:
VM1, VM2, VM3

To:
VM1 only
```

---

# 🔄 Horizontal Scaling with Virtualization

Because virtualization allows multiple VMs on one physical server:

```
Physical Server
   ↓
Hypervisor
   ↓
VM1  VM2  VM3  VM4
```

If load increases:

* Create more VMs
* Deploy same banking binary
* Attach to Load Balancer

---

# 🆚 Vertical vs Horizontal Scaling

| Feature        | Vertical Scaling   | Horizontal Scaling    |
| -------------- | ------------------ | --------------------- |
| Method         | Increase hardware  | Increase instances    |
| Downtime       | Usually Yes        | No (mostly)           |
| Cost           | Expensive hardware | Commodity servers     |
| Failure Impact | Full system down   | Partial impact        |
| Limit          | Hardware limit     | Practically unlimited |

---

# 🧠 Evolution Summary

### 1️⃣ Waterfall Era

* Monolithic apps
* Physical servers
* Vertical scaling

### 2️⃣ Virtualization Era

* Multiple VMs per server
* Better utilization
* Horizontal scaling possible

### 3️⃣ Cloud & DevOps Era

* Containers
* Microservices
* Auto-scaling
* Elastic infrastructure

---

# 🎯 Final Big Picture

* **Monolith** → Single binary
* **Vertical Scaling** → Scale up/down hardware
* **Virtualization** → Multiple VMs on one server
* **Horizontal Scaling** → Scale out/in instances
* **Guest OS** → OS inside VM
* **Hypervisor** → Creates VMs

---
