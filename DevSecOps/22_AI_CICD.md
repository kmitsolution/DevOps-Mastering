# AI in CI/CD and DevSecOps

AI in CI/CD and DevSecOps means using artificial intelligence to help developers build, test, review, secure, and deploy applications more efficiently.

In a traditional CI/CD pipeline, most stages follow predefined rules and commands. AI can assist with interpreting failures, recommending fixes, generating tests, analyzing security findings, and helping engineers troubleshoot pipelines.

Important: AI assistance does not replace automated security scanners or pipeline policies. Deterministic checks should still decide whether a build or deployment is allowed.

## 1. Where AI fits into the CI/CD pipeline

1\. Code development

AI code suggestions and secure coding guidance

2\. Build and test

Test generation, failure explanation, build troubleshooting

3\. Security analysis

AI-assisted findings review plus SAST, SCA, secret detection, and Trivy

4\. Merge request review

AI review suggestions and human approval

5\. Deploy

Policy-controlled deployment and post-deployment monitoring

## 2. Key use cases

| Use case                 | How AI helps                                       | Example                                  |
| ------------------------ | -------------------------------------------------- | ---------------------------------------- |
| Code generation          | Suggests implementation and secure coding patterns | Generate input validation for a .NET API |
| Test generation          | Creates candidate unit and integration tests       | Generate MSTest cases for invalid inputs |
| Pipeline troubleshooting | Explains logs and proposes corrections             | Diagnose a failed `dotnet publish` job   |
| Security remediation     | Explains vulnerabilities and suggests fixes        | Recommend a safer dependency version     |
| Merge request review     | Highlights possible bugs and security issues       | Review an API endpoint before merging    |
| Deployment assistance    | Helps interpret deployment errors                  | Diagnose a Kubernetes rollout failure    |
| Operations               | Helps summarize logs and investigate anomalies     | Identify recurring application errors    |

Availability depends on the AI product and its integrations. Some tasks require an engineer to supply logs or findings to an AI assistant; they are not necessarily performed automatically inside CI/CD.

## 3. Practical example using your GitLab DevSecOps project

For your .NET 10 API and Angular UI, consider this workflow:

Step 1 — Developer writes code

Use GitLab Duo or a supported AI coding assistant to suggest code and generate candidate tests.

Step 2 — Create a merge request

AI-assisted review can highlight risky changes. A developer reviews the findings and decides what to fix.

Step 3 — Execute CI/CD checks

Run .NET tests, Angular tests, SAST, dependency scanning, secret detection, and Trivy image scanning.

Step 4 — Enforce security policies

Block the merge or deployment when mandatory checks fail, according to your project's configured rules.

## 4. Example: AI-assisted troubleshooting of a failed pipeline

Suppose your GitLab pipeline fails with:

```
error NETSDK1045:
The current .NET SDK does not support targeting .NET 10.0.
```

You could ask an AI assistant:

Analyze this GitLab CI/CD error:

NETSDK1045: The current .NET SDK does not support targeting .NET 10.0.

The project targets net10.0. Identify the likely cause, recommend the minimum safe correction to the pipeline, and provide a corrected YAML example. Explain how to verify the fix without weakening security controls.

The assistant may recommend using a compatible .NET 10 SDK image, such as:

```
dotnet_build:
  stage: build
  image: mcr.microsoft.com/dotnet/sdk:10.0
  script:
    - dotnet --info
    - dotnet restore DevSecOpsDemo.csproj
    - dotnet build DevSecOpsDemo.csproj --no-restore
    - dotnet test
```

This is an illustrative example; adapt the test command to your actual test-project structure.

The key point: AI proposes the correction, but the pipeline verifies whether the correction works.

## 5. AI versus traditional automation

| Traditional CI/CD and security tools     | AI-assisted DevSecOps                           |
| ---------------------------------------- | ----------------------------------------------- |
| Execute predefined commands and rules    | Explain failures in natural language            |
| Run repeatable tests                     | Generate candidate tests                        |
| Identify known vulnerable dependencies   | Help interpret findings and remediation options |
| Fail jobs based on configured thresholds | Suggest possible fixes for failed jobs          |
| Enforce configured approval policies     | Assist reviewers in evaluating changes          |

AI can make the process more efficient, but it may hallucinate, overlook vulnerabilities, or recommend unsafe changes. Keep independent security checks and human accountability in place.

## 6. Hands-on learning plan

For a practical class, cover these exercises in order:

1. Use AI to generate unit tests for the .NET API.
2. Use AI to review a merge request for security risks.
3. Ask AI to explain a failed GitLab build job.
4. Scan the application dependencies and Docker image.
5. Ask AI to explain a real scanner finding and suggest remediation.
6. Apply the fix and rerun the same automated checks.
7. Demonstrate that a failed mandatory security check blocks progression.

This gives students a complete picture of AI-assisted development while preserving the core DevSecOps principle: automate verification, enforce policy, and keep humans accountable.
