# Creating Your First GitLab CI Pipeline Using Docker Images

In GitLab CI/CD, every job runs inside an **execution environment**. When using the **Docker Executor**, this environment is created from a **Docker image**.

The Docker image determines what software is available during pipeline execution. If the required tools are not present in the image, the pipeline will fail.

In this example, we will first use the **Alpine Linux** image, which does **not** include Node.js. The pipeline will fail because the `node` and `npm` commands are unavailable.

We will then replace the image with the **Node.js 22** image, which includes both Node.js and npm, and the pipeline will execute successfully.

---

# Step 1: Create the Pipeline File

Create a file named:

```text
.gitlab-ci.yml
```

Place it in the root directory of your repository.

---

# Step 2: Pipeline Using Alpine Linux

Create the following pipeline:

```yaml
image: alpine

stages:
  - verify

verify-node:
  stage: verify

  script:
    - node --version
    - npm --version
```

---

# Understanding the Pipeline

## image

```yaml
image: alpine
```

This line instructs GitLab Runner to:

* Pull the **Alpine Linux** Docker image.
* Create a temporary container.
* Execute all pipeline commands inside that container.

Alpine is a lightweight Linux distribution commonly used for containers.

However, it contains only a minimal set of Linux utilities.

It **does not include**:

* Node.js
* npm

---

## stages

```yaml
stages:
  - verify
```

The pipeline contains a single stage named **verify**.

---

## Job

```yaml
verify-node:
```

This is the job that will execute inside the Runner.

---

## Script

```yaml
script:
  - node --version
  - npm --version
```

The Runner attempts to execute two Linux commands:

* Display the Node.js version.
* Display the npm version.

---

# What Happens During Execution?

```text
Developer Pushes Code
        │
        ▼
GitLab Creates Pipeline
        │
        ▼
Runner Pulls Alpine Image
        │
        ▼
Creates Alpine Container
        │
        ▼
Runs node --version
        │
        ▼
Command Not Found
        │
        ▼
Job Failed
        │
        ▼
Pipeline Failed
```

---

# Expected Error

Since Alpine does not contain Node.js, the Runner displays an error similar to:

```text
$ node --version
/bin/sh: node: not found

ERROR: Job failed: exit code 127
```

Or:

```text
$ npm --version
/bin/sh: npm: not found
```

---

# Why Did the Pipeline Fail?

The GitLab Runner successfully created the Docker container.

However, the container was created from the **Alpine Linux image**, which contains only a minimal Linux installation.

Since Node.js and npm are not installed inside the container, the commands cannot be executed.

The Runner can execute **only the software available inside the Docker image**.

---

# Step 3: Use the Correct Docker Image

Replace the Alpine image with the official Node.js image.

```yaml
image: node:22

stages:
  - verify

verify-node:
  stage: verify

  script:
    - node --version
    - npm --version
```

---

# What Changed?

Only one line changed:

Before:

```yaml
image: alpine
```

After:

```yaml
image: node:22
```

The **Node.js 22 Docker image** already includes:

* Node.js
* npm

Therefore, the Runner can execute both commands successfully.

---

# Pipeline Execution

```text
Developer Pushes Code
        │
        ▼
GitLab Creates Pipeline
        │
        ▼
Runner Pulls node:22 Image
        │
        ▼
Creates Docker Container
        │
        ▼
Runs node --version
        │
        ▼
Displays Node Version
        │
        ▼
Runs npm --version
        │
        ▼
Displays npm Version
        │
        ▼
Job Succeeded
        │
        ▼
Pipeline Succeeded
```

---

# Expected Output

```text
$ node --version
v22.x.x

$ npm --version
10.x.x

Job succeeded
```

(The exact version numbers may vary depending on the specific `node:22` image.)

---

# Why Does This Work?

The official **Node.js Docker image** is built specifically for JavaScript applications.

It already contains:

* Node.js runtime
* npm
* Required Linux libraries

When the Runner creates a container from this image, all Node.js commands are immediately available.

---

# Understanding the Role of the Docker Image

Think of the Docker image as a **pre-configured virtual environment** for your job.

### Alpine Image

```text
Docker Image
┌──────────────────────────────┐
│ Alpine Linux                 │
│                              │
│ ✓ Linux Shell                │
│ ✓ Basic Linux Commands       │
│ ✗ Node.js                    │
│ ✗ npm                        │
└──────────────────────────────┘
```

---

### Node.js Image

```text
Docker Image
┌──────────────────────────────┐
│ Node.js 22                   │
│                              │
│ ✓ Linux Shell                │
│ ✓ Node.js                    │
│ ✓ npm                        │
│ ✓ JavaScript Runtime         │
└──────────────────────────────┘
```

---

# How GitLab Runner Uses the Image

For every job, the Runner performs these steps:

```text
Read .gitlab-ci.yml
        │
        ▼
Pull Docker Image
        │
        ▼
Create Temporary Container
        │
        ▼
Clone Repository
        │
        ▼
Execute Script Commands
        │
        ▼
Display Logs
        │
        ▼
Delete Container
```

The software available to your pipeline depends entirely on the Docker image you choose.

---

# Key Takeaways

* Every GitLab CI job runs inside an execution environment.
* With the Docker executor, this environment is a **Docker container**.
* The **`image`** keyword specifies which Docker image the Runner should use.
* **`alpine`** is a minimal Linux image and does not include Node.js or npm.
* Attempting to run `node --version` or `npm --version` in an Alpine container results in a **command not found** error.
* Replacing `image: alpine` with `image: node:22` provides a container with Node.js and npm pre-installed, allowing the pipeline to execute successfully.

---

# Summary

This example demonstrates one of the most important concepts in GitLab CI/CD: **the Docker image defines the execution environment**. When the pipeline uses the `alpine` image, the GitLab Runner creates a lightweight Linux container that lacks Node.js, causing the `node` and `npm` commands to fail. By changing the image to `node:22`, the Runner creates a container with Node.js and npm already installed, enabling the same commands to execute successfully. This illustrates why selecting the correct Docker image is essential when designing CI/CD pipelines.
