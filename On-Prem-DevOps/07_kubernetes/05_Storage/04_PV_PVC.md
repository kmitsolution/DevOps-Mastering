## Persistent Storage in Kubernetes (PV + PVC with Case Study)

Now let’s continue the same learning pattern and understand **Persistent Storage**, which solves the problems we saw with `emptyDir` and `hostPath`.

---

# What is Persistent Storage?

Persistent storage means:

👉 Data **survives**:

* Container restart ✅
* Pod deletion ✅
* Pod rescheduling to another node ✅

---

## Core Components

### 1. PersistentVolume (PV)

* Represents **actual storage**
* Created by admin or dynamically
* Backed by:

  * AWS EBS (in EKS)
  * NFS
  * Azure Disk
  * GCP Disk

---

### 2. PersistentVolumeClaim (PVC)

* Request for storage by user
* Defines:

  * Size
  * Access mode

---

### 3. Binding

```text
PVC → PV → Actual Storage
```

---
## Access Modes in Persistent Volumes (PV)

Access modes define **how a volume can be mounted by Pods** — basically:

👉 **Who can use the volume and in what way (read/write)**

---

# Types of Access Modes

Kubernetes supports mainly **3 access modes**:

---

## 1. ReadWriteOnce (RWO)

### Meaning

👉 Volume can be mounted as:

* **Read + Write**
* By **only one node at a time**

---

### Important Clarification

* Multiple Pods can use it ✅
* But **only if they are on the same node**

---

### Example Use Case

* Databases (MySQL, PostgreSQL)
* AWS EBS volumes (in EKS)

---

### Simple Understanding

```text
1 Node → Many Pods → Read/Write ✅  
Multiple Nodes → Not allowed ❌
```

---

## 2. ReadOnlyMany (ROX)

### Meaning

👉 Volume can be mounted as:

* **Read-only**
* By **multiple nodes**

---

### Key Points

* No Pod can write ❌
* All Pods can only read ✅

---

### Example Use Case

* Shared configuration files
* Static content (HTML, images)

---

### Simple Understanding

```text
Multiple Nodes → Read only ✅  
Write → Not allowed ❌
```

---

## 3. ReadWriteMany (RWX)

### Meaning

👉 Volume can be mounted as:

* **Read + Write**
* By **multiple nodes at the same time**

---

### Key Points

* Fully shared storage
* Multiple Pods across nodes can read/write

---

### Example Use Case

* Shared file systems
* NFS, EFS (AWS), GlusterFS

---

### Simple Understanding

```text
Multiple Nodes → Read/Write ✅
```

---


# How It Fits in PVC

When you create a PVC:

```yaml
accessModes:
  - ReadWriteOnce
```

👉 Kubernetes:

* Finds a PV that supports this mode
* Binds PVC to that PV

---

# Summary Table

| Access Mode | Nodes | Read | Write | Use Case       |
| ----------- | ----- | ---- | ----- | -------------- |
| RWO         | 1     | ✅    | ✅     | Databases      |
| ROX         | Many  | ✅    | ❌     | Static content |
| RWX         | Many  | ✅    | ✅     | Shared storage |

---

# Final Understanding

* **RWO** → One node, read/write
* **ROX** → Many nodes, read-only
* **RWX** → Many nodes, read/write

👉 Choosing the correct access mode depends on:

* Your application needs
* Your storage backend

---

# Case Study (Step-by-Step)

We’ll use the same idea:

* Create file `/myapp/myapp.java`
* Restart container
* Delete Pod
* Recreate Pod

👉 This time data should **persist fully**

---

## Step 1: Create PersistentVolume (PV)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/pv-storage
```

---

### Explanation

* 1Gi storage
* Stored on node (for demo)
* In real-world (EKS) → EBS is used

---

## Step 2: Create PersistentVolumeClaim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

---

👉 Kubernetes will:

* Find matching PV
* Bind PVC to PV

---

## Step 3: Create Pod using PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: myvolume
      mountPath: /myapp
  volumes:
  - name: myvolume
    persistentVolumeClaim:
      claimName: my-pvc
```

---

## Step 4: Apply All Resources

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f pod.yaml
```

---

## Step 5: Create File

```bash
kubectl exec -it mypod -- /bin/bash
```

Inside container:

```bash
echo "Persistent Data" > /myapp/myapp.java
ls /myapp
```

👉 Output:

```
myapp.java
```

---

## Step 6: Restart Container (Change Image)

```bash
kubectl set image pod/mypod nginx=tomcat
```

---

## Step 7: Check Again

```bash
kubectl exec -it mypod -- ls /myapp
```

👉 Output:

```
myapp.java ✅
```

---

## Step 8: Delete Pod

```bash
kubectl delete pod mypod
```

---

## Step 9: Recreate Pod

```bash
kubectl apply -f pod.yaml
```

---

## Step 10: Check Again

```bash
kubectl exec -it mypod -- ls /myapp
```

👉 Output:

```
myapp.java ✅
```

---

# Why Data Persists?

Because:

* Data is stored in **PersistentVolume**
* PV is independent of Pod lifecycle
* PVC ensures same storage is reused

---

# Behavior Summary

```text
Container restart → data persists ✅  
Pod deletion → data persists ✅  
Pod recreated → data persists ✅  
```

---

# Real-World (EKS Understanding)

In AWS EKS:

* PV is backed by **EBS volume**
* PVC triggers **dynamic provisioning**
* Storage moves with Pod across nodes

---

# Comparison (All Storage Types)

| Feature            | emptyDir | hostPath | Persistent Volume |
| ------------------ | -------- | -------- | ----------------- |
| Container restart  | ✅ Yes    | ✅ Yes    | ✅ Yes             |
| Pod deletion       | ❌ No     | ✅ Yes    | ✅ Yes             |
| Cross-node support | ✅ Yes    | ❌ No     | ✅ Yes             |
| Production ready   | ❌ No     | ❌ No     | ✅ Yes             |

---

# Final Understanding

* **emptyDir** → temporary (Pod-level)
* **hostPath** → node-level (not portable)
* **PV + PVC** → real persistent storage

👉 This is what you use for:

* Databases
* Stateful apps
* Production workloads

---
