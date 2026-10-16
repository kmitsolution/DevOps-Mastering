# Lesson 4 — Software Composition Analysis (SCA) with GitLab CI/CD and .NET 10

Hands-on DevSecOps lab · Beginner-friendly · 30–45 minutes

Raman, let's rewrite the entire lesson using your actual project name, `DevSecOpsDemo.csproj`, with a simple, controlled vulnerable NuGet dependency. Students will add the dependency, run the scan, observe the vulnerability warning, enforce a security policy in GitLab CI/CD, and then fix the vulnerable package.

## 1. What is SCA?

Software Composition Analysis (SCA) identifies known security vulnerabilities in third-party and open-source libraries used by an application.

For example, your .NET application may use NuGet packages for JSON processing, logging, authentication, or web functionality. Even if your own code is secure, an outdated package might have a published vulnerability.

DevSecOpsDemo.csproj

Our .NET 10 application

NuGet packages

Direct and transitive dependencies

NuGet vulnerability audit

Checks package versions against known advisories

No blocking finding

Pipeline continues

High/critical finding

Pipeline fails according to policy

### SCA versus SAST

| SAST                                                             | SCA                                                          |
| ---------------------------------------------------------------- | ------------------------------------------------------------ |
| Analyzes your application's source code for security weaknesses. | Analyzes dependencies for known vulnerabilities.             |
| Example: unsafe handling of user input.                          | Example: a NuGet package with a published security advisory. |
| Tools: GitLab SAST, Semgrep.                                     | Tools: NuGet auditing, dependency scanners.                  |

## 2. Prepare your project

We'll use the following example layout. Adjust the path if your repository is organized differently.

```
DevSecOpsDemo/
├── .gitlab-ci.yml
├── DevSecOpsDemo/
│   ├── DevSecOpsDemo.csproj
│   └── Program.cs
└── tests/
```

Open a terminal in the repository root and verify your .NET SDK:

```
dotnet --version
```

You should see a .NET 10 SDK version, such as `10.0.402`.

List your project's current dependencies:

```
dotnet list DevSecOpsDemo/DevSecOpsDemo.csproj package
```

This command lists the direct NuGet packages referenced by the application.

## 3. Enable NuGet auditing

Open `DevSecOpsDemo/DevSecOpsDemo.csproj`.

Inside the existing `PropertyGroup`, add these settings. Keep your existing project properties and package references rather than replacing the entire file blindly.

```
<NuGetAudit>true</NuGetAudit>
<NuGetAuditMode>all</NuGetAuditMode>
<NuGetAuditLevel>low</NuGetAuditLevel>
<RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
```

For example, the relevant part of the project file should look like this:

```
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>

    <NuGetAudit>true</NuGetAudit>
    <NuGetAuditMode>all</NuGetAuditMode>
    <NuGetAuditLevel>low</NuGetAuditLevel>
    <RestorePackagesWithLockFile>true</RestorePackagesWithLockFile>
  </PropertyGroup>

</Project>
```

These settings enable NuGet auditing, check the full dependency graph, report vulnerabilities starting at low severity, and generate a package lock file.

## 4. Add an intentionally vulnerable package for the demonstration

We'll use `System.Text.Encodings.Web` version `4.5.0` as a training example. This older version has a published security advisory. Keep it in a disposable training branch only; do not deploy an application with this dependency.

Add the following `ItemGroup` to your existing `.csproj` file:

```
<ItemGroup>
  <!-- TRAINING ONLY: intentionally vulnerable dependency -->
  <PackageReference Include="System.Text.Encodings.Web"
                    Version="4.5.0" />
</ItemGroup>
```

Your project file now has the auditing properties from Step 3 and this additional package reference. Do not remove any packages the application already needs.

Why add a vulnerable package? It gives the class a concrete dependency to inspect instead of relying on an existing project that may have no findings.

## 5. Run the scan locally

From your repository root, restore the project:

```
dotnet restore DevSecOpsDemo/DevSecOpsDemo.csproj
```

Look for NuGet audit warnings during restore.

Now list vulnerable direct and transitive dependencies:

```
dotnet list DevSecOpsDemo/DevSecOpsDemo.csproj package --vulnerable --include-transitive
```

The output format varies with the SDK and available vulnerability data. Students should look for:

- The affected package name.
- The resolved version.
- The vulnerability severity.
- The advisory or URL, when available.

If the command reports no vulnerabilities, check that the package was actually restored and that the configured package source provides vulnerability data. Do not assume that a successful command means no vulnerabilities exist in all possible sources.

