# DevSecOps Fundamentals

## 1. What is DevSecOps?

**DevSecOps = Development + Security + Operations**

It is an approach where **security is integrated into every stage of the software development and deployment lifecycle**, rather than checking security only after the application is developed.

### Traditional approach

```text
Developer
   ↓
Code
   ↓
Build
   ↓
Test
   ↓
Deploy
   ↓
Security Testing
   ↓
Production
```

The problem is that security issues are discovered **very late**.

For example:

```text
Developer writes vulnerable code
          ↓
Application is developed
          ↓
Application is deployed
          ↓
Security team discovers vulnerability
          ↓
Developer fixes code
          ↓
Build again
          ↓
Deploy again
```

This is expensive and slow.

---

# 2. DevOps vs DevSecOps

### DevOps

```text
PLAN
 ↓
CODE
 ↓
BUILD
 ↓
TEST
 ↓
RELEASE
 ↓
DEPLOY
 ↓
OPERATE
 ↓
MONITOR
```

### DevSecOps

Security is integrated into every stage:

```text
              SECURITY
                  ↓
PLAN → CODE → BUILD → TEST → RELEASE → DEPLOY → OPERATE
       ↓       ↓       ↓       ↓         ↓
     SAST     SCA    Secrets   DAST      Runtime
```

The important idea is:

> **Security is everyone's responsibility, not only the security team's responsibility.**

---

# 3. Why Do We Need DevSecOps?

Modern applications are very complex.

A single application might contain:

```text
Application
│
├── .NET / Java / Node.js
│
├── Third-party libraries
│
├── Docker containers
│
├── Kubernetes
│
├── Terraform
│
├── Cloud infrastructure
│
├── APIs
│
└── Databases
```

Every component can introduce security risks.

For example:

```text
Source Code
    ↓
Vulnerable Code

Dependency
    ↓
Known CVE

Docker Image
    ↓
Vulnerable OS Package

Terraform
    ↓
Public Security Group

Kubernetes
    ↓
Privileged Container

Git Repository
    ↓
Leaked Password
```

DevSecOps attempts to identify these problems **before they reach production**.

---

# 4. The Shift-Left Security Concept

One of the most important concepts in DevSecOps is:

## Shift Left

Traditional security:

```text
Development ------------------------→ Production
                                      ↓
                                   Security
```

DevSecOps:

```text
Development
    ↓
Security
    ↓
Build
    ↓
Security
    ↓
Test
    ↓
Security
    ↓
Deploy
    ↓
Security
    ↓
Production
```

We move security activities **earlier in the lifecycle**.

### Example

Suppose a developer commits:

```python
query = "SELECT * FROM users WHERE id=" + user_id
```

A SAST tool can detect a potential SQL Injection **during the CI pipeline**.

Instead of discovering it after production deployment, we discover it within minutes of the code being committed.

---

# 5. DevSecOps Lifecycle

A typical DevSecOps lifecycle looks like this:

```text
                 ┌─────────────┐
                 │     PLAN    │
                 └──────┬──────┘
                        ↓
                 ┌─────────────┐
                 │     CODE    │
                 └──────┬──────┘
                        ↓
                 ┌─────────────┐
                 │    BUILD    │
                 └──────┬──────┘
                        ↓
                 ┌─────────────┐
                 │    TEST     │
                 └──────┬──────┘
                        ↓
                 ┌─────────────┐
                 │   RELEASE   │
                 └──────┬──────┘
                        ↓
                 ┌─────────────┐
                 │   DEPLOY    │
                 └──────┬──────┘
                        ↓
                 ┌─────────────┐
                 │   OPERATE   │
                 └──────┬──────┘
                        ↓
                 ┌─────────────┐
                 │   MONITOR   │
                 └──────┬──────┘
                        │
                        └──────────→ Feedback
```

Security is applied across all these stages.

---

# 6. What Security Checks Are Performed?

Let's map security tools to the lifecycle.

| Stage          | Security Activity        | Example Tool               |
| -------------- | ------------------------ | -------------------------- |
| Plan           | Threat modeling          | Threat modeling techniques |
| Code           | SAST                     | SonarQube, Semgrep         |
| Code           | Secret scanning          | Gitleaks                   |
| Build          | SCA                      | Dependency-Check, Trivy    |
| Build          | Container scanning       | Trivy                      |
| Package        | Artifact security        | JFrog                      |
| Infrastructure | IaC scanning             | Checkov                    |
| Kubernetes     | Configuration scanning   | Trivy, Kubescape           |
| Test           | DAST                     | OWASP ZAP                  |
| Deploy         | Policy enforcement       | OPA / Kyverno              |
| Runtime        | Monitoring               | Falco                      |
| Production     | Vulnerability management | Various platforms          |

---

# 7. Important DevSecOps Terminology

You should become comfortable with these terms.

### SAST

**Static Application Security Testing**

Analyzes source code.

```text
Source Code
     ↓
SAST
     ↓
Security Vulnerabilities
```

Example:

**SonarQube**

---

### DAST

**Dynamic Application Security Testing**

Tests a running application.

```text
Running Application
        ↓
       DAST
        ↓
Security Vulnerabilities
```

