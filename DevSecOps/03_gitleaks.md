# Lesson 2 — Secret Scanning with Gitleaks

In Lesson 1, we created our basic CI/CD pipeline:

```text
Git Push
   ↓
Build
   ↓
Test
```

Now we are going to introduce our **first DevSecOps security control: Secret Scanning**.

We will use **Gitleaks** and deliberately create a fake secret in a file called `test-secret.txt`.

The goal is to see the complete security flow:

```text
Developer
    ↓
Git Push
    ↓
Gitleaks Secret Scan
    ↓
Secret Found?
   ↙       ↘
 YES        NO
 ↓           ↓
FAIL       BUILD
 ❌          ↓
           TEST
```

---

# 1. What is Secret Scanning?

Secret scanning is the process of automatically looking for sensitive information such as:

* Passwords
* API keys
* Access tokens
* Cloud credentials
* Database credentials
* SSH private keys
* Authentication tokens
* Connection strings

For example, a developer might accidentally write:

```text
DB_PASSWORD=MySuperSecretPassword
```

and commit it to Git.

That creates a security risk because anyone who can access the repository may potentially obtain the credential.

---

# 2. Why is Secret Scanning Important?

Consider this situation:

```text
Developer
    ↓
Writes application code
    ↓
Accidentally adds API key
    ↓
git commit
    ↓
git push
    ↓
GitLab
```

Without secret scanning:

```text
Secret
   ↓
Git Repository
   ↓
CI/CD
   ↓
Docker Image
   ↓
Production
```

With DevSecOps:

```text
Secret
   ↓
Git Push
   ↓
Gitleaks
   ↓
🚨 SECRET DETECTED
   ↓
Pipeline FAILED
```

The pipeline prevents the application from continuing.

This is our first **security gate**.

---

# 3. What is Gitleaks?

**Gitleaks** is an open-source tool that scans source code and Git repositories for credentials and other sensitive information.

Conceptually:

```text
Repository
     ↓
   Gitleaks
     ↓
┌────┴─────┐
│          │
No Leak   Leak
│          │
↓          ↓
PASS      FAIL
```

Gitleaks uses rules and patterns to identify strings that look like secrets.

---

# 4. Create a Test Secret

For this lesson, we are intentionally going to create a **fake secret**.

> ⚠️ Never use a real password, API key, AWS key, GitHub token, or production credential for this exercise.

At the root of your project, create:

```text
test-secret.txt
```

Your project might now look like:

```text
devsecops-demo/
│
├── devsecops-demo/
│   ├── Program.cs
│   ├── appsettings.json
│   └── devsecops-demo.csproj
│
├── tests/
│   └── tests.csproj
│
├── test-secret.txt
│
└── .gitlab-ci.yml
```

Put the following **fake token** into `test-secret.txt`:

```text
github_token=ghp_123456789012345678901234567890123456
```

Again, this is deliberately fake test data.

---

# 5. Test Gitleaks Locally

Before putting the scanner into GitLab, it's useful to test it locally.

If Gitleaks is installed on your machine, run:

```bash
gitleaks detect --source . --verbose
```

Gitleaks scans the current directory.

You should get a result indicating that a potential secret was detected.

The exact output can vary depending on the Gitleaks version and rules.

---

# 6. Add Gitleaks to GitLab CI

Now we add a new stage:

```yaml
stages:
  - security
  - build
  - test
```

Our pipeline becomes:

```text
security
   ↓
build
   ↓
test
```

Create/update `.gitlab-ci.yml`:

