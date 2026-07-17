Yes. Below is a complete and corrected `.gitlab-ci.yml` for your ASP.NET Core project with:

✅ Build
✅ Unit Test
✅ Code Coverage
✅ Coverage Threshold
✅ Package ZIP
✅ Upload to JFrog Artifactory

---

## First Create GitLab Variables

Go to:

```text
Project → Settings → CI/CD → Variables
```

Create:

| Variable        | Value                                                        |
| --------------- | ------------------------------------------------------------ |
| JF_URL          | [https://trialmkwslq.jfrog.io](https://trialmkwslq.jfrog.io) |
| JF_ACCESS_TOKEN | Your New Access Token                                        |
| JF_REPO         | my-local-dotnet-generic-local                                |

**Important:** The token you posted is now exposed. Revoke it and generate a new token in JFrog.

---

# Complete `.gitlab-ci.yml`

```yaml
stages:
  - build
  - test
  - package
  - upload

variables:
  PACKAGE_NAME: "ProductApi-${CI_PIPELINE_ID}.zip"

############################################################
# BUILD
############################################################

build_app:
  stage: build
  image: mcr.microsoft.com/dotnet/sdk:9.0

  script:
    - dotnet --version
    - dotnet restore
    - dotnet publish ProductApi/ProductApi.csproj -c Release -o publish
    - ls -lrt
    - ls -lrt publish

  artifacts:
    paths:
      - publish/
    expire_in: 1 day

############################################################
# UNIT TEST + COVERAGE
############################################################

unit_test:
  stage: test
  image: mcr.microsoft.com/dotnet/sdk:9.0

  before_script:
    - dotnet tool install --global trx2junit
    - dotnet tool install --global dotnet-reportgenerator-globaltool
    - export PATH="$PATH:/root/.dotnet/tools"

  script:

    - dotnet test --logger "trx;LogFileName=test_results.trx" --collect:"XPlat Code Coverage"

    # Convert TRX to JUnit XML
    - find . -name "*.trx" -exec trx2junit {} \;

    # Generate Coverage Report
    - |
      reportgenerator \
      "-reports:**/coverage.cobertura.xml" \
      "-targetdir:coveragereport" \
      "-reporttypes:HtmlInline;Cobertura"

    # Find Coverage File
    - find . -name coverage.cobertura.xml

    # Coverage Validation
    - |
      FILE=$(find . -name coverage.cobertura.xml | head -1)

      echo "Coverage File : $FILE"

      COVERAGE=$(grep -oP 'line-rate="\K[^"]+' "$FILE")

      PERCENT=$(awk -v cov="$COVERAGE" \
      'BEGIN { printf "%.2f", cov*100 }')

      echo "Coverage = $PERCENT %"

      if awk -v c="$PERCENT" \
      'BEGIN { exit !(c < 10) }'
      then
        echo "Coverage below 10%"
        exit 1
      fi

  coverage: '/Coverage = (\d+\.\d+)/'

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
        path: ProductUnitTest/TestResults/**/coverage.cobertura.xml

############################################################
# PACKAGE
############################################################

package_app:
  stage: package
  image: mcr.microsoft.com/dotnet/sdk:9.0

  needs:
    - job: build_app
      artifacts: true

  before_script:
    - apt-get update
    - apt-get install -y zip

  script:
    - zip -r "$PACKAGE_NAME" publish/
    - ls -lrt

  artifacts:
    paths:
      - "*.zip"
    expire_in: 7 days

############################################################
# UPLOAD TO JFROG
############################################################

upload_jfrog:
  stage: upload

  image:
    name: releases-docker.jfrog.io/jfrog/jfrog-cli-v2-jf:latest

  needs:
    - job: package_app
      artifacts: true

  script:

    - jf --version

    - |
      jf config add artifactory \
      --url="$JF_URL" \
      --access-token="$JF_ACCESS_TOKEN" \
      --interactive=false

    - jf config show

    - ls -lrt

    - |
      jf rt upload \
      "*.zip" \
      "${JF_REPO}/ProductApi/"

    - |
      jf rt search \
      "${JF_REPO}/ProductApi/"
```

---

# Pipeline Flow

```text
Git Push
    │
    ▼
Build
    │
    ▼
publish/
    │
    ▼
Unit Test
    │
    ▼
Coverage Reports
    │
    ▼
ZIP Package
    │
    ▼
JFrog Upload
```

---

# JFrog Repository Structure

After upload:

```text
my-local-dotnet-generic-local
│
└── ProductApi
      │
      ├── ProductApi-12.zip
      ├── ProductApi-13.zip
      └── ProductApi-14.zip
```

---

# Recommended Next Step

After this works, we can improve it further by adding:

* Semantic versioning (`1.0.${CI_PIPELINE_IID}`)
* Upload Build Information to JFrog
* Xray security scanning
* Download artifact from JFrog for deployment
* Dev → QA → Prod promotion flow

For now, first verify that:

```text
build → test → package → upload
```

runs successfully end-to-end.
