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

# Practical Lab: GitLab SAST, Dependency Scanning, SBOM and Merge Request Security

Raman, let's implement this practically in your existing private GitLab repository using your .NET 10 API (`DevSecOpsDemo.csproj`) and Angular UI (`angular-ui`).

We'll do one step at a time, verify each result, and then move to the next. We won't change your repository visibility or use SonarQube Cloud.

## Lab roadmap

### 1

Enable GitLab SAST

Scan C# and Angular TypeScript source code

### 2

Scan dependencies

Check NuGet and npm packages for known vulnerabilities

### 3

Generate an SBOM

Create a machine-readable software inventory

### 4

Create a Merge Request

Review security findings before merging

### 5

Enforce security checks

Require pipeline success and understand approval options

## Step 1 — Check your repository structure

Your repository should have a layout similar to this:

```
DevSecOpsDemo/
├── .gitlab-ci.yml
├── DevSecOpsDemo/
│   └── DevSecOpsDemo.csproj
└── angular-ui/
    ├── package.json
    ├── package-lock.json
    └── src/
```

The paths are examples based on our earlier lessons. If your `.csproj` is directly at the repository root or the Angular folder has a different name, we'll adjust the configuration.

Open `.gitlab-ci.yml` and keep your existing Gitleaks, build and test jobs. For this first step, we will only add GitLab's SAST template.

### 1.1 Add GitLab SAST

Add this block at the top level of `.gitlab-ci.yml`:

```
include:
  - template: Jobs/SAST.gitlab-ci.yml
```

For example, if you already have a `stages` section, it might look like:

```
stages:
  - security
  - build
  - test

include:
  - template: Jobs/SAST.gitlab-ci.yml
```

Don't create a second `stages` section if one already exists. GitLab's standard SAST template uses its configured stage defaults; ensure that your stage list includes `test` or configure the relevant SAST stage to match your pipeline.

### 1.2 Commit and push

```
git checkout -b feature/enable-gitlab-sast
git add .gitlab-ci.yml
git commit -m "Enable GitLab SAST"
git push -u origin feature/enable-gitlab-sast
```

If you already have a working feature branch, use it instead of creating another one.

### 1.3 Verify the result

In GitLab:

1. Open Build → Pipelines.
2. Select the latest pipeline.
3. Look for the SAST job or jobs created by the template.
4. Open the logs and check which analyzers ran.

A pipeline may include an analyzer that finds no applicable files, depending on the detected languages and configuration. We need to inspect the job logs rather than assume every language was scanned.

