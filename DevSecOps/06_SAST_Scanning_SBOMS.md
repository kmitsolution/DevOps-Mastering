# DevSecOps Lesson 5 — GitLab SAST, Dependency Scanning, SBOMs, and Merge Request Security

Before moving to Docker container scanning, let's cover these three important DevSecOps topics using your existing private GitLab repository with a .NET 10 API and Angular UI.

1\. GitLab SAST

Detect security weaknesses in .NET and Angular source code using rules, configuration and tuning.

2\. Dependency scanning and SBOM

Identify vulnerable NuGet and npm packages, and generate a software inventory.

3\. False positives and merge request approvals

Review findings, suppress justified false positives, and prevent risky code from merging.

## Part 1 — GitLab SAST for .NET and Angular

### 1. What is SAST?

Static Application Security Testing (SAST) analyzes source code without needing to run the application.

For example, imagine your application contains:

- A .NET API that builds SQL queries using user input.
- An Angular component that renders untrusted HTML.
- Hardcoded credentials or insecure cryptographic operations.

SAST can detect certain insecure patterns before the application is deployed.

SAST does not guarantee that every vulnerability will be detected. Results depend on the analyzer, its supported languages, and the rules enabled.

### 2. How GitLab SAST works

Developer creates a branch

Changes .NET API or Angular UI

Merge Request pipeline

GitLab SAST jobs analyze source code

Security report

Findings, severity, affected files and locations

Review and merge decision

Fix, investigate, or formally accept an exception

GitLab's standard SAST template uses supported analyzers, including Semgrep-based analysis. Language support and security-report features depend on the GitLab version and subscription. Check the current supported-language matrix before promising coverage for a particular framework. [GitLab SAST documentation](https://docs.gitlab.com/user/application_security/sast/?utm_source=chatgpt.com).

### 3. Enable GitLab SAST in your project

In your repository root, open `.gitlab-ci.yml` and add the GitLab SAST template:

```
include:
  - template: Jobs/SAST.gitlab-ci.yml
```

Keep your existing jobs for Gitleaks, .NET build, unit tests and dependency scanning. Merge this `include` into your current YAML rather than replacing the file.

GitLab's standard SAST analyzer supports C#, JavaScript and TypeScript, so it can analyze source files from both your .NET API and Angular UI. Advanced cross-file analysis is a separate capability with additional tier requirements.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+1



For example, your repository might contain:

```
DevSecOpsDemo/
├── .gitlab-ci.yml
├── DevSecOpsDemo/
│   ├── DevSecOpsDemo.csproj
│   └── Controllers/
└── angular-ui/
    ├── package.json
    └── src/
```

GitLab selects supported analyzers based on the detected source files. Check the pipeline job names and logs to verify which analyzers actually ran.

### 4. Understanding SAST rules

A rule is a check that identifies a potentially unsafe code pattern.

Here are simplified examples of the kinds of issues security rules may detect:

Example A — C# SQL injection risk

```
var sql = "SELECT * FROM Products WHERE Id = "
          + userInput;
```

Risk: building SQL statements directly from untrusted input. Use parameterized queries instead.

Example B — Angular unsafe HTML

```
element.innerHTML = userInput;
```

Risk: inserting untrusted HTML may allow cross-site scripting (XSS). Prefer safe rendering and avoid bypassing Angular's security mechanisms.

These are illustrative risky patterns; whether a particular GitLab rule flags them depends on the analyzer and context.

A rule typically identifies:

- The weakness category, such as SQL injection or XSS.
- The file and line containing the code.
- A severity and supporting explanation.
- Sometimes, a code path showing how untrusted data reaches a dangerous operation.

### 5. Tune SAST for your repository

Security scanners can produce noise from generated code, build output and test fixtures. You can exclude irrelevant paths.

For example:

```
include:
  - template: Jobs/SAST.gitlab-ci.yml

variables:
  SAST_EXCLUDED_PATHS: "node_modules,dist,bin,obj,coverage"
```

This example excludes common generated directories. Review the exclusions carefully: don't exclude your real application source just to reduce the number of findings.

