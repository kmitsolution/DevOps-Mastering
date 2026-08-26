Yes. One important security point first: **the runner token you pasted is a real authentication credential**. I will not reproduce it in the documentation. If that token is still active, **rotate/revoke it in GitLab and create a new runner authentication token** before using the documentation.

Below is a documentation-style guide based on the commands you used, updated to the **current GitLab Runner authentication-token workflow**. GitLab now recommends runner authentication tokens with the `glrt-` prefix; the older registration-token workflow is deprecated. ([GitLab Docs][1])

# GitLab Runner Installation and Runner Creation

## 1. Introduction

A **GitLab Runner** is an agent that executes CI/CD jobs defined in the `.gitlab-ci.yml` file.

The basic flow is:

```text
Developer
    |
    | git push
    v
GitLab Repository
    |
    | Pipeline
    v
.gitlab-ci.yml
    |
    | Job
    v
GitLab Runner
    |
    | Executes commands
    v
Build / Test / Deploy
```

GitLab provides different runner scopes:

* **Instance Runner** – available across the GitLab instance.
* **Group Runner** – available to projects within a group.
* **Project Runner** – associated with a specific project.

For learning and project-specific CI/CD, a **Project Runner** is a good starting point. ([GitLab Docs][2])

---

# 2. Prerequisites

For this example:

* Ubuntu/Debian Linux server
* Internet connectivity
* GitLab.com account
* GitLab project
* Permission to create a project runner
* `sudo` access on the Linux server

The Linux server can be:

```text
Azure VM
AWS EC2
Virtual Machine
On-Premises Server
Local Ubuntu Machine
```

The runner machine communicates with GitLab over the network.

---

# 3. Architecture

The setup will look like this:

```text
                    GitLab.com
                        |
                        |
                 Project Repository
                        |
                 .gitlab-ci.yml
                        |
                        v
                 GitLab Runner
                        |
              --------------------
              |                  |
           Build              Test
              |                  |
              --------------------
```

The runner does not normally wait for GitLab to connect directly to it.

Instead, the runner continuously communicates with GitLab and requests jobs that it is allowed to execute.

---

# 4. Step 1 – Update Ubuntu Packages

Run:

```bash
sudo apt update
sudo apt upgrade -y
```

Verify the operating system:

```bash
cat /etc/os-release
```

Example:

```text
NAME="Ubuntu"
VERSION="24.04 LTS"
```

---

# 5. Step 2 – Add the GitLab Runner Repository

GitLab provides an official package repository for Debian/Ubuntu-based systems.

Download the repository installation script:

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" -o script.deb.sh
```

GitLab recommends inspecting the downloaded script before executing it:

```bash
less script.deb.sh
```

Run the repository setup script:

```bash
sudo bash script.deb.sh
```

This adds the official GitLab Runner package repository to the Linux machine. ([GitLab Docs][3])

---

# 6. Step 3 – Install GitLab Runner

Install the GitLab Runner package:

```bash
sudo apt install gitlab-runner -y
```

Verify the installation:

```bash
gitlab-runner --version
```

Example:

```text
Version:      18.x.x
Git revision: xxxxxxxx
GO version:   go1.x
```

You can also check the available commands:

```bash
gitlab-runner --help
```

---

# 7. Step 4 – Check GitLab Runner Service

GitLab Runner is installed as a service.

Check its status:

```bash
sudo systemctl status gitlab-runner
```

You should see something similar to:

```text
Active: active (running)
```

If it is not running:

```bash
sudo systemctl start gitlab-runner
```

Enable it to start automatically when the server boots:

```bash
sudo systemctl enable gitlab-runner
```

Verify again:

```bash
sudo systemctl status gitlab-runner
```

---

# 8. Step 5 – Create a Runner in GitLab

Now we create the runner from the GitLab web interface.

Open your GitLab project.

Navigate to:

```text
Project
   |
   +-- Settings
          |
          +-- CI/CD
                 |
                 +-- Runners
```

Expand the **Runners** section.

Select:

```text
Create project runner
```

GitLab's current workflow creates a runner first and then provides an authentication token for registration. ([GitLab Docs][2])

---

# 9. Step 6 – Configure the Project Runner

When creating the runner, GitLab asks for runner configuration.

Example:

```text
Runner description:
ubuntu-gitlab-runner

Tags:
ubuntu

