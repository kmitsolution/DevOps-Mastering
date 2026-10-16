# Lesson 5 — Docker Image Security Scanning with Trivy and GitLab CI/CD

DevSecOps hands-on lab · .NET 10 · Beginner-friendly · Approximately 30–40 minutes

Even if your source code and NuGet packages pass security checks, your Docker image can still contain vulnerable operating-system packages, outdated libraries, or other known vulnerabilities.

## 1. What is container image scanning?

Imagine your application is built into a Docker image using a base image such as `mcr.microsoft.com/dotnet/aspnet:10.0`.

That image contains more than your application—it also includes a runtime and operating-system components. Trivy scans the image for known vulnerabilities and can report misconfigurations and exposed secrets as well.

DevSecOpsDemo

.NET 10 application

Docker image

Application + .NET runtime + OS packages

Trivy scanner

Checks known vulnerabilities in image components

Pass

No findings match the blocking policy

Fail

Blocking vulnerability found

## 2. What is Trivy?

Trivy is an open-source security scanner that can inspect:

- Container images for known vulnerabilities.
- OS packages and application dependencies.
- Infrastructure-as-code configurations.
- Secrets and certain configuration risks.

In this lesson, we'll focus on container image vulnerabilities.

## 3. Create a Dockerfile for `DevSecOpsDemo`

In the same directory as `DevSecOpsDemo.csproj`, create a file named `Dockerfile`.

```
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

COPY DevSecOpsDemo.csproj .
RUN dotnet restore DevSecOpsDemo.csproj

COPY . .
RUN dotnet publish DevSecOpsDemo.csproj \
    -c Release \
    -o /app/publish \
    --no-restore

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS runtime
WORKDIR /app

COPY --from=build /app/publish .
EXPOSE 8080

ENTRYPOINT ["dotnet", "DevSecOpsDemo.dll"]
```

This assumes the project is an ASP.NET Core web application. If your project uses a different output assembly name, update `DevSecOpsDemo.dll` accordingly.

The multi-stage build keeps the SDK in the build stage and uses the smaller ASP.NET Core runtime image for execution.

## 4. Build and run the image locally

Open a terminal in the directory containing the Dockerfile.

Build the image:

```
docker build -t devsecops-demo:lesson5 .
```

Run the container:

```
docker run --rm -p 8080:8080 devsecops-demo:lesson5
```

Open `http://localhost:8080` if your application has a suitable endpoint. Stop the container with `Ctrl+C`.

## 5. Scan the image using Trivy

You can use Trivy locally or inside a GitLab job. For a local demonstration, after installing Trivy, run:

```
trivy image devsecops-demo:lesson5
```

To focus on high and critical findings:

```
trivy image --severity HIGH,CRITICAL devsecops-demo:lesson5
```

This filters the report to those severities; it does not necessarily fail the command when findings exist.

To fail the command if high or critical vulnerabilities are found:

```
trivy image \
  --severity HIGH,CRITICAL \
  --exit-code 1 \
  devsecops-demo:lesson5
```

Trivy's exit-code option lets you turn vulnerability findings into a pipeline gate.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

+1



## 6. Integrate image scanning into GitLab CI/CD

For this simple lab, we'll build the image and scan it in one job. This example uses Docker-in-Docker, which requires a compatible GitLab Runner configuration. GitLab documents the runner requirements and security trade-offs.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

+1



Add the following job to your `.gitlab-ci.yml`:

```
container_scan:
  stage: test
  image: docker:27-cli

  services:
    - name: docker:27-dind
      alias: docker

  variables:
    DOCKER_HOST: tcp://docker:2375
    DOCKER_TLS_CERTDIR: ""
    IMAGE_NAME: "devsecops-demo:$CI_COMMIT_SHA"

  before_script:
    - apk add --no-cache curl tar
    - docker info
    - |
      TRIVY_VERSION="0.69.3"
      curl -sfL \
        "https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-64bit.tar.gz" \
        -o /tmp/trivy.tar.gz
      tar -xzf /tmp/trivy.tar.gz -C /tmp trivy
      install /tmp/trivy /usr/local/bin/trivy

  script:
    - docker build -t "$IMAGE_NAME" ./DevSecOpsDemo
    - trivy image --severity HIGH,CRITICAL --exit-code 1 "$IMAGE_NAME"
```

Before using this example, verify the Trivy release version and compatible Docker image tags for your environment. If your project directory differs, adjust the Docker build context.

Runner requirement: Docker-in-Docker commonly requires privileged runner configuration. If your GitLab shared runner doesn't permit it, this job may fail before Trivy runs. You can instead build with an approved rootless image builder or scan an image that has already been pushed to your registry.

## 7. What should students observe?

Case A — No high or critical findings

Trivy completes successfully and the pipeline continues. This doesn't mean the image is guaranteed vulnerability-free.

Case B — High or critical findings

Trivy exits with code 1 and the GitLab job fails, preventing later dependent stages from proceeding.

If no findings appear, don't deliberately downgrade the base image just to force a failure. You can demonstrate the blocking behavior using a known vulnerable image in a separate, isolated test job, without deploying it.

## 8. How do we fix vulnerabilities?

When Trivy reports a vulnerability:

1. Identify the affected package and CVE.
2. Check whether a patched package or base-image update is available.
3. Update the Dockerfile's base image or the relevant application dependency.
4. Rebuild the image.
5. Run Trivy again and verify the result.

Avoid suppressing findings without a documented reason and review process.

## 9. DevSecOps pipeline progress

| Lesson | Security check                       |
| ------ | ------------------------------------ |
| 2      | Gitleaks — committed secrets         |
| 3      | SAST / source-code analysis          |
| 4      | NuGet dependency vulnerabilities     |
| 5      | Docker image vulnerabilities — Trivy |

Practical task: Create the Dockerfile, build `devsecops-demo:lesson5`, scan it locally, then add the GitLab job. Once this works, the next lesson can cover Infrastructure-as-Code security scanning with Checkov or Trivy, including Terraform misconfigurations.
