# 🐳 Dockerfile – Definition, Commands & Complete Example

## 📘 What is a Dockerfile?

A **Dockerfile** is a text file that contains instructions to automatically build a Docker image.

Instead of manually creating a container and using `docker commit`, we define everything in code.

Build command:

```bash
docker build -t myapp:1.0 .
```

Docker reads Dockerfile → Creates layered image → Stores it in Docker Engine.

---

# 🧱 Common Dockerfile Instructions

We will cover:

* FROM
* RUN
* ADD
* COPY
* USER
* EXPOSE
* CMD
* ENTRYPOINT
* WORKDIR

---

# 1️⃣ FROM

## 🔹 Definition

Specifies the **base image**.

It must be the first instruction.

## 🔹 Example

```dockerfile
FROM ubuntu:22.04
```

This pulls `ubuntu:22.04` image and starts building from it.

---

# 2️⃣ RUN

## 🔹 Definition

Executes commands during image build time.

Creates a new layer.

## 🔹 Example

```dockerfile
RUN apt update && apt install -y nginx
```

Best Practice:

```dockerfile
RUN apt update && apt install -y nginx && rm -rf /var/lib/apt/lists/*
```

Combining commands reduces layers.

---

# 3️⃣ COPY

## 🔹 Definition

Copies files from local system into container image.

## 🔹 Syntax

```dockerfile
COPY <source> <destination>
```

## 🔹 Example

```dockerfile
COPY index.html /var/www/html/
```

---

# 4️⃣ ADD

## 🔹 Definition

Similar to COPY, but has extra features:

* Can extract tar files automatically
* Can download from URL

## 🔹 Example

```dockerfile
ADD myapp.tar.gz /app/
```

⚠️ Recommendation: Use COPY unless you specifically need ADD features.

---

# 5️⃣ WORKDIR

## 🔹 Definition

Sets working directory inside container.

Equivalent to `cd` in Linux.

## 🔹 Example

```dockerfile
WORKDIR /app
```

All subsequent commands run inside `/app`.

---

# 6️⃣ USER

## 🔹 Definition

Sets the user to run commands inside container.

Default is root.

## 🔹 Example

```dockerfile
RUN useradd -m appuser
USER appuser
```

Improves security.

---

# 7️⃣ EXPOSE

## 🔹 Definition

Documents which port the container will use.

It does NOT publish port automatically.

## 🔹 Example

```dockerfile
EXPOSE 80
```

To publish:

```bash
docker run -p 80:80 myimage
```

---

# 8️⃣ CMD

## 🔹 Definition

Defines default command executed when container starts.

Only one CMD allowed (last one wins).

## 🔹 Example

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

Can be overridden at runtime.

---

# 9️⃣ ENTRYPOINT

## 🔹 Definition

Defines fixed executable that always runs.

Unlike CMD, it cannot be easily overridden.

## 🔹 Example

```dockerfile
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

Execution:

```text
nginx -g "daemon off;"
```

---

# 🆚 CMD vs ENTRYPOINT

| Feature                | CMD             | ENTRYPOINT      |
| ---------------------- | --------------- | --------------- |
| Can override easily    | ✅ Yes           | ❌ No            |
| Sets default arguments | ✅ Yes           | Sometimes       |
| Best use               | Default command | Main executable |

---

# 🏗️ Complete Example – Apache Web Server

Let’s build a real Dockerfile.

---

## 📁 Project Structure

```
project/
 ├── Dockerfile
 └── index.html
```

---

## 📄 index.html

```html
<h1>Hello from Dockerfile Example</h1>
```

---

## 📄 Dockerfile

```dockerfile
# Base Image
FROM ubuntu:22.04

