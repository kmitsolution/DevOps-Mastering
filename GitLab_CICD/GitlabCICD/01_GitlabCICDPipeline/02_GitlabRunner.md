# GitLab Runner

## What is a GitLab Runner?

A **GitLab Runner** is an application (or agent) that executes the jobs defined in your **`.gitlab-ci.yml`** file.

GitLab itself **does not execute your build, test, or deployment commands**. Instead, GitLab sends the job to a registered **GitLab Runner**, which performs the actual work.

Think of it this way:

* **GitLab Server** → Manages repositories, pipelines, users, and jobs.
* **GitLab Runner** → Executes the jobs.

Without a runner, a pipeline can be created, but none of its jobs will run.

---

## How Does a GitLab Runner Work?

When a developer pushes code to a GitLab repository, the following process occurs:

1. GitLab detects the code change.
2. GitLab creates a new pipeline.
3. The pipeline contains one or more jobs.
4. GitLab assigns each job to an available GitLab Runner.
5. The Runner downloads the project source code.
6. The Runner executes the commands specified in the **`script`** section of the job.
7. The Runner uploads logs, artifacts, and job status back to GitLab.
8. GitLab displays the results in the pipeline interface.

---

## What Actually Happens Inside the Runner?

A GitLab Runner does not simply execute commands directly. The execution depends on the **executor** configured for that runner.

The most common executor is the **Docker Executor**.

With the Docker executor, the process is:

1. GitLab sends the job to the Runner.
2. The Runner checks the Docker image specified in the job (for example, `ubuntu:22.04`, `python:3.12`, or `maven:3.9`).
3. If the image is not available locally, the Runner pulls it from a container registry.
4. The Runner creates a **temporary Docker container** from that image.
5. The project source code is copied or mounted into the container.
6. The Runner executes the Linux commands from the `script` section **inside the container**.
7. After the job finishes, the container is stopped and removed (unless configured otherwise).

This ensures every pipeline job runs in a clean, isolated, and reproducible environment.

---

## Example

Suppose your `.gitlab-ci.yml` file contains:

```yaml
build:
  image: ubuntu:22.04

  script:
    - pwd
    - ls
    - echo "Building Application"
```

The Runner performs these steps:

* Receives the job from GitLab.
* Pulls the `ubuntu:22.04` image (if needed).
* Creates a new Docker container.
* Downloads the repository into the container.
* Runs the Linux commands:

  * `pwd`
  * `ls`
  * `echo "Building Application"`
* Sends the output back to GitLab.
* Deletes the temporary container.

---

## Why Does the Runner Use Docker?

Running jobs inside Docker containers provides several advantages:

* **Isolation:** Each job runs in its own container without affecting other jobs.
* **Consistency:** Every pipeline uses the same environment regardless of the host machine.
* **Clean Environment:** Each job starts with a fresh container, avoiding leftover files or processes.
* **Portability:** The same pipeline behaves consistently on different machines.

---

## Runner Execution Flow

```text
Developer Pushes Code
          │
          ▼
      GitLab Server
          │
          ▼
    Pipeline Created
          │
          ▼
Job Assigned to GitLab Runner
          │
          ▼
Runner Pulls Docker Image
          │
          ▼
Creates Temporary Docker Container
          │
          ▼
Downloads Repository
          │
          ▼
Runs Linux Commands from script
          │
          ▼
Uploads Logs & Artifacts to GitLab
          │
          ▼
Deletes Container
          │
          ▼
Pipeline Completed
```

---

## Key Points to Remember

* A **GitLab Runner** is the execution engine for GitLab CI/CD jobs.
* GitLab creates and manages pipelines, while the Runner performs the actual execution.
* With the **Docker executor**, the Runner creates a temporary Docker container for each job.
* The Runner downloads the project, executes the Linux commands defined in the `script` section, collects the results, and reports them back to GitLab.
* After the job completes, the temporary container is usually removed, ensuring a clean environment for the next pipeline run.
