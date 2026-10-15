# Lesson 3 — SAST with SonarQube Cloud and GitLab CI/CD

Our next objective is to scan the .NET application for potential security vulnerabilities using SonarQube Cloud, without exposing the SonarQube server on your laptop.

## 1. Our current pipeline

We already have these stages:

```
GitLab Repository
       |
       v
  Gitleaks Scan
       |
       v
   Build (.NET 10)
       |
       v
  Unit Test + Coverage
```

We'll add SAST to this pipeline.

```
GitLab Repository
       |
       v
  Gitleaks Scan
       |
       v
 SonarQube SAST
       |
       v
 Security / Quality Gate
       |
       v
   Build + Test
```

One important detail: for .NET, SonarScanner needs to wrap the build process (`begin` → build → `end`). Therefore, we will integrate SonarScanner into the build job rather than treating it as an entirely independent scan job.

## 2. Create your SonarQube Cloud account

Open the official service:

[SonarQube Cloud — Official Website](https://sonarcloud.io/)

Follow these steps:

1. Sign in using your GitLab account.
2. Select the option to analyze your first projects.
3. Import or connect your GitLab repository, if the onboarding flow offers it.
4. Select your `devsecops-demo` project.
5. Create or select a SonarQube Cloud organization.
6. Note the project key and organization key displayed during setup.

The project key and organization key are different values. We will need both in our CI configuration. SonarQube Cloud's official GitLab onboarding guide explains the repository connection process.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.sonarsource.com\&sz=32)

Sonar Documentation



If the onboarding flow asks you to import a GitLab group and you don't have the required group ownership, use the available project onboarding route or follow the permissions requested by SonarQube Cloud.

## 3. Generate a SonarQube Cloud token

The GitLab runner needs a token to submit analysis results.

In SonarQube Cloud, open your account or organization settings and locate the token management section. Generate a token appropriate for analysis, following the permissions available on your plan.

Store the token securely. Do not commit it to Git.

You should now have three values:

| Setting          | Example                            |
| ---------------- | ---------------------------------- |
| Organization key | `your-organization`                |
| Project key      | `your-organization_devsecops-demo` |
| Analysis token   | Your generated token               |

These are examples, not your actual values. Copy the keys from your SonarQube Cloud project.

## 4. Configure GitLab CI/CD variables

Open your project in GitLab:

`Settings → CI/CD → Variables`

Add these variables:

Variable 1 — `SONAR_HOST_URL`

Value

`https://sonarcloud.io`

Variable 2 — `SONAR_TOKEN`

Value

Your generated SonarQube Cloud analysis token.

Variable 3 — `SONAR_ORGANIZATION`

Value

Your actual SonarQube Cloud organization key.

Variable 4 — `SONAR_PROJECT_KEY`

Value

Your actual SonarQube Cloud project key.

For `SONAR_TOKEN`, enable masking if GitLab accepts the token's format. Restrict it to protected branches if your pipeline runs on those branches only. Ensure the variable is available to the pipeline you are testing.

Do not paste your actual token into this conversation or into `.gitlab-ci.yml`.

## 5. Update `.gitlab-ci.yml`

Now we will integrate SonarScanner for .NET into the existing pipeline.

Our pipeline will contain three stages:

1. `security` — Gitleaks scans for secrets.
2. `build` — SonarScanner analyzes the .NET application during the build.
3. `test` — Unit tests run and generate test and coverage reports.

SonarScanner for .NET must wrap the build with `begin` and `end`. We will follow this sequence in the build job.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.sonarsource.com\&sz=32)

Sonar Documentation

+1



Use this complete YAML, retaining your current test-report configuration:

```

stages:
  - security
  - build
  - test

variables:
  SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"
  DOTNET_CLI_TELEMETRY_OPTOUT: "1"

secret_scan:
  stage: security
  image:
    name: zricethezav/gitleaks:latest
    entrypoint: [""]

  script:
    - gitleaks version
    - gitleaks detect --source . --verbose --exit-code 1

build_app:
  stage: build
  image: mcr.microsoft.com/dotnet/sdk:10.0

  before_script:
    - dotnet tool install --global dotnet-sonarscanner
    - export PATH="$PATH:/root/.dotnet/tools"

  script:
    - dotnet --version
    - dotnet sonarscanner begin /k:"$SONAR_PROJECT_KEY" /o:"$SONAR_ORGANIZATION" /d:sonar.host.url="$SONAR_HOST_URL" /d:sonar.token="$SONAR_TOKEN" /d:sonar.qualitygate.wait=true
    - dotnet restore
    - dotnet build --no-restore
    - dotnet publish devsecops-demo/devsecops-demo.csproj -c Release --no-restore -o publish
    - dotnet sonarscanner end /d:sonar.token="$SONAR_TOKEN"

  artifacts:
    paths:
      - publish/
    expire_in: 1 day

unit_test:
  stage: test
  image: mcr.microsoft.com/dotnet/sdk:10.0

  before_script:
    - dotnet tool install --global trx2junit
    - dotnet tool install --global dotnet-reportgenerator-globaltool
    - export PATH="$PATH:/root/.dotnet/tools"

  script:
    - dotnet test --logger "trx;LogFileName=test_results.trx" --collect:"XPlat Code Coverage"
    - find . -name "*.trx"
    - find . -name "coverage.cobertura.xml"
    - find . -name "*.trx" -exec trx2junit {} \;
    - |
      reportgenerator \
        "-reports:**/coverage.cobertura.xml" \
        "-targetdir:coveragereport" \
        "-reporttypes:HtmlInline;Cobertura"

  artifacts:
    when: always
    paths:
      - coveragereport/
      - "**/*.trx"
      - "**/*.xml"
    reports:
      junit:
        - "**/*.xml"
      coverage_report:
        coverage_format: cobertura
        path: "**/coverage.cobertura.xml"
    expire_in: 7 days

```

