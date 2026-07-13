# Adding Code Coverage in ASP.NET Core + MSTest
```
In your MSTest project (ProductUnitTest.csproj), ensure you have:
<ItemGroup>
  <PackageReference Include="coverlet.collector" Version="6.*">
    <PrivateAssets>all</PrivateAssets>
  </PackageReference>
</ItemGroup>
```
Now that your build and unit testing pipeline is working, the next step is **Code Coverage**.

Code Coverage tells us:

* How much of the source code is executed by our unit tests.
* Which methods are tested.
* Which methods are not tested.

Example:

```text id="b2nq4v"
Controller.cs

Method1()  ✓ Covered
Method2()  ✓ Covered
Method3()  ✗ Not Covered
```

---

# Part 1 : Check Code Coverage in Visual Studio 2022

## Step 1 : Install Required NuGet Package

Open your **MSTest Project**.

Install:

```bash id="m16thd"
dotnet add package coverlet.collector
```

or using NuGet Package Manager install:

```text id="v2gwyk"
coverlet.collector
```

You should see:

```xml id="mp6u83"
<PackageReference Include="coverlet.collector" Version="6.*" />
```

inside:

```text id="ec4s7m"
ProductUnitTest.csproj
```

---

## Step 2 : Run Tests with Coverage

From terminal:

```bash id="n9qg6m"
dotnet test --collect:"XPlat Code Coverage"
```

After execution:

```text id="2g1ru1"
TestResults/
```

will be created.

Inside:

```text id="c6r7y5"
TestResults
│
└── GUID
      │
      ├── coverage.cobertura.xml
      └── test_results.trx
```

---

# Part 2 : View Coverage Report Locally

The XML file is difficult to read.

Install ReportGenerator.

```bash id="o53l0j"
dotnet tool install -g dotnet-reportgenerator-globaltool
```

---

Generate HTML Report:

```bash id="bwl34t"
reportgenerator \
-reports:**/coverage.cobertura.xml \
-targetdir:coveragereport \
-reporttypes:Html
```

This creates:

```text id="mnzxmh"
coveragereport/
```

Open:

```text id="6ef8sp"
coveragereport/index.html
```

You will get a nice report.

Example:

```text id="v2j9y8"
Line Coverage : 85%
Branch Coverage : 80%
```

---

# Using Visual Studio GUI

In VS2022:

```text id="s7wft4"
Test
   ↓
Analyze Code Coverage
   ↓
All Tests
```

Visual Studio Enterprise supports built-in coverage visualization.

For Community/Professional editions, Coverlet is usually preferred.

---

# Part 3 : Add Code Coverage to GitLab Pipeline

---

# Step 1

Install ReportGenerator in Pipeline.

---

## Updated Test Job

```yaml id="k3v4ja"
unit_test:
  stage: test
  image: mcr.microsoft.com/dotnet/sdk:9.0

  before_script:
    - dotnet tool install --global trx2junit
    - dotnet tool install --global dotnet-reportgenerator-globaltool

    - export PATH="$PATH:/root/.dotnet/tools"

  script:
    - dotnet test
      --logger "trx;LogFileName=test_results.trx"
      --collect:"XPlat Code Coverage"

    - find . -name "*.trx" -exec trx2junit {} \;

    - reportgenerator
      -reports:**/coverage.cobertura.xml
      -targetdir:coveragereport
      -reporttypes:HtmlInline;Cobertura
```

---

# What Happens?

```text id="k8jzcg"
dotnet test
        │
        ▼
coverage.cobertura.xml
        │
        ▼
reportgenerator
        │
        ▼
coveragereport/
```

---

# Step 2 : Upload Coverage Reports

```yaml id="d3obzk"
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
```

---

# Complete Job

```yaml id="rcc31f"
unit_test:
  stage: test
  image: mcr.microsoft.com/dotnet/sdk:9.0

  before_script:
    - dotnet tool install --global trx2junit
    - dotnet tool install --global dotnet-reportgenerator-globaltool

    - export PATH="$PATH:/root/.dotnet/tools"

  script:
    - dotnet test --logger "trx;LogFileName=test_results.trx" --collect:"XPlat Code Coverage"

    - find . -name "*.trx" -exec trx2junit {} \;

    - reportgenerator \
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
```

---

# Pipeline Flow

```text id="h4slvo"
Build
   │
   ▼
Unit Test
   │
   ├── Execute Tests
   ├── Generate TRX
   ├── Generate Coverage XML
   ├── Convert TRX → JUnit
   └── Generate HTML Report
            │
            ▼
      Upload Artifacts
```

---

# Where to See Coverage?

### 1. Pipeline Artifacts

Download:

```text id="ejt7zx"
coveragereport/index.html
```

Open it in browser.

---

### 2. Merge Request Coverage

GitLab can show coverage inside merge requests if the report is configured correctly.

---

### 3. GitLab UI

GitLab Premium/Ultimate can display richer coverage information directly in the UI.

---

# Example Folder Structure

```text id="2h89pi"
TestResults/
│
├── test_results.trx
└── coverage.cobertura.xml

coveragereport/
│
├── index.html
├── classes.html
└── files.html
```

---

# Sample Coverage Report

```text id="v5hgs3"
ProductController.cs

Lines Covered : 90%
Branches Covered : 85%
Methods Covered : 100%
```

---

# Recommended NuGet Packages

Install in test project:

```xml id="yyxmkx"
<PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.*" />
<PackageReference Include="MSTest.TestAdapter" Version="3.*" />
<PackageReference Include="MSTest.TestFramework" Version="3.*" />
<PackageReference Include="coverlet.collector" Version="6.*" />
```

---

# Final CI/CD Flow

```text id="m2i3ak"
Developer Push
       │
       ▼
Build Stage
       │
       ▼
Unit Test Stage
       │
       ├── dotnet test
       ├── Generate TRX
       ├── Generate Coverage XML
       ├── Generate HTML Report
       └── Upload Reports
               │
               ▼
         GitLab Artifacts
```
