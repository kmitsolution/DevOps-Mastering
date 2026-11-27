# Docker MCQ Practice Set 02 – Docker Images and Containers

**KMIT Solutions | Docker Training | Interview Preparation**

| Details | Description |
|---|---|
| Topic | Docker Images and Containers |
| Difficulty | Beginner to Intermediate |
| Total Questions | 15 |
| Total Marks | 15 |
| Question Type | MCQ + Practical Scenarios |

## Instructions

- Each question carries 1 mark.
- Each question has four options.
- Select only one correct answer.
- Questions include practical Docker scenarios.
- Answer key with one-line explanations is provided at the end.

---

# Section 1: Docker Images

### Q1. A developer wants to download the NGINX image from Docker Hub without creating a container. Which command should the developer execute?

A. `docker run nginx`

B. `docker pull nginx`

C. `docker start nginx`

D. `docker create nginx`

### Q2. A developer executes the following command:

```bash
docker pull ubuntu:24.04
```

What does `24.04` represent?

A. Container ID

B. Image ID

C. Image Tag

D. Docker Version

### Q3. You have downloaded three Docker images on your system. Which command will display all locally available images?

A. `docker ps`

B. `docker images`

C. `docker containers`

D. `docker list`

### Q4. A developer wants to create a new tag named `myapp:v1` for an existing image named `nginx:latest`.

Which command should be used?

A. `docker tag nginx:latest myapp:v1`

B. `docker rename nginx:latest myapp:v1`

C. `docker create nginx:latest myapp:v1`

D. `docker push nginx:latest myapp:v1`

### Q5. You want to understand the configuration details of a Docker image, including its environment variables, exposed ports, entry point and working directory.

Which command should you use?

A. `docker ps nginx`

B. `docker inspect nginx`

C. `docker start nginx`

D. `docker logs nginx`

---

# Section 2: Docker Containers and Commands

### Q6. A developer executes the following command:

```bash
docker run -d --name webserver nginx
```

What will happen?

A. Docker downloads the image only.

B. Docker creates and starts a container in detached mode with the name `webserver`.

C. Docker creates a Virtual Machine named `webserver`.

D. Docker starts an existing container named `webserver`.

### Q7. You have created a container using the following command:

```bash
docker create --name mycontainer nginx
```

The container is not running. Which command will start this existing container?

A. `docker run mycontainer`

B. `docker start mycontainer`

C. `docker create mycontainer`

D. `docker pull mycontainer`

### Q8. A container named `web1` is currently running. You want to stop it gracefully.

Which command should you execute?

A. `docker rm web1`

B. `docker kill web1`

C. `docker stop web1`

D. `docker pause web1`

### Q9. A developer wants to execute an interactive Bash shell inside an already running Ubuntu container named `ubuntu1`.

Which command should be used?

A. `docker run -it ubuntu1 bash`

B. `docker exec -it ubuntu1 bash`

C. `docker pull ubuntu1 bash`

D. `docker start -it ubuntu1 bash`

### Q10. You want to see both running and stopped containers on your Docker host.

Which command should you execute?

A. `docker ps`

B. `docker images`

C. `docker ps -a`

D. `docker inspect`

---

# Section 3: Practical Scenario-Based Questions

### Q11. Scenario: Multiple Containers from One Image

A developer downloads the Ubuntu image once:

```bash
docker pull ubuntu
```

Then creates three containers:

```bash
docker run -it --name ubuntu1 ubuntu bash
docker run -it --name ubuntu2 ubuntu bash
docker run -it --name ubuntu3 ubuntu bash
```

Which statement is correct?

A. Docker downloads three separate Ubuntu images.

B. All three containers share the same writable layer.

C. All three containers use the same image but have independent writable container layers.

D. Docker creates three Virtual Machines.

### Q12. Scenario: Data Loss in Containers

A developer creates an Ubuntu container and creates a file inside it:

```bash
docker run -it --name testcontainer ubuntu bash
```

Inside the container:

```bash
echo "Hello Docker" > /tmp/test.txt
exit
```

The developer then removes the container:

```bash
docker rm testcontainer
```

What happens to the file `/tmp/test.txt`?

A. The file is automatically saved in the Docker image.

B. The file is permanently available in the next container.

C. The file is lost because it was stored in the container's writable layer, which was removed.

D. The file is automatically uploaded to Docker Hub.