Example:

**OWASP ZAP**

---

### SCA

**Software Composition Analysis**

Analyzes third-party dependencies.

```text
Application
     ↓
Dependencies
     ↓
SCA
     ↓
Known Vulnerabilities
```

---

### Secret Scanning

Looks for accidentally exposed credentials.

```text
Git Repository
      ↓
Secret Scanner
      ↓
API Key / Password / Token
```

Example:

**Gitleaks**

---

### Container Scanning

Analyzes container images.

```text
Docker Image
     ↓
   Trivy
     ↓
CVE / Malware / Secrets / Misconfiguration
```

---

### IaC Scanning

Analyzes Infrastructure as Code.

For example:

```text
Terraform
   ↓
Checkov
   ↓
Security Misconfiguration
```

---

# 8. What is a Vulnerability?

A **vulnerability** is a weakness that can potentially be exploited.

Example:

```text
Application
     ↓
SQL Injection vulnerability
     ↓
Attacker manipulates SQL query
     ↓
Unauthorized database access
```

Another example:

```text
AWS Security Group
       ↓
Port 22
       ↓
0.0.0.0/0
       ↓
SSH exposed to Internet
```

That configuration represents a security risk.

---

# 9. What is a CVE?

**CVE = Common Vulnerabilities and Exposures**

A CVE is a publicly identified security vulnerability.

For example:

```text
CVE-2026-XXXX
```

You will frequently see CVEs when using tools such as Trivy or dependency scanners.

Example:

```text
Library: example-library
Version: 1.2.3

CVE-2026-XXXX
Severity: HIGH
```

The scanner tells you:

> "This particular software version has a known security vulnerability."

---

# 10. What is CVSS?

**CVSS = Common Vulnerability Scoring System**

It provides a severity score for vulnerabilities.

Generally:

```text
0.0        → None
0.1–3.9    → Low
4.0–6.9    → Medium
7.0–8.9    → High
9.0–10.0   → Critical
```

For example:

```text
CVE-XXXX
CVSS: 9.8
Severity: CRITICAL
```

A DevSecOps pipeline can use this information to decide whether the pipeline should continue.

---

# 11. Security Gate

This is a very important DevSecOps concept.

Imagine your GitLab pipeline:

```text
Code
 ↓
Build
 ↓
SAST
 ↓
Security Gate
```

Suppose SAST finds:

```text
Critical = 2
High     = 5
```

Your security policy might say:

```text
Critical > 0
      ↓
Pipeline FAILED
```

So:

```text
Developer
    ↓
Git Push
    ↓
GitLab Pipeline
    ↓
SAST
    ↓
CRITICAL vulnerability
    ↓
❌ PIPELINE FAILED
```

The vulnerable application never reaches production.

---

# 12. DevSecOps and the Shared Responsibility Model

In DevOps, developers, operations and infrastructure teams work together.

In DevSecOps:

```text
Developer
     +
Operations
     +
Security
     +
DevOps
     ↓
Shared Security Responsibility
```

Developers should understand secure coding.

DevOps engineers should understand:

* Secure CI/CD
* Secrets
* Container security
* Kubernetes security
* IaC security

Security teams should work with developers and operations rather than acting only as a final approval gate.

---

# 13. DevSecOps vs Traditional Security

| Traditional Security        | DevSecOps                     |
| --------------------------- | ----------------------------- |
| Security at the end         | Security throughout lifecycle |
| Manual testing              | Automated testing             |
| Security team owns security | Shared responsibility         |
| Find problems late          | Find problems early           |
| Long remediation cycle      | Faster remediation            |
| Manual approvals            | Automated security gates      |
| Production-focused          | Developer-to-production       |

---

# 14. A Real DevSecOps Pipeline

Eventually, your pipeline could look like this:

```text
Developer
    ↓
Git Push
    ↓
┌───────────────────────┐
│ Secret Scanning       │
│ Gitleaks              │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ SAST                  │
│ SonarQube             │
└───────────┬───────────┘
            ↓
┌───────────────────────┐
│ SCA                   │
│ Dependency Scanning   │
└───────────┬───────────┘
            ↓
          Build
            ↓
        Unit Tests
            ↓
       Docker Build
            ↓
┌───────────────────────┐
│ Container Scan        │
│ Trivy                 │
└───────────┬───────────┘
            ↓
           SBOM
            ↓
        JFrog Registry
            ↓
┌───────────────────────┐
│ IaC Scan              │
│ Checkov               │
└───────────┬───────────┘
            ↓
       Kubernetes
            ↓
┌───────────────────────┐
│ Kubernetes Security   │
└───────────┬───────────┘
            ↓
          Deploy
            ↓
┌───────────────────────┐
│ DAST                  │
│ OWASP ZAP             │
└───────────┬───────────┘
            ↓
       Production
```

This is the architecture we can build **hands-on**, step by step.

## Next step

I recommend we now start the practical lab with:

**Lesson 1: Set up the DevSecOps Lab — GitLab + Sample Application + GitLab Runner**

Then we will add **Gitleaks → SonarQube/SAST → SCA → Trivy → SBOM → Checkov → Kubernetes Security → OWASP ZAP**, one stage at a time.