Official reference: [GitLab SAST documentation](https://docs.gitlab.com/user/application_security/sast/?utm_source=chatgpt.com).

Checkpoint 1: SAST is integrated when the pipeline configuration validates and the relevant SAST jobs run. If the pipeline fails, copy the job name and error message before changing anything else.

## Step 2 — Add dependency scanning for .NET and Angular

We'll use the tools already available in your project ecosystems. This gives us a straightforward first practical scan without depending on SonarCloud permissions.

### 2.1 Scan .NET NuGet dependencies

Add the following job to `.gitlab-ci.yml`:

```
nuget_audit:
  stage: security
  image: mcr.microsoft.com/dotnet/sdk:10.0

  script:
    - dotnet restore DevSecOpsDemo/DevSecOpsDemo.csproj
    - dotnet list DevSecOpsDemo/DevSecOpsDemo.csproj package --vulnerable --include-transitive
```

Adjust the project path if your `.csproj` is located elsewhere.

This checks the resolved NuGet dependency graph for known vulnerability advisories. NuGet's report command is informational: it may return exit code 0 even when vulnerabilities are listed.

To enforce a high/critical vulnerability policy, you can promote the NuGet warning codes `NU1903` and `NU1904` to errors during restore, as we covered in Lesson 4.

### 2.2 Scan Angular npm dependencies

Add a second job:

```
npm_audit:
  stage: security
  image: node:22-alpine

  script:
    - cd angular-ui
    - npm ci
    - npm audit --audit-level=high
```

This requires `angular-ui/package-lock.json` to be committed. `npm ci` installs the versions recorded in the lock file, and `npm audit --audit-level=high` fails the job when npm reports a high- or critical-severity vulnerability.

If your Angular folder is named differently, update `cd angular-ui`.

### 2.3 Verify both jobs

Push the changes and open the pipeline.

You should now see jobs for:

- SAST
- `nuget_audit`
- `npm_audit`

If an audit fails, open its logs and identify the package, installed version, severity and recommended patched version. Don't blindly apply all suggested upgrades; review compatibility first.

## Step 3 — Generate an SBOM

An SBOM is a machine-readable inventory of the software components in your application. We'll generate a CycloneDX SBOM for the Angular dependency graph using the npm ecosystem.

Add this job:

```
angular_sbom:
  stage: security
  image: node:22-alpine

  script:
    - cd angular-ui
    - npm ci
    - npx --yes @cyclonedx/cyclonedx-npm --output-file ../angular-sbom.json

  artifacts:
    when: always
    paths:
      - angular-sbom.json
    expire_in: 7 days
```

The tool reads the npm dependency information and produces a CycloneDX JSON inventory. For reproducible training, you can pin a reviewed version of the SBOM generator instead of always using the latest release.

After the pipeline runs:

1. Open the `angular_sbom` job.
2. Find the Job artifacts section.
3. Download `angular-sbom.json`.
4. Open it and look for the `components` array, where package names and versions are listed.

The SBOM is an inventory, not a guarantee that every component is secure. We use `npm audit` to check known npm advisories in the preceding job.

For the .NET application, we can add a corresponding CycloneDX SBOM generator once this Angular example is working.

## Step 4 — Create a Merge Request and review security findings

Now we'll demonstrate how security checks fit into the developer workflow.

### 4.1 Create a feature branch

```
git checkout -b feature/security-lab
```

Make a small harmless code change in your .NET API or Angular UI, then commit it:

```
git add .
git commit -m "Add security scanning lab"
git push -u origin feature/security-lab
```

In GitLab, open Code → Merge requests → New merge request and create a merge request from `feature/security-lab` into your default branch.

### 4.2 Review the pipeline

Inside the merge request, check the pipeline status and any available security reports.

| Result                                   | What to do                                              |
| ---------------------------------------- | ------------------------------------------------------- |
| SAST finds a potential vulnerability     | Review the rule, source location and code context.      |
| NuGet audit reports a vulnerable package | Upgrade or otherwise remediate the affected dependency. |
| npm audit fails                          | Inspect the dependency path and available fixes.        |
| SBOM job succeeds                        | Download the artifact and verify the package inventory. |

Security report visibility in the merge request depends on the report format and GitLab features available to your project.

### 4.3 Understand false positives

Suppose SAST flags a line of code that appears unsafe, but investigation shows that the relevant input is safely handled elsewhere.

Don't immediately suppress the finding. Instead:

1. Inspect the reported code and the rule.
2. Check the data flow and relevant application context.
3. Fix the issue if it is genuine.
4. If it is genuinely a false positive, document the evidence and use the supported dismissal or exception workflow available in your GitLab instance.

Keep a record of the reason, reviewer and any follow-up required.

## Step 5 — Enforce security checks before merging

Start with a basic merge policy that is useful even without advanced security approval features.

1. In GitLab, open Settings → Merge requests.
2. Look for the merge checks available to your project.
3. Enable the requirement for a successful pipeline before merging, if available.
4. Keep the SAST and audit jobs blocking where you want them to enforce policy—don't set `allow_failure: true` for those jobs.
5. Test the policy using a feature branch and verify that a failed required job prevents the merge.

For additional security approval rules, GitLab's available features depend on your subscription. Security approval policies are an Ultimate-tier capability; don't assume that installing the SAST template automatically creates mandatory security approvals.

Official reference: [GitLab security policies documentation](https://docs.gitlab.com/user/application_security/policies/?utm_source=chatgpt.com).

## Step 6 — Final practical checklist

Lab progress

0/6 complete

SAST jobs run in GitLab CI/CD

NuGet audit runs for DevSecOpsDemo.csproj

npm audit runs for Angular dependencies

CycloneDX SBOM artifact is generated

Merge Request pipeline displays its status

Required pipeline success is enforced before merging

### Your first action

Start with Step 1 only: add the GitLab SAST template, push the feature branch, and verify the pipeline.

If it fails, share the error and your current `stages` section. We'll fix that before adding the dependency audit jobs, so you can learn and validate each part in sequence.

Raman, thanks for sharing your project structure. We'll adapt the practical to what you actually have and create a new Angular project alongside your existing .NET project.

Your current project is a .NET-only application, and your `.gitlab-ci.yml` already includes GitLab SAST and NuGet dependency scanning. We'll keep those and add Angular, npm dependency scanning, SBOM generation, and merge request security checks.

## Updated project structure

Currently, your repository looks like this:

```
DEVSECOPS/
└── DevSecOpsDemo/
    ├── .gitlab-ci.yml
    ├── DevSecOpsDemo.csproj
    ├── Program.cs
    ├── appsettings.json
    ├── packages.lock.json
    └── test-secret.txt
```

We'll expand it to:

```
DEVSECOPS/
└── DevSecOpsDemo/
    ├── .gitlab-ci.yml
    ├── DevSecOpsDemo.csproj
    ├── Program.cs
    ├── packages.lock.json
    ├── angular-ui/
    │   ├── package.json
    │   ├── package-lock.json
    │   ├── angular.json
    │   └── src/
    └── ...
```

Important: Your screenshot shows `test-secret.txt`, which was used in our earlier Gitleaks exercise. Make sure it contains only fake test data. Never commit a real credential; if one was exposed, revoke or rotate it.

## Step 1 — Create the Angular project

Open the VS Code terminal in the directory that contains `DevSecOpsDemo.csproj`.

Verify Node.js and npm are installed:

```
node --version
npm --version
```

If either command is not recognized, install a supported Node.js LTS release first.

Now create the Angular application inside your existing repository:

```
npx @angular/cli@latest new angular-ui --routing --style css --skip-git
```

When prompted about enabling Server-Side Rendering (SSR), choose No for this simple lab.

This creates a new Angular project without creating a separate Git repository inside your existing GitLab project.

When it finishes, your structure will include:

```
DevSecOpsDemo/
├── .gitlab-ci.yml
├── DevSecOpsDemo.csproj
├── Program.cs
└── angular-ui/
    ├── package.json
    ├── package-lock.json
    ├── angular.json
    └── src/
```

Test that Angular can build:

```
cd angular-ui
npm ci
npm run build
cd ..
```

If the build succeeds, the Angular project is ready for the security lab.

## Step 2 — Update GitLab SAST and dependency scanning

Replace the current `.gitlab-ci.yml` with the following version. It preserves your .NET dependency audit and adds Angular dependency scanning and SBOM generation.

```
stages:
  - security
  - build
  - test

include:
  - template: Jobs/SAST.gitlab-ci.yml

variables:
  DOTNET_CLI_TELEMETRY_OPTOUT: "1"

# 1. Existing .NET dependency scan
nuget_dependency_scan:
  stage: security
  image: mcr.microsoft.com/dotnet/sdk:10.0
  script:
    - dotnet --version
    - dotnet restore DevSecOpsDemo.csproj
    - dotnet list DevSecOpsDemo.csproj package --vulnerable --include-transitive
  allow_failure: false

# 2. Angular dependency scan
npm_dependency_scan:
  stage: security
  image: node:22-alpine
  script:
    - cd angular-ui
    - npm ci
    - npm audit --audit-level=high
  allow_failure: false

# 3. Generate an Angular SBOM
angular_sbom:
  stage: security
  image: node:22-alpine
  script:
    - cd angular-ui
    - npm ci
    - npx --yes @cyclonedx/cyclonedx-npm --output-file ../angular-sbom.json
  artifacts:
    when: always
    paths:
      - angular-sbom.json
    expire_in: 7 days

# 4. Build the .NET application
dotnet_build:
  stage: build
  image: mcr.microsoft.com/dotnet/sdk:10.0
  script:
    - dotnet restore DevSecOpsDemo.csproj
    - dotnet build DevSecOpsDemo.csproj --no-restore

# 5. Build the Angular application
angular_build:
  stage: build
  image: node:22-alpine
  script:
    - cd angular-ui
    - npm ci
    - npm run build

# 6. Run .NET tests if a test project exists
# Add your test project path here when ready.
```

Important details:

- Your `.csproj` is at the repository root, so the .NET commands correctly use `DevSecOpsDemo.csproj`.
- `npm audit --audit-level=high` fails the job when high- or critical-severity vulnerabilities are reported. It may fail if a newly generated Angular dependency has an advisory; investigate the finding rather than automatically suppressing it.
- The SAST template adds its own jobs. Check the pipeline to confirm which analyzers run and whether your GitLab tier exposes the desired reports.
- The SBOM job produces `angular-sbom.json` as a downloadable artifact. An SBOM is an inventory of components, not a guarantee that they are secure.

## Step 3 — Push the changes to GitLab

Run these commands from the repository root, not from inside `angular-ui`:

```
git status
git add .gitlab-ci.yml angular-ui
git commit -m "Add Angular and DevSecOps security scanning"
git push
```

If Git reports that there is no upstream branch, push the branch with `git push -u origin <branch-name>`.

## Step 4 — Verify the pipeline

Open GitLab → Build → Pipelines and check the jobs.

| Job                                | Expected result                            |
| ---------------------------------- | ------------------------------------------ |
| `semgrep-sast` or another SAST job | Scans supported source files               |
| `nuget_dependency_scan`            | Lists vulnerable .NET dependencies, if any |
| `npm_dependency_scan`              | Reports Angular dependency vulnerabilities |
| `angular_sbom`                     | Produces `angular-sbom.json`               |
| `dotnet_build`                     | Compiles your .NET API                     |
| `angular_build`                    | Compiles the Angular application           |

Job names generated by the SAST template may vary. If a job fails, open its logs to identify whether it is a configuration problem, an actual finding, or a network/package-source issue.

## Step 5 — Create a Merge Request

Once the pipeline runs, create a feature branch:

```
git checkout -b feature/security-mr-demo
```

Make a harmless change to an Angular component, then commit and push:

```
git add angular-ui/src
git commit -m "Update Angular component"
git push -u origin feature/security-mr-demo
```

Open a merge request in GitLab targeting your default branch.

Review the pipeline status and available security reports. If a finding appears, investigate it, fix genuine issues, and rerun the pipeline.

To enforce successful pipelines before merging, open Settings → Merge requests and look for the merge-check option requiring a successful pipeline. The exact settings available depend on your GitLab tier and project configuration.

## Step 6 — Your classroom demonstration

The practical is complete when you can show these four things:

1. GitLab SAST jobs run against the .NET and Angular source code.
2. NuGet and npm audits inspect third-party dependencies.
3. `angular-sbom.json` can be downloaded from the SBOM job's artifacts.
4. A merge request displays the pipeline status, and your configured merge checks enforce the required pipeline result.

One final note: SAST and dependency scanning are complementary. SAST looks for insecure source-code patterns, dependency scanning identifies known vulnerable libraries, and the SBOM records the components included in the software.

Yes, Raman. Since all your jobs are passing, let's create a controlled security finding so you can demonstrate to students how GitLab CI/CD blocks a pipeline.

We'll start with the easiest and most reliable example: deliberately fail the SAST job using a known insecure code pattern, then test dependency scanning separately. Use a training branch so the demonstration doesn't affect your main branch.

## Example 1 — Make the Angular SAST job detect a vulnerability

We'll add an intentionally unsafe Angular code pattern that can lead to Cross-Site Scripting (XSS).

### Step 1: Find an Angular component

Look inside:

```
angular-ui/src/app/
```

Find the main component file, usually `app.ts` in newer Angular projects or `app.component.ts` in older ones.

Add this method to the component class:

```
showUserInput(userInput: string): void {
  const element = document.getElementById('output');

  if (element) {
    element.innerHTML = userInput;
  }
}
```

This uses `innerHTML` with untrusted input, which is a potentially unsafe pattern.

Important: This is intentionally insecure training code. Do not use it in a production application.

### Step 2: Push the change and check SAST

```
git add angular-ui/src
git commit -m "Add insecure XSS example for SAST lab"
git push
```

Open GitLab → Build → Pipelines and inspect the SAST job.

If the analyzer detects the unsafe `innerHTML` pattern, it may report an XSS-related finding. However, SAST findings do not always make the job fail automatically; this depends on the analyzer and GitLab configuration.

## Example 2 — Guarantee a failed security job

For a classroom demonstration, add this temporary job to `.gitlab-ci.yml`. It deliberately fails when the unsafe pattern is present.

```
security_policy_demo:
  stage: security
  image: alpine:3.22

  script:
    - apk add --no-cache grep
    - |
      if grep -R -n \
        --include='*.ts' \
        'innerHTML = userInput' angular-ui/src; then
        echo "SECURITY POLICY FAILED: Unsafe HTML assignment detected!"
        exit 1
      else
        echo "Security policy passed."
      fi

  allow_failure: false
```

This is a simple demonstration gate, not a replacement for SAST. It detects this specific training pattern, not every possible XSS vulnerability.

### Step 3: Run the pipeline

Commit and push the YAML:

```
git add .gitlab-ci.yml
git commit -m "Add security policy failure demonstration"
git push
```

The `security_policy_demo` job should fail with output similar to:

```
SECURITY POLICY FAILED: Unsafe HTML assignment detected!
ERROR: Job failed: exit code 1
```

This proves that a security check can block the pipeline when a policy violation is detected.

### Step 4: Fix the issue and make the job pass

Remove the intentionally insecure `showUserInput` method from the Angular component, then push the fix:

```
git add angular-ui/src
git commit -m "Remove insecure XSS training example"
git push
```

The security-policy job should now pass because the targeted pattern is gone.

For your class: show the failing pipeline first, explain why `innerHTML` with untrusted input is dangerous, remove the unsafe example, and rerun the pipeline to demonstrate remediation. Once finished, remove the temporary `security_policy_demo` job so it doesn't become a permanent substitute for proper security analysis.
