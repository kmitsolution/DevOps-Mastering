## hostPath Volume in Kubernetes (Persistent Storage on node)

---

# What is hostPath?

A **hostPath volume** allows a Pod to **access a directory or file from the Node (host machine)**.

👉 Instead of storing data inside:

* Container ❌
* Pod (like emptyDir) ❌

➡️ Data is stored on the **Node’s filesystem** ✅

---

## Key Idea

```text
Pod → hostPath → Node filesystem
```

👉 The data physically exists on the **worker node**

---

# Why is it called Non-Persistent?

Even though data is stored on the Node:

* If Pod is deleted → data remains on node ✅
* BUT if Pod is scheduled to another node → data is NOT available ❌

👉 So it is **not reliable for production** → considered non-persistent in Kubernetes design

---

# Characteristics of hostPath

* Tied to a **specific node**
* Not portable across nodes
* Useful for:

  * Testing
  * Debugging
  * Accessing node files

---

# Case Study: hostPath Example

---

## Step 1: Create Pod with hostPath Volume

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
    hostPath:
      path: /data/myapp
      type: DirectoryOrCreate
```

---

## Explanation

* `mountPath: /myapp` → inside container
* `path: /data/myapp` → on node

👉 Kubernetes will:

* Create `/data/myapp` on the node (if not exists)
* Mount it inside container at `/myapp`

---

## Step 2: Apply Pod

```bash
kubectl apply -f pod.yaml
```

---

## Step 3: Create File

```bash
kubectl exec -it mypod -- /bin/bash
```

Inside container:

```bash
echo "Hello from hostPath" > /myapp/myapp.java
ls /myapp
```

👉 Output:

```
myapp.java
```

---

## Step 4: Verify on Node (Important Concept)

👉 On the Kubernetes node:

```bash
ls /data/myapp
```

👉 Output:

```
myapp.java
```

✔ Same file exists on Node filesystem

---

## Step 5: Restart Container (Image Change)

```bash
kubectl set image pod/mypod nginx=tomcat
```

---

## Step 6: Check Again

```bash
kubectl exec -it mypod -- ls /myapp
```

👉 Output:

```
myapp.java ✅
```

👉 Data persists because it is stored on Node

---

## Step 7: Delete Pod

```bash
kubectl delete pod mypod
```

---

## Step 8: Recreate Pod (same YAML)

```bash
kubectl apply -f pod.yaml
```

---

## Step 9: Check Again

```bash
kubectl exec -it mypod -- ls /myapp
```

👉 Output:

```
myapp.java ✅
```

👉 Data still exists because:

* It is stored on Node (not Pod)

---

## Important Problem

👉 What if Pod is scheduled to another node?

* New node does NOT have `/data/myapp`
* Data will be missing ❌

---

# hostPath Behavior Summary

```text
Container restart → data persists ✅  
Pod deletion → data persists ✅  
Pod moves to another node → data lost ❌
```

---

# When to Use hostPath?

### Good for:

* Local development (Minikube, kind)
* Debugging node files
* Testing

### Not recommended for:

* Production ❌
* Distributed systems ❌

---

# Comparison (emptyDir vs hostPath)

| Feature             | emptyDir | hostPath |
| ------------------- | -------- | -------- |
| Stored where        | Pod      | Node     |
| Survives restart    | ✅ Yes    | ✅ Yes    |
| Survives pod delete | ❌ No     | ✅ Yes    |
| Works across nodes  | ✅ Yes    | ❌ No     |

---

# Final Understanding

* **hostPath**
  → Uses Node storage
  → More durable than emptyDir
  → But not portable

👉 That’s why:

* Still considered **non-persistent in Kubernetes design**

---