### Understand the important commands

Start the analysis

```
dotnet sonarscanner begin \
  /k:"$SONAR_PROJECT_KEY" \
  /o:"$SONAR_ORGANIZATION" \
  /d:sonar.host.url="$SONAR_HOST_URL" \
  /d:sonar.token="$SONAR_TOKEN" \
  /d:sonar.qualitygate.wait=true
```

- `/k` identifies your SonarQube Cloud project.
- `/o` identifies your organization.
- `sonar.host.url` identifies the cloud service.
- `sonar.token` authenticates the analysis.
- `sonar.qualitygate.wait=true` asks the scanner to wait for the Quality Gate result and fail the job if the gate fails.

Build while analysis is active

```
dotnet restore
dotnet build --no-restore
```

The scanner collects analysis data during the build.

Submit the analysis

```
dotnet sonarscanner end /d:sonar.token="$SONAR_TOKEN"
```

This submits the analysis to SonarQube Cloud. The quality-gate setting allows the build job to fail if the resulting gate fails.

Important: If SonarQube Cloud has automatic analysis enabled for this project, disable it before using this CI-based scanner workflow. The two analysis modes should not run concurrently.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.sonarsource.com\&sz=32)

Sonar Documentation



## 6. Commit and run the pipeline

After updating the YAML, commit and push:

```
git add .gitlab-ci.yml
git commit -m "Add SonarQube Cloud SAST"
git push
```

Open your GitLab project:

`Build → Pipelines`

You should see the following execution order:

Stage 1 — Secret scanning

Gitleaks checks the repository.

Stage 2 — Build + SAST

SonarScanner analyzes the .NET project and submits results.

Stage 3 — Unit tests

Tests run and generate TRX and coverage reports.

The green icons illustrate the intended flow, not a claim that your pipeline has already passed.

## 7. Verify the analysis in SonarQube Cloud

Open your SonarQube Cloud project and review:

- Overview: analysis status and project metrics.
- Issues: detected bugs, vulnerabilities and code-quality issues.
- Security Hotspots: security-sensitive code that needs review.
- Quality Gate: whether the configured conditions passed.

You should see a new analysis after the GitLab job completes successfully.

## 8. Test the security gate

Once the first analysis succeeds, check your project's Quality Gate configuration.

For a meaningful exercise:

1. Review the default Quality Gate.
2. Understand which conditions it evaluates.
3. Introduce a controlled example of insecure code in a separate branch.
4. Push the change and observe the analysis.
5. Check whether the finding affects the configured Quality Gate.

Don't assume every reported issue will fail the pipeline. That depends on the rules, severity and Quality Gate conditions.

Also, the current pipeline performs SAST in the build job, but it does not yet feed the test-stage coverage report into SonarQube. We can integrate coverage into the scanner workflow in a later improvement.

## 9. Troubleshooting checklist

| Error                             | What to check                                                                |
| --------------------------------- | ---------------------------------------------------------------------------- |
| `401 Unauthorized`                | Verify `SONAR_TOKEN` and its permissions.                                    |
| Project or organization not found | Verify the exact keys from SonarQube Cloud.                                  |
| Connection refused or timeout     | Check the runner's Internet access and `SONAR_HOST_URL`.                     |
| Quality Gate failed               | Open the Quality Gate and inspect the failed conditions.                     |
| No analysis appears               | Check the `sonarscanner end` logs and ensure automatic analysis is disabled. |
| Gitleaks fails first              | Investigate the detected finding before continuing.                          |

If your Gitleaks job still fails because of a test secret committed in earlier history, deleting the file alone may not resolve it. We should address that separately before diagnosing SonarScanner.

## Lesson 3 completion checklist

SonarQube Cloud project created

GitLab CI/CD variables configured

SonarScanner integrated with the .NET build

Analysis visible in SonarQube Cloud

Quality Gate result understood

0 of 5 completed

Next lesson: Lesson 4 — SCA (Software Composition Analysis). We will scan the NuGet dependencies of your .NET application for known vulnerabilities and learn how to prevent vulnerable dependencies from progressing through the pipeline.