Generate or update the lock file:

```
dotnet restore DevSecOpsDemo/DevSecOpsDemo.csproj
```

Check for:

```
DevSecOpsDemo/packages.lock.json
```

Commit the lock file so dependency resolution can be tracked.

## 6. Add the SCA job to GitLab CI/CD

Open `.gitlab-ci.yml` and add the following job. This is a reporting job: it restores the application and prints the vulnerability report.

```
stages:
  - security
  - build
  - test

nuget_dependency_scan:
  stage: security
  image: mcr.microsoft.com/dotnet/sdk:10.0

  script:
    - dotnet --version
    - dotnet restore DevSecOpsDemo/DevSecOpsDemo.csproj
    - dotnet list DevSecOpsDemo/DevSecOpsDemo.csproj package --vulnerable --include-transitive

  allow_failure: false
```

If you already have `stages` defined, merge `security` into your existing list. Do not delete your Gitleaks, build or test jobs. The path in the commands must match your actual `.csproj` location.

Commit and push the changes:

```
git add .gitlab-ci.yml DevSecOpsDemo/DevSecOpsDemo.csproj
git add DevSecOpsDemo/packages.lock.json
git commit -m "Add NuGet dependency vulnerability scan"
git push
```

Then open GitLab → Build → Pipelines → latest pipeline → `nuget_dependency_scan`.

At this point, the job displays the vulnerability report, but it may still succeed even if vulnerabilities are found. We need a separate blocking policy.

## 7. Block high and critical vulnerabilities

NuGet uses these warning codes:

| Code     | Severity |
| -------- | -------- |
| `NU1901` | Low      |
| `NU1902` | Moderate |
| `NU1903` | High     |
| `NU1904` | Critical |

To make high and critical findings fail the restore, add this property group to your `.csproj`:

```
<PropertyGroup Condition="'$(AuditPipeline)' == 'true'">
  <WarningsAsErrors>$(WarningsAsErrors);NU1903;NU1904</WarningsAsErrors>
</PropertyGroup>
```

Now change the restore command in the GitLab job to:

```
- dotnet restore DevSecOpsDemo/DevSecOpsDemo.csproj -p:AuditPipeline=true
```

When NuGet reports a high or critical vulnerability as `NU1903` or `NU1904`, restore should fail and GitLab marks the job as failed.

Teaching point: This restore may fail before the separate package-report command runs. That is expected for a blocking policy. The restore log itself should contain the relevant warning. You can run the reporting command locally to inspect the full dependency report.

For the first demonstration, you can leave the blocking policy out until the reporting job works. Then add it and run the pipeline again.

## 8. Fix the vulnerability and rerun the pipeline

After students have observed the finding, remove the training-only vulnerable package reference and use a patched compatible version. For this demonstration, change the package version to `10.0.0`:

```
<ItemGroup>
  <PackageReference Include="System.Text.Encodings.Web"
                    Version="10.0.0" />
</ItemGroup>
```

This version change is an example of remediation, not a guarantee that every package version or dependency combination is free of vulnerabilities. Confirm the selected version and compatibility against the current NuGet advisory before using it.

Run the scan again:

```
dotnet restore DevSecOpsDemo/DevSecOpsDemo.csproj
dotnet list DevSecOpsDemo/DevSecOpsDemo.csproj package --vulnerable --include-transitive
```

Then commit and push the fix:

```
git add DevSecOpsDemo/DevSecOpsDemo.csproj
git add DevSecOpsDemo/packages.lock.json
git commit -m "Remediate vulnerable NuGet dependency"
git push
```

The pipeline should pass the high/critical gate if no other high or critical vulnerability warnings are reported.

## 9. Final DevSecOps pipeline

### 1

Secret scanning

Gitleaks

Detect committed credentials

### 2

Dependency scanning

NuGet audit

Detect vulnerable package versions

### 3

Build

dotnet build

Compile the .NET 10 application

### 4

Unit tests

dotnet test

Verify application behavior

### 5

Container scanning

Trivy — next lesson

Inspect image and OS package vulnerabilities

## 10. Questions to ask students

1. What is a NuGet package?
2. What is the difference between direct and transitive dependencies?
3. What is the purpose of `dotnet list package --vulnerable`?
4. Why should high and critical vulnerabilities block a pipeline?
5. Why should a vulnerable dependency be upgraded instead of simply ignoring the warning?

Lesson outcome: Students will have a working NuGet dependency scan in GitLab CI/CD, understand the difference between reporting and blocking findings, and demonstrate remediation by upgrading a vulnerable dependency.