Run untagged jobs:
Enable if required

Operating system:
Linux
```

A runner description is simply a friendly name that helps identify the machine.

For example:

```text
ubuntu-runner-01
```

---

# 10. Understanding Runner Tags

Tags are very important in GitLab CI/CD.

Suppose the runner has this tag:

```text
ubuntu
```

Then a job can request that runner:

```yaml
build:
  tags:
    - ubuntu

  script:
    - echo "Building application"
```

GitLab looks for a runner that has the required tag.

For example:

```text
GitLab Job
    |
    | tag: ubuntu
    v
Runner 1 -> docker
Runner 2 -> windows
Runner 3 -> ubuntu  <-- selected
```

Tags are useful when you have different runners for different workloads.

For example:

```text
docker
linux
windows
aws
azure
kubernetes
dotnet
node
```

---

# 11. Step 7 – Copy the Runner Authentication Token

After creating the runner, GitLab provides a **runner authentication token**.

It normally starts with:

```text
glrt-
```

Example format:

```text
glrt-XXXXXXXXXXXXXXXXXXXXXXXX
```

**Do not commit this token to Git.**

Do not put it directly inside:

```text
.gitlab-ci.yml
GitHub
GitLab repository
Public documentation
Screenshots
```

The token should be treated as a secret credential.

GitLab documents `glrt-` tokens as runner authentication tokens. ([GitLab Docs][4])

---

# 12. Step 8 – Register the Runner

On the Ubuntu server run:

```bash
sudo gitlab-runner register
```

GitLab Runner starts interactive registration.

It will ask for information similar to:

```text
Enter the GitLab instance URL:
https://gitlab.com
```

Enter:

```text
https://gitlab.com
```

Next, enter the authentication token:

```text
Enter the runner authentication token:
glrt-XXXXXXXXXXXXXXXX
```

Use the token generated by your GitLab project.

Next:

```text
Enter a name for the runner:
ubuntu-gitlab-runner
```

Then provide the executor.

For a simple Linux-based runner:

```text
shell
```

GitLab currently documents interactive registration using:

```bash
sudo gitlab-runner register
```

and then asking for the GitLab URL, authentication token, description, tags, maintenance note, and executor. ([GitLab Docs][4])

---

# 13. Shell Executor

For this example we use:

```text
shell
```

The Shell executor runs CI/CD commands directly on the runner machine.

For example, if `.gitlab-ci.yml` contains:

```yaml
build:
  script:
    - echo "Hello GitLab Runner"
    - dotnet --version
```

The commands execute directly on the Ubuntu server.

Architecture:

```text
GitLab
   |
   | CI Job
   v
GitLab Runner
   |
   | Shell Executor
   v
Ubuntu Linux
   |
   +-- echo
   +-- dotnet
   +-- docker
   +-- npm
   +-- kubectl
```

---

# 14. Alternative – Non-Interactive Registration

Instead of answering questions interactively, GitLab Runner can also be registered using command-line parameters.

Example:

```bash
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "$RUNNER_TOKEN" \
  --executor "shell" \
  --description "ubuntu-gitlab-runner"
```

First set the token as an environment variable:

```bash
export RUNNER_TOKEN="glrt-XXXXXXXXXXXXXXXX"
```

Then run:

```bash
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "$RUNNER_TOKEN" \
  --executor "shell" \
  --description "ubuntu-gitlab-runner"
```

Using an environment variable is preferable to permanently putting the token directly into a command or script.

GitLab supports both interactive and non-interactive runner registration. ([GitLab Docs][4])

---

# 15. Step 9 – Verify Runner Registration

After registration, check the configured runners:

```bash
sudo gitlab-runner list
```

Example:

```text
Listing configured runners

ubuntu-gitlab-runner
```

You can also verify the runner:

```bash
sudo gitlab-runner verify
```

A successful verification indicates that the runner can communicate with GitLab.

GitLab provides `list` and `verify` commands specifically for checking registered runners. ([GitLab Docs][5])

---

# 16. Step 10 – Check Runner Configuration

The runner configuration is normally stored in:

```text
/etc/gitlab-runner/config.toml
```

View it:

```bash
sudo cat /etc/gitlab-runner/config.toml
```

Example:

```toml
concurrent = 1

[[runners]]
  name = "ubuntu-gitlab-runner"
  url = "https://gitlab.com"
  token = "glrt-XXXXXXXXXXXXXXXX"
  executor = "shell"
