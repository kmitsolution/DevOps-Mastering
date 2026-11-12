# 🐳 Docker Storage – Types of Storage 

In Docker Engine, storage is divided into two main categories:

```
1️⃣ Non-Persistent Storage
2️⃣ Persistent Storage
```

Let’s understand both deeply with step-by-step practical examples.

---

# 🧱 1️⃣ Non-Persistent Storage

> Data is lost when the container is deleted.

This includes:

* Container Writable Layer
* tmpfs (memory-based storage)

---

## 🔹 1.1 Default Container Storage (Writable Layer)

When you run a container:

```bash
docker container run -it --name tmpcontainer -d ubuntu
```

Docker creates:

```
Read-only Image Layers
        +
Writable Container Layer
```

All new files go into writable layer.

---

### ✅ Practical Example (Non-Persistent)

### Step 1: Create Container

```bash
docker container run -it --name tmpcontainer -d ubuntu
```

### Step 2: Enter Container

```bash
docker exec -it tmpcontainer bash
```

### Step 3: Create Files

```bash
mkdir test
cd test
touch file1 file2 file3 file4
ls
exit
```

### Step 4: Stop Container

```bash
docker stop tmpcontainer
```

### Step 5: Start Again

```bash
docker start tmpcontainer
docker exec -it tmpcontainer bash
ls test
```

👉 Files still exist (because container still exists).

### Step 6: Delete Container

```bash
docker rm -f tmpcontainer
```

👉 Data is permanently lost ❌

Because writable layer was deleted.

---

# 🔹 1.2 tmpfs (Memory-Based Storage)

> Data stored in RAM only
> Data disappears when container stops
> Linux only feature

---

## Example: tmpfs Mount

### Step 1: Create Container with tmpfs

```bash
docker container run -it \
  --name tmpcontainer \
  --mount type=tmpfs,destination=/test \
  -d ubuntu
```

### Step 2: Enter Container

```bash
docker exec -it tmpcontainer bash
```

### Step 3: Create Files

```bash
cd /test
touch file1 file2 file3 file4
ls
exit
```

### Step 4: Stop Container

```bash
docker stop tmpcontainer
```

### Step 5: Start Container Again

```bash
docker start tmpcontainer
docker exec -it tmpcontainer bash
ls /test
```

👉 Files are gone ❌

Because tmpfs is memory-based.

---

## tmpfs with Permission Mode

```bash
docker container run -it \
  --name c1 \
  --mount type=tmpfs,destination=/tmp1,tmpfs-mode=1700 \
  -d ubuntu
```

`tmpfs-mode=1700` sets directory permissions.

---

# 🧱 2️⃣ Persistent Storage

> Data survives container deletion.

Includes:

* Docker Volume
* Bind Mount
* NFS Volume

---

# 🔹 2.1 Docker Volume

> Managed by Docker daemon
> Default location:

```
/var/lib/docker/volumes/
```

---

## 🔹 Volume Commands

```bash
docker volume --help
```

Important commands:

```
create
inspect
ls
prune
rm
```

---

## Example: Docker Volume (demo-vol)

---

### Step 0: Remove Unused Volumes

```bash
docker volume prune
```

---

### Step 1: Create Volume

```bash
docker volume create demo-vol
```

---

### Step 2: List Volumes

```bash
docker volume ls
```

---

### Step 3: Check Default Path

```bash
ls /var/lib/docker/volumes/demo-vol/_data
```

---

### Step 4: Attach Volume to Container

```bash
docker container run -it \
  --name c1 \
  --mount source=demo-vol,destination=/app \
  -d ubuntu
```

---

### Step 5: Enter Container

```bash
docker exec -it c1 bash
```

---

### Step 6: Create Files

```bash
cd /app
touch file1 file2 file3 file4
exit
```

---

### Step 7: Verify on Host

```bash
ls /var/lib/docker/volumes/demo-vol/_data
```

Files exist ✅

---

### Step 8: Modify from Host

```bash
rm /var/lib/docker/volumes/demo-vol/_data/file4
docker exec -it c1 bash
ls /app
```

file4 is removed inside container ✅

---

### Step 9: Add File Inside Container

```bash
docker exec -it c1 bash
cd /app
touch file5
exit
```

Check on host:

```bash
ls /var/lib/docker/volumes/demo-vol/_data
```

file5 exists ✅

---

### Step 10: Delete Container

```bash
docker rm -f c1
```

---

### Step 11: Check Volume Data

```bash
ls /var/lib/docker/volumes/demo-vol/_data
```

👉 Data still exists ✅

---

### Step 12: Attach Volume to New Container

```bash
docker container run -it \
  --name c2 \
  --mount source=demo-vol,destination=/demo \
  -d centos
```

---

### Step 13: Verify Data

```bash
docker exec -it c2 bash
ls /demo
```

All files exist ✅

---

### Step 14: Remove Volume

```bash
docker rm -f c2
docker volume rm demo-vol
```

Data deleted ❌

---

# 🔹 Alternative -v Option

```bash
docker container run -it --name c3 -v demo-vol:/demo -d centos
```

---

# 🔹 Volume Mounted to Root

```bash
docker container run -it --name c3 -v demo-vol:/root -d centos
```

All root directory files stored in volume.

---

# 🔹 2.2 Bind Mount

> Managed by host
> Direct mapping of host directory
> No Docker lifecycle management

---

## Example: Bind Mount

---

### Step 1: Create Host Directory

```bash
mkdir /home/vagrant/myfiles
```

---

### Step 2: Run Container with Bind Mount

Using -v:

```bash
docker container run -it \
  --name bindmountcontainer \
  -v /home/vagrant/myfiles:/app1 \
  -d ubuntu
```

Using --mount:

```bash
docker container run -it \
  --name bindmountcontainer \
  --mount type=bind,source=/home/vagrant/myfiles,target=/app1 \
  -d ubuntu
```

---

### Step 3: Create Files Inside Container

```bash
docker exec -it bindmountcontainer bash
cd /app1
touch file1 file2 file3 file4
exit
```

---

### Step 4: Verify on Host

```bash
ls /home/vagrant/myfiles
```

Files exist ✅

---

# 🔹 2.3 Docker Volume with NFS

Docker volumes can use remote storage:

* NFS
* CIFS
* Cloud storage

Example (conceptual):

```bash
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.10,rw \
  --opt device=:/path/to/nfs \
  nfs-volume
```

---

# 📊 Storage Comparison Table

| Type           | Persistent | Location        | Managed By | Best Use                  |
| -------------- | ---------- | --------------- | ---------- | ------------------------- |
| Writable Layer | ❌ No       | overlay2        | Docker     | Temporary                 |
| tmpfs          | ❌ No       | RAM             | Docker     | In-memory processing      |
| Volume         | ✅ Yes      | /var/lib/docker | Docker     | Databases                 |
| Bind Mount     | ✅ Yes      | Host directory  | Admin      | Development               |
| NFS Volume     | ✅ Yes      | Remote storage  | External   | Production shared storage |

---

# 🎯 Final Summary

```
Non-Persistent:
  - Writable Layer
  - tmpfs

Persistent:
  - Docker Volume
  - Bind Mount
  - NFS Volume
```

---

# 🧠 Interview-Level Explanation

> Docker provides non-persistent storage through the container writable layer and tmpfs, where data is lost when the container stops or is deleted. Persistent storage is achieved using Docker volumes and bind mounts. Volumes are managed by Docker and stored under /var/lib/docker/volumes, while bind mounts map host directories directly into containers.

---
