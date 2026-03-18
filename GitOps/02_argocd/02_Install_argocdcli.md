#  What is Argo CD CLI?

The CLI for Argo CD lets you:

* Login to Argo CD server
* Create/manage applications
* Sync deployments
* Automate GitOps workflows

---

# 🐧 Install Argo CD CLI (Linux – Ubuntu)

## 🔹 Step 1: Download Binary

```bash
curl -sSL -o argocd \
https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```

---

## 🔹 Step 2: Make it Executable

```bash
chmod +x argocd
```

---

## 🔹 Step 3: Move to PATH

```bash
sudo mv argocd /usr/local/bin/
```

---

## 🔹 Step 4: Verify Installation

```bash
argocd version
```

✅ You should see client version output

---

# 🔐 Login to Argo CD Server

## 🔹 Step 5: Login

If you used port-forward:

```bash
argocd login localhost:8080
```

---

## ⚠️ If you see certificate warning

Use:

```bash
argocd login localhost:8080 --insecure
```

---

## 🔑 Username & Password

* Username: `admin`
* Password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
-o jsonpath="{.data.password}" | base64 -d
```

---

# 🧪 Test CLI

```bash
argocd app list
argocd account list
argocd account update-password
```

---

# 🔄 Optional: Install via Package Manager (Alternative)

### Using Homebrew (Mac)

```bash
brew install argocd
```

---

### Manual Download (Specific Version)

```bash
VERSION=v2.11.0   # example
curl -sSL -o argocd \
https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64
```

---

# 🧠 Useful CLI Commands

### List apps

```bash
argocd app list
```

### Create app

```bash
argocd app create my-app \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

### Sync app

```bash
argocd app sync my-app
```

---

# ⚠️ Common Issues

### ❌ `command not found`

👉 PATH issue → ensure:

```bash
/usr/local/bin
```

---

### ❌ Cannot connect to server

👉 Check:

* Port-forward running
* Correct IP/port

---

# ✅ Quick Summary

1. Download CLI
2. Make executable
3. Move to `/usr/local/bin`
4. Login
5. Use commands

---
