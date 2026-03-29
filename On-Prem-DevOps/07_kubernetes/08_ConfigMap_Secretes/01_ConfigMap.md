---

# ❓ What is a ConfigMap?

A **ConfigMap** is used to store **configuration data (key-value pairs)** in Kubernetes.

👉 It helps you:

* Separate config from application code
* Avoid hardcoding values
* Reuse configuration

---

## 🧠 Example

Instead of hardcoding:

```text
DB_HOST=localhost
```

👉 Store in ConfigMap and use in Pod

---

# 🛠️ Ways to Create ConfigMap

---

# ✅ 1. From Literal Values

```bash
kubectl create configmap my-config \
  --from-literal=env=dev \
  --from-literal=app=nginx
```

---

## 🔍 Check

```bash
kubectl get configmap
kubectl describe configmap my-config
```

---

# ✅ 2. From File

---

## Create a File

```bash
echo "db_host=localhost" > config.txt
echo "db_port=3306" >> config.txt
```

---

## Create ConfigMap

```bash
kubectl create configmap my-config-file --from-file=config.txt
```

---

👉 Key = filename
👉 Value = file content

---

# ✅ 3. From YAML (Recommended)

---

## configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config-yaml
data:
  DB_HOST: "localhost"
  DB_PORT: "3306"
  ENV: "dev"
```

---

## Apply

```bash
kubectl apply -f configmap.yaml
```

---

# 📥 How to Use ConfigMap in Pod

---

# 🔹 Method 1: Environment Variables

---

## pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-env-pod
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: my-config-yaml
          key: DB_HOST
```

---

👉 Inside container:

```text
DB_HOST=localhost
```

---

# 🔹 Method 2: Use Entire ConfigMap as Env

```yaml
envFrom:
- configMapRef:
    name: my-config-yaml
```

---

# 🔹 Method 3: Mount as Volume

---

## pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-volume-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: my-config-yaml
```

---

## Inside Container

```bash
ls /etc/config
```

👉 Output:

```text
DB_HOST
DB_PORT
ENV
```

---

## File Content Example

```bash
cat /etc/config/DB_HOST
```

👉 Output:

```text
localhost
```

---

# 🔄 Important Concept: Savepoint (Update Behavior)

---

## ❓ What Happens When ConfigMap Changes?

👉 Kubernetes **does NOT automatically restart Pods**

---

### Case 1: Env Variables

* ConfigMap updated ❌
* Pod still uses old values ❌

👉 Need restart:

```bash
kubectl delete pod <pod-name>
```

---

### Case 2: Volume Mount

* ConfigMap updated ✅
* File inside container updates automatically (after some time)

---

## 🧠 Key Point

```text
Env → static (needs restart)  
Volume → dynamic (auto update)
```

---

# 🔥 Real Example Flow

```text
ConfigMap → Pod → Application reads config
```

---

# ⚠️ Common Mistakes

* Wrong key name ❌
* ConfigMap not created ❌
* Namespace mismatch ❌

---

# 🧾 Interview Answer

> “ConfigMap is used to store configuration data in Kubernetes. It can be consumed as environment variables or mounted as files inside containers.”

---

# 👍 Final Summary

* Create ConfigMap:

  * Literal
  * File
  * YAML
* Use in Pod:

  * Env variables
  * Volume mount
* Updates:

  * Env → restart needed
  * Volume → auto update

---


