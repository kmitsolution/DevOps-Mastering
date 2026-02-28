# 🐳 Docker `save`, `load`, `export`, `import` Commands

These commands are used for **backup, migration, and offline transfer** of Docker images and containers in Docker Engine.

They are often used when:

* No internet access (air-gapped servers)
* Moving images between environments
* Backup & restore
* CI/CD image transfer

---

# 🧱 1️⃣ `docker save`

## 📘 Definition

`docker save` exports a **Docker image** (including all its layers & metadata) into a tar file.

## 🔹 Syntax

```bash
docker save -o <filename>.tar <image_name>
```

## 🔹 Example

```bash
docker pull nginx
docker save -o nginx.tar nginx
```

Now `nginx.tar` contains:

* Image layers
* Tags
* Metadata

You can move this file to another server.

---

## 🔹 Save Multiple Images

```bash
docker save -o images.tar nginx ubuntu redis
```

---

# 📦 2️⃣ `docker load`

## 📘 Definition

`docker load` restores an image from a tar file created using `docker save`.

## 🔹 Syntax

```bash
docker load -i <filename>.tar
```

## 🔹 Example

```bash
docker load -i nginx.tar
```

Verify:

```bash
docker images
```

Image is restored with same tag.

---

# 🧠 save & load Summary

| Command | Works On | Includes                 |
| ------- | -------- | ------------------------ |
| save    | Image    | Layers + Metadata + Tags |
| load    | Image    | Restores complete image  |

---

# 🗂️ 3️⃣ `docker export`

## 📘 Definition

`docker export` exports a **running or stopped container’s filesystem** into a tar file.

⚠️ Important:

* It exports container filesystem only
* Does NOT include image metadata
* Does NOT preserve history/layers

---

## 🔹 Syntax

```bash
docker export <container_name> -o <file>.tar
```

---

## 🔹 Example

```bash
docker run -it --name c1 ubuntu
```

Inside container:

```bash
touch file1.txt
exit
```

Now export:

```bash
docker export c1 -o container.tar
```

This tar contains:

* Entire container filesystem
* All changes made inside container

---

# 📥 4️⃣ `docker import`

## 📘 Definition

`docker import` creates a new image from a tar file exported using `docker export`.

## 🔹 Syntax

```bash
docker import <file>.tar <new_image_name>
```

## 🔹 Example

```bash
docker import container.tar myimage:v1
```

Check:

```bash
docker images
```

New image created.

---

# ⚠️ Difference Between save/load and export/import

| Feature                | save/load | export/import |
| ---------------------- | --------- | ------------- |
| Works on               | Images    | Containers    |
| Preserves Layers       | ✅ Yes     | ❌ No          |
| Preserves History      | ✅ Yes     | ❌ No          |
| Preserves Metadata     | ✅ Yes     | ❌ No          |
| Recommended for backup | ✅ Yes     | ❌ Usually No  |

---

# 🔬 Practical Example (Migration Scenario)

## Scenario: Move Image to Air-Gapped Server

### On Source Server:

```bash
docker save -o app.tar myapp:1.0
```

Transfer `app.tar` to target server.

---

### On Target Server:

```bash
docker load -i app.tar
docker run myapp:1.0
```

---

# 🧠 Internal View

## save/load Flow

```text
Image Layers + Metadata
         ↓
docker save
         ↓
image.tar
         ↓
docker load
         ↓
Restored Image
```

---

## export/import Flow

```text
Running Container
         ↓
docker export
         ↓
container filesystem.tar
         ↓
docker import
         ↓
New Single-Layer Image
```

Notice:

Imported image becomes **single layer image**.

---

# 📦 Check Image History

After import:

```bash
docker history myimage:v1
```

You will see:

* Only one layer
* No build history

---

# 🎯 When to Use What?

### Use `docker save/load` when:

* Backing up images
* Moving images between servers
* Maintaining exact image version

### Use `docker export/import` when:

* Capturing container state snapshot
* Creating quick base image from running container

---

# 🚀 Example Full Workflow

```bash
# Build image
docker build -t myapp:1.0 .

# Save image
docker save -o myapp.tar myapp:1.0

# Load image
docker load -i myapp.tar

# Run container
docker run -d myapp:1.0
```

---

# 🧠 Interview-Level Answer

> `docker save` and `docker load` are used to export and import images including all layers and metadata. `docker export` and `docker import` work on containers and export only filesystem content without preserving layers or history. For production image migration, save/load is recommended.

---