```yaml
stages:
  - security
  - build
  - test


secret_scan:
  stage: security

  image:
    name: zricethezav/gitleaks:latest
    entrypoint: [""]

  script:
    - gitleaks version
    - echo "Running secret scan..."
    - gitleaks detect --source . --verbose --exit-code 1


build_app:
  stage: build

  image: mcr.microsoft.com/dotnet/sdk:10.0

  script:
    - dotnet --version
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
    - dotnet tool install --global dotnet-reportgenerator-globaltool
    - export PATH="$PATH:/root/.dotnet/tools"

  script:
    - dotnet --version
    - dotnet test --logger "trx;LogFileName=test_results.trx" --collect:"XPlat Code Coverage"
    - find . -name "*.trx"
    - find . -name "coverage.cobertura.xml"
    - find . -name "*.trx" -exec trx2junit {} \;

    - |
      reportgenerator \
        "-reports:**/coverage.cobertura.xml" \
        "-targetdir:coveragereport" \
        "-reporttypes:HtmlInline;Cobertura"

  artifacts:
    when: always

    paths:
      - coveragereport/
      - "**/*.trx"
      - "**/*.xml"

    reports:
      junit:
        - "**/*.xml"

      coverage_report:
        coverage_format: cobertura
        path: "**/coverage.cobertura.xml"

    expire_in: 7 days
```

---

# 7. Understand the Gitleaks Job

Let's understand each section.

### Job name

```yaml
secret_scan:
```

This is the GitLab CI job name.

---

### Stage

```yaml
stage: security
```

This places the job in the security stage.

Because `security` comes before `build`:

```yaml
stages:
  - security
  - build
  - test
```

GitLab will run security first.

---

### Gitleaks Docker Image

```yaml
image:
  name: zricethezav/gitleaks:latest
  entrypoint: [""]
```

GitLab Runner starts a container containing Gitleaks.

Conceptually:

```text
GitLab Runner
      ↓
Gitleaks Container
      ↓
Repository
      ↓
Scan
```

---

# 8. Gitleaks Command

The most important command is:

```bash
gitleaks detect --source . --verbose --exit-code 1
```

Let's break it down.

### `gitleaks`

Runs Gitleaks.

### `detect`

Tells Gitleaks to detect potential secrets.

### `--source .`

Scan the current directory.

### `--verbose`

Show detailed output.

### `--exit-code 1`

This is particularly important for CI/CD.

It means:

> If Gitleaks detects a leak, return exit code `1`.

GitLab interprets a non-zero exit code as a failed job.

---

# 9. Commit and Push

Now commit the test secret and pipeline:

```bash
git add .
git commit -m "Add Gitleaks secret scanning"
git push
```

Go to:

```text
GitLab
   ↓
Your Project
   ↓
Build
   ↓
Pipelines
```

You should see:

```text
Pipeline
   │
   ├── secret_scan
   ├── build_app
   └── unit_test
```

---

# 10. Expected Result — Secret Detected

Because we deliberately created:

```text
test-secret.txt
```

containing:

```text
github_token=ghp_123456789012345678901234567890123456
```

Gitleaks should detect it.

The pipeline should behave approximately like:

```text
                    Git Push
                       ↓
                ┌──────────────┐
                │ secret_scan   │
                │   Gitleaks    │
                └──────┬───────┘
                       ↓
                🚨 Secret Found
                       ↓
                 Exit Code 1
                       ↓
                ❌ JOB FAILED
                       ↓
                 Pipeline Stops
```

The later stages should not proceed.

---

# 11. Why Does the Pipeline Stop?

GitLab executes stages sequentially:

```text
security
   ↓
build
   ↓
test
```

If the security stage fails:

```text
security ❌
    ↓
build ⏭️
    ↓
test ⏭️
```

This is our first **DevSecOps security gate**.

The principle is:

> **Don't allow potentially compromised code to continue through the pipeline.**

---

# 12. Now Remove the Test Secret

After confirming that Gitleaks detects it, remove the test file:

```bash
git rm test-secret.txt
```

Then:

```bash
git add .
git commit -m "Remove test secret"
git push
```

Now the pipeline should become:

```text
Git Push
   ↓
Gitleaks
   ↓
✅ No leaks found
   ↓
Build
   ↓
Test
   ↓
Pipeline SUCCESS
```

---

# 13. Important: Deleting a Secret Is Not Enough

This is an extremely important DevSecOps concept.

Imagine:

