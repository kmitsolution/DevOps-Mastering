# Lesson 1 — DevSecOps Lab Setup

In this lesson, we will **build the foundation for the entire DevSecOps practical course**.

Our goal is not just to install tools. We will create a small application and a GitLab CI/CD pipeline that we will progressively secure in later lessons.

## 1. What We Are Going to Build

Our final DevSecOps lab will look like this:

```text
Developer
    |
    | git push
    v
GitLab Repository
    |
    v
GitLab CI/CD
    |
    +--> Secret Scan
    |
    +--> SAST
    |
    +--> SCA
    |
    +--> Build & Test
    |
    +--> Docker Build
    |
    +--> Container Scan
    |
    +--> SBOM
    |
    +--> IaC Scan
    |
    +--> Kubernetes Security
    |
    +--> DAST
    |
    v
Production
```

We will **add these security stages one by one**, so you can clearly understand what each tool is doing.

---

# 2. Lab Architecture

For the initial lab, let's keep the environment simple.

```text
                    Your Laptop
                         |
                ┌────────┴────────┐
                |                 |
             GitLab            Docker
                |                 |
                |          DevSecOps Tools
                |
          GitLab Runner
                |
                v
        Pipeline Execution
```

You can run the lab using:

* VirtualBox
* Linux VM
* Docker
* GitLab
* GitLab Runner

Since you already have a VirtualBox environment and Kubernetes knowledge, you can use your existing Linux VM.

---

# 3. Prerequisites

Before starting, make sure you have:

### Required

* Git
* Docker
* Linux VM
* GitLab account
* GitLab project
* GitLab Runner

### Later

We will install/use:

* Gitleaks
* SonarQube
* Trivy
* OWASP Dependency-Check
* Checkov
* OWASP ZAP
* Kubernetes
* Helm
* JFrog Artifactory

**Don't install everything now.**

We will introduce each tool when we reach that lesson.

---

# 4. Create the Application

Let's use a very simple application initially.

Since you already work with **ASP.NET Core**, we can use a small .NET API.

Our project structure will eventually look like:

```text
devsecops-demo/
│
├── DevSecOpsDemo/
│   ├── Program.cs
│   └── DevSecOpsDemo.csproj
│
├── tests/
│   └── DevSecOpsDemo.Tests/
│
├── Dockerfile
│
├── .gitignore
│
└── .gitlab-ci.yml
```

The application doesn't need to be complicated.

For example:

```text
GET /hello
```

returns:

```text
Hello from DevSecOps!
```

The application itself is not the focus.

**The CI/CD security pipeline is the focus.**

---

# 5. Create the ASP.NET Core Application

On your Linux machine:

```bash
dotnet new webapi -n DevSecOpsDemo
```

Go inside the project:

```bash
cd DevSecOpsDemo
```

Run it:

```bash
dotnet run
```

You should see something similar to:

```text
Now listening on: http://localhost:5000
```

Open another terminal and test the application.

Depending on the generated project, use one of its available endpoints, or create a simple endpoint such as:

```csharp
app.MapGet("/hello", () => "Hello from DevSecOps!");
```

Then test:

```text
http://localhost:5000/hello
```

Expected:

```text
Hello from DevSecOps!
```

---

# 6. Initialize Git

Inside the project:

```bash
git init
```

Check:

```bash
git status
```

Add files:

```bash
git add .
```

Commit:

```bash
git commit -m "Initial DevSecOps application"
```

---

# 7. Create GitLab Repository

Go to your GitLab account and create a new project.

For example:

```text
Project name:

devsecops-demo
```

Choose:

```text
Visibility: Private
```

Don't initialize the repository with another README if you already have your local project.

You will get a GitLab repository URL similar to:

```text
https://gitlab.com/<username>/devsecops-demo.git
```

Add it as the remote:

```bash
git remote add origin https://gitlab.com/<username>/devsecops-demo.git
```

Verify:

```bash
git remote -v
```

Then push:

```bash
git branch -M main
git push -u origin main
```

Now your application should be available in GitLab.

---

# 8. Create the First CI/CD Pipeline

