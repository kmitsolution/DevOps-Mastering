


# GitLab CI/CD: Build, Test, and DAST Security Scanning

## 1. Overview

This pipeline implements a basic DevSecOps workflow using GitLab CI/CD. It builds a Docker image for the .NET 10 application, pushes the image to the GitLab Container Registry, executes unit tests, and performs Dynamic Application Security Testing (DAST) against the running application.

The pipeline also includes GitLab Static Application Security Testing (SAST) to analyze source code for potential security vulnerabilities.

### Objectives

- Build and package the application as a Docker image.
- Store the image in the GitLab Container Registry.
- Execute automated .NET unit tests.
- Perform SAST using GitLab's security analyzer.
- Run DAST against the application running in a service container.
- Publish test results as pipeline artifacts.
- Use a unique image tag for each commit.

## 2. Pipeline Architecture

```
Developer Pushes Code
         |
         v
   BUILD STAGE
   ├── Print build information
   └── Build Docker image
       and push to GitLab Registry
         |
         v
    TEST STAGE
    └── Restore dependencies
        and execute .NET unit tests
         |
         v
     DAST STAGE
     └── Start application as a service
         and scan its HTTP endpoint
         |
         v
   Review Pipeline and Security Results
```

GitLab SAST is included as a separate security analyzer job. Its exact stage and execution order depend on the included template and the pipeline configuration.

## 3. Prerequisites

Before running the pipeline, ensure that the following requirements are met:

1. A GitLab project with a repository containing the .NET application.
2. A valid `Dockerfile` in the repository root.
3. GitLab Container Registry enabled for the project.
4. A GitLab Runner capable of running Docker-based jobs and services.
5. The GitLab subscription and version required for the selected DAST analyzer.
6. A working HTTP endpoint in the application, such as `/hello`.
7. A test project at `tests/DevSecOpsDemo.Tests/DevSecOpsDemo.Tests.csproj`.

The runner must be able to pull the application image and start service containers. If using a self-managed runner, its executor and Docker configuration must support the required services and networking.

## 4. Pipeline Stages

```
stages:
  - build
  - test
  - dast

include:
  - template: Security/SAST.gitlab-ci.yml
  - template: DAST.gitlab-ci.yml

variables:
  TRIVY_NO_PROGRESS: "true"
  APP_NAME: "devsecops-demo"
  APP_VERSION: "1.0.$CI_PIPELINE_IID"
  DOTNET_CLI_TELEMETRY_OPTOUT: "1"
  NUGET_PACKAGES: "$CI_PROJECT_DIR/.nuget/packages"

print_build_info:
  stage: build
  script:
    - 'echo "App: $APP_NAME"'
    - 'echo "Build number: $CI_PIPELINE_IID"'
    - 'echo "Version: $APP_VERSION"'
    - 'echo "Commit: $CI_COMMIT_SHORT_SHA"'

# Build the application image and push it to the project's container registry
# so the DAST job can run it as a service.
build_image:
  stage: build
  image: docker:27
  services:
    - docker:27-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin "$CI_REGISTRY"
  script:
    - docker build -t "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA" .
    - docker push "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA"
  rules:
    - if: $CI_COMMIT_BRANCH

unit_tests:
  stage: test
  image: mcr.microsoft.com/dotnet/sdk:10.0
  cache:
    key: nuget-$CI_COMMIT_REF_SLUG
    paths:
      - .nuget/packages/
  script:
    - dotnet restore tests/DevSecOpsDemo.Tests/DevSecOpsDemo.Tests.csproj
    - >
      dotnet test tests/DevSecOpsDemo.Tests/DevSecOpsDemo.Tests.csproj
      --no-restore
      --configuration Release
      --logger "trx;LogFileName=test-results.trx"
      --results-directory TestResults
  artifacts:
    when: always
    expire_in: 1 week
    paths:
      - TestResults/

# DAST: scan the application running as a service container (port 8080)
dast:
  stage: dast
  dependencies: []
  services:
    - name: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      alias: devsecops-app
  variables:
    DAST_TARGET_URL: "http://devsecops-app:8080/hello"
    DAST_FULL_SCAN_ENABLED: "false"

```
The pipeline defines three explicit stages:

```
stages:
  - build
  - test
  - dast
```

### 4.1 Build Stage

The build stage prints application metadata and creates the Docker image.

#### Print build information

The `print_build_info` job displays the application name, pipeline number, application version, and commit SHA.

