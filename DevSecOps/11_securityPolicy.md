# DevSecOps: Policy & Compliance in GitLab CI/CD — Step-by-Step Example

In this lesson, we'll implement a practical security policy in your existing GitLab DevSecOps project, which contains a .NET 10 application and an Angular frontend.

Goal: prevent a pipeline from passing if the source code violates a security rule or the Docker image contains HIGH or CRITICAL vulnerabilities.

We'll start with simple, custom policies that work without requiring a paid compliance platform. Later, you can extend them to enterprise compliance frameworks.

## 1. What are policy and compliance in a pipeline?

- Policy: a rule that defines what is allowed or prohibited.
- Compliance: evidence that the project follows required rules and controls.
- Pipeline enforcement: automatically passes or fails a pipeline according to those rules.

Examples:

| Policy                                    | Expected behavior                               |
| ----------------------------------------- | ----------------------------------------------- |
| No hardcoded secrets                      | Fail when a secret is detected                  |
| No unsafe HTML assignment                 | Fail when a prohibited code pattern is detected |
| No HIGH or CRITICAL image vulnerabilities | Fail the Trivy scan                             |
| Tests must pass                           | Block progression when tests fail               |
| Security jobs must not be optional        | Ensure failures cannot be ignored               |

We'll implement two policies first: a source-code security rule and a Trivy image vulnerability policy.

## 2. Example 1 — Create a custom security policy

We'll enforce this rule:

> The Angular application must not assign user input directly to `innerHTML`.

This is a simplified training policy. It detects a specific unsafe coding pattern; it is not a complete security analyzer.

### Step 1 — Create a policy script

In your repository root, create this file:

`security/policy-check.sh`

Add the following code:

```

#!/bin/sh
set -eu

SOURCE_DIR="angular-ui/src"

echo "Checking application security policy..."

if [ ! -d "$SOURCE_DIR" ]; then
  echo "ERROR: Source directory $SOURCE_DIR not found."
  exit 1
fi

if grep -R -n \
  --include='*.ts' \
  'innerHTML[[:space:]]*=' \
  "$SOURCE_DIR"; then
  echo "POLICY FAILED: Direct innerHTML assignment detected."
  echo "Review the finding and use safe rendering practices."
  exit 1
fi

echo "POLICY PASSED: No matching innerHTML assignment found."

```

This script searches TypeScript source files for assignments to `innerHTML`. It intentionally uses a broad pattern for demonstration, so it may flag safe or trusted assignments too. In a real project, review the finding and use a proper static analyzer to distinguish unsafe data flows.

### Step 2 — Test the policy locally

From the repository root, run:

```
bash security/policy-check.sh
```

If no matching pattern is found, you should see:

```
Checking application security policy...
POLICY PASSED: No matching innerHTML assignment found.
```

Now, to test the failure path, create a temporary TypeScript file:

`angular-ui/src/policy-demo.ts`

Add:

```
const userInput = "<img src=x onerror=alert(1)>";
const element = document.createElement("div");
element.innerHTML = userInput;
```

Run the script again:

```
bash security/policy-check.sh
```

It should fail with exit code `1` and report the matching line. This is a deliberately vulnerable training example; don't use this code in the actual application.

After testing, delete `angular-ui/src/policy-demo.ts`.

## 3. Example 2 — Enforce the policy in GitLab CI/CD

Your current pipeline uses Trivy to scan a Docker image. We can add a policy job without bringing back the other dependency scans and build jobs.

Update `.gitlab-ci.yml` to the following:

```

stages:
  - security

variables:
  TRIVY_NO_PROGRESS: "true"

# Policy 1: Check application source code
source_security_policy:
  stage: security
  image: alpine:3.22
  script:
    - apk add --no-cache grep
    - sh security/policy-check.sh
  allow_failure: false

# Policy 2: Reject high/critical image vulnerabilities
trivy_image_scan:
  stage: security
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  variables:
    DOCKER_IMAGE: "registry.gitlab.com/YOUR_GROUP/YOUR_PROJECT/devsecops-demo:1.0"
  script:
    - trivy image --severity HIGH,CRITICAL --exit-code 1 "$DOCKER_IMAGE"
  allow_failure: false

```

Replace `YOUR_GROUP` and `YOUR_PROJECT` with the actual GitLab registry path. The image must already be pushed to the registry, and the runner must be able to authenticate if the image is private. Trivy cannot scan an image that exists only on your Ubuntu machine unless that image is made available to the job.

What does `allow_failure: false` do?

It ensures that a failing security job causes the pipeline to fail. In this example, the policy job and Trivy scan can run in parallel because they share the same stage.

## 4. Test the policy in GitLab

1. Commit the new policy script and pipeline file to a feature branch.
2. Push the branch to GitLab.
3. Open a Merge Request.
4. Open Build → Pipelines and select the pipeline.
5. Inspect `source_security_policy` and `trivy_image_scan`.
6. Introduce the temporary unsafe assignment from Step 2 in a test branch. Confirm the policy job fails.
7. Remove the test code and push again. Confirm the policy passes.
8. Review the Trivy report and verify whether any HIGH or CRITICAL findings caused the scan to fail.

A failed job means the pipeline fails. To actually prevent merging, your project must also require a successful pipeline before merge in its merge request settings.

## 5. What compliance evidence should you retain?

For each pipeline, retain:

- Commit SHA and branch.
- Pipeline status and timestamps.
- Policy script version.
- Trivy scan output or JSON report.
- Findings, remediation decisions, and exceptions.
- Approval and merge history, where applicable.

This creates a useful audit trail. A green pipeline alone does not prove compliance with an entire standard such as ISO 27001 or SOC 2; the organization must map its required controls to enforced checks and retain appropriate evidence.

## Your expected result

Policy passes

No matching prohibited source pattern; Trivy finds no HIGH or CRITICAL vulnerabilities.

Pipeline fails

A policy violation is detected or Trivy finds a HIGH/CRITICAL vulnerability.

Next step: implement the source policy first and test both its pass and fail paths. Once that works, connect the Trivy job to your GitLab Container Registry image and verify the vulnerability policy end to end.
