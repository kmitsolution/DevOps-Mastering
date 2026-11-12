# 🐳 Docker Image Commands – Complete Practical Guide

Docker image management is done using the CLI of Docker Engine.
By default, images are pulled from Docker Hub.

---

# 📥 1️⃣ `docker pull` – Download an Image

## Pull Latest Version

```bash
docker pull nginx
```

If no tag is mentioned → Docker pulls:

```text
nginx:latest
```

---

## 📌 Pull with Specific Tag (Version)

```bash
docker pull nginx:1.25
```

Tag format:

```text
repository:tag
```

Example:

```bash
docker pull ubuntu:22.04
```

---

# 🔍 2️⃣ Search Docker Image

Search images on Docker Hub:

```bash
docker search redis
```

Output columns:

```text
NAME          DESCRIPTION          STARS     OFFICIAL
redis         Redis is an open...  15000     [OK]
```

* `[OK]` → Official image
* STARS → Popularity indicator

---

# 🖼️ 3️⃣ List Docker Images

```bash
docker images
```

Output:

```text
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    605c77e624dd   2 weeks ago   142MB
```

---

# 🔎 4️⃣ Show Full Image ID (No Truncate)

By default, Docker shortens image ID.

To see full SHA256:

```bash
docker images --no-trunc
```

Example output:

```text
sha256:605c77e624ddb75e4b3...
```

---

# 🔐 5️⃣ Docker Digest

Digest is a **content-based hash** of the image.

Pull image with digest:

```bash
docker pull nginx@sha256:<digest>
```

See digest:

```bash
docker images --digests
```

Output:

```text
REPOSITORY   TAG       DIGEST                                                                    IMAGE ID
nginx        latest    sha256:abc123xyz...                                                       605c77e624dd
```

👉 Digest ensures immutability (used in production).

---

# 🏷️ 6️⃣ Image Placeholders (Formatting Output)

Docker allows custom output format.

Example:

```bash
docker images --format "{{.Repository}}:{{.Tag}}"
```

Common placeholders:

| Placeholder   | Meaning      |
| ------------- | ------------ |
| `.Repository` | Image name   |
| `.Tag`        | Version tag  |
| `.ID`         | Image ID     |
| `.Digest`     | Image digest |
| `.Size`       | Image size   |

Example:

```bash
docker images --format "{{.ID}} {{.Size}}"
```

---

# 🗑️ 7️⃣ Dangling Images

Dangling images are:

* Untagged images
* Leftover layers
* Usually `<none>:<none>`

Example:

```text
<none>    <none>   3a5f8d2e3c1a
```

---

## Find Dangling Images

```bash
docker images -f "dangling=true"
```

---

## Remove Dangling Images

```bash
docker image prune
```

Remove all unused images:

```bash
docker image prune -a
```

---

# 🔎 8️⃣ Filter Docker Images

You can filter images using:

```bash
docker images --filter
```

---

## Filter by Dangling

```bash
docker images --filter "dangling=true"
```

---

## Filter by Reference

```bash
docker images --filter=reference="nginx*"
```

---

## Filter by Before Image

```bash
docker images --filter "before=nginx:latest"
```

---

## Filter by Since Image

```bash
docker images --filter "since=ubuntu:20.04"
```

---

# 🧪 9️⃣ Inspect Docker Image

To see detailed metadata:

```bash
docker inspect nginx
```

You’ll see:

* Layers
* Environment variables
* Entrypoint
* Working directory
* Architecture

---

# 📦 🔟 Remove Image

Remove by name:

```bash
docker rmi nginx
```

Remove by image ID:

```bash
docker rmi 605c77e624dd
```

Force remove:

```bash
docker rmi -f nginx
```

---

# 🏗️ Image Lifecycle Summary

```text
docker search → docker pull → docker images → docker run
                         ↓
                    docker inspect
                         ↓
                    docker rmi
```

---

# 🎯 Important Production Notes

* Use **specific tags**, not `latest`
* Use **digest** for immutable deployments
* Regularly clean dangling images
* Monitor disk usage:

```bash
docker system df
```

---

# 🧠 Interview-Ready Explanation

> Docker image commands include docker pull to download images, docker images to list them, --no-trunc to show full image ID, --digests to display image digest, docker search to find images, filtering using --filter, and docker image prune to remove dangling images. Digest ensures immutability, and dangling images are untagged unused layers.

