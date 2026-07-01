# Creating Your First GitLab CI/CD Pipeline

In this example, we'll create a very simple GitLab pipeline that creates a file named `context.txt` and writes **"Hello World"** into it.

This example helps you understand how a GitLab pipeline works before moving on to real-world build and deployment pipelines.

---

# Step 1: Create the Pipeline File

Inside the root of your GitLab repository, create a file named:

```text
.gitlab-ci.yml
```

GitLab automatically looks for this file whenever code is pushed to the repository.

---

# Step 2: Write the Pipeline

```yaml
stages:
  - build

create-file:
  stage: build

  script:
    - echo "Hello World" > context.txt
    - cat context.txt
    - pwd
    - ls -l
```

---

# Understanding Every Part of the Pipeline

## 1. stages

```yaml
stages:
  - build
```

A **stage** is a logical phase of the pipeline.

Think of stages as chapters in a book.

Example:

```text
Build
 ↓
Test
 ↓
Deploy
```

A pipeline can contain multiple stages.

In our example, we have only one stage:

```text
Build
```

Since there is only one stage, the pipeline will execute only the Build phase.

---

## 2. Job

```yaml
create-file:
```

This is the **job name**.

A job is an individual unit of work executed by the GitLab Runner.

Think of a stage as a folder and jobs as the files inside that folder.

Example:

```text
Stage
 ├── Job 1
 ├── Job 2
 └── Job 3
```

Here, the job is named:

```text
create-file
```

You can choose any meaningful name.

Examples:

```text
compile

run-tests

build-image

deploy-app
```

---

## 3. stage

```yaml
stage: build
```

This tells GitLab that the **create-file** job belongs to the **build** stage.

If multiple stages exist, GitLab executes them in the order listed under `stages:`.

Example:

```yaml
stages:
  - build
  - test
  - deploy
```

Execution order:

```text
Build
   ↓
Test
   ↓
Deploy
```

---

## 4. script

```yaml
script:
```

The **script** section contains the commands that the GitLab Runner executes.

Every command is executed in sequence.

Example:

```yaml
script:
  - command1
  - command2
  - command3
```

---

# Step-by-Step Execution of Our Script

### Command 1

```bash
echo "Hello World" > context.txt
```

Creates a file named:

```text
context.txt
```

and writes:

```text
Hello World
```

into it.

---

### Command 2

```bash
cat context.txt
```

Displays the contents of the file.

Output:

```text
Hello World
```

---

### Command 3

```bash
pwd
```

Prints the current working directory inside the Runner's execution environment.

Example:

```text
/builds/group/project
```

---

### Command 4

```bash
ls -l
```

Lists all files in the current directory.

Example output:

```text
context.txt

.gitlab-ci.yml

README.md
```

---

# What Happens When You Push This File?

Suppose you commit the `.gitlab-ci.yml` file and push it to GitLab.

The following happens automatically:

```text
Developer Pushes Code
          │
          ▼
GitLab Detects Push
          │
          ▼
Reads .gitlab-ci.yml
          │
          ▼
Creates Pipeline
          │
          ▼
Creates Build Stage
          │
          ▼
Creates create-file Job
          │
          ▼
Assigns Job to GitLab Runner
          │
          ▼
Runner Starts Docker Container
          │
          ▼
Downloads Repository
          │
          ▼
Runs Script Commands
          │
          ▼
Creates context.txt
          │
          ▼
Displays Output in Job Log
          │
          ▼
Job Succeeds
          │
          ▼
Pipeline Completed
```

---

# Understanding Pipeline Components

## Pipeline

A **pipeline** is the complete automation workflow.

Example:

```text
Pipeline
```

contains one or more stages.

---

## Stage

A **stage** groups related jobs together.

Example:

```text
Build
Test
Deploy
```

Stages execute one after another.

---

## Job

A **job** is an individual task executed by the GitLab Runner.

Example:

```text
create-file
```

A stage can have one or many jobs.

---

## Script (Steps)

The **script** section contains the individual Linux commands that the Runner executes.

Example:

```yaml
script:
  - echo "Hello"
  - pwd
  - ls
```

Each line under `script` is a **step** (or command). They are executed from top to bottom.

---

# Pipeline Hierarchy

The relationship between the different components can be visualized as follows:

```text
Pipeline
│
├── Stage (Build)
│     │
│     └── Job (create-file)
│             │
│             ├── Step 1 → echo "Hello World" > context.txt
│             ├── Step 2 → cat context.txt
│             ├── Step 3 → pwd
│             └── Step 4 → ls -l
```

---

# Execution Order

```text
Pipeline Starts
      │
      ▼
Build Stage Starts
      │
      ▼
create-file Job Starts
      │
      ▼
Step 1 → echo "Hello World" > context.txt
      │
      ▼
Step 2 → cat context.txt
      │
      ▼
Step 3 → pwd
      │
      ▼
Step 4 → ls -l
      │
      ▼
Job Completed
      │
      ▼
Build Stage Completed
      │
      ▼
Pipeline Completed
```

---

# Expected Job Log Output

When you open the job log in GitLab, you'll see output similar to this:

```text
$ echo "Hello World" > context.txt

$ cat context.txt
Hello World

$ pwd
/builds/group/project

$ ls -l
total 8
-rw-r--r-- 1 root root   12 Jun 30 10:15 context.txt
-rw-r--r-- 1 root root  126 Jun 30 10:15 .gitlab-ci.yml
-rw-r--r-- 1 root root   54 Jun 30 10:15 README.md

Job succeeded
```

---

# Key Takeaways

* **Pipeline**: The complete CI/CD workflow.
* **Stage**: A logical phase in the pipeline (for example, Build, Test, Deploy).
* **Job**: A single unit of work within a stage, executed by a GitLab Runner.
* **Script**: A list of Linux commands that the Runner executes.
* **Steps**: Each individual command under the `script` section.

In this example, the GitLab Runner starts a temporary execution environment (typically a Docker container when using the Docker executor), checks out your repository, executes each script command in order, streams the output to GitLab, and then cleans up the environment after the job completes. This simple pipeline forms the foundation for more advanced pipelines that compile code, run tests, build Docker images, and deploy applications.
