## Storage in Kubernetes 

In Kubernetes, storage is broadly divided into two types:

👉 **1. Non-Persistent Storage**
👉 **2. Persistent Storage**

This classification is based on whether data **survives Pod restarts** or not.

---

# 1. Non-Persistent Storage

### What is it?

Non-persistent storage is **temporary storage** that exists **only for the lifetime of a Pod**.

👉 If the Pod is:

* Deleted ❌
* Restarted ❌
* Rescheduled ❌

➡️ The data is **lost permanently**

---

### Key Characteristics

* Tied to **Pod lifecycle**
* Data is **ephemeral (temporary)**
* Fast and simple
* No external dependency

---

### Common Types

#### 1. emptyDir

* Created when a Pod starts
* Shared between containers in the same Pod
* Deleted when Pod is removed

👉 Use case:

* Temporary files
* Caching
* Scratch space

---

#### 2. container filesystem

* Each container has its own writable layer
* Data disappears when container restarts

👉 Use case:

* Logs (temporary)
* Intermediate processing

---

### Example Use Cases

* Cache storage
* Temporary computation data
* Buffer storage

---

### Simple Understanding

```
Pod starts → storage created  
Pod deleted → storage deleted ❌
```

---

# 2. Persistent Storage

### What is it?

Persistent storage is **long-term storage** that exists **independent of Pods**.

👉 Even if:

* Pod is deleted ✅
* Pod is recreated ✅

➡️ Data is **retained**

---

### Key Characteristics

* Independent of Pod lifecycle
* Data durability
* Can be reused by multiple Pods
* Backed by real storage systems

---

## Core Components of Persistent Storage

### 1. PersistentVolume (PV)

* Represents actual storage in the cluster
* Created by:

  * Admin (static)
  * Or dynamically (via StorageClass)

👉 Example sources:

* AWS EBS
* NFS
* Azure Disk
* GCP Persistent Disk

---

### 2. PersistentVolumeClaim (PVC)

* A request for storage by a user (Pod)
* Defines:

  * Size
  * Access mode

👉 Kubernetes binds:

```
PVC → PV
```

---

### 3. StorageClass

* Defines **how storage is provisioned**
* Enables **dynamic provisioning**

---

## Access Modes (Important Concept)

* **ReadWriteOnce (RWO)**
  → Mounted by one node at a time

* **ReadOnlyMany (ROX)**
  → Multiple nodes, read-only

* **ReadWriteMany (RWX)**
  → Multiple nodes, read & write

---

## How It Works

```
Pod → PVC → PV → Actual Storage (EBS/NFS/etc.)
```

---

## Example Use Cases

* Databases (MySQL, MongoDB)
* File storage
* Application state
* Logs (long-term)

---

## Simple Understanding

```
Pod deleted → data still exists ✅  
New Pod → reuses same data ✅
```

---

# Key Differences

| Feature         | Non-Persistent Storage | Persistent Storage       |
| --------------- | ---------------------- | ------------------------ |
| Data lifetime   | Pod lifecycle          | Independent of Pod       |
| Data durability | ❌ No                   | ✅ Yes                    |
| Use case        | Temp data              | Databases, stateful apps |
| Complexity      | Simple                 | More structured          |
| Components      | emptyDir, container FS | PV, PVC, StorageClass    |

---

## Final Summary

* **Non-Persistent Storage**

  * Temporary
  * Deleted with Pod
  * Used for caching, temp data

* **Persistent Storage**

  * Permanent
  * Survives Pod restarts
  * Used for databases and important data

---
