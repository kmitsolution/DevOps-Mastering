
## 1. What is DAST?

DAST stands for Dynamic Application Security Testing.

DAST tests a running application from the outside to identify security vulnerabilities. Instead of analyzing source code, it sends requests to the application and examines how the application responds.

Think of it this way:

- SAST examines your source code.
- SCA examines third-party libraries and dependencies.
- Trivy image scanning examines container images for known vulnerabilities and other supported issues.
- DAST tests the running application for potential security weaknesses.

For your .NET 10 API and Angular application, DAST can test the application after it has been built and deployed to a test environment.

## 2. How DAST works

1\. Start the application

Run your .NET API and Angular UI in a test environment.

2\. DAST scanner sends requests

Tests endpoints, inputs, headers, and application behavior.

3\. Analyze responses

Looks for signs of vulnerabilities and security misconfiguration.

4\. Generate a security report

Lists findings, evidence, severity, and recommended remediation.

DAST usually doesn't need access to your application's source code. However, it does need a reachable running application. Authentication and API-specific configuration may also be needed to test protected functionality.

## 3. Common vulnerabilities DAST can detect

| Vulnerability              | Example                                                        |
| -------------------------- | -------------------------------------------------------------- |
| Cross-Site Scripting (XSS) | User input is rendered as executable JavaScript                |
| SQL Injection              | Application constructs unsafe database queries from user input |
| Security misconfiguration  | Debug pages or sensitive error details are exposed             |
| Missing security headers   | HTTP responses lack recommended security headers               |
| Insecure cookies           | Session cookies lack appropriate security attributes           |
| Authentication weaknesses  | Login or session behavior has security problems                |

DAST cannot reliably detect every vulnerability. For example, it may not discover SQL injection if the scanner cannot reach the relevant endpoint or exercise the vulnerable input.

## 4. DAST tools you should know

[ZAP | Ahmed Bouayed](https://images.openai.com/static-rsc-4/L5a0un05-fAv2jXrQ_LRKxrfEKu9gAp_f-xgRJJWOm3Kz99UExVXcipmcYCSt2VZa4qZbwFV4W1XK9nwK2rD_HVp9B6HMQf4oRr2Njp2j87RVLM0pRzcYeqRsZa5Y_A3SAAbnIHzhx2ytVIOutzIg-Qzgc524kvA8yN1hW020b4?purpose=inline)

OWASP ZAP

An open-source web application scanner. Its Baseline Scan is a good starting point for CI/CD because it performs crawling and passive analysis without actively attacking the application.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.zaproxy.org\&sz=32)

ZAP Docker Documentation

+1



Recommended for your hands-on learning.

[SkillHigh – Learn Skills. Get Hired. Grow Fast.](https://images.openai.com/static-rsc-4/iXIkr0bdtiLg8gvOs6Ae8MuwUuUEoZMpXkPEfxNUZ48QbXLSNgHMFpVi2voLh2opQ2CviCklT6iZccKl132-Gwkz8j7rBuv6Qfgm51hmJx1YuQXbqC2ka8FF04Hp7RrQqKf9F1L2VVlMF0I0ul1MZd3W4xs3APJ2N2-WxmGv2fE?purpose=inline)

Burp Suite

A widely used web security testing platform with manual and automated testing capabilities. It is useful for learning how to investigate requests, responses, and application behavior.

[Gitlab CI - Opsera](https://images.openai.com/static-rsc-4/lUcuW2mMvIk8ubK7avmLbJLsOA99RdHE5vcL9YmPb0AKQebKNvKoCe6ZOQl42_Vo5Xl21XDz9seTi666JCOyoVjOTf5MUmPFlENeTxBXoI1xLlCuN_9x1zEloftChYcqRwdpHu3cNpl5IMX31JrRE9PZFPb3rMz7-xjVTkv4Nwo?purpose=inline)

GitLab DAST

GitLab offers DAST tooling that integrates security scans with CI/CD pipelines and security reporting. Current GitLab DAST features have tier and configuration requirements, so we can use ZAP directly for a simpler, open-source exercise.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+1



## 5. Practical lab — run your first DAST scan

We'll use ZAP Baseline Scan against a running test application.

### Step 1 — Start your application

For example, if your .NET application is running locally on port 8080, verify it first:

```
curl -I http://localhost:8080
```

If your application uses a different port or route, adjust the URL accordingly.

### Step 2 — Run ZAP in Docker

If the application is running directly on your Ubuntu host, run:

```
docker run --rm \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://172.17.0.1:8080
```

This uses a common Linux Docker bridge gateway address, `172.17.0.1`. If your Docker network uses a different gateway, substitute the correct address. If the application is in another container, use a shared Docker network and its container/service name instead.

The Baseline Scan crawls the target and performs passive analysis. It does not perform the active attack scan.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.zaproxy.org\&sz=32)

ZAP Docker Documentation

+1



Important: Scan only applications you own or are explicitly authorized to test. Start with a test environment, not production.

### Step 3 — Save an HTML report

Create a report directory:

```
mkdir -p zap-reports
```

Then run:

```
docker run --rm \
  -v "$PWD/zap-reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://172.17.0.1:8080 \
  -r report.html
```

After the scan, open `zap-reports/report.html` to review the findings.

### Step 4 — Understand the results

ZAP may report alerts as `PASS`, `INFO`, `WARN`, or `FAIL`. A warning is not automatically a confirmed exploitable vulnerability; review the evidence and recommendation.

## 6. How DAST fits into your DevSecOps pipeline

For your .NET and Angular project, the typical sequence is:

1. SAST checks source code.
2. SCA checks dependencies.
3. Trivy scans the Docker image.
4. The application is deployed to a test environment.
5. ZAP performs DAST against the running application.
6. Findings are reviewed, fixed, and rescanned before release.

One important distinction: DAST needs a running target URL. Building a Docker image alone is not enough.

For the next step, start your .NET application and confirm which URL works in your browser—`http://localhost:8080`, another port, or a deployed test URL. Then we can configure the DAST scan for your actual application.