```

The exact contents can vary depending on your runner configuration.

**Important:** The configuration file contains the runner authentication credential. Do not publish it or commit it to source control.

---

# 17. Step 11 – Restart GitLab Runner

After configuration changes, restart the service:

```bash
sudo systemctl restart gitlab-runner
```

Check:

```bash
sudo systemctl status gitlab-runner
```

Expected:

```text
Active: active (running)
```

---

# 18. Step 12 – Verify Runner in GitLab

Go back to:

```text
GitLab Project
   |
   +-- Settings
          |
          +-- CI/CD
                 |
                 +-- Runners
```

You should see your runner.

Example:

```text
ubuntu-gitlab-runner
     |
     +-- Online
     |
     +-- Tags: ubuntu
```

A green/online runner indicates that GitLab can use the runner for jobs.

GitLab notes that an active runner with a green status is available to process CI/CD jobs. ([GitLab Docs][6])

---

# 19. Step 13 – Create a Test Pipeline

Create a file in your GitLab repository:

```text
.gitlab-ci.yml
```

Use this simple pipeline:

```yaml
stages:
  - test

test_runner:
  stage: test

  tags:
    - ubuntu

  script:
    - echo "GitLab Runner is working!"
    - hostname
    - whoami
    - date
```

Commit the file.

GitLab automatically creates a pipeline.

---

# 20. Pipeline Execution

The complete flow is:

```text
Developer
    |
    | git push
    v
GitLab Repository
    |
    | .gitlab-ci.yml
    v
GitLab Pipeline
    |
    | test_runner
    v
Runner searches for jobs
    |
    | tag = ubuntu
    v
ubuntu-gitlab-runner
    |
    | Shell Executor
    v
Commands execute
```

Expected output:

```text
GitLab Runner is working!
```

Then:

```text
hostname
```

prints the hostname of your runner machine.

And:

```text
whoami
```

shows the Linux user under which the job is executing.

---

# 21. Understanding Your Commands

The commands you used can be explained as follows.

### Download repository script

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" -o script.deb.sh
```

Downloads the official GitLab Runner repository setup script.

### Inspect the script

```bash
less script.deb.sh
```

Allows you to inspect the script before execution.

### Add GitLab repository

```bash
sudo bash script.deb.sh
```

Adds the GitLab Runner package repository.

### Install GitLab Runner

```bash
sudo apt install gitlab-runner
```

Installs GitLab Runner.

### Display GitLab Runner commands

```bash
gitlab-runner
```

Displays the available GitLab Runner commands.

### Register runner

```bash
gitlab-runner register
```

Starts interactive runner registration.

### Register using authentication token

Your original command was:

```bash
gitlab-runner register --url https://gitlab.com --token <TOKEN>
```

This associates the Linux runner with GitLab using the runner authentication token.

For security, use:

```bash
sudo gitlab-runner register \
  --url "https://gitlab.com" \
  --token "$RUNNER_TOKEN"
```

### Run the runner manually

```bash
gitlab-runner run
```

This starts the runner process in the foreground.

However, when GitLab Runner is installed as a system service, you normally don't need to run:

```bash
gitlab-runner run
```

manually.

Instead use:

```bash
sudo systemctl status gitlab-runner
sudo systemctl start gitlab-runner
sudo systemctl restart gitlab-runner
```

---

# 22. `gitlab-runner run` vs Systemd Service

There are two concepts to understand.

### Manual execution

```bash
gitlab-runner run
```

The runner process runs in the current terminal.

If you close the terminal, the process can stop.

### Service execution

```bash
sudo systemctl start gitlab-runner
```

The runner runs as a Linux service.

This is the preferred approach for a permanent runner.

Enable it during system startup:

```bash
sudo systemctl enable gitlab-runner
```

Therefore:

```text
Development / Testing
        |
        v
gitlab-runner run
```

can be useful for troubleshooting.

For production:

```text
systemd
   |
   v
gitlab-runner service
   |
   v
GitLab
```

---

# 23. Useful GitLab Runner Commands

### Check version

```bash
gitlab-runner --version
```

### List runners

```bash
sudo gitlab-runner list
```

### Verify runners

```bash
sudo gitlab-runner verify
```

### Register runner

```bash
sudo gitlab-runner register
```