```text
Commit 1
   ↓
Secret added 🔴
   ↓
Commit 2
   ↓
Secret deleted
```

The secret may still exist in:

```text
Commit 1
```

Git stores history.

Therefore:

```text
Current source code
        ≠
Complete Git history
```

If a **real credential** is ever committed, simply deleting the file is not enough.

You should:

```text
1. Revoke the credential
        ↓
2. Rotate the credential
        ↓
3. Remove the secret from repository/history
        ↓
4. Investigate whether it was used
        ↓
5. Store the new credential securely
```

For our lab, we're using only fake credentials.

---

# 14. Why Not Put Passwords in `.gitlab-ci.yml`?

Never do this:

```yaml
variables:
  DB_PASSWORD: "MyRealPassword"
```

The password is now part of the repository.

Instead, use GitLab's protected/masked CI/CD variables or an external secrets manager.

Conceptually:

```text
GitLab
   │
   └── Settings
          ↓
       CI/CD Variables
          ↓
    ┌───────────────┐
    │ DB_PASSWORD   │
    │ API_TOKEN     │
    │ DB_USERNAME   │
    └───────────────┘
```

Later in the course we will cover:

```text
GitLab Variables
       ↓
Azure Key Vault
       ↓
HashiCorp Vault
       ↓
Kubernetes Secrets
```

and discuss when each approach makes sense.

---

# 15. What If Gitleaks Finds a False Positive?

Sometimes a scanner may flag something that isn't actually a secret.

For example:

```text
password=example
```

might simply be test data.

Don't immediately disable the scanner.

Instead:

```text
Finding
   ↓
Investigate
   ↓
Is it a real secret?
   ↓
 ┌───────┴────────┐
 YES              NO
 ↓                 ↓
Rotate          Configure
secret          exception
```

Later we will create a `.gitleaks.toml` configuration to handle legitimate exceptions.

---

# 16. Our Pipeline After Lesson 2

We have now transformed our pipeline from:

```text
Git Push
   ↓
Build
   ↓
Test
```

into:

```text
                    Git Push
                       ↓
              ┌────────────────┐
              │ Secret Scanning│
              │    Gitleaks    │
              └───────┬────────┘
                      ↓
                 Security Gate
                      ↓
              ┌───────┴────────┐
              ↓                ↓
            PASS              FAIL
              ↓                ↓
            Build             STOP
              ↓
            Test
```

This is the beginning of our actual **DevSecOps pipeline**.

---

# 17. Lesson 2 — Key Takeaways

By the end of this lesson, you should be able to explain:

### What is Secret Scanning?

Automatically searching source code and Git repositories for potentially sensitive credentials.

### What is Gitleaks?

A tool that detects secrets using predefined detection rules and patterns.

### What is a Security Gate?

A condition that must pass before the CI/CD pipeline can continue.

### Why should secrets not be stored in Git?

Because Git preserves repository history, and exposed credentials can be copied, abused, or remain accessible even after deletion.

### What should you do if a real secret is exposed?

**Revoke → Rotate → Remove → Investigate → Securely store the replacement.**

---

# Final Lab State

Your project should now have:

```text
devsecops-demo/
│
├── devsecops-demo/
│   ├── Program.cs
│   ├── appsettings.json
│   └── devsecops-demo.csproj
│
├── tests/
│   └── tests.csproj
│
└── .gitlab-ci.yml
```

And your GitLab pipeline:

```text
┌─────────────────────────┐
│ SECURITY                │
│                         │
│ Gitleaks                │
│ Secret Scanning         │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ BUILD                   │
│                         │
│ .NET 10                 │
│ dotnet publish          │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│ TEST                    │
│                         │
│ Unit Test               │
│ TRX                     │
│ Code Coverage           │
└─────────────────────────┘
```

**Next: Lesson 3 — SAST with SonarQube.** We will add static code security analysis to this same pipeline and create a **Quality/Security Gate** so that vulnerable code can automatically fail the GitLab pipeline.
