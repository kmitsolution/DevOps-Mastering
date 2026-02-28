# 🔎 Scan Docker Images for Vulnerabilities Using Trivy (Step-by-Step Guide)

![Image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2019/11/22/lizrice2.png)

![Image](https://devopscube.com/content/images/2025/03/trivy-blog-image-1.gif)

![Image](https://res.cloudinary.com/canonical/image/fetch/f_auto%2Cq_auto%2Cfl_sanitize%2Cw_819/https%3A%2F%2Fdashboard.snapcraft.io%2Fsite_media%2Fappmedia%2F2023%2F08%2Fiac_7fSo6VN.jpg)

![Image](https://0x1.gitlab.io/img/blog/trivy.png)

To scan Docker images for vulnerabilities, we commonly use:

👉 Trivy
👉 Scanning images built using Docker Engine

Trivy is:

* Open-source
* Fast
* Easy to use
* Supports container images, filesystems, Git repos, Kubernetes

---

# 🧠 What Does Trivy Scan?

Trivy scans for:

* OS package vulnerabilities (Alpine, Ubuntu, Debian, etc.)
* Application dependencies (npm, pip, Maven, etc.)
* Misconfigurations
* Secrets
* License issues

---

# 🐧 Step 1: Install Trivy (Ubuntu / Debian)

## 🔹 Option 1: Install Using Official Repository (Recommended)

```bash
sudo apt update
sudo apt install wget apt-transport-https gnupg lsb-release -y
```

Add Trivy GPG key:

```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
```

Add repository:

```bash
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
```

Install Trivy:

```bash
sudo apt update
sudo apt install trivy -y
```

---

## 🔹 Verify Installation

```bash
trivy --version
```

Expected output:

```text
Version: 0.xx.x
```

---

# 🐳 Step 2: Pull a Docker Image to Scan

Example:

```bash
docker pull nginx
```

Verify:

```bash
docker images
```

---

# 🔎 Step 3: Scan Docker Image

Basic scan:

```bash
trivy image nginx
```

What happens internally:

1. Trivy downloads vulnerability database (first time only)
2. Scans image layers
3. Detects OS & packages
4. Matches against CVE database

---

# 📊 Sample Output Explanation

Example output:

```text
nginx:latest

Total: 10 (CRITICAL: 2, HIGH: 5, MEDIUM: 3)

┌──────────┬────────────┬──────────┬─────────┬──────────────┬───────────────┐
│ Library  │ Vulnerability │ Severity │ Installed │ Fixed Version │
└──────────┴────────────┴──────────┴─────────┴──────────────┴───────────────┘
```

### Severity Levels

| Level    | Meaning                |
| -------- | ---------------------- |
| CRITICAL | Immediate fix required |
| HIGH     | Serious issue          |
| MEDIUM   | Moderate               |
| LOW      | Minor                  |

---

# 🎯 Step 4: Scan Specific Severity Only

Example: Show only HIGH and CRITICAL

```bash
trivy image --severity HIGH,CRITICAL nginx
```

---

# 📄 Step 5: Generate Report File

## Save as table output

```bash
trivy image -o report.txt nginx
```

---

## Save as JSON (for CI/CD)

```bash
trivy image -f json -o report.json nginx
```

---

# 🚫 Step 6: Fail Build if Vulnerabilities Found (CI/CD)

```bash
trivy image --exit-code 1 --severity CRITICAL nginx
```

If CRITICAL found → exit code 1 → pipeline fails.

---

# 🧪 Step 7: Scan Local Dockerfile (Optional)

```bash
trivy config .
```

Scans Dockerfile misconfigurations.

---

# 🧰 Step 8: Scan Running Container

```bash
docker ps
trivy image container_id
```

Or scan filesystem:

```bash
trivy fs /
```

---

# 🔄 Step 9: Update Vulnerability Database

```bash
trivy image --download-db-only
```

---

# 🏗️ Example Full Workflow

```bash
# Build image
docker build -t myapp:1.0 .

# Scan image
trivy image myapp:1.0

# Fail if critical found
trivy image --exit-code 1 --severity CRITICAL myapp:1.0
```

---

# 🧠 How Trivy Works Internally

```text
Docker Image
     ↓
Extract Layers
     ↓
Detect OS & Packages
     ↓
Match Against CVE Database
     ↓
Generate Vulnerability Report
```

---

# 🔐 Best Practices in Production

✔ Scan images before pushing to Docker Hub
✔ Use minimal base images (alpine)
✔ Fix vulnerabilities regularly
✔ Integrate Trivy in CI/CD
✔ Avoid using `latest` tag

---

# 🧠 Interview-Level Answer

> Trivy is an open-source vulnerability scanner used to scan Docker images for OS and dependency vulnerabilities. We install it using the official repository, then run `trivy image <image_name>`. It scans layers and matches packages against CVE databases. It can also fail CI/CD pipelines if critical vulnerabilities are detected.

---

