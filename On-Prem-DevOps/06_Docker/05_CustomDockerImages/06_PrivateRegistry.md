# 🐳 Docker Private Registry – Complete Step-by-Step Guide

A **Private Docker Registry** allows you to store Docker images inside your organization instead of pushing to public registries like Docker Hub.

Docker provides an official registry image:

👉 Docker Registry

---

# 🏗️ 1️⃣ Create a Local Private Registry (Single System)

We will run registry on:

```
localhost:5000
```

---

## 🔹 Step 1: Pull Registry Image

```bash
docker pull registry:2
```

---

## 🔹 Step 2: Run Registry Container

```bash
docker run -d \
  -p 5000:5000 \
  --name myregistry \
  registry:2
```

Now registry is running.

---

## 🔹 Step 3: Verify Registry

Open browser or curl:

```bash
curl http://localhost:5000/v2/_catalog
```

Output:

```json
{"repositories":[]}
```

Registry is working ✅

---

# 📦 Push Image to Local Registry

## Step 1: Tag Image

```bash
docker tag nginx localhost:5000/nginx:v1
```

---

## Step 2: Push Image

```bash
docker push localhost:5000/nginx:v1
```

---

## Step 3: Verify

```bash
curl http://localhost:5000/v2/_catalog
```

Output:

```json
{"repositories":["nginx"]}
```

---

# ⚠️ Insecure Registry (HTTP)

By default Docker expects HTTPS.

If using HTTP registry (like localhost), configure insecure registry.

---

## 🔹 Configure Insecure Registry Using daemon.json

Edit:

```bash
sudo nano /etc/docker/daemon.json
```

Add:

```json
{
  "insecure-registries": ["localhost:5000"]
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

Verify:

```bash
docker info | grep -i insecure
```

---

# 🔐 2️⃣ Private Registry with SSL (HTTPS)

Production registry must use SSL.

---

## 🔹 Step 1: Create Self-Signed Certificate

```bash
mkdir certs
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
  -x509 -days 365 -out certs/domain.crt
```

Enter:

* Common Name → Your IP or domain

---

## 🔹 Step 2: Run Registry with SSL

```bash
docker run -d \
  -p 5000:5000 \
  --name secure-registry \
  -v $(pwd)/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```

Registry now runs on:

```
https://localhost:5000
```

---

## 🔹 Step 3: Add Certificate to Docker

Create directory:

```bash
sudo mkdir -p /etc/docker/certs.d/localhost:5000
```

Copy cert:

```bash
sudo cp certs/domain.crt /etc/docker/certs.d/localhost:5000/ca.crt
```

Restart Docker:

```bash
sudo systemctl restart docker
```

Now Docker trusts your registry.

---

# 🔐 3️⃣ Private Registry with Basic Authentication

We will secure registry using username/password.

---

## 🔹 Step 1: Install htpasswd Tool

Ubuntu:

```bash
sudo apt install apache2-utils -y
```

---

## 🔹 Step 2: Create Authentication File

```bash
mkdir auth
htpasswd -Bc auth/htpasswd admin
```

Enter password.

---

## 🔹 Step 3: Run Registry with Authentication

```bash
docker run -d \
  -p 5000:5000 \
  --name auth-registry \
  -v $(pwd)/auth:/auth \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2
```

---

## 🔹 Step 4: Login to Registry

```bash
docker login localhost:5000
```

Enter:

* Username: admin
* Password

---

## 🔹 Step 5: Push Image

```bash
docker tag nginx localhost:5000/nginx:v2
docker push localhost:5000/nginx:v2
```

Without login → push fails ❌
After login → push succeeds ✅

---

# 🧠 Complete Architecture

```text
Docker Client
      ↓
Docker Daemon
      ↓
Private Registry (localhost:5000)
      ↓
Image Storage
```

With SSL + Auth:

```text
Docker Client
      ↓
HTTPS + Basic Auth
      ↓
Secure Private Registry
```

---

# 📌 Registry Data Persistence (Important)

To persist images:

```bash
docker run -d \
  -p 5000:5000 \
  -v /opt/registry-data:/var/lib/registry \
  --name myregistry \
  registry:2
```

Images stored in:

```
/opt/registry-data
```

---

# 🆚 Private Registry vs Docker Hub

| Feature           | Docker Hub    | Private Registry |
| ----------------- | ------------- | ---------------- |
| Public Access     | Yes           | No               |
| Internet Required | Yes           | No               |
| Security Control  | Limited       | Full             |
| Best For          | Public images | Enterprise       |

---

# 🎯 Interview-Level Explanation

> A Docker private registry is a self-hosted image repository using the official registry:2 image. It can run locally on port 5000. For production, we configure HTTPS using SSL certificates and enable Basic Authentication using htpasswd. Insecure registries must be added in daemon.json.

---