Create this file in the root of your project:

```text
.gitlab-ci.yml
```

For our **first pipeline**, don't add security yet.

Start with:

```yaml
stages:
  - build
  - test

build_app:
  stage: build
  image: mcr.microsoft.com/dotnet/sdk:10.0

  script:
    - dotnet --version
    - dotnet --info
    - dotnet restore
    - dotnet publish devsecops-demo/devsecops-demo.csproj -c Release -o publish

  artifacts:
    paths:
      - publish/
    expire_in: 1 day


unit_test:
  stage: test
  image: mcr.microsoft.com/dotnet/sdk:10.0

  before_script:
    - dotnet tool install --global trx2junit
    - export PATH="$PATH:/root/.dotnet/tools"

  script:
    - dotnet --version
    - dotnet test --logger "trx;LogFileName=test_results.trx"
    - find . -name "*.trx"
    - find . -name "*.trx" -exec trx2junit {} \;

  artifacts:
    when: always

    paths:
      - "**/*.trx"
      - "**/*.xml"

    reports:
      junit:
        - "**/*.xml"```

Commit:

```bash
git add .gitlab-ci.yml
git commit -m "Add initial CI pipeline"
git push
```

---

# 9. What Happens Now?

GitLab detects:

```text
.gitlab-ci.yml
```

and starts the pipeline.

The pipeline looks like:

```text
Pipeline
   |
   +---- Build
   |
   +---- Test
```

The build job executes:

```bash
dotnet restore
dotnet build
```

The test job executes:

```bash
dotnet test
```

---

# 10. GitLab Runner

Now an important question:

### Who actually executes these commands?

GitLab itself doesn't normally execute your:

```bash
dotnet build
dotnet test
docker build
trivy
gitleaks
```

A **GitLab Runner** executes them.

Architecture:

```text
             GitLab
                |
                | Job
                v
         GitLab Runner
                |
                v
       Execution Environment
                |
        ┌───────┴────────┐
        |                |
     dotnet            tools
```

The runner receives the job and executes the commands defined in `.gitlab-ci.yml`.

---

# 11. Check Runner Status

In GitLab, open:

```text
Project
   ↓
Settings
   ↓
CI/CD
   ↓
Runners
```

You should have an available runner.

Depending on your GitLab setup, you may be able to use GitLab-hosted runners, or you can register your own runner on your Linux VM.

For a serious DevSecOps lab, I recommend eventually having your **own Docker-based GitLab Runner**, because later we will execute tools such as:

```text
Gitleaks
Trivy
Checkov
OWASP ZAP
SonarScanner
```

---

# 12. Why Are We Starting Without Security?

This is very important for understanding DevSecOps.

First establish:

```text
Code
 ↓
Build
 ↓
Test
 ↓
Pipeline
```

Then we progressively transform it into:

```text
Code
 ↓
Secret Scan
 ↓
SAST
 ↓
SCA
 ↓
Build
 ↓
Test
 ↓
Docker
 ↓
Container Scan
 ↓
SBOM
 ↓
IaC Scan
 ↓
Kubernetes Security
 ↓
DAST
 ↓
Production
```

This allows you to understand exactly **where security is being inserted into a DevOps pipeline**.

---

# 13. Your First DevSecOps Exercise

Before moving to Lesson 2, make sure you can demonstrate:

```text
☐ ASP.NET Core application created
☐ Git repository initialized
☐ GitLab repository created
☐ Code pushed to GitLab
☐ .gitlab-ci.yml created
☐ Build job successful
☐ Test job successful
☐ GitLab Runner executing jobs
```

Once this works, your **DevSecOps foundation is ready**.

---

# Next: Lesson 2 — Secret Scanning

In Lesson 2, we will deliberately put a fake credential into the project and see how a security scanner detects it.

We'll use:

**Gitleaks**

The flow will become:

```text
Developer
    ↓
Git Commit
    ↓
Gitleaks
    ↓
🚨 Secret Detected
    ↓
Pipeline FAILED
```

Then we'll modify the pipeline so that **a developer cannot accidentally push credentials through the CI/CD process**.
