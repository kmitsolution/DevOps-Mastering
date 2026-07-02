# Storing Build Output Using GitLab Artifacts

In the previous pipeline, the React application was successfully built, and the **`build`** folder was created inside the GitLab Runner's Docker container.

However, once the job completed, the Docker container was removed. As a result, the **`build`** folder was also deleted because it existed only inside that temporary container.

To preserve the build output after the job finishes, GitLab provides a feature called **Artifacts**.

---

# What are Artifacts?

**Artifacts** are files or directories that GitLab saves after a job completes successfully.

Artifacts allow you to:

* Preserve build outputs.
* Share files between different stages.
* Download generated files from the GitLab UI.
* Deploy the generated application in later pipeline stages.

For a React application, the **`build`** folder is the most common artifact because it contains the production-ready website.

---

# Why Do We Need Artifacts?

Without artifacts, the pipeline works like this:

```text
GitLab Runner
      │
      ▼
Creates Docker Container
      │
      ▼
Runs npm ci
      │
      ▼
Runs npm run build
      │
      ▼
Creates build/
      │
      ▼
Job Completed
      │
      ▼
Docker Container Deleted
      │
      ▼
build/ Folder Lost ❌
```

The build folder is removed along with the temporary container.

---

# Using Artifacts

Modify the pipeline as follows:

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

  artifacts:
    paths:
      - build/
```

---

# Understanding the Artifacts Section

## artifacts

```yaml
artifacts:
```

This section tells GitLab to preserve files after the job completes.

---

## paths

```yaml
paths:
```

The `paths` keyword specifies which files or directories should be saved.

---

## build/

```yaml
- build/
```

This tells GitLab:

> "After the job finishes, save the entire **build** directory as an artifact."

GitLab compresses the folder and stores it with the pipeline.

---

# Pipeline Execution

```text
Developer Pushes Code
          │
          ▼
GitLab Creates Pipeline
          │
          ▼
Runner Pulls node:22-alpine
          │
          ▼
Creates Docker Container
          │
          ▼
Runs npm ci
          │
          ▼
Creates node_modules/
          │
          ▼
Runs npm run build
          │
          ▼
Creates build/
          │
          ▼
GitLab Saves build/ as Artifact
          │
          ▼
Container Deleted
          │
          ▼
Artifact Still Available ✅
```

---

# What Happens Internally?

```text
Temporary Docker Container
│
├── Repository
├── node_modules/
├── src/
├── build/
│
└── Job Completed
        │
        ▼
GitLab Copies build/
        │
        ▼
Stores Artifact
        │
        ▼
Deletes Docker Container
```

Even though the container is removed, GitLab has already copied the **build** folder to its artifact storage.

---

# Verifying the Artifact

After the pipeline completes successfully:

1. Open your GitLab project.
2. Navigate to **Build → Pipelines**.
3. Open the completed pipeline.
4. Click the **build_app** job.
5. On the job page, look for the **Artifacts** section.
6. Download the artifact ZIP file.
7. Extract it and verify that it contains the **build** folder.

Typical contents:

```text
build/
│
├── index.html
├── asset-manifest.json
├── favicon.ico
└── static/
    ├── css/
    ├── js/
    └── media/
```

This confirms that GitLab successfully stored the production build.

---

# Why Are Artifacts Important?

Artifacts are commonly used to:

* Preserve build outputs.
* Share files with later pipeline stages.
* Deploy applications.
* Store reports (test reports, coverage reports, security scan reports).
* Download generated files from GitLab.

For example:

```text
Build Stage
      │
      ▼
Creates build/
      │
      ▼
Stores Artifact
      │
      ▼
Deploy Stage
      │
      ▼
Downloads build/
      │
      ▼
Deploys Application
```

Without artifacts, the Deploy stage would have nothing to deploy.

---

# Expected Pipeline Output

```text
$ npm ci
added 1350 packages...

$ npm run build

Creating an optimized production build...

Compiled successfully.

The build folder is ready to be deployed.

Uploading artifacts...
build/: found 25 matching files

Uploading artifacts as "archive"...

Job succeeded
```

The message **"Uploading artifacts..."** confirms that GitLab is saving the `build` directory.

---

# Summary

By adding the `artifacts` section to the pipeline, GitLab preserves the **build** directory after the job completes. Although the GitLab Runner deletes the temporary Docker container at the end of the job, the `build` folder is copied to GitLab's artifact storage before the container is removed. These artifacts can be downloaded from the pipeline, shared with later stages such as deployment, or used for further processing. This is a fundamental feature of GitLab CI/CD because it allows build outputs to persist beyond the lifetime of the Runner's temporary execution environment.
