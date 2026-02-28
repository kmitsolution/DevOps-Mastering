# 🐳 Docker Installation Guide

( Ubuntu | CentOS | Windows | macOS )

We will install **Docker Engine** on Linux servers and **Docker Desktop** on Windows & macOS.

---

# 🐧 1️⃣ Docker Installation on Ubuntu

> Recommended for Ubuntu 20.04 / 22.04 / 24.04

# Simple installation
```
 apt update
 apt install docker.io -y
```

## OR

Follow below steps to install the latest version of docker.

## Step 1: Remove Old Versions (if any)

```bash
sudo apt remove docker docker-engine docker.io containerd runc -y
```

---

## Step 2: Update Packages

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg -y
```

---

## Step 3: Add Docker Official GPG Key

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

---

## Step 4: Add Docker Repository

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## Step 5: Install Docker

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

---

## Step 6: Verify Installation

```bash
sudo systemctl status docker
sudo docker run hello-world
```

---

## Optional: Run Docker Without sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

# 🐧 2️⃣ Docker Installation on CentOS / RHEL

> Works for CentOS 7/8 and RHEL

---

## Step 1: Remove Old Versions

```bash
sudo yum remove docker docker-client docker-common docker-engine -y
```

---

## Step 2: Install Required Packages

```bash
sudo yum install -y yum-utils
```

---

## Step 3: Add Docker Repository

```bash
sudo yum-config-manager \
--add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

---

## Step 4: Install Docker

```bash
sudo yum install docker-ce docker-ce-cli containerd.io -y
```

---

## Step 5: Start Docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## Step 6: Test

```bash
sudo docker run hello-world
```

---

# 🪟 3️⃣ Docker Installation on Windows

> For Windows 10/11 (64-bit) with WSL2 enabled

## Step 1: Enable WSL2

Open PowerShell as Admin:

```powershell
wsl --install
```

Restart system.

---

## Step 2: Download Docker Desktop

Install:

👉 Docker Desktop

From official website.

---

## Step 3: Configure WSL2 Backend

* Open Docker Desktop
* Enable WSL2 integration
* Start Docker

---

## Step 4: Verify

Open PowerShell:

```powershell
docker --version
docker run hello-world
```

---

# 🍎 4️⃣ Docker Installation on macOS

> Works on Intel & Apple Silicon

---

## Step 1: Download Docker Desktop

Install:

👉 Docker Desktop

---

## Step 2: Install & Launch

* Drag to Applications
* Open Docker
* Grant permissions

---

## Step 3: Verify

```bash
docker --version
docker run hello-world
```

---

# 🏗️ What Gets Installed?

## On Linux (Ubuntu/CentOS)

```text
Docker CLI
Docker Daemon (dockerd)
containerd
runc
```

Runs directly on Linux kernel.

---

## On Windows/macOS

Because they don’t use Linux kernel directly:

```text
Windows/macOS
    ↓
Lightweight VM (Linux)
    ↓
Docker Engine
```

Docker Desktop creates a small Linux VM internally.

---

# 📊 Linux vs Windows/macOS Difference

| Feature                    | Linux           | Windows/macOS   |
| -------------------------- | --------------- | --------------- |
| Native Linux Kernel        | ✅ Yes           | ❌ No            |
| VM Required                | ❌ No            | ✅ Yes           |
| Performance                | Slightly better | Slight overhead |
| Recommended for Production | ✅ Yes           | ❌ Mostly Dev    |

---

# 🚀 Production Recommendation

* ✅ Use Ubuntu / CentOS / RHEL for servers
* ✅ Use Docker Desktop for local development
* ❌ Avoid Windows/macOS for production container hosting

---

# 🎯 Quick Interview Answer

> On Linux, Docker Engine runs directly on the host kernel.
> On Windows and macOS, Docker Desktop runs Docker inside a lightweight Linux VM because containers require a Linux kernel.

---

