# DevSecOps Lesson: Installing and Using Trivy on Ubuntu

## 1. What is Trivy?

[Trivy: Enhancing DevSecOps with Automated Container Security Scanning | by Jake C | Medium](https://images.openai.com/static-rsc-4/wnYpd40udIwL5wJLergqr33Hb1lDqRLGOk6xTpsTwxpnSsSBn_sG_GivN2ZsO-7bN3VdqR2-kVD5k18QOrcATWMa8Ja4xPXDkmEiKLmSLuXFKyEVrZnrO8lTuJe9F1HcbmHED5tgX4VMwoP5W7EVm7rncmYeKBMF0FNWG4SFEu8?purpose=inline)

Trivy is an open-source security scanner developed by Aqua Security. It helps identify security risks in applications, dependencies, container images, filesystems, and infrastructure configuration.

In your DevSecOps project, we'll use Trivy to scan the Docker image of your `.NET 10` application before deploying it.

### What can Trivy detect?

Container image vulnerabilities

Detects known vulnerabilities in operating-system packages and application dependencies inside Docker images.

Dependency vulnerabilities

Checks supported package manifests and lock files, such as NuGet and npm dependencies.

Secret detection

Searches supported targets for exposed credentials, API keys, and other secrets.

Misconfiguration scanning

Checks supported configuration files, including Dockerfiles and Kubernetes manifests, for security misconfigurations.

### How Trivy fits into your DevSecOps pipeline

1\. Developer commits code

2\. GitLab CI builds the Docker image

3\. Trivy scans the image

No blocking findings

Continue toward deployment

Blocking findings

Fail the pipeline

Trivy uses vulnerability databases and package metadata to identify known issues. A finding is not automatically proof that your application is exploitable; you must evaluate severity, package versions, and whether the vulnerable component is actually used.

## 2. Install Trivy on Ubuntu

We'll use the official Trivy APT repository so you can install and update Trivy using Ubuntu's package manager. These are the installation steps documented by Trivy.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.trivy.dev\&sz=32)

Trivy

+1



### Step 1 — Update Ubuntu

Open your terminal and run:

```
sudo apt update
sudo apt install -y wget gnupg
```

### Step 2 — Add the Trivy repository signing key

```
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
```

This lets APT verify packages from the repository.

### Step 3 — Add the Trivy repository

```
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" \
  | sudo tee /etc/apt/sources.list.d/trivy.list
```

### Step 4 — Install Trivy

```
sudo apt update
sudo apt install -y trivy
```

### Step 5 — Verify the installation

```
trivy --version
```

You should see the installed Trivy version.

## 3. Run your first Trivy scan

Let's start with a public image, so you can learn the tool without needing to build your own image first.

```
trivy image nginx:latest
```

The first scan may take some time because Trivy needs to download its vulnerability database and the image layers.

The output lists detected vulnerabilities, including their IDs, severity, affected package and available fixed versions where known.

## 4. Scan your own .NET Docker image

Once you have successfully built your .NET image on Ubuntu, run:

```
docker images
```

If your image is named `devsecops-demo:1.0`, scan it with:

```
trivy image devsecops-demo:1.0
```

To show only high and critical vulnerabilities:

```
trivy image --severity HIGH,CRITICAL devsecops-demo:1.0
```

To make the command return a failure exit code when high or critical vulnerabilities are detected:

```
trivy image --exit-code 1 --severity HIGH,CRITICAL devsecops-demo:1.0
```

This last command is useful in CI/CD pipelines: a matching finding makes the scan fail. It does not mean every finding is automatically exploitable.

## 5. Understand the results

| Severity   | Meaning                            | Typical response                 |
| ---------- | ---------------------------------- | -------------------------------- |
| `CRITICAL` | Extremely serious potential impact | Prioritize immediate remediation |
| `HIGH`     | Significant security risk          | Investigate and fix promptly     |
| `MEDIUM`   | Moderate risk                      | Assess and plan remediation      |
| `LOW`      | Lower-severity issue               | Review and track                 |
| `UNKNOWN`  | Severity not determined            | Investigate the finding          |


# DevSecOps Lesson: What is a CVE, and how is it connected to Trivy?

## 1. What is a CVE?

CVE stands for Common Vulnerabilities and Exposures.

A CVE is a publicly recorded identifier assigned to a known security vulnerability in software, libraries, operating systems, or other components.

Think of a CVE as a unique reference number for a known security weakness. It allows developers, security teams, and tools such as Trivy to refer to the same vulnerability consistently.

For example, a CVE identifier looks like this:

`CVE-2021-44228`

This is the well-known Log4Shell vulnerability in Apache Log4j, a Java logging library.

