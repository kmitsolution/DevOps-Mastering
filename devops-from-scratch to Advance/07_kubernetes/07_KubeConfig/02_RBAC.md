Here’s a **clean, structured RBAC lab documentation** based on your steps — perfect for notes, teaching, or sharing 👇

---

# 🔐 Kubernetes RBAC (Role-Based Access Control) – Hands-on Lab

---

## 🧠 What is RBAC?

**RBAC (Role-Based Access Control)** is used to **control access** to Kubernetes resources based on user roles.

👉 It defines:

* **Who (User/ServiceAccount)**
* Can do **What (verbs)**
* On **Which resources**
* In **Which namespace**

---

# 🎯 Lab Objective

* Create a namespace → `finance`
* Create a user → `john`
* Grant access to manage deployments, pods, replicasets
* Verify access using kubeconfig

---

# 🧩 Step 1: Create Namespace

```bash
kubectl create ns finance
```

---

# 🔑 Step 2: Create User (Certificates)

---

## Generate Private Key

```bash
openssl genrsa -out john.key 2048
```

---

## Create CSR (Certificate Signing Request)

```bash
openssl req -new -key john.key -out john.csr -subj "/CN=john/O=javadeveloper"
```

👉 CN = username (`john`)
👉 O = group (`javadeveloper`)

---

## Sign Certificate using Kubernetes CA

```bash
openssl x509 -req -in john.csr \
-CA /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key \
-CAcreateserial \
-out john.crt \
-days 500
```

---

# 🧾 Step 3: Create Role

👉 Role defines **permissions inside a namespace**

---

## role.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: finance
  name: deployment-manager
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

---

## Apply Role

```bash
kubectl create -f role.yaml
```

---

# 🔗 Step 4: Create RoleBinding

👉 Binds **user → role**

---

## rolebinding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployment-manager-binding
  namespace: finance
subjects:
- kind: User
  name: john
  apiGroup: ""
roleRef:
  kind: Role
  name: deployment-manager
  apiGroup: ""
```

---

## Apply RoleBinding

```bash
kubectl create -f rolebinding.yaml
```

---

# ⚙️ Step 5: Configure kubeconfig for User

---

## Add Credentials

```bash
kubectl config set-credentials john \
--client-certificate=/home/ubuntu/temp/john.crt \
--client-key=/home/ubuntu/temp/john.key
```

---

## Create Context

```bash
kubectl config set-context developer-context \
--cluster=kubernetes \
--namespace=finance \
--user=john
```

---

## Switch Context

```bash
kubectl config use-context developer-context
```

---

## (Optional) Remove Admin Context

```bash
kubectl config delete-context kubernetes-admin@kubernetes
```

---

# 🧪 Step 6: Test Access

---

## Check Pods in finance namespace

```bash
kubectl get pods -n finance
```

👉 Should work ✅

---

## Try Accessing Other Namespace

```bash
kubectl get pods -n default
```

👉 Should fail ❌ (no permission)

---

# 💻 Optional: Install kubectl (Client Setup)

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

```bash
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

---

## Verify

```bash
kubectl version --client
```

---

# 📁 kubeconfig Setup

---

## Create Directory

```bash
mkdir -p $HOME/.kube
```

---

## View Config

```bash
kubectl config view
```

---

# 🧠 How RBAC Works (Flow)

```text
User (john)
   ↓
RoleBinding
   ↓
Role (permissions)
   ↓
Namespace (finance)
```

---

# 🔥 Key Concepts

| Component   | Purpose              |
| ----------- | -------------------- |
| Role        | Defines permissions  |
| RoleBinding | Assigns role to user |
| User        | Identity (john)      |
| Namespace   | Scope of access      |

---

# 🎯 Final Result

* User `john` can:

  * Create pods ✅
  * Manage deployments ✅
* But only in:

  * `finance` namespace ✅

---

# 🧾 Interview Answer

> “RBAC in Kubernetes controls access using Roles and RoleBindings. Roles define permissions, and RoleBindings assign those permissions to users within a namespace.”

---

# 👍 Summary

* RBAC = security mechanism
* Role = what actions allowed
* RoleBinding = who gets access
* Namespace = scope

---

If you want next, I can explain:
👉 **ClusterRole vs Role (very important interview question)** 🚀
