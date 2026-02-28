# 🐳 Docker Daemon Configuration File (`daemon.json`)

When you install Docker Engine, the Docker daemon (`dockerd`) runs in the background and manages containers.

You can customize its behavior using a configuration file called:

```bash
/etc/docker/daemon.json
```

This file controls:

* Storage driver
* Logging driver
* Registry settings
* Cgroup driver
* Insecure registries
* DNS
* Data root directory
* Default runtime

---

# 📁 Location of Docker Daemon Config File

| OS                     | Location                                   |
| ---------------------- | ------------------------------------------ |
| Linux                  | `/etc/docker/daemon.json`                  |
| Windows                | `C:\ProgramData\docker\config\daemon.json` |
| macOS (Docker Desktop) | GUI settings or internal VM                |

---

# 🧱 Basic Structure of daemon.json

The file uses **JSON format**:

```json
{
  "key": "value",
  "key2": "value2"
}
```

⚠️ Important:

* Proper JSON format required
* No trailing commas
* Use double quotes only

---

# 🔧 Example 1️⃣: Set Storage Driver

Recommended storage driver on Linux:

```json
{
  "storage-driver": "overlay2"
}
```

After editing:

```bash
sudo systemctl restart docker
```

Verify:

```bash
docker info | grep "Storage Driver"
```

---

# 🔧 Example 2️⃣: Configure Log Driver

By default, Docker uses `json-file`.

To limit log size:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

This prevents log files from filling disk.

---

# 🔧 Example 3️⃣: Configure Insecure Registry

If you have a private registry without SSL:

```json
{
  "insecure-registries": ["192.168.1.100:5000"]
}
```

⚠️ Use only in internal trusted networks.

---

# 🔧 Example 4️⃣: Change Docker Data Directory

Default Docker root:

```bash
/var/lib/docker
```

To move it:

```json
{
  "data-root": "/mnt/docker-data"
}
```

Steps:

```bash
sudo systemctl stop docker
sudo rsync -aP /var/lib/docker/ /mnt/docker-data
sudo systemctl start docker
```

Verify:

```bash
docker info | grep "Docker Root Dir"
```

---

# 🔧 Example 5️⃣: Set Cgroup Driver

Important for Kubernetes compatibility.

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

Verify:

```bash
docker info | grep Cgroup
```

---

# 🔧 Example 6️⃣: Configure DNS

```json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```

Useful in corporate environments.

---

# 🔧 Example 7️⃣: Enable Live Restore

Prevents containers from stopping if daemon restarts:

```json
{
  "live-restore": true
}
```

Very useful in production.

---

# 🔧 Example 8️⃣: Set Default Address Pool

Avoid IP conflicts:

```json
{
  "default-address-pools": [
    {
      "base": "10.10.0.0/16",
      "size": 24
    }
  ]
}
```

---

# 📌 Full Production Example daemon.json

```json
{
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "5"
  },
  "exec-opts": ["native.cgroupdriver=systemd"],
  "live-restore": true,
  "data-root": "/var/lib/docker"
}
```

---

# 🔄 After Any Change

Always restart Docker:

```bash
sudo systemctl restart docker
```

Check logs if issues:

```bash
sudo journalctl -u docker
```

---

# 🧪 Validate Configuration

Test JSON syntax:

```bash
sudo dockerd --validate --config-file /etc/docker/daemon.json
```

Or:

```bash
sudo cat /etc/docker/daemon.json | python -m json.tool
```

---

# 🧠 Internal Architecture Reference

```text
Docker CLI
   ↓
dockerd (reads daemon.json)
   ↓
containerd
   ↓
runc
   ↓
Linux Kernel
```

`daemon.json` directly affects how `dockerd` behaves.

---

# 🚨 Common Mistakes

❌ Missing comma
❌ Extra comma at end
❌ Using single quotes
❌ Not restarting Docker
❌ Wrong JSON format

---

# 🎯 Interview-Level Answer

> The Docker daemon configuration file is `/etc/docker/daemon.json`. It is a JSON file used to configure storage drivers, log drivers, insecure registries, cgroup drivers, DNS, and other daemon-level settings. After modification, Docker service must be restarted for changes to take effect.

