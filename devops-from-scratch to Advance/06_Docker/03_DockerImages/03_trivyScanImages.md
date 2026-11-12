## Trivy is one of the most commonly used container vulnerability scanners. It can scan:

* Docker Images
* Running Containers
* Filesystems
* Git Repositories
* Kubernetes
* IaC files (Terraform, Kubernetes YAML, Dockerfile)
* Secrets
* SBOMs ([Fossies][1])

---

# Step 1: Install Trivy

### Ubuntu

```bash
sudo apt-get update
sudo apt-get install wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
| sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] \
https://aquasecurity.github.io/trivy-repo/deb generic main" \
| sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt-get update

sudo apt-get install trivy
```

Verify:

```bash
trivy --version
```

---

# Step 2: Build a Docker Image

Suppose your Dockerfile is

```dockerfile
FROM nginx:latest

COPY . /usr/share/nginx/html
```

Build it

```bash
docker build -t myweb:v1 .
```

Check

```bash
docker images
```

Output

```
REPOSITORY   TAG

myweb        v1
```

---

# Step 3: Scan the Docker Image

Basic command

```bash
trivy image myweb:v1
```

or

```bash
trivy image nginx:latest
```

Trivy will:

* Download the vulnerability database (first run only)
* Scan OS packages
* Scan application packages
* Show vulnerabilities

Example

```
Total: 18

CRITICAL: 2
HIGH: 7
MEDIUM: 9
```

([Trivy][2])

---

# Step 4: Show Only High and Critical

```bash
trivy image \
--severity HIGH,CRITICAL \
myweb:v1
```

Output

```
HIGH
CRITICAL
```

Useful in CI/CD.

---

# Step 5: Ignore Unfixed Vulnerabilities

Many vulnerabilities don't have patches.

```bash
trivy image \
--ignore-unfixed \
myweb:v1
```

Only fixable vulnerabilities are shown. ([Trivy][2])

---

# Step 6: Exit with Error

Useful for GitLab/Jenkins.

```bash
trivy image \
--exit-code 1 \
--severity CRITICAL \
myweb:v1
```

If any CRITICAL vulnerability exists

```
Exit Code = 1
```

Pipeline fails.

Otherwise

```
Exit Code = 0
```

---

# Step 7: Generate JSON Report

```bash
trivy image \
--format json \
-o report.json \
myweb:v1
```

Open

```bash
cat report.json
```

Useful for dashboards.

---

# Step 8: Generate Table Report

```bash
trivy image \
--format table \
myweb:v1
```

This is the default.

---

# Step 9: Generate SBOM

```bash
trivy image \
--format cyclonedx \
-o sbom.json \
myweb:v1
```

or

```bash
trivy image \
--format spdx-json \
-o sbom.json \
myweb:v1
```

([Trivy][2])

---

# Step 10: Scan an Image Tar File

Save image

```bash
docker save myweb:v1 -o myweb.tar
```

Scan

```bash
trivy image --input myweb.tar
```

Useful when Docker isn't installed on the scanning machine. ([Trivy][2])

---

# Step 11: Scan a Running Container

Find the container

```bash
docker ps
```

Example

```
CONTAINER ID

3e5d84c12
```

Inspect the image used

```bash
docker inspect 3e5d84c12
```

Then scan the image

```bash
trivy image nginx:latest
```

Trivy scans images rather than live container state by default.

---

# Step 12: Scan Local Filesystem

```bash
trivy fs .
```

or

```bash
trivy filesystem .
```

This scans:

* npm packages
* Maven
* NuGet
* Python
* Go
* OS packages
* Secrets

([Trivy][3])

---

# Step 13: Scan a Dockerfile

```bash
trivy config .
```

Detects issues like:

```
Running as root

No USER instruction

Latest tag used

Privileged container

Missing HEALTHCHECK
```

---

# Step 14: Scan for Secrets

```bash
trivy secret .
```

Finds:

* AWS Keys
* Azure Keys
* GitHub Tokens
* Passwords
* API Keys

---

# Step 15: Scan a Git Repository

```bash
trivy repo https://github.com/user/project
```

---

# Step 16: Cache the Vulnerability Database

Download the database in advance:

```bash
trivy image --download-db-only
```

This speeds up future scans.

---

# Common Docker + Trivy Workflow

```bash
# Build image
docker build -t employee-api:v1 .

# Verify
docker images

# Scan
trivy image employee-api:v1

# Scan only critical
trivy image --severity CRITICAL employee-api:v1

# Fail if critical issues exist
trivy image \
--severity CRITICAL \
--exit-code 1 \
employee-api:v1

# Generate JSON report
trivy image \
--format json \
-o trivy-report.json \
employee-api:v1
```

---

# Using Trivy as a Docker Container

If you don't want to install Trivy on your host, you can run it in a container:

```bash
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $HOME/.cache:/root/.cache \
  aquasec/trivy image nginx:latest
```

This uses the Docker socket to access local images and stores the vulnerability database in your local cache. ([Docker Hub][4])

---

## Interview Questions

**Q1. What is Trivy?**

* An open-source vulnerability scanner for container images, filesystems, repositories, Kubernetes, IaC, and secrets.

**Q2. How do you scan a Docker image?**

```bash
trivy image myimage:latest
```

**Q3. How do you fail a CI/CD pipeline when critical vulnerabilities are found?**

```bash
trivy image --severity CRITICAL --exit-code 1 myimage:latest
```

**Q4. Can Trivy scan Dockerfiles and Terraform?**

* Yes, using:

```bash
trivy config .
```

**Q5. Can Trivy generate an SBOM?**

* Yes, in formats such as CycloneDX and SPDX. ([Trivy][2])

As a next step, I can also show you **how to integrate Trivy into a GitLab CI/CD pipeline** so the pipeline automatically scans your Docker image and fails the build if HIGH or CRITICAL vulnerabilities are detected.

[1]: https://fossies.org/linux/trivy/docs/docs/references/configuration/cli/trivy.md?utm_source=chatgpt.com "trivy: docs/docs/references/configuration/cli/trivy.md | Fossies"
[2]: https://trivy.dev/docs/dev/docs/references/configuration/cli/trivy_image/?utm_source=chatgpt.com "Trivy image - Trivy"
[3]: https://www.trivy.dev/docs/latest/guide/references/configuration/cli/trivy_filesystem/?utm_source=chatgpt.com "Filesystem - Trivy"
[4]: https://hub.docker.com/hardened-images/catalog/dhi/trivy/guides?utm_source=chatgpt.com "Hardened Images catalog | Trivy | Guides | Docker Hub"