### Q13. Scenario: Container Port Mapping

A developer wants to access an NGINX container from a web browser using:

```text
http://localhost:8080
```

The NGINX application listens on port 80 inside the container.

Which command should be used?

A. `docker run -d -p 8080:80 nginx`

B. `docker run -d -p 80:8080 nginx`

C. `docker run -d -p 8080 nginx`

D. `docker run -d --port 8080:80 nginx`

### Q14. Scenario: Removing an Image

A developer executes:

```bash
docker rmi nginx
```

Docker displays an error saying that the image is being used by an existing container.

What should the developer do?

A. Use `docker pull nginx`.

B. Remove the container using that image, then remove the image if no other containers depend on it.

C. Execute `docker start nginx`.

D. Execute `docker inspect nginx`.

### Q15. Scenario: Image vs Container Modification

A developer creates a container from an Ubuntu image:

```bash
docker run -it --name myubuntu ubuntu bash
```

Inside the container, the developer installs a package and creates a new file.

Which statement is correct?

A. The original Ubuntu image is modified automatically.

B. All other containers created from the same image automatically receive these changes.

C. Changes are stored in the container's writable layer and do not automatically modify the original image.

D. Docker automatically pushes the modified image to Docker Hub.

---

# Answer Key – Docker Images and Containers

**Check your answers only after attempting all 15 questions.**

| Question | Correct Answer | Explanation |
|---|---|---|
| Q1 | B | `docker pull` downloads an image without creating a container. |
| Q2 | C | `24.04` is the image tag identifying the Ubuntu image version. |
| Q3 | B | `docker images` displays locally available images. |
| Q4 | A | `docker tag` creates another tag referencing the existing image. |
| Q5 | B | `docker inspect` displays detailed configuration and metadata. |
| Q6 | B | `-d` runs the container in detached mode and `--name` assigns its name. |
| Q7 | B | `docker start` starts an existing stopped container. |
| Q8 | C | `docker stop` gracefully stops a running container. |
| Q9 | B | `docker exec -it` executes a command interactively inside a running container. |
| Q10 | C | `docker ps -a` lists running and stopped containers. |
| Q11 | C | Containers created from the same image have separate writable layers. |
| Q12 | C | Removing a container removes its writable layer and its unpersisted data. |
| Q13 | A | `-p 8080:80` maps host port 8080 to container port 80. |
| Q14 | B | An image cannot normally be removed while dependent containers still reference it. |
| Q15 | C | Container modifications are stored in its writable layer, not in the original image. |

---

# Score Evaluation

| Score | Performance | Feedback |
|---|---|---|
| 13–15 | Excellent | Strong understanding of Docker Images and Containers. |
| 10–12 | Good | Concepts understood. Practice more Docker commands. |
| 7–9 | Average | Revise image and container lifecycle concepts. |
| 0–6 | Needs Improvement | Revisit the Docker fundamentals lectures. |

---

# Bonus: Hands-on Docker Lab

After completing this MCQ assessment, perform these practical tasks.

### Task 1: Download NGINX Image

```bash
docker pull nginx
```

### Task 2: List Docker Images

```bash
docker images
```

### Task 3: Create a Container

```bash
docker run -d --name web1 nginx
```

### Task 4: Check Running Containers

```bash
docker ps
```

### Task 5: Inspect the Container

```bash
docker inspect web1
```

### Task 6: Check Container Logs

```bash
docker logs web1
```

### Task 7: Execute a Command Inside the Container

```bash
docker exec -it web1 bash
```

### Task 8: Stop the Container

```bash
docker stop web1
```

### Task 9: Restart the Container

```bash
docker start web1
```

### Task 10: Remove the Container

```bash
docker stop web1
docker rm web1
```

### Task 11: Remove the NGINX Image

```bash
docker rmi nginx
```

---

# Learning Outcomes

After completing this assessment, students should be able to:

1. Download Docker images from Docker Hub.
2. Understand Docker image tags.
3. List and inspect Docker images.
4. Create multiple containers using the same image.
5. Understand the difference between `docker run` and `docker create`.
6. Manage container lifecycle using start, stop and remove commands.
7. Execute commands inside running containers.
8. Understand container writable layers.
9. Map container ports to host ports.
10. Understand the relationship between Docker Images and Containers.

---

**KMIT Solutions – Empowering Your IT Career Through Practical Learning!**