### Start service

```bash
sudo systemctl start gitlab-runner
```

### Stop service

```bash
sudo systemctl stop gitlab-runner
```

### Restart service

```bash
sudo systemctl restart gitlab-runner
```

### Check service

```bash
sudo systemctl status gitlab-runner
```

### Enable at boot

```bash
sudo systemctl enable gitlab-runner
```

### View logs

```bash
sudo journalctl -u gitlab-runner
```

Follow logs in real time:

```bash
sudo journalctl -u gitlab-runner -f
```

---

# 24. Troubleshooting

## Problem 1 – Runner is Offline

Check:

```bash
sudo systemctl status gitlab-runner
```

If stopped:

```bash
sudo systemctl start gitlab-runner
```

Then:

```bash
sudo gitlab-runner verify
```

---

## Problem 2 – Job Stuck

If the pipeline shows:

```text
This job is stuck because the project doesn't have any runners online assigned to it.
```

Check:

1. Runner is online.
2. Runner is assigned to the project.
3. Runner tags match the job.
4. The runner accepts untagged jobs if your job has no tags.

For example, if the runner has:

```text
Tag: ubuntu
```

your job should contain:

```yaml
tags:
  - ubuntu
```

---

# 25. Problem 3 – Shell Command Not Found

Suppose the pipeline contains:

```yaml
script:
  - dotnet --version
```

and GitLab reports:

```text
dotnet: command not found
```

This means the required software is not installed or isn't available in the runner's execution environment/PATH.

Install the required software on the runner machine.

For example:

```bash
dotnet --version
```

should work directly on the runner server before expecting the CI job to use it.

---

# 26. Problem 4 – Permission Denied

Check which user is executing the runner:

```bash
ps aux | grep gitlab-runner
```

Also check:

```bash
sudo -u gitlab-runner whoami
```

The Shell executor generally executes commands using the runner service account, so that account needs appropriate permissions for the operations your pipeline performs.

---

# 27. Problem 5 – Runner Token Exposed

If a runner authentication token is accidentally published:

```text
Git repository
GitHub
GitLab
Screenshot
Documentation
Chat
Public website
```

treat the token as compromised.

Do not continue using the exposed token.

Instead:

1. Remove the exposed credential from public locations.
2. Revoke/delete the affected runner if appropriate.
3. Create a new runner/authentication token.
4. Register the runner again.
5. Protect secrets using GitLab CI/CD variables.

Runner authentication tokens are credentials and should be protected accordingly.

---

# 28. Shell Executor vs Docker Executor

The Shell executor is the easiest executor for learning.

### Shell Executor

```text
GitLab
   |
   v
Runner
   |
   v
Ubuntu
   |
   +-- dotnet
   +-- docker
   +-- npm
```

Commands execute directly on the runner host.

### Docker Executor

```text
GitLab
   |
   v
GitLab Runner
   |
   v
Docker
   |
   +-- Container
          |
          +-- Build
          +-- Test
```

Docker executor provides more isolation and reproducibility.

For a DevOps learning path, a good progression is:

```text
1. Shell Executor
        |
        v
2. Docker Executor
        |
        v
3. Kubernetes Executor
```

---

# 29. Recommended Learning Exercise

After successfully creating the runner, create the following pipeline:

```yaml
stages:
  - build
  - test

build:
  stage: build

  tags:
    - ubuntu

  script:
    - echo "Building application"
    - hostname
    - whoami

test:
  stage: test

  tags:
    - ubuntu

  script:
    - echo "Running tests"
    - echo "Tests completed successfully"
```

The pipeline will execute:

```text
Pipeline
   |
   +---- build
   |       |
   |       +-- Building application
   |
   +---- test
           |
           +-- Running tests
```

This demonstrates the relationship between:

```text
GitLab
  |
  +-- Repository
  |
  +-- .gitlab-ci.yml
  |
  +-- Pipeline
  |
  +-- Job
  |
  +-- Runner
  |
  +-- Executor
  |
  +-- Script
```

---

# 30. Final Setup Checklist

