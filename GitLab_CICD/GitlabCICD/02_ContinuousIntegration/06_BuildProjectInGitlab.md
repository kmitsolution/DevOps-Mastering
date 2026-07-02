# Building a React Application Using GitLab CI/CD

Now that we have verified Node.js is available inside our GitLab Runner, let's build the React application automatically using a GitLab CI pipeline.

Instead of running the build commands manually on our local machine, GitLab Runner will execute them inside a Docker container whenever code is pushed to the repository.

---

# Objective

In this example, the pipeline will:

* Use the **Node.js 22 Alpine** Docker image.
* Verify the installed Node.js version.
* Verify the installed npm version.
* Install all project dependencies.
* Build the React application.

---

# Create the GitLab CI Pipeline

Create or update the **`.gitlab-ci.yml`** file with the following content:

```yaml
stages:
  - build

build_app:
  stage: build
  image: node:22-alpine

  script:
    - node --version
    - npm --version
    - npm ci
    - npm run build
```

---

# Understanding the Pipeline

## Stage

```yaml
stages:
  - build
```

This pipeline contains a single stage called **build**.

Its purpose is to compile the React application and generate the production-ready build files.

---

## Job

```yaml
build_app:
```

This is the job name.

The **build_app** job performs all the tasks required to build the application.

---

## Docker Image

```yaml
image: node:22-alpine
```

GitLab Runner performs the following actions:

* Pulls the **node:22-alpine** Docker image (if it is not already available).
* Creates a temporary Docker container.
* Executes all the commands inside that container.

The image already includes:

* Node.js 22
* npm
* Alpine Linux

This lightweight image is ideal for Node.js and React applications because it is smaller and downloads faster than the full Node.js image.

---

# Script Commands

## 1. Verify Node.js Version

```bash
node --version
```

Displays the installed Node.js version.

Example:

```text
v22.18.0
```

This confirms that Node.js is available inside the container.

---

## 2. Verify npm Version

```bash
npm --version
```

Displays the installed npm version.

Example:

```text
10.9.3
```

This confirms that npm is installed.

---

## 3. Install Dependencies

```bash
npm ci
```

### What is `npm ci`?

`npm ci` stands for **Clean Install**.

It installs all project dependencies using the **package-lock.json** file.

Unlike `npm install`, `npm ci`:

* Deletes the existing `node_modules` folder (if present).
* Installs the exact dependency versions specified in `package-lock.json`.
* Produces faster and more reliable installations.
* Ensures consistent builds across all environments.

For CI/CD pipelines, **`npm ci` is recommended over `npm install`** because it provides deterministic and repeatable builds.

After execution, the Runner creates the **node_modules** directory inside the container.

---

## 4. Build the Application

```bash
npm run build
```

This command executes the **build** script defined in the project's `package.json`.

Example:

```json
"scripts": {
  "build": "react-scripts build"
}
```

The build process:

* Compiles the React source code.
* Bundles JavaScript and CSS files.
* Optimizes the application for production.
* Minifies assets.
* Generates static files.

A new **build** directory is created containing the production-ready application.

Typical structure:

```text
build/
│
├── index.html
├── asset-manifest.json
├── static/
│   ├── css/
│   ├── js/
│   └── media/
└── favicon.ico
```

---

# Pipeline Execution Flow

```text
Developer Pushes Code
        │
        ▼
GitLab Detects Commit
        │
        ▼
Reads .gitlab-ci.yml
        │
        ▼
Creates Pipeline
        │
        ▼
Starts Build Stage
        │
        ▼
Starts build_app Job
        │
        ▼
Pulls node:22-alpine Image
        │
        ▼
Creates Temporary Docker Container
        │
        ▼
Runs node --version
        │
        ▼
Runs npm --version
        │
        ▼
Runs npm ci
        │
        ▼
Creates node_modules
        │
        ▼
Runs npm run build
        │
        ▼
Creates build/
        │
        ▼
Job Completed Successfully
        │
        ▼
Pipeline Succeeded
```

---

# What Happens Inside the Docker Container?

```text
Temporary Docker Container
│
├── Clone Repository
│
├── node --version
│
├── npm --version
│
├── npm ci
│      │
│      └── Creates node_modules/
│
├── npm run build
│      │
│      └── Creates build/
│
└── Job Finished
```

After the job completes, the temporary container is removed. Since we have not configured **artifacts**, the generated `build` folder is also discarded when the job finishes.

---

# Expected Pipeline Output

```text
$ node --version
v22.18.0

$ npm --version
10.9.3

$ npm ci

added 1350 packages...

$ npm run build

Creating an optimized production build...

Compiled successfully.

File sizes after gzip:

...

The build folder is ready to be deployed.

Job succeeded
```

---

# Why Use `npm ci` Instead of `npm install`?

| `npm ci`                           | `npm install`                           |
| ---------------------------------- | --------------------------------------- |
| Uses `package-lock.json` exactly   | May update dependency versions          |
| Faster installation                | Slightly slower                         |
| Consistent and reproducible builds | May produce different results over time |
| Recommended for CI/CD pipelines    | Commonly used for local development     |

---

# Summary

In this pipeline, GitLab Runner uses the **`node:22-alpine`** Docker image to create a lightweight execution environment with Node.js and npm pre-installed. The **build** stage contains a single job, **build_app**, which verifies the Node.js and npm versions, installs project dependencies using **`npm ci`**, and builds the React application with **`npm run build`**. During the build process, the `node_modules` directory is created for project dependencies, and the `build` directory is generated as the production-ready output. This demonstrates how GitLab CI/CD can automate the application build process without requiring manual intervention on a developer's machine.
