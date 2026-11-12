# 🐳 What is Docker Hub?

![Image](https://www.docker.com/app/uploads/2022/05/docker-homepage.png)

![Image](https://docs.docker.com/get-started/docker-concepts/the-basics/images/create-hub-repository.webp)

![Image](https://i.sstatic.net/Wc7Uq.png)

![Image](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/img/oci-docker-push.png)

## 📘 Definition

Docker Hub is the **official public container image registry** provided by Docker.

It is used to:

* Store Docker images
* Share images publicly
* Maintain private repositories
* Version control images
* Distribute images globally

👉 Think of Docker Hub like **GitHub for Docker images**.

---

# 🧱 Key Concepts in Docker Hub

## 1️⃣ Repository

A repository is a collection of image versions (tags).

Example:

```text
username/myapp
```

---

## 2️⃣ Image Tag

```text
username/myapp:1.0
username/myapp:latest
```

Each tag represents a version.

---

## 3️⃣ Public vs Private Images

| Type    | Access                |
| ------- | --------------------- |
| Public  | Anyone can pull       |
| Private | Only authorized users |

---

# 👤 How to Create Docker Hub Account

1️⃣ Go to Docker Hub website
2️⃣ Click **Sign Up**
3️⃣ Provide:

* Username
* Email
* Password
  4️⃣ Verify email

Your Docker Hub username becomes part of image name:

```text
username/myimage:tag
```

---

# 🔐 Login to Docker Hub from CLI

```bash
docker login
```

Enter:

* Username
* Password

If successful:

```text
Login Succeeded
```

---

# 🚀 How to Push Docker Image to Docker Hub

Let’s do complete workflow step-by-step.

---

## ✅ Step 1: Build Docker Image

Example Dockerfile:

```dockerfile
FROM nginx
COPY index.html /usr/share/nginx/html/
```

Build image:

```bash
docker build -t mynginx:1.0 .
```

Check:

```bash
docker images
```

---

## ✅ Step 2: Tag Image with Docker Hub Username

Important format:

```text
<username>/<repository>:<tag>
```

Example:

```bash
docker tag mynginx:1.0 username/mynginx:1.0
```

Now verify:

```bash
docker images
```

You’ll see:

```text
REPOSITORY              TAG
username/mynginx        1.0
```

---

## ✅ Step 3: Push Image

```bash
docker push username/mynginx:1.0
```

Docker will:

1. Authenticate
2. Upload layers
3. Skip layers already existing
4. Push metadata

Output example:

```text
Pushed
```

---

## ✅ Step 4: Verify on Docker Hub Website

Go to Docker Hub
You’ll see:

```text
username/mynginx
```

With tag `1.0`.

---

# 🔄 How Others Pull Your Image

Anyone can run:

```bash
docker pull username/mynginx:1.0
```

---

# 🧠 Complete Workflow Summary

```text
Create Dockerfile
        ↓
docker build
        ↓
docker tag username/repo:tag
        ↓
docker login
        ↓
docker push
        ↓
Available on Docker Hub
```

---

# 📦 Example: Full Real Demo

```bash
# Build image
docker build -t app:v1 .

# Tag image
docker tag app:v1 username/app:v1

# Login
docker login

# Push image
docker push username/app:v1
```

---

# 🧬 Important Notes

✔ Always tag properly before push
✔ Avoid pushing `latest` blindly in production
✔ Use versioning (1.0, 1.1, etc.)
✔ Use private repo for company apps

---

# 🔐 Push to Private Repository

When creating repository on Docker Hub, choose:

* Public
* Private

Push process remains same.

Only authorized users can pull private image.

---

# 🧠 Interview-Level Explanation

> Docker Hub is a cloud-based container image registry provided by Docker. It allows users to store, manage, and distribute Docker images. To push an image, we first build it, tag it using the Docker Hub username format, log in using docker login, and then push using docker push.

---

# 🔎 Common Errors

### ❌ Error: denied: requested access to the resource is denied

Reason:

* Image not tagged with correct username

Fix:

```bash
docker tag image username/image:tag
```

---

### ❌ Error: authentication required

Fix:

```bash
docker login
```

---

