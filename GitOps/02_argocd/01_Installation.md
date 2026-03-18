# 📘 Documentation: Installing Helm and Argo CD on Kubernetes

---

#  Overview

This guide explains how to install:

* **Helm** → Kubernetes package manager
* **Argo CD** → GitOps continuous delivery tool

on a Kubernetes cluster, along with explanations of each step.

---

# 🧩 Part 1: Installing Helm

## 🔹 What is Helm?

Helm is a package manager for Kubernetes that allows you to install applications using **charts** (pre-configured templates).

---

## 🔹 Step 1: Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### ✅ What this does:

* Downloads Helm installation script
* Installs Helm v3 binary on your system

---

## 🔹 Step 2: Verify Installation

```bash
helm version
```

### ✅ Expected:

Displays Helm version → confirms installation success

---

## 🔹 Step 3: Install Git (Optional but useful)

```bash
apt install git -y
```

### ✅ Why:

* Needed for cloning repositories
* Useful for GitOps workflows (Argo CD)

---

# 📦 Part 2: Installing Argo CD using Helm

---

## 🔹 What is Argo CD?

Argo CD is a GitOps tool that:

* Pulls Kubernetes manifests from Git
* Automatically syncs them with your cluster

---

## 🔹 Step 1: Add Helm Repository

❌ Incorrect command (you tried):

```bash
helm repo add argo https:/argoproj.github.io/argo-helm
```

✔ Correct command:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
```

---

## 🔹 Step 2: Update Helm Repositories

```bash
helm repo update
```

### ✅ What this does:

* Fetches latest chart versions
* Makes charts searchable/installable

---

## 🔹 Step 3: Search Available Versions

```bash
helm search repo argo/argo-cd --versions
```

### ✅ Why:

* Lists all available versions of Argo CD chart
* Helps choose stable version (e.g., `9.4.13`)

---

## 🔹 Step 4: Create Namespace

```bash
kubectl create ns argocd
```

### ✅ Why:

* Keeps Argo CD resources isolated
* Best practice for Kubernetes apps

---

## 🔹 Step 5: Install Argo CD via Helm

```bash
helm upgrade argocd argo/argo-cd \
  --version 9.4.13 \
  --install \
  -n argocd
```

---

### 🔍 Explanation of Command

| Part               | Meaning                             |
| ------------------ | ----------------------------------- |
| `helm upgrade`     | Upgrades if exists, installs if not |
| `argocd`           | Release name                        |
| `argo/argo-cd`     | Chart name                          |
| `--version 9.4.13` | Specific chart version              |
| `--install`        | Install if not already present      |
| `-n argocd`        | Target namespace                    |

---

## 🔹 Step 6: Verify Installation

```bash
kubectl get po -n argocd
```

### ✅ Expected:

All pods should be:

```
Running
```

---

# 🔄 Additional Commands You Used (Explained)

---

## 🔹 List Helm Releases

```bash
helm list
```

### ✅ Shows:

* Installed applications via Helm
* Release names and status

---

## 🔹 Uninstall Release

```bash
helm uninstall my-argo-cd
```

### ✅ Removes:

* All resources created by that Helm release

---

# 🌐 Accessing Argo CD UI

## 🔹 Port Forward

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:80 --address 0.0.0.0
```

Open:

```
https://localhost:8080
```

---

## 🔐 Get Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

---

## 🔑 Login

* Username: `admin`
* Password: (from above command)

---

# 🧠 Key Concepts

---

## 🔹 Helm Concepts

* **Chart** → Application template
* **Release** → Installed instance of chart
* **Repo** → Collection of charts

---

## 🔹 Argo CD Concepts

* **Application** → Git repo + manifests
* **Sync** → Apply Git → Kubernetes
* **Auto-sync** → Continuous deployment

---

# ⚠️ Common Mistakes (From Your Commands)

---

### ❌ Wrong repo URL

```bash
https:/argoproj.github.io/argo-helm
```

✔ Fix:

```bash
https://argoproj.github.io/argo-helm
```

---

### ❌ Installing without namespace

Always specify:

```bash
-n argocd
```

---

### ❌ Confusion between OCI and Helm repo

* OCI → `oci://...`
* Repo → `argo/argo-cd`

Use one method consistently.

---

# ✅ Final Workflow Summary

1. Install Helm
2. Add Argo repo
3. Update repo
4. Create namespace
5. Install Argo CD with Helm
6. Verify pods
7. Access UI

---

