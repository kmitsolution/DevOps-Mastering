# 🐳 Docker Installation Using Convenience Script (Linux)

Docker provides an **automated installation script** for quick setup of Docker Engine.

This method is mainly used for:

* Testing environments
* Labs
* Quick demos
* Learning purposes

⚠️ Not recommended for strict production environments (manual repo setup is preferred there).

---

# 🐧 Supported OS

The convenience script works on:

* Ubuntu
* Debian
* CentOS
* RHEL
* Fedora

---

# 🚀 Step-by-Step Installation

## Step 1️⃣ Download the Script

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```

---

## Step 2️⃣ Run the Script

```bash
sudo sh get-docker.sh
```

What this script does automatically:

* Detects your OS
* Adds Docker official repository
* Installs:

  * docker-ce
  * docker-ce-cli
  * containerd
* Starts Docker service

---

## Step 3️⃣ Verify Installation

```bash
docker --version
sudo docker run hello-world
```

Expected output:

```text
Hello from Docker!
```

---

# 🔍 What Happens Internally?

The script:

```text
1. Detect OS type
2. Add Docker GPG key
3. Add Docker repository
4. Install latest stable Docker Engine
5. Enable Docker service
```

You can inspect the script before running:

```bash
cat get-docker.sh
```

---

# 👤 Optional: Run Docker Without sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Then test:

```bash
docker run hello-world
```

---

# 🔄 Install Specific Docker Version (Optional)

After running the script, you can check available versions:

```bash
apt-cache madison docker-ce
```

Then install specific version:

```bash
sudo apt install docker-ce=<VERSION>
```

---

# 🗑️ Uninstall Docker (If Needed)

Ubuntu/Debian:

```bash
sudo apt remove docker-ce docker-ce-cli containerd.io
```

CentOS/RHEL:

```bash
sudo yum remove docker-ce docker-ce-cli containerd.io
```

---

# ⚠️ When NOT to Use Convenience Script

Avoid in:

* Enterprise production systems
* Air-gapped environments
* Strict compliance systems

Because:

* Less control over versioning
* Installs latest version automatically

---

# 🎯 Interview Explanation (Short Version)

> Docker provides a convenience script available at get.docker.com which automatically installs Docker Engine by detecting the OS, adding the repository, and installing required packages. It is mainly used for testing and quick setup, not recommended for controlled production environments.

---

