# 🐳 Docker Image `tag` and `commit` Commands

In Docker, images are central to container creation. Here we’ll deeply understand:

* `docker image tag`
* `docker commit`

Both are part of image management in Docker Engine and are commonly used with Docker Hub.

---

# 🏷️ 1️⃣ Docker Image Tag Command

## 🔹 What is Tag?

A **tag** is simply a label (version name) given to a Docker image.

Format:

```text
repository:tag
```

Example:

```text
nginx:latest
ubuntu:22.04
myapp:1.0
```

If no tag is mentioned → Docker assumes `latest`.

---

## 🔹 Syntax

```bash
docker tag SOURCE_IMAGE TARGET_IMAGE
```

---

## 🔹 Example 1: Tag Local Image

Check images:

```bash
docker images
```

Suppose we have:

```text
REPOSITORY   TAG     IMAGE ID
nginx        latest  abc123
```

Now tag it:

```bash
docker tag nginx:latest mynginx:v1
```

Verify:

```bash
docker images
```

Output:

```text
REPOSITORY   TAG     IMAGE ID
nginx        latest  abc123
mynginx      v1      abc123
```

👉 Same IMAGE ID
👉 Two different names
👉 No duplicate storage (same layers)

---

## 🔹 Example 2: Tag Before Pushing to Docker Hub

To push image to Docker Hub:

```bash
docker tag myapp username/myapp:1.0
```

Then:

```bash
docker push username/myapp:1.0
```

---

## 🔹 What Happens Internally?

`docker tag` does NOT create new image.

It just creates another reference to same image layers.

```text
Image Layers (shared)
        ↑
Multiple tags pointing to same image ID
```

---

# 📸 2️⃣ Docker Commit Command

## 🔹 What is Docker Commit?

`docker commit` creates a **new image from a running container**.

It captures container’s writable layer and saves it as a new image.

---

## 🔹 Syntax

```bash
docker commit CONTAINER_NAME NEW_IMAGE_NAME
```

---

## 🔹 Example: Commit a Modified Container

Step 1: Run container

```bash
docker run -it --name c1 ubuntu
```

Step 2: Install software inside container

```bash
apt update
apt install nginx -y
```

Step 3: Exit container

```bash
exit
```

Step 4: Commit container to new image

```bash
docker commit c1 ubuntu-nginx:v1
```

Step 5: Verify image

```bash
docker images
```

You will see:

```text
REPOSITORY      TAG    IMAGE ID
ubuntu-nginx    v1     xyz456
```

---

# 🧱 What Happens Internally?

Container structure:

```text
Writable Layer (Container Changes)
-----------------------------------
Base Image Layers (ubuntu)
```

When you commit:

```text
New Image =
Base Image Layers +
Container Writable Layer
```

Docker converts writable layer into new read-only image layer.

---

# 📌 Difference Between Tag and Commit

| Feature               | docker tag | docker commit     |
| --------------------- | ---------- | ----------------- |
| Creates new layers?   | ❌ No       | ✅ Yes             |
| Creates new image ID? | ❌ No       | ✅ Yes             |
| Used for versioning?  | ✅ Yes      | ❌ Not ideal       |
| Used in production?   | ✅ Yes      | ❌ Not recommended |
| Recommended approach  | Tagging    | Use Dockerfile    |

---

# ⚠️ Why `docker commit` is Not Recommended in Production

Problems:

* Not reproducible
* No Dockerfile history
* Hard to track changes
* Bad DevOps practice

Best practice:

```text
Always use Dockerfile to build images
```

---

# 🧠 Real Example (Bad vs Good)

## ❌ Bad Practice

```bash
docker run ubuntu
apt install nginx
docker commit
```

No documentation of what was installed.

---

## ✅ Good Practice

Dockerfile:

```dockerfile
FROM ubuntu
RUN apt update && apt install -y nginx
```

Build:

```bash
docker build -t ubuntu-nginx:v1 .
```

Reproducible and version-controlled.

---

# 🔍 Inspect Image After Commit

```bash
docker history ubuntu-nginx:v1
```

You will see new layer created by commit.

---

# 🗑️ Remove Image Tag

Remove only tag reference:

```bash
docker rmi mynginx:v1
```

If multiple tags point to same image → image removed only when last tag removed.

---

# 🧠 Interview-Level Answer

> `docker tag` is used to create another reference to an existing image without creating new layers. It is commonly used before pushing images to a registry. `docker commit` creates a new image from a container’s writable layer, but it is not recommended for production because it is not reproducible. Instead, Dockerfile should be used.

---

# 📌 Final Visualization

```text
docker tag:
Image ID abc123
  ├── nginx:latest
  └── mynginx:v1

docker commit:
Base Image Layers
        +
Writable Layer
        =
New Image (new Image ID)
```

---

