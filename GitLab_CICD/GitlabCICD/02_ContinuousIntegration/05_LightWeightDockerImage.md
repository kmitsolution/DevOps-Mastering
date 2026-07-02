# Using Lightweight Docker Images in GitLab CI/CD

When creating CI/CD pipelines, one of the first decisions is **which Docker image to use**.

Docker images come in different sizes:

* **Full Images** – Include the operating system and many pre-installed packages.
* **Slim Images** – Smaller versions with only essential packages.
* **Alpine Images** – Extremely lightweight images based on Alpine Linux.

Using a smaller image reduces download time and makes pipelines execute faster.

---

# Pipeline 1 – Using Alpine Linux

Let's start with the Alpine Linux image.

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

## Pipeline Result

The pipeline fails.

Example:

```text
$ node --version
/bin/sh: node: not found

ERROR: Job failed: exit code 127
```

### Why?

The **Alpine Linux** image is designed to be a minimal Linux distribution.

It contains:

* Linux shell
* Basic Linux utilities

It does **not** contain:

* Node.js
* npm

Therefore, the Runner cannot execute the `node` or `npm` commands.

---

# Solution 1 – Use the Official Node.js Image

Replace the image with:

```yaml
image: node:22
```

Pipeline:

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

Output:

```text
$ node --version
v22.x.x

$ npm --version
10.x.x

Job succeeded
```

This works because the official Node.js image already includes Node.js and npm.

---

# Solution 2 – Use the Lightweight Node Alpine Image (Recommended)

Instead of using the full Node.js image, we can use the Alpine-based Node.js image.

```yaml
image: node:22-alpine

stages:
  - verify

verify-node:
  stage: verify

  script:
    - node --version
    - npm --version
```

Output:

```text
$ node --version
v22.x.x

$ npm --version
10.x.x

Job succeeded
```

This pipeline also succeeds because **Node.js and npm are already installed**, but the underlying operating system is Alpine Linux, making the image much smaller.

---

# Why Use `node:22-alpine`?

The image:

```text
node:22
```

is based on a larger Linux distribution and includes additional libraries and packages.

The image:

```text
node:22-alpine
```

is built on Alpine Linux and contains:

* Alpine Linux
* Node.js 22
* npm

while removing unnecessary packages to keep the image lightweight.

---

# Image Comparison

| Image            | Node.js | npm | Approximate Size | Recommended Use                          |
| ---------------- | ------- | --- | ---------------- | ---------------------------------------- |
| `alpine`         | ❌       | ❌   | ~5 MB            | Basic Linux commands only                |
| `node:22-alpine` | ✅       | ✅   | ~70–90 MB        | **Recommended for Node.js CI pipelines** |
| `node:22`        | ✅       | ✅   | ~350–450 MB      | When additional OS libraries are needed  |

> **Note:** Image sizes vary over time as new versions are published, but `node:22-alpine` remains significantly smaller than `node:22`.

---

# Visual Comparison

```text
                 Docker Images

┌────────────────────────────┐
│ alpine                     │
├────────────────────────────┤
│ Alpine Linux               │
│ Basic Linux Commands       │
│                            │
│ ✗ Node.js                  │
│ ✗ npm                      │
└────────────────────────────┘


┌────────────────────────────┐
│ node:22-alpine            │
├────────────────────────────┤
│ Alpine Linux               │
│ Node.js                    │
│ npm                        │
│ Lightweight                │
└────────────────────────────┘


┌────────────────────────────┐
│ node:22                    │
├────────────────────────────┤
│ Debian/Linux               │
│ Node.js                    │
│ npm                        │
│ Extra Libraries            │
│ Larger Size                │
└────────────────────────────┘
```

---

# Which Image Should You Use?

* Use **`alpine`** when you only need a minimal Linux environment and will install the required software yourself.
* Use **`node:22-alpine`** for most Node.js and React projects because it is lightweight, downloads quickly, and already includes Node.js and npm.
* Use **`node:22`** when your application requires additional system libraries that may not be available in Alpine Linux.

---

# Key Takeaways

* Every GitLab CI job runs inside a Docker container.
* The **Docker image defines the execution environment**.
* `alpine` is a lightweight Linux image but **does not include Node.js or npm**, so Node.js commands fail.
* `node:22` includes Node.js and npm, so the pipeline succeeds.
* `node:22-alpine` combines the benefits of Alpine's small size with Node.js and npm pre-installed, making it the preferred choice for most React and Node.js CI/CD pipelines.
