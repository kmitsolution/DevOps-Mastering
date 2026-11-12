# 🐳 What Are Docker Images?


## 🔹 Definition

A **Docker Image** is a **read-only template** used to create containers.

It contains:

* Application code
* Runtime (Java, Python, Node, etc.)
* Libraries
* System tools
* Environment variables

When you run:

```bash
docker run nginx
```

Docker:

1. Pulls the image (if not present)
2. Creates a container from that image
3. Runs it

---

## 🧱 Image vs Container

| Docker Image       | Docker Container      |
| ------------------ | --------------------- |
| Read-only template | Running instance      |
| Like a class       | Like an object        |
| Stored in registry | Runs on Docker Engine |

---

# 🌍 What is Docker Hub?

Docker Hub is the **official public registry** provided by Docker.

It is like **GitHub for container images**.

Docker Hub allows you to:

* Store images
* Share images
* Pull images
* Manage public/private repositories

---

# 📥 How to Download Images from Docker Hub

## Step 1️⃣ Search Image

```bash
docker search nginx
```

## Step 2️⃣ Pull Image

```bash
docker pull nginx
```

Docker pulls image from Docker Hub by default.

Internally:

```text
Docker CLI
   ↓
dockerd
   ↓
Docker Hub
   ↓
Image downloaded to /var/lib/docker
```

---

## Step 3️⃣ Verify Downloaded Image

```bash
docker images
```

Output:

```text
REPOSITORY   TAG       IMAGE ID
nginx        latest    xxxxxxx
```

---

## 📌 Pull Specific Version

```bash
docker pull nginx:1.25
```

Tag = version.

---

# 🏷️ Official Images on Docker Hub

Official images are:

* Maintained by Docker or upstream maintainers
* High quality
* Regularly updated
* Security scanned

Example Official Images:

* nginx
* mysql
* redis
* ubuntu
* node
* python

They appear as:

```text
nginx
```

(No username prefix)

---

# ✅ Verified Publisher Images

Verified Publisher images are:

* Provided by trusted companies
* Official vendor-supported images
* Signed & verified

Example companies:

* Microsoft
* Red Hat
* Oracle
* IBM

Example:

```text
microsoft/mssql-server
```

These images have a **Verified Publisher badge** on Docker Hub.

---

# 👤 Creating Docker Hub Account

## Step 1️⃣ Go to Docker Hub website

Create account on Docker Hub.

## Step 2️⃣ Provide:

* Username
* Email
* Password

## Step 3️⃣ Verify Email

---

## Step 4️⃣ Login from CLI

```bash
docker login
```

Enter:

* Username
* Password

You’ll see:

```text
Login Succeeded
```

---

# 📤 Push Image to Docker Hub

After building image:

```bash
docker build -t myapp .
```

Tag image with username:

```bash
docker tag myapp username/myapp:1.0
```

Push:

```bash
docker push username/myapp:1.0
```

---

# 🌐 Public vs Private Images

## 🔓 Public Images

* Anyone can pull
* Visible to everyone
* Used for open-source or public apps

Example:

```bash
docker pull nginx
```

---

## 🔒 Private Images

* Only authorized users can pull
* Used for company applications
* Requires login

To pull private image:

```bash
docker login
docker pull username/privateapp
```

---

# 📦 Image Naming Convention

```text
<registry>/<username>/<repository>:<tag>
```

Example:

```text
docker.io/nginx:latest
docker.io/username/myapp:1.0
```

If registry not mentioned → defaults to Docker Hub.

---

# 🧠 Where Images Are Stored Locally?

On Linux:

```bash
/var/lib/docker/
```

Uses layered filesystem (overlay2 recommended).

---

# 🏗️ How Docker Images Work Internally

Images are built in **layers**.

Example Dockerfile:

```dockerfile
FROM ubuntu
RUN apt update
RUN apt install nginx
```

Each instruction creates a layer.

Benefits:

* Faster builds
* Layer caching
* Efficient storage

---

# 🔐 Security in Docker Hub

* Image vulnerability scanning
* Content trust (image signing)
* Access control for private repos

---

# 🎯 Interview-Ready Answer

> Docker images are read-only templates used to create containers. They are stored in registries like Docker Hub. Official images are maintained by Docker or upstream maintainers, while Verified Publisher images are provided by trusted companies. Docker Hub supports public and private repositories. Images can be pulled using `docker pull` and pushed using `docker push`.

---