These values help identify which source revision produced a particular build.

#### Build and push the Docker image

The `build_image` job:

1. Uses the Docker CLI image.
2. Starts Docker-in-Docker (DinD).
3. Authenticates to the GitLab Container Registry.
4. Builds the Docker image.
5. Pushes the image to the registry.

The image is tagged using the commit SHA:

```
$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

For example, the resulting image reference has this structure:

```
registry.gitlab.com/group/project:<commit-sha>
```

The commit-based tag helps ensure that the DAST job scans the image associated with the pipeline's source revision.

The `rules` section restricts this job to pipelines where `CI_COMMIT_BRANCH` is available.

### 4.2 Test Stage

The `unit_tests` job uses the .NET 10 SDK image.

It restores dependencies and executes tests from:

```
tests/DevSecOpsDemo.Tests/DevSecOpsDemo.Tests.csproj
```

The command runs the tests in Release configuration:

```
dotnet test tests/DevSecOpsDemo.Tests/DevSecOpsDemo.Tests.csproj \
  --no-restore \
  --configuration Release \
  --logger "trx;LogFileName=test-results.trx" \
  --results-directory TestResults
```

The `--no-restore` option avoids a second dependency restore within that job.

#### NuGet caching

The job caches packages under:

```
.nuget/packages/
```

This can reduce dependency-download time on subsequent pipeline runs.

The cache key uses the branch name slug. For more precise cache invalidation, consider incorporating the NuGet lockfile into the cache key.

#### Test artifacts

The job uploads test results from `TestResults/`, even when the test job fails.

Artifacts are retained for one week:

```
artifacts:
  when: always
  expire_in: 1 week
  paths:
    - TestResults/
```

This allows developers to inspect test results after the job completes.

### 4.3 DAST Stage

The DAST job scans the application while it is running in a service container.

Its target configuration is:

```
dast:
  stage: dast
  dependencies: []
  services:
    - name: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
      alias: devsecops-app
  variables:
    DAST_TARGET_URL: "http://devsecops-app:8080/hello"
    DAST_FULL_SCAN_ENABLED: "false"
```

#### How the service container works

The service declaration starts the application image that was pushed by the build job.

```
services:
  - name: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    alias: devsecops-app
