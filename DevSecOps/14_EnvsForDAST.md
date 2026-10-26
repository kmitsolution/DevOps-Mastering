# DevSecOps Lesson: Test Environments, Test Data, and Safe-Scan Practices

When we run DAST tools such as OWASP ZAP against an application, we must control where we scan, what data we use, and which tests we allow.

A security scanner can send unexpected requests, trigger application errors, submit forms, or create significant traffic. A safe DevSecOps workflow tests these behaviors in a controlled environment before applying security checks to production.

## 1. Test environments

A test environment is an isolated environment where you can validate an application without affecting real users or production data.

Development environment

Used by developers for local testing. For example, running your .NET API and Angular application on your Ubuntu machine.

Good for learning

Test / staging environment

A separate deployment that resembles production but uses test credentials, synthetic data, and controlled network access.

Recommended for DAST

Production environment

Serves real users and real business data. Active security testing can cause outages, change data, or trigger business operations.

Requires explicit authorization and controls

### Example: Your DevSecOps project

You have a .NET 10 API, an Angular UI, Docker, GitLab CI/CD, Trivy, and ZAP.

A suitable workflow is:

1. Build the .NET and Angular applications.
2. Scan source code and dependencies.
3. Build and scan the Docker image with Trivy.
4. Deploy the image to a temporary test environment.
5. Run ZAP against the test URL.
6. Review findings and clean up the environment.

The key principle is that DAST must target the running application, not merely the Docker image.

## 2. Test data: What should you use?

Test data is the information used when exercising application functionality and security controls.

Use synthetic data—data created specifically for testing—rather than copying real customer records into your lab.

| Data type            | Example                       | Safe practice                                 |
| -------------------- | ----------------------------- | --------------------------------------------- |
| User accounts        | `testuser@example.test`       | Use dedicated test accounts                   |
| Passwords            | Dummy credentials             | Never reuse real passwords                    |
| Personal information | Fictional names and addresses | Avoid real customer data                      |
| Database records     | Sample products or orders     | Use isolated test databases                   |
| API tokens           | Short-lived test tokens       | Restrict permissions and expire them          |
| Payment data         | Mock payment responses        | Never use real cardholder data in a basic lab |

For example, your .NET API could use a fictional test user:

```
{
  "username": "testuser@example.test",
  "role": "Tester",
  "isTestAccount": true
}
```

This is only sample application data, not an actual authentication credential.

Important: Never put real secrets in test fixtures, Git repositories, Docker images, screenshots, or scan reports. If real credentials are accidentally exposed, revoke or rotate them.

## 3. Safe-scan practices

Before starting a DAST scan, follow this checklist.

Pre-scan checklist

3/7 completed

I own the application or have written authorization to scan it.

I have confirmed the exact target URL and allowed scope.

The target is a development or staging environment.

I am using synthetic data and test-only credentials.

I have disabled destructive tests and uncontrolled form submissions.

I have set reasonable scan duration and request limits.

I have checked logs, monitoring, and cleanup procedures.

### Passive versus active scanning

| Scan type    | What it does                                                                   | Recommended use                                    |
| ------------ | ------------------------------------------------------------------------------ | -------------------------------------------------- |
| Passive scan | Inspects requests and responses without deliberately attacking the application | Good starting point                                |
| Active scan  | Sends attack-like requests to test for vulnerabilities                         | Only in an authorized, controlled test environment |

An active scan can trigger unexpected behavior, so don't run it against production without an approved testing plan.

## 4. Practical lab: Safely scan your NGINX container with ZAP

You previously scanned NGINX at `http://172.17.0.1:802`. Let's use that as a controlled lab target, provided it is your own test service.

### Step 1 — Confirm the target

```
curl -I http://172.17.0.1:802
```

Verify that this address reaches the intended NGINX container and not an unrelated service.

### Step 2 — Run the ZAP Baseline Scan

```
mkdir -p zap-reports

docker run --rm \
  -v "$PWD/zap-reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://172.17.0.1:802 \
  -m 2 \
  -r report.html
```

This runs a baseline scan with a short maximum crawl duration setting. It performs passive analysis rather than the active attack scan. The scan duration is not a guarantee of total runtime, and the target may still receive multiple requests.

### Step 3 — Review the report

Open the generated file:

```
zap-reports/report.html
```

Review each alert, its evidence, and the recommended remediation. Don't assume every warning is exploitable, and don't dismiss an alert solely because the scan finished successfully.

### Step 4 — Clean up

If you created a temporary container or test deployment specifically for the exercise, stop and remove it when testing is complete.

```
docker ps
```

Identify the relevant container before stopping it. Don't remove containers that host other applications.

## 5. Integrating safe scanning into GitLab CI/CD

A good DAST job should:

- Target a dedicated staging URL or a temporary test deployment.
- Run only for the intended branches or merge requests.
- Use test credentials stored securely in GitLab CI/CD variables, if authentication is needed.
- Apply appropriate timeouts and scanning limits.
- Save the report as a pipeline artifact without exposing credentials.
- Fail the pipeline according to a clearly defined security policy.

For example, a baseline scan can be added after the application has been deployed to a test environment:

```
dast_baseline:
  stage: security
  image:
    name: ghcr.io/zaproxy/zaproxy:stable
    entrypoint: [""]
  variables:
    DAST_TARGET: "https://staging.example.test"
  script:
    - mkdir -p /zap/wrk
    - zap-baseline.py -t "$DAST_TARGET" -m 2 -r report.html
  artifacts:
    when: always
    paths:
      - report.html
    expire_in: 7 days
  allow_failure: true
```

Replace the example URL with your authorized staging URL. The runner must be able to reach it. This configuration collects the report but does not automatically enforce a blocking security policy, because `allow_failure: true` allows the job to fail without failing the pipeline. Once you understand the scan results, you can introduce an explicit alert threshold and make the job blocking.

## Key takeaway

- Test environments isolate the scan from real users and production services.
- Test data prevents unnecessary exposure of sensitive information.
- Safe-scan practices limit the scope, impact, and risk of security testing.

The next practical step is to run ZAP against your own .NET/Angular test deployment and configure GitLab to retain the report and enforce a security threshold.