GitLab also allows supported analyzers to be configured through CI/CD variables and analyzer-specific settings. Rule identifiers and tuning mechanisms can differ between analyzers. See the [SAST configuration documentation](https://docs.gitlab.com/user/application_security/sast/?utm_source=chatgpt.com).&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+1



Classroom exercise: Add the template, push a branch with a harmless source-code change, and inspect the SAST job log. Confirm that the .NET and Angular source files are being scanned before attempting any rule-specific tuning.

## Part 2 — Dependency scanning and SBOM concepts

### 6. What is dependency scanning?

SAST examines how your own code is written. Dependency scanning checks the external packages your application relies on for known vulnerabilities.

For your application, that means:

- .NET: NuGet dependencies.
- Angular: npm dependencies.

For example, a direct npm dependency might itself use several other packages. Those additional packages are called transitive dependencies.

Angular application

package.json

Direct dependency

A package your project explicitly requests

Transitive dependencies

Packages required by your direct dependencies

Dependency scanner

Compares package identities and versions with vulnerability advisories

For the existing NuGet lab, you already used:

```
dotnet list DevSecOpsDemo/DevSecOpsDemo.csproj package --vulnerable --include-transitive
```

For Angular, use the lock file to make dependency resolution reproducible:

```
cd angular-ui
npm ci
npm audit
```

`npm audit` checks npm dependencies against the configured registry's advisory data. Review proposed fixes before running `npm audit fix`, especially if it might introduce breaking version changes.

### 7. What is an SBOM?

SBOM means Software Bill of Materials. It is a machine-readable inventory of software components included in an application or software release.

Think of it as an ingredient list for software.

An SBOM can include:

- Package name and version.
- Package ecosystem.
- Component relationships.
- Identifiers and other metadata.
- License information, when available.

Two common SBOM formats are:

- CycloneDX
- SPDX

An SBOM is not itself a vulnerability scan. It records what components are present; a scanner or vulnerability database can then use that inventory to identify known risks.

For example, an SBOM might record that your Angular application contains a particular npm package at version `x.y.z`. If an advisory later identifies that version as vulnerable, the inventory helps you identify which application needs attention.

### 8. How GitLab uses SBOMs

GitLab's SBOM-based dependency-scanning workflow can produce CycloneDX reports and use them to identify vulnerable dependencies. Reports and features available in the interface depend on the GitLab version, configuration and subscription.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+1



For a project with both .NET and Angular, keep the relevant dependency manifests and lock files committed:

```
DevSecOpsDemo/
├── DevSecOpsDemo/
│   ├── DevSecOpsDemo.csproj
│   └── packages.lock.json
└── angular-ui/
    ├── package.json
    └── package-lock.json
```

The exact .NET lock-file location depends on where the project file resides.

Classroom takeaway: Dependency scanning answers “Which dependencies have known vulnerabilities?” An SBOM answers “Which software components are present?”

## Part 3 — False positives, merge requests and approvals

### 9. What is a false positive?

A false positive occurs when a scanner reports a potential vulnerability that, after investigation, is not a valid issue in the specific context.

For example, a rule might flag an input-handling pattern as potentially unsafe, but the team verifies that the input has already been safely validated by a trusted component.

A finding should not be dismissed merely because it is inconvenient.

Use this workflow:

#### 1

Review the finding

Inspect the rule, code location, data flow and severity.

#### 2

Validate the risk

Determine whether the vulnerable pattern actually exists and whether the code is reachable.

#### 3

Choose an action

Fix the code, upgrade the dependency, or document a justified exception.

#### 4

Record the decision

Document the reason, owner and any follow-up date for accepted risk.

GitLab provides vulnerability triage and dismissal workflows. Available statuses and permissions depend on your GitLab version and role. See [GitLab vulnerability management](https://docs.gitlab.com/user/application_security/vulnerabilities/?utm_source=chatgpt.com).&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs



### 10. What happens when a developer creates a Merge Request?

Suppose a developer modifies `angular-ui/src/app/product.service.ts` and creates a merge request.

The merge request pipeline can run:

1. Unit tests.
2. SAST.
3. Dependency scanning.
4. Other configured security checks.

The results help reviewers understand whether the proposed change introduces new security findings. GitLab can display supported security reports in merge request views, allowing reviewers to investigate findings before merging.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+1



A simple example:

Merge Request: Add product search

Unit tests passed

SAST detected a potential SQL injection

Dependency scan found a vulnerable package

Decision: Review and remediate the findings before merging, according to your project's security policy.

### 11. How do we require approvals?

There are two different concepts:

- Required pipeline success: A failed security job can prevent the merge when the project requires successful pipelines.
- Security approval policy: A rule can require an eligible reviewer to approve a merge request based on defined security findings.

The second option is not available in every GitLab tier. GitLab's security approval policies are documented as an Ultimate-tier feature.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs



For your private GitLab training project, start with the simplest enforcement:

1. Add SAST and dependency scanning to the pipeline.
2. Open a merge request from a feature branch.
3. Wait for the security jobs to finish.
4. Inspect the findings.
5. Fix the issue and rerun the pipeline.
6. Configure the project to require successful pipelines before merging, if that option is available to your project.

If you have access to security approval policies, you can later configure approval requirements based on vulnerability severity or other policy criteria.

Don't assume that adding a scanner automatically blocks every vulnerable merge request. Detection, reporting and enforcement are separate configurations.

## 12. Hands-on checklist for your class

Lesson completion

0 of 8

Add GitLab's SAST template to .gitlab-ci.yml

Verify scanning of C# and Angular TypeScript files

Review the analyzer logs and tune unnecessary exclusions

Run NuGet and npm dependency audits

Understand the difference between a dependency report and an SBOM

Create a feature branch and open a merge request

Review a security finding and document the remediation

Confirm how pipeline success is enforced before merging

### What comes next?

Once these source-code and dependency security controls are understood, we'll move to Docker container image scanning with Trivy. That lesson will cover the next layer: detecting vulnerabilities in the operating-system packages and runtime components bundled into your container image.
