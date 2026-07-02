# Verifying the Build Output Using a Test Stage

After building the React application and storing the **build** folder as an artifact, the next step is to verify that the build was created successfully.

One simple way to validate the build is to check whether the **`index.html`** file exists inside the **build** directory.

If the file exists, the test should pass.

If the file does not exist, the pipeline should fail.

---

# Updated GitLab CI Pipeline

```yaml
stages:
  - build
  - test

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

verify_build:
  stage: test
  image: node:22-alpine

  script:
    - test -f build/index.html
```

---

# Understanding the Test Stage

## Stage

```yaml
- test
```

A new stage named **test** has been added.

The pipeline execution order becomes:

```text
Build
   │
   ▼
Test
```

GitLab executes the **Test** stage only after the **Build** stage completes successfully.

---

## Test Job

```yaml
verify_build:
```

This job verifies whether the application was built successfully.

---

## Script

```bash
test -f build/index.html
```

The Linux `test` command is used to check conditions.

### Meaning of the options

```text
test
```

Checks whether a condition is true or false.

```text
-f
```

Checks whether the specified path exists **and is a regular file**.

---

## What is Being Checked?

```bash
test -f build/index.html
```

This command checks whether the following file exists:

```text
build/index.html
```

If the file exists:

* Exit code = **0**
* Job succeeds
* Pipeline continues

If the file does not exist:

* Exit code ≠ **0**
* Job fails
* Pipeline stops

---

# Why Does This Work?

During the **Build** stage, GitLab stores the **build/** directory as an artifact.

Before the **Test** stage starts, GitLab automatically downloads the artifacts from the previous stage into the Runner's working directory.

Therefore, the **verify_build** job has access to:

```text
build/
│
├── index.html
├── static/
├── favicon.ico
└── asset-manifest.json
```

It can then verify that `build/index.html` exists.

---

# Pipeline Execution Flow

```text
Developer Pushes Code
          │
          ▼
Build Stage
          │
          ▼
npm ci
          │
          ▼
npm run build
          │
          ▼
Creates build/
          │
          ▼
Uploads build/ as Artifact
          │
          ▼
Test Stage Starts
          │
          ▼
Downloads build/ Artifact
          │
          ▼
Runs:
test -f build/index.html
          │
          ▼
File Exists?
     ┌───────────────┐
     │               │
    Yes             No
     │               │
     ▼               ▼
 Job Passes     Job Fails
```

---

# Successful Pipeline

If `build/index.html` exists:

```text
$ test -f build/index.html

Job succeeded
```

The command produces no output because the condition is true.

---

# Failed Pipeline

If the file is missing:

```text
$ test -f build/index.html

ERROR: Job failed: exit code 1
```

GitLab marks the job and the pipeline as **Failed**.

---

# Making the Output More Readable (Optional)

Instead of using only `test -f`, you can display a message:

```yaml
verify_build:
  stage: test
  image: node:22-alpine

  script:
    - |
      if test -f build/index.html; then
        echo "Build verification passed."
      else
        echo "Build verification failed."
        exit 1
      fi
```

This produces clearer pipeline logs while still failing the job if the file is missing.

---

# Complete Pipeline

```yaml
stages:
  - build
  - test

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

verify_build:
  stage: test
  image: node:22-alpine

  script:
    - test -f build/index.html
```

---

# Summary

In this pipeline, a new **Test** stage is added after the **Build** stage to verify that the React application was built successfully. The **build_app** job generates the production-ready **build/** directory and stores it as a GitLab artifact. Before the **verify_build** job starts, GitLab automatically downloads this artifact. The command `test -f build/index.html` checks whether the `index.html` file exists inside the `build` directory. If the file is found, the command returns a success exit code (`0`), and the job passes. If the file is missing, the command returns a non-zero exit code, causing the job and the pipeline to fail. This simple validation demonstrates how artifacts can be passed between stages and how basic file checks can be used to verify build outputs in a CI pipeline.
