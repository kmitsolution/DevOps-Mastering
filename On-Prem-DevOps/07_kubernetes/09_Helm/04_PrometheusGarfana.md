##  Install Helm (Kubernetes Package Manager)

---

# ❓ What is Helm?

👉 **Helm** is a package manager for Kubernetes
👉 It helps you install applications using **Helm charts**

Example:

* Prometheus
* Grafana
* NGINX

---

# 🧠 Why Helm?

* Simplifies deployments
* Reusable configurations
* Easy upgrades/rollbacks

---

# 🛠️ Installation Methods

---

# ✅ Method 1: Using Script (Recommended)

👉 Works on most Linux systems

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## ✔️ Verify Installation

```bash
helm version
```

👉 Output should show version info

---

# ✅ Method 2: Manual Download

---

## Step 1: Download Helm

```bash
curl -LO https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz
```

---

## Step 2: Extract

```bash
tar -xvf helm-v3.14.0-linux-amd64.tar.gz
```

---

## Step 3: Move Binary

```bash
sudo mv linux-amd64/helm /usr/local/bin/helm
```

---

## Step 4: Verify

```bash
helm version
```

---

# ✅ Method 3: Using Package Manager (Ubuntu)

```bash
sudo apt update
sudo apt install helm -y
```

👉 (May install older version)

---

# ⚙️ Basic Setup After Installation

---

## Add Helm Repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

---

## Update Repos

```bash
helm repo update
```

---

## Search Charts

```bash
helm search repo prometheus
```

---

# 🚀 Test Helm Installation

Install a sample app:

```bash
helm install my-release prometheus-community/prometheus
```

---

## Check

```bash
helm list
```

---

# 🧠 Important Concepts

| Term       | Meaning                     |
| ---------- | --------------------------- |
| Chart      | Package (like app template) |
| Release    | Installed instance of chart |
| Repository | Collection of charts        |

---

# 🧾 Interview Answer

> “Helm is a package manager for Kubernetes that allows you to deploy applications using reusable templates called charts.”

---