# Install Apache
RUN apt update && \
    apt install -y apache2 && \
    rm -rf /var/lib/apt/lists/*

# Set Working Directory
WORKDIR /var/www/html

# Copy Website Files
COPY index.html .

# Expose Port
EXPOSE 80

# Run Apache in Foreground
CMD ["apachectl", "-D", "FOREGROUND"]
```

---

# 🛠️ Build the Image

```bash
docker build -t myapache:1.0 .
```

---

# ▶️ Run Container

```bash
docker run -d -p 8080:80 --name web1 myapache:1.0
```

Open browser:

```
http://localhost:8080
```

You will see:

```
Hello from Dockerfile Example
```

---

# 🧬 Internal Layer Structure

```text
Layer 1 → ubuntu base
Layer 2 → apt install apache
Layer 3 → WORKDIR
Layer 4 → COPY index.html
Layer 5 → CMD metadata
```

---

# 🎯 Best Practices

✔ Use specific base image version
✔ Combine RUN commands
✔ Remove unnecessary files
✔ Use non-root USER
✔ Use COPY instead of ADD (unless required)
✔ Keep images small

---
# 🐳 What is a Multi-Stage Dockerfile?

![Image](https://labs.iximiuz.com/content/files/tutorials/docker-multi-stage-builds/__static__/multi-stage-build.png)

![Image](https://cdn.prod.website-files.com/68232df8de1a79da2c32a09e/68473e96bab1a481e47a648c_66fdaf3d58acc627f76db63a_66fdaf35614e560cc54df420_Docker%252520Multi-Stage%252520Builds%252520-%252520Image%252520Size%252520Diagram.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1286/1%2Aw_McJn3masV9PLfJp9JGjQ.png)

![Image](https://containers-at-tacc.readthedocs.io/en/latest/_images/multi-stage.jpg)

## 📘 Definition

A **Multi-Stage Dockerfile** is a Dockerfile that uses **multiple `FROM` instructions** to create separate build stages.

It allows you to:

* Build application in one stage
* Copy only required artifacts into final stage
* Remove build dependencies
* Reduce final image size

👉 Introduced in Docker 17.05.

---

# 🎯 Why Do We Need Multi-Stage Build?

In traditional Dockerfile:

```dockerfile
FROM node:18
RUN npm install
RUN npm run build
```

Problem:

* Node
* npm
* build tools
* source code
* dev dependencies

All remain in final image ❌

Image becomes large.

---

# ✅ Multi-Stage Solution

Separate:

1️⃣ Build stage
2️⃣ Runtime stage

Only copy final compiled output.

---

# 🧱 Basic Multi-Stage Structure

```dockerfile
FROM base-image AS builder
# Build steps

FROM runtime-image
# Copy built artifacts from builder
COPY --from=builder /path /destination
```

---

# 🔥 Example 1️⃣ – Node.js Application

## 📁 Project Structure

```text
app/
 ├── package.json
 ├── package-lock.json
 ├── index.js
 └── Dockerfile
```

---

## 🐳 Multi-Stage Dockerfile

```dockerfile
# Stage 1 - Build Stage
FROM node:18 AS builder

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build


# Stage 2 - Production Stage
FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app .

EXPOSE 3000

CMD ["node", "index.js"]
```

---

# 🔍 What Happens Here?

### Stage 1 (builder)

* Installs dependencies
* Builds application
* Contains all build tools

### Stage 2 (runtime)

* Lightweight alpine image
* Copies only built app
* Does NOT contain:

  * npm cache
  * dev tools
  * unnecessary files

Final image is smaller 🚀

---

# 📦 Example 2️⃣ – Java Application (Maven)

Without multi-stage:

* Maven
* JDK
* Source code
* Build files

All included ❌

---

## 🐳 Multi-Stage Java Dockerfile

```dockerfile
# Stage 1 - Build
FROM maven:3.9.6-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn clean package


# Stage 2 - Runtime
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

COPY --from=builder /app/target/myapp.jar .

EXPOSE 8080

CMD ["java", "-jar", "myapp.jar"]
```

---

# 📊 Size Comparison Example

| Build Type        | Image Size |
| ----------------- | ---------- |
| Normal Dockerfile | 800MB      |
| Multi-Stage       | 200MB      |

Huge reduction 🔥

---

# 🧠 How It Works Internally

```text
Stage 1:
Layer 1 → maven base
Layer 2 → dependencies
Layer 3 → compiled JAR

Stage 2:
Layer 1 → jre base
Layer 2 → copied JAR only
```

Only Stage 2 becomes final image.

Stage 1 is discarded automatically.

---

# 🎯 Advantages

✔ Smaller image size
✔ Better security (no build tools)
✔ Faster deployment
✔ Reduced attack surface
✔ Production-ready images

---

# ⚠️ Important Notes

* Each `FROM` creates a new stage
* Use `AS <name>` to name stages
* Use `COPY --from=<stage>` to copy artifacts
* Only last stage becomes final image

---

# 🧠 Interview-Level Explanation

> A multi-stage Dockerfile uses multiple FROM statements to separate build and runtime environments. The build stage compiles the application and the runtime stage copies only required artifacts, reducing image size and improving security.

---

# 📌 Visual Representation

```text
Build Stage (Heavy)
   ↓
Compile Application
   ↓
Copy Only Output
   ↓
Runtime Stage (Lightweight)
   ↓
Final Production Image
```




