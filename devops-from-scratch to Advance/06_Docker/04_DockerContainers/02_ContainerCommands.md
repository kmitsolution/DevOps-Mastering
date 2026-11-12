# 🐳 Docker Container Commands – 

This document covers commonly used **Docker container management commands** using Docker Engine.

---

# 📘 1️⃣ Get Help for Docker Container Commands

## 🔹 Help for All Container Commands

```bash
docker container --help
```

Shows all available subcommands like:

* run
* ls
* stop
* rm
* exec
* inspect
* logs
* pause
* restart
* stats

---

## 🔹 Help for Specific Command

Example: Help for listing containers

```bash
docker container ls --help
```

Shows available options like:

* --all
* --quiet
* --filter
* --format

---

# ▶️ 2️⃣ Run a Container (Attached Mode)

By default, Docker runs in foreground (attached mode).

```bash
docker container run -it ubuntu
```

### Options:

* `-i` → Interactive
* `-t` → Allocate terminal

You cannot return to shell until you exit container.

Exit:

```bash
exit
```

---

# 📴 3️⃣ Run Container in Detached Mode (-d)

Runs in background.

```bash
docker container run -it -d ubuntu
```

* Prints container ID
* Returns to shell immediately

---

# 🏷️ 4️⃣ Run Container with Name

If name not given → Docker assigns random name.

```bash
docker container run -it --name c1 -d ubuntu
```

Now you can use:

```bash
docker stop c1
```

Instead of container ID.

---

# 🧠 5️⃣ Run Container with Memory Limit

```bash
docker run -m 8m -dit --name web1 nginx
```

* `-m 8m` → Limit memory to 8MB
* `-d` → Detached
* `-i -t` → Interactive terminal

Check usage:

```bash
docker stats web1
```

---

# ⚙️ 6️⃣ Run Container with CPU Allocation

CPU shares default value = 1024.

```bash
docker run -c 614 -dit --name db nginx
docker run -c 410 -dit --name web nginx
```

* db gets ~60%
* web gets ~40%

(614 ≈ 60% of 1024)

---

# 📊 7️⃣ Container Statistics

```bash
docker stats web1
```

Shows:

* CPU %
* Memory usage
* Network IO
* Block IO

---

# 🔍 8️⃣ Inspect a Container

```bash
docker inspect web
```

Displays:

* IP address
* Mounts
* Volumes
* Network details
* Environment variables

---

# 📜 9️⃣ View Container Logs

```bash
docker logs web
```

Follow logs live:

```bash
docker logs -f web
```

---

# 📋 🔟 List Containers

## Running Containers

Using classic command:

```bash
docker ps
```

Using management command:

```bash
docker container ls
```

---

## All Containers (Including Exited)

```bash
docker ps -a
```

or

```bash
docker container ls --all
```

---

# 🔄 1️⃣1️⃣ Interact with Running Container

```bash
docker exec -it c1 bash
```

You enter container shell.

Exit:

```bash
exit
```

---

# 🛑 1️⃣2️⃣ Stop a Container

Graceful stop:

```bash
docker stop c1
```

Force stop:

```bash
docker kill c1
```

Exit code 137 = Killed forcefully.

---

# 🗑️ 1️⃣3️⃣ Remove Containers

Remove stopped container:

```bash
docker rm c1
```

Force remove running container:

```bash
docker rm -f c1
```

Remove all containers:

```bash
docker rm -f $(docker ps -a -q)
```

---

# 🌐 1️⃣4️⃣ Port Forwarding (Port Mapping)

```bash
docker container run -it --name webserver -p 80:80 -d ubuntu
```

Format:

```text
-p HostPort:ContainerPort
```

Host port 80 → Container port 80

---

# 🌍 Example: Run Apache in Container

Step 1:

```bash
docker exec -it webserver bash
```

Step 2 (inside container):

```bash
apt update
apt install apache2 -y
service apache2 start
```

Now open browser:

```
http://<PublicIP>
```

You will see Apache default page.

---

# 📁 1️⃣5️⃣ Copy File to Container

Create file locally:

```bash
nano test.html
```

Copy to container:

```bash
docker cp test.html webserver:/var/www/html/
```

Access in browser:

```
http://<PublicIP>/test.html
```

---

# ▶️ 1️⃣6️⃣ Start a Container

If container is in Exited state:

```bash
docker start c1
```

---

# 🔄 1️⃣7️⃣ Restart a Container

```bash
docker restart c1
```

* Stops container
* Starts again

---

# ⏸️ 1️⃣8️⃣ Pause a Container

```bash
docker pause c1
```

Freezes container processes.

---

# ▶️ 1️⃣9️⃣ Unpause a Container

```bash
docker unpause c1
```

Resumes execution.

---

# 🔥 2️⃣0️⃣ Kill a Container

Forcefully stops:

```bash
docker kill c1
```

---

# 📌 Container Lifecycle

```text
Create → Running → Paused → Stopped → Removed
```

Commands involved:

```text
docker run
docker pause
docker unpause
docker stop
docker start
docker restart
docker rm
```

---

# 🧠 Interview-Level Summary

> Docker container commands allow us to create, run, inspect, monitor, and manage containers. We can run containers in attached or detached mode, limit CPU and memory using cgroups, inspect using docker inspect, view logs using docker logs, manage lifecycle using stop/start/restart/pause/unpause, and expose ports using -p option.

---