A CVE record typically includes an identifier, a description, affected software, and references to further information. Severity scores and fixed versions may also be available from vulnerability databases and related advisories.

## 2. How does a CVE get connected to Trivy?

Trivy is a scanner. A CVE is a vulnerability identifier. Trivy checks software components against vulnerability information to determine whether they are affected by known vulnerabilities.

Your Docker image

Contains OS packages and application libraries

Trivy identifies installed packages

Package names and versions

Vulnerability database lookup

Matches packages and versions to known vulnerability records

CVE findings in the scan report

CVE ID, severity, affected package and possible fix

For example, if an image contains a library version known to be affected by a CVE, Trivy can report that vulnerability. The result depends on the image contents, available vulnerability data, and the package ecosystem.

## 3. Understand a Trivy finding

A Trivy report may look similar to this illustrative example:

| Field             | Example           | Meaning                                          |
| ----------------- | ----------------- | ------------------------------------------------ |
| Library           | `example-library` | A detected software package                      |
| Vulnerability     | `CVE-2025-12345`  | Identifier for a reported vulnerability          |
| Severity          | `HIGH`            | Risk classification from the applicable advisory |
| Installed version | `1.2.0`           | Version detected in the image                    |
| Fixed version     | `1.2.5`           | Version containing a fix, if available           |

The CVE and package values above are illustrative, not an actual scan result.

When you see a CVE, investigate the affected package, installed version, available fix, and relevant advisory before deciding how to remediate it. Trivy uses multiple advisory sources, including ecosystem-specific databases for .NET/NuGet packages.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://trivy.dev\&sz=32)

Trivy

+1



## 4. Try it on your Ubuntu machine

Run this command against your own Docker image:

```
trivy image devsecops-demo:1.0
```

To focus on high and critical findings:

```
trivy image --severity HIGH,CRITICAL devsecops-demo:1.0
```

To save the complete scan report as JSON:

```
trivy image --format json --output trivy-report.json devsecops-demo:1.0
```

Look for these fields in the output:

- `VulnerabilityID` — the CVE or advisory identifier.
- `PkgName` — the affected package.
- `InstalledVersion` — the version found.
- `FixedVersion` — the version to upgrade to, if a fix is available.
- `Severity` — the reported severity.

## 5. CVE versus Trivy

| CVE                                    | Trivy                                                |
| -------------------------------------- | ---------------------------------------------------- |
| Identifies a known vulnerability       | Scans software for known vulnerabilities             |
| Provides a common identifier           | Uses vulnerability data from relevant sources        |
| Does not scan your Docker image        | Examines your image and its components               |
| Helps teams discuss and track an issue | Helps identify which issues may affect your software |

Remember: CVE is the vulnerability's identifier; Trivy is the tool that can discover and report it in your software.

Replace your existing `.gitlab-ci.yml` with this simplified version. It removes GitLab SAST, NuGet scanning, npm scanning, SBOM generation, the security policy demo, and the build/test jobs. It keeps only a Trivy Docker image scan.

This pipeline scans the existing Docker image `devsecops-demo:1.0` and fails if it finds HIGH or CRITICAL vulnerabilities.

```

stages:
  - security

variables:
  DOCKER_IMAGE: "devsecops-demo:1.0"
  TRIVY_NO_PROGRESS: "true"
  TRIVY_CACHE_DIR: ".trivycache/"

trivy_image_scan:
  stage: security
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]

  script:
    # Scan the Docker image for HIGH and CRITICAL vulnerabilities
    - trivy image --cache-dir "$TRIVY_CACHE_DIR" --severity HIGH,CRITICAL --exit-code 1 "$DOCKER_IMAGE"

  allow_failure: false

```

## Important: Make the image available to GitLab CI

The YAML above assumes the image already exists in the environment where the GitLab job runs. A Docker image built on your Ubuntu machine is not automatically available to a GitLab runner.

For a reliable GitLab pipeline, push the image to a container registry, such as the GitLab Container Registry, and scan that registry image.

For example, the image reference would look like:

```
registry.gitlab.com/<your-group>/<your-project>/devsecops-demo:1.0
```

You would then update `DOCKER_IMAGE` to that full registry path and configure authentication for a private image.

## How the scan behaves

- No HIGH or CRITICAL findings: the scan passes.
- At least one HIGH or CRITICAL finding: Trivy exits with code `1`, and the job fails.
- MEDIUM and LOW findings: they don't fail this particular scan.
- Image not found or registry authentication failure: the job also fails, so check the logs to distinguish a scanning failure from a vulnerability finding.

One thing to confirm: Have you already pushed `devsecops-demo:1.0` to GitLab Container Registry, or is it currently only on your Ubuntu machine? That determines the exact final configuration you need.


