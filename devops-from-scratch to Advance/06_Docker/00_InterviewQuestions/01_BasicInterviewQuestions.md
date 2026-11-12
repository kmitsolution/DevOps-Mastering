### 1. What is Docker?

Docker is a **containerization platform** used to build, package, and run applications in **containers**. Containers include the application along with its dependencies, ensuring it runs the same in any environment.

---

### 2. What is a Container?

A **container** is a lightweight, portable environment that includes an application and all its dependencies needed to run it.

Example: Instead of installing software manually on every server, we run it inside a container.

---

### 3. What is a Docker Image?

A **Docker Image** is a **read-only template** used to create containers.
It contains application code, libraries, dependencies, and configuration files.

Example:
Ubuntu image, Nginx image, MySQL image.

---

### 4. What is the difference between Docker Image and Docker Container?

| Docker Image              | Docker Container          |
| ------------------------- | ------------------------- |
| Blueprint/template        | Running instance of image |
| Read-only                 | Writable                  |
| Used to create containers | Runs the application      |

Example:
Image = Class
Container = Object

---

### 5. What is a Dockerfile?

A **Dockerfile** is a script that contains instructions to build a Docker image automatically.

Example:

```
FROM ubuntu
RUN apt update
CMD ["echo","Hello Docker"]
```

---

### 6. What is Docker Hub?

Docker Hub is a **public registry** where users can **store and share Docker images**.

Example:
You can pull images like:

```
docker pull nginx
docker pull ubuntu
```

---

### 7. What is the difference between Virtual Machine and Container?

| Virtual Machine | Container      |
| --------------- | -------------- |
| Runs full OS    | Shares host OS |
| Heavy           | Lightweight    |
| Slow startup    | Fast startup   |
| More resources  | Less resources |

---

### 8. What is Docker Volume?

Docker Volume is used to **store persistent data outside containers**.

If a container is deleted, the data stored in the volume **remains safe**.

Example:

```
docker volume create myvolume
```

---

### 9. What is Docker Networking?

Docker networking allows **containers to communicate with each other and with external systems**.

Common network types:

* Bridge
* Host
* None
* Overlay

---

### 10. What is the difference between `docker run` and `docker start`?

| Command      | Purpose                              |
| ------------ | ------------------------------------ |
| docker run   | Creates and starts a new container   |
| docker start | Starts an existing stopped container |

Example:

```
docker run ubuntu
docker start container_id
```

---
