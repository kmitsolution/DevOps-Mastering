---

# 🔐 Kubernetes Secrets – Complete Guide + Lab

---

## 🧠 What are Secrets?

**Kubernetes Secrets** are used to store **sensitive data**, such as:

* Passwords
* API tokens
* SSH keys
* Docker registry credentials

👉 Safer than:

* Hardcoding in Pod YAML ❌
* Storing in images ❌

---

## 🔑 Key Features of Secrets

1. Store **confidential data**
2. Data is **base64 encoded** (not encrypted by default ⚠️)
3. Stored as **key-value pairs**
4. Can be used in Pods:

   * As **environment variables**
   * As **volumes (files)**
5. Max size: **1 MB**
6. Stored in **etcd database**
7. Can be created:

   * From **file**
   * From **literal**
   * From **YAML**

---

# 🧪 LAB 1: Secret from File + Volume Mount

---

## Step 1: Create Secret Files

```bash id="pjkrj9"
echo -n 'admin' > username.txt
echo -n 'pa$$w00rd' > password.txt
```

---

## Step 2: Create Secret

```bash id="s0w72q"
kubectl create secret generic nginx-secret-vol \
  --from-file=username.txt \
  --from-file=password.txt
```

---

## Verify

```bash id="nqyz2j"
kubectl get secrets
kubectl describe secret nginx-secret-vol
```

---

## Step 3: Use Secret in Pod (Volume)

---

### nginx-pod-secret-vol.yaml

```yaml id="k40ebf"
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-secret-vol
spec:
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: test-vol
      mountPath: "/etc/confidential"
      readOnly: true
  volumes:
  - name: test-vol
    secret:
      secretName: nginx-secret-vol
```

---

## Create Pod

```bash id="7g9c3t"
kubectl create -f nginx-pod-secret-vol.yaml
```

---

## Validate (Inside Pod)

```bash id="yixb3h"
kubectl exec -it nginx-pod-secret-vol -- /bin/sh
cd /etc/confidential
ls
cat username.txt
cat password.txt
```

---

## Validate (Outside Pod)

```bash id="pq3m21"
kubectl exec nginx-pod-secret-vol ls /etc/confidential
kubectl exec nginx-pod-secret-vol cat /etc/confidential/username.txt
```

---

# 🧪 LAB 2: Secret from YAML + Environment Variables

---

## Step 1: Encode Data

```bash id="mpoj7o"
echo -n 'admin' | base64
echo -n 'pa$$w00rd' | base64
```

---

## Step 2: Create Secret YAML

---

### redis-secret-env.yaml

```yaml id="l1y8yt"
apiVersion: v1
kind: Secret
metadata:
  name: redis-secret-env
type: Opaque
data:
  username: YWRtaW4=
  password: cGEkJHcwMHJk
```

---

## Apply

```bash id="n3t5qe"
kubectl create -f redis-secret-env.yaml
```

---

## Step 3: Use Secret in Pod (Env Variables)

---

### redis-pod-secret-env.yaml

```yaml id="g7sl0f"
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod-secret-env
spec:
  containers:
  - name: redis-container
    image: redis
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: redis-secret-env
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: redis-secret-env
          key: password
  restartPolicy: Never
```

---

## Create Pod

```bash id="x9z9h4"
kubectl create -f redis-pod-secret-env.yaml
```

---

## Validate

```bash id="k3ps4z"
kubectl exec redis-pod-secret-env env | grep SECRET
```

---

# 🔍 Decode Secret

```bash id="zq3h3v"
kubectl get secret redis-secret-env -o yaml
```

```bash id="3yk4j9"
echo 'cGEkJHcwMHJk' | base64 --decode
```

---

# 🧪 LAB 3: Docker Registry Secret

---

## Step 1: Create Secret

```bash id="9q9y2p"
kubectl create secret docker-registry pullsecret \
  --docker-username=<username> \
  --docker-password=<password>
```

---

## Step 2: Use in Pod

```yaml id="c3c1vf"
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  imagePullSecrets:
  - name: pullsecret
  containers:
  - name: pod1
    image: <your-private-image>
    command: ["sleep", "3600"]
```

---

## Create Pod

```bash id="p7s9rz"
kubectl create -f pod.yaml
```

---

# 🧪 LAB 4: Using ServiceAccount

---

## Add Secret to Default ServiceAccount

```bash id="b4qv8q"
kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "pullsecret"}]}'
```

---

👉 Now pods automatically use secret

---

# 🧠 How Secrets Work (Flow)

```text
Secret → Pod → (Volume / Env) → Application
```

---

# ⚠️ Important Notes

* Base64 ≠ Encryption (just encoding ⚠️)
* Use **RBAC + encryption at rest** for security
* Avoid exposing secrets in logs

---

# 🔄 Cleanup

```bash id="d2m8wr"
kubectl delete secrets nginx-secret-vol redis-secret-env
kubectl delete pods nginx-pod-secret-vol redis-pod-secret-env
```

---

# 🧾 Interview Answer

> “Secrets in Kubernetes are used to store sensitive information securely. They can be consumed as environment variables or mounted as volumes inside Pods.”

---

# 👍 Final Summary

* Secrets = sensitive data
* Stored in base64
* Used via:

  * Env variables
  * Volumes
* Supports:

  * File
  * Literal
  * YAML

---
