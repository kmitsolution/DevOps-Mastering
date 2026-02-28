# 🐳 Docker Container Overview (Deep Explanation with Examples)
---

# 🧱 1️⃣ What is a Docker Container?

A **Docker container** is a **running instance of a Docker image**.

👉 Image = Template
👉 Container = Running process created from that template

Example:

```bash
docker pull nginx
docker run -d nginx
```

* `nginx` image → read-only template
* `docker run` → creates container

---

# 🖼️ 2️⃣ Relationship Between Image and Container

Think like this:

```text
Image → Blueprint
Container → Running House built from blueprint
```

Technically:

```text
Docker Image (Read-only layers)
             +
Writable Layer
             =
Docker Container
```

When you run:

```bash
docker run ubuntu
```

Docker:

1. Takes image layers
2. Adds new writable layer
3. Starts container process

---

# ✍️ 3️⃣ Docker Container Writable Layer

When container runs, Docker adds a:

> **Thin writable layer on top of image layers**

Structure:

```text
Writable Layer (Container)
---------------------------
Image Layer 3
Image Layer 2
Image Layer 1
```

Image layers:

* Read-only
* Shared across containers

Writable layer:

* Unique per container
* Stores changes

---

## Example

Run container:

```bash
docker run -it ubuntu bash
```

Inside container:

```bash
touch file1.txt
```

Where is file stored?

👉 Not in image
👉 Stored in writable layer

---

# 🔄 4️⃣ What is Copy-On-Write (CoW)?

Copy-On-Write means:

> If a container modifies a file from image, Docker copies it to writable layer first.

---

## Example

Image contains:

```text
/etc/config.conf
```

Inside container:

```bash
echo "new config" >> /etc/config.conf
```

What happens internally?

1. Docker copies file from image layer
2. Places it into writable layer
3. Applies modification

Original image remains unchanged.

This is called:

> Copy-On-Write mechanism

---

# 💾 5️⃣ How Docker Container Size Is Created

When you run:

```bash
docker run -it ubuntu
```

Container size initially:

```bash
docker ps -s
```

Example output:

```text
CONTAINER ID   SIZE
abc123         0B (virtual 72MB)
```

Explanation:

* 72MB = Image size
* 0B = Writable layer size (no changes yet)

---

## After Creating Files

Inside container:

```bash
dd if=/dev/zero of=bigfile bs=1M count=100
```

Now check:

```bash
docker ps -s
```

Output:

```text
SIZE
100MB (virtual 172MB)
```

Explanation:

* 72MB → image size
* 100MB → writable layer size
* 172MB → virtual size (image + writable)

---

# 📊 6️⃣ Virtual Size vs Container Size

| Term          | Meaning                       |
| ------------- | ----------------------------- |
| Image Size    | Base image layers             |
| Writable Size | Data created inside container |
| Virtual Size  | Image + Writable size         |

Check disk usage:

```bash
docker system df
```

---

# 🧬 7️⃣ Multiple Containers Sharing Same Image

Example:

```bash
docker run nginx
docker run nginx
docker run nginx
```

All containers share:

```text
Same nginx image layers
```

But each has its own:

```text
Separate writable layer
```

Efficient storage usage.

---

# 🗑️ 8️⃣ What Happens When Container Is Removed?

If you remove container:

```bash
docker rm container_id
```

👉 Writable layer is deleted
👉 Image layers remain

---

# 📁 9️⃣ Where Writable Layer Stored?

On Linux (overlay2):

```bash
/var/lib/docker/overlay2/
```

Each container has:

* upperdir → writable changes
* lowerdir → image layers

---

# 🔥 Real Example Walkthrough

Step 1:

```bash
docker run -it ubuntu bash
```

Step 2:

```bash
apt update
apt install nginx
```

Step 3:

Check container size:

```bash
docker ps -s
```

You will see writable layer increased.

---

# ⚠️ Important Production Concept

Container writable layer:

* Slower than volumes
* Not meant for persistent data
* Deleted when container deleted

For persistent storage → use Docker Volumes.

---

# 🧠 Internal Flow

When container starts:

```text
Image Layers (Read-only)
         +
Writable Layer (CoW)
         +
Linux Kernel (Namespaces + cgroups)
         =
Running Container Process
```

---

# 📌 Key Concepts Summary

| Concept          | Meaning                         |
| ---------------- | ------------------------------- |
| Docker Image     | Read-only template              |
| Docker Container | Running instance                |
| Writable Layer   | Stores container changes        |
| Copy-On-Write    | File copied before modification |
| Virtual Size     | Image + Writable size           |
| Image Layers     | Shared across containers        |

---

# 🎯 Interview-Level Explanation

> A Docker container is a running instance of a Docker image. The image consists of read-only layers, and when a container starts, Docker adds a writable layer on top. Any modification inside the container is stored in the writable layer using a Copy-On-Write mechanism. The container size consists of image size plus writable layer size.

---