```

The alias `devsecops-app` acts as the hostname through which the DAST job can reach the service container.

The application must listen on port `8080` and expose a reachable HTTP endpoint.

#### DAST target URL

```
DAST_TARGET_URL: "http://devsecops-app:8080/hello"
```

This tells the scanner which application URL to test.

Replace `/hello` with a valid endpoint in your application if necessary. Verify that the application starts successfully and that the runner's service networking allows the scanner to reach it.

#### Dependencies

```
dependencies: []
```

This prevents the DAST job from downloading artifacts from earlier jobs through the legacy artifact-dependency mechanism. It does not control job ordering; the stage configuration controls ordering.

The DAST job still requires the application image to have been pushed successfully.

#### Scan mode

```
DAST_FULL_SCAN_ENABLED: "false"
```

This variable is intended to configure a non-full scan in analyzer versions that support it. Confirm that the variable name and behavior match the GitLab DAST template and analyzer version in use. Do not assume that this setting alone guarantees a passive-only scan.

## 5. GitLab Security Templates

The pipeline includes these templates:

```
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: DAST.gitlab-ci.yml
```

### SAST

Static Application Security Testing analyzes source code for potential vulnerabilities without requiring the application to be running.

Examples of issues it may detect include insecure coding patterns and certain injection vulnerabilities.

### DAST

Dynamic Application Security Testing interacts with a running application over HTTP or HTTPS to identify potential runtime security issues.

Examples include missing security headers, insecure cookie configurations, and other detectable web application weaknesses.

**Note:** GitLab template paths and analyzer variables depend on the GitLab version and product offering. Confirm the supported DAST template for your installation before running the pipeline. The browser-based GitLab DAST analyzer has specific subscription and runner requirements.

## 6. Environment Variables

| Variable                      | Purpose                                                                                                               |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `TRIVY_NO_PROGRESS`           | Controls Trivy progress output if Trivy is used elsewhere in the pipeline. It does not enable a Trivy scan by itself. |
| `APP_NAME`                    | Application name used for build metadata.                                                                             |
| `APP_VERSION`                 | Application version derived from the pipeline IID.                                                                    |
| `DOTNET_CLI_TELEMETRY_OPTOUT` | Disables .NET CLI telemetry.                                                                                          |
| `NUGET_PACKAGES`              | Defines the NuGet package-cache directory.                                                                            |
| `CI_REGISTRY_IMAGE`           | GitLab-provided project registry image path.                                                                          |
| `CI_REGISTRY_USER`            | GitLab-provided registry username for CI jobs.                                                                        |
| `CI_REGISTRY_PASSWORD`        | GitLab-provided registry password or token used by CI jobs.                                                           |
| `CI_COMMIT_SHA`               | Commit identifier used as the image tag.                                                                              |
| `DAST_TARGET_URL`             | URL the DAST analyzer scans.                                                                                          |

GitLab provides the registry variables when the Container Registry is enabled and the job has the required permissions.

## 7. Execution Flow

When a developer pushes code to a branch, GitLab creates a pipeline.

1. Build metadata is printed.
2. The application image is built and pushed to the registry.
3. The unit-test job restores dependencies and executes the test suite.
4. The DAST job starts the application image as a service.
5. The scanner accesses the configured target URL.
6. Developers review pipeline logs, test artifacts, and available security reports.

Jobs in the same stage may execute in parallel. Jobs in later stages normally wait for earlier stages to complete successfully, subject to GitLab rules and failure-handling configuration.

## 8. Troubleshooting

| Problem                                | Possible cause                                                                                                | Recommended check                                                                     |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Docker login fails                     | Registry credentials or permissions are incorrect                                                             | Check the GitLab registry variables and project settings.                             |
| Docker image push fails                | Registry is disabled or inaccessible                                                                          | Verify registry availability and runner network access.                               |
| DAST cannot pull the service image     | The image was not pushed, the tag differs, or registry authentication is unavailable to the service container | Confirm the image exists and configure the required service-container authentication. |
| DAST cannot connect to the application | Wrong port, startup failure, or service networking problem                                                    | Check application logs, port configuration, and the target URL.                       |
| Unit tests fail                        | Application regression or test configuration issue                                                            | Review the test logs and TRX artifacts.                                               |
| No DAST findings appear                | Limited crawl coverage, unreachable pages, or no detected issues                                              | Verify the scan logs and reachable application endpoints.                             |

## 9. Security Best Practices

- Use commit-based image tags to identify the exact build being scanned.
- Keep registry credentials out of source code and logs.
- Use protected and masked variables for additional secrets.
- Run DAST against a dedicated test environment with synthetic data.
- Review security findings before promoting the application.
- Keep the scanner and base images updated according to your organization's change-management process.
- Do not interpret a successful scan as proof that the application is free of vulnerabilities.
- Ensure the CI pipeline cannot bypass mandatory security checks.

## 10. Important Pipeline Improvements

The supplied pipeline is a useful starting point, but several details should be verified before relying on it for enforcement:

1. **DAST template path:** Confirm whether your GitLab version supports `DAST.gitlab-ci.yml` or requires a different template path.
2. **DAST configuration:** Verify that `DAST_FULL_SCAN_ENABLED` is supported by the selected analyzer version.
3. **Service readiness:** Ensure the application has started and is ready to accept requests before scanning.
4. **Registry authentication:** Confirm that the service container can pull a private project image. Authentication in the `build_image` job does not automatically guarantee that the service container can authenticate.
5. **Pipeline job rules:** Ensure the build, test, and DAST jobs run under the same intended branch and merge-request conditions.
6. **Security report enforcement:** Decide which findings must block a merge or deployment and configure appropriate policies.
7. **SAST stage placement:** Inspect the merged CI configuration to confirm where the included SAST job runs.

## 11. Conclusion

This pipeline demonstrates a practical DevSecOps workflow by combining container image delivery, automated unit testing, source-code analysis, and dynamic application scanning.

The most important concept is that the DAST job scans the actual application image running as a service, rather than scanning source code alone. This helps identify security issues that only become visible when the application is running.

The resulting workflow provides an effective foundation for extending the project with dependency scanning, secret detection, Trivy image scanning, merge-request approvals, and deployment security gates.

Important implementation note: In your original YAML, `build_image` and `unit_tests` are in separate stages, so DAST only starts after the test stage succeeds. However, the `build_image` job currently has branch-only rules; check merge-request pipeline behavior and the exact DAST template path supported by your GitLab version before treating this as a production-ready security gate.
