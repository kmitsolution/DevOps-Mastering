# Docker Fundamentals – MCQ Practice Set 01

**KMIT Solutions | Docker Training | Interview Preparation**

| Details         | Description                     |
| --------------- | ------------------------------- |
| Topic           | Docker Fundamentals             |
| Difficulty      | Beginner                        |
| Total Questions | 15                              |
| Total Marks     | 15                              |
| Question Type   | Multiple Choice Questions (MCQ) |

## Instructions

- Each question carries 1 mark.
- Select only one correct answer.
- Try solving all questions before checking the answer key.
- Answers with explanations are provided at the end.

---

## Section 1: Docker Fundamentals

### Q1. What is Docker?

A. A programming language used to develop applications.

B. A containerization platform used to build, package, and run applications.

C. A database management system.

D. A hypervisor used to create virtual machines.

### Q2. What is a Docker Container?

A. A template used to create virtual machines.

B. A running instance of a Docker Image that provides an isolated environment for an application.

C. A physical server used to host applications.

D. A configuration file used to install Docker.

### Q3. What is the major difference between a Virtual Machine and a Docker Container?

A. Both require a separate guest operating system.

B. Containers virtualize hardware, whereas VMs virtualize applications.

C. Virtual Machines run a guest OS, whereas containers share the host OS kernel.

D. Containers require more resources than Virtual Machines in every situation.

### Q4. Why are Docker Containers generally lightweight compared to Virtual Machines?

A. Containers do not require an application.

B. Containers share the host operating system kernel instead of running a separate guest OS.

C. Containers do not require memory.

D. Containers do not require a CPU.

### Q5. Which component is responsible for managing Docker objects such as images, containers, networks, and volumes?

A. Docker Client

B. Docker Daemon

C. Docker Hub

D. Docker Registry

---

## Section 2: Docker Engine and Architecture

### Q6. What is Docker Engine?

A. A tool used only for downloading Docker images.

B. The core containerization technology that builds and runs containers.

C. A graphical interface for managing Virtual Machines.

D. A programming language for writing Dockerfiles.

### Q7. What is the primary responsibility of Docker Daemon (`dockerd`)?

A. To execute SQL queries.

B. To manage Docker objects and respond to Docker API requests.

C. To compile application source code.

D. To create Kubernetes clusters.

### Q8. How does the Docker Client communicate with the Docker Daemon?

A. Through HTTP/HTTPS API requests using Docker's API.

B. Through database queries.

C. Through SSH only.

D. Through FTP.

### Q9. Which command is used to verify whether Docker is installed and check the Docker Client and Server versions?

A. `docker check`

B. `docker status`

C. `docker version`

D. `docker verify`

### Q10. Which command can be used to check whether Docker is working correctly by running a test container?

A. `docker test`

B. `docker run hello-world`

C. `docker start docker`

D. `docker install test`

---

## Section 3: Docker Images and Containers

### Q11. What is a Docker Image?

A. A running application process.

B. A read-only template used to create Docker Containers.

C. A virtual machine with a complete operating system.

D. A Docker network configuration.

### Q12. Which command is used to download an image from Docker Hub without creating a container?

A. `docker download nginx`

B. `docker get nginx`

C. `docker pull nginx`

D. `docker start nginx`

### Q13. You want to download the Ubuntu 24.04 image from Docker Hub. Which command should you execute?

A. `docker pull ubuntu:24.04`

B. `docker run ubuntu:24.04`

C. `docker start ubuntu:24.04`

D. `docker install ubuntu:24.04`

### Q14. What happens when you execute the following command?

```
docker run nginx
```

A. Docker only downloads the NGINX image.

B. Docker creates and starts a container using the NGINX image. If the image is not available locally, Docker attempts to pull it first.

C. Docker installs NGINX directly on the host operating system.

D. Docker creates a Virtual Machine.

### Q15. Consider the following scenario:

A developer downloads the Ubuntu image using `docker pull ubuntu`. Later, the developer creates three containers using the same Ubuntu image.

Which statement is correct?

A. Three separate Ubuntu images must be downloaded.

B. All three containers share the same writable layer.

C. All three containers can be created from the same image, but each gets its own writable container layer.

D. Docker creates three separate Virtual Machines.

---

# Answer Key – Docker Fundamentals

| Question | Correct Answer | Explanation                                                          |
| -------- | -------------- | -------------------------------------------------------------------- |
| Q1       | B              | Docker is a containerization platform.                               |
| Q2       | B              | A container is a running instance of an image.                       |
| Q3       | C              | VMs have guest OSs; containers share the host kernel.                |
| Q4       | B              | Containers avoid running a separate guest OS.                        |
| Q5       | B              | Docker Daemon manages Docker objects.                                |
| Q6       | B              | Docker Engine provides core container functionality.                 |
| Q7       | B              | dockerd manages images, containers, networks and volumes.            |
| Q8       | A              | Docker Client communicates through the Docker API.                   |
| Q9       | C              | `docker version` displays client and server versions.                |
| Q10      | B              | `hello-world` tests basic Docker functionality.                      |
| Q11      | B              | An image is a read-only template for containers.                     |
| Q12      | C              | `docker pull` downloads images.                                      |
| Q13      | A              | Downloads the Ubuntu 24.04 image.                                    |
| Q14      | B              | Docker pulls the image if needed and creates and starts a container. |
| Q15      | C              | Containers share image layers but have independent writable layers.  |

---

## Score Evaluation

| Score | Performance       | Feedback                                     |
| ----- | ----------------- | -------------------------------------------- |
| 13–15 | Excellent         | Strong understanding of Docker fundamentals. |
| 10–12 | Good              | Concepts understood. Practice more commands. |
| 7–9   | Average           | Revise Docker fundamentals.                  |
| 0–6   | Needs Improvement | Revisit the introductory Docker lectures.    |

---

## Bonus: Practical Interview Questions

After completing these MCQs, perform the following tasks on your system:

**Task 1:** Verify Docker installation.

```
docker version
```

**Task 2:** Execute the Docker test container.

```
docker run hello-world
```

**Task 3:** Download the Ubuntu image.

```
docker pull ubuntu
```

**Task 4:** List all locally available images.

```
docker images
```

**Task 5:** Create two containers from the same Ubuntu image.

```
docker run -it --name ubuntu1 ubuntu bash
```

```
docker run -it --name ubuntu2 ubuntu bash
```

**Task 6:** Explain the difference between the following commands:

```
docker images
docker ps
docker ps -a
```

---

## Learning Outcome

After completing this assessment, students should be able to:

1. Explain Docker and containerization.
2. Differentiate between Virtual Machines and Containers.
3. Understand Docker Engine, Docker Client and Docker Daemon.
4. Verify Docker installation and functionality.
5. Understand Docker Images and Containers.
6. Download images from Docker Hub.
7. Create multiple containers using the same image.

---

**KMIT Solutions – Empowering Your IT Career Through Practical Learning!**

**Suggested GitHub filename:** `Docker-Fundamentals-MCQ.md`

I recommend maintaining separate Markdown files for each MCQ assessment and case study so students can follow the Docker course as a structured practice repository.