* [ ] Create Linux/Ubuntu VM
* [ ] Update Linux packages
* [ ] Add GitLab Runner repository
* [ ] Install GitLab Runner
* [ ] Verify GitLab Runner version
* [ ] Check GitLab Runner systemd service
* [ ] Create a project runner in GitLab
* [ ] Configure runner tags
* [ ] Generate runner authentication token
* [ ] Register runner on Linux
* [ ] Select Shell executor
* [ ] Verify runner using `gitlab-runner verify`
* [ ] Check runner status in GitLab
* [ ] Create `.gitlab-ci.yml`
* [ ] Run a test pipeline
* [ ] Verify that the job executes on your Linux runner

---

# 31. Key Commands – Quick Reference

```bash
# Add GitLab Runner repository
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" -o script.deb.sh

# Inspect script
less script.deb.sh

# Configure repository
sudo bash script.deb.sh

# Install GitLab Runner
sudo apt install gitlab-runner -y

# Check version
gitlab-runner --version

# Check service
sudo systemctl status gitlab-runner

# Register runner
sudo gitlab-runner register

# List runners
sudo gitlab-runner list

# Verify runner
sudo gitlab-runner verify

# Restart runner
sudo systemctl restart gitlab-runner

# Enable runner at boot
sudo systemctl enable gitlab-runner

# View logs
sudo journalctl -u gitlab-runner -f
```

---

# 32. Important Note About Runner Registration Tokens

Older GitLab tutorials often show:

```bash
gitlab-runner register \
  --registration-token "XXXX"
```

This is the **legacy runner registration-token workflow**.

The current recommended workflow is:

```bash
gitlab-runner register \
  --url "https://gitlab.com" \
  --token "$RUNNER_TOKEN"
```

where `$RUNNER_TOKEN` contains a runner authentication token beginning with:

```text
glrt-
```

GitLab has deprecated registration tokens and recommends the newer runner creation/authentication workflow. ([GitLab Docs][1])

---

# 33. Official GitLab Documentation

GitLab Runner installation:

[Install GitLab Runner on Linux](https://docs.gitlab.com/runner/install/linux-repository/?utm_source=chatgpt.com)

Create and register a project runner:

[Create, register, and run a project runner](https://docs.gitlab.com/tutorials/create_register_first_runner/?utm_source=chatgpt.com)

Runner registration:

[Registering GitLab Runners](https://docs.gitlab.com/runner/register/?utm_source=chatgpt.com)

Runner management:

[Manage GitLab Runners](https://docs.gitlab.com/ci/runners/runners_scope/?utm_source=chatgpt.com)

---

# Conclusion

A GitLab Runner is the machine/agent that actually executes the CI/CD jobs defined in `.gitlab-ci.yml`.

The complete process is:

```text
1. Install GitLab Runner
          |
          v
2. Create Runner in GitLab
          |
          v
3. Get Authentication Token
          |
          v
4. Register Runner
          |
          v
5. Select Executor
          |
          v
6. Runner becomes Online
          |
          v
7. Create .gitlab-ci.yml
          |
          v
8. Push Code
          |
          v
9. GitLab creates Pipeline
          |
          v
10. Runner picks up Job
          |
          v
11. Runner executes Script
```

This forms the foundation for the next topics:

```text
GitLab Runner
      |
      +-- Shell Executor
      |
      +-- Docker Executor
      |
      +-- Docker-in-Docker
      |
      +-- GitLab Runner Tags
      |
      +-- Variables & Secrets
      |
      +-- Artifacts
      |
      +-- Cache
      |
      +-- Parallel Jobs
      |
      +-- Protected Runners
      |
      +-- Autoscaling
      |
      +-- Kubernetes Runner
```

[1]: https://docs.gitlab.com/ci/runners/runners_scope/?utm_source=chatgpt.com "Manage runners | GitLab Docs"
[2]: https://docs.gitlab.com/tutorials/create_register_first_runner/?utm_source=chatgpt.com "Tutorial: Create, register, and run your own project runner | GitLab Docs"
[3]: https://docs.gitlab.com/runner/install/linux-repository/?utm_source=chatgpt.com "Install GitLab Runner using the official GitLab repositories | GitLab Docs"
[4]: https://docs.gitlab.com/runner/register/?utm_source=chatgpt.com "Registering runners | GitLab Docs"
[5]: https://docs.gitlab.com/runner/commands/?utm_source=chatgpt.com "GitLab Runner commands | GitLab Docs"
[6]: https://docs.gitlab.com/ci/quick_start/?utm_source=chatgpt.com "Tutorial: Create and run your first GitLab CI/CD pipeline | GitLab Docs"
