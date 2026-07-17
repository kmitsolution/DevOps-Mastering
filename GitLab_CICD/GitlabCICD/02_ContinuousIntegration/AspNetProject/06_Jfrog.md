**JFrog Cloud Free Trial**, let's first understand only the **JFrog steps**, without GitLab CI/CD for now.

---

# Architecture

For your current ASP.NET Core API project:

```text id="mivh8t"
Developer Machine
        │
        ▼
dotnet publish
        │
        ▼
ProductApi.zip
        │
        ▼
JFrog Artifactory
```

Later GitLab will automate this.

---

# Step 1 : Login to JFrog Cloud

Open:

```text id="vyh15n"
https://<your-company>.jfrog.io
```

You will see:

```text id="icn0n3"
Platform
├── Artifactory
├── Xray
├── Pipelines
└── Distribution
```

Open:

```text id="n8xkqj"
Artifactory
```

---

# Step 2 : Understand Repository Types

You will see:

```text id="lhyrx5"
Repositories
├── Local
├── Remote
└── Virtual
```

---

## Local Repository

Stores your packages.

Example:

```text id="9y4c6s"
mycompany-nuget-local
dotnet-generic-local
```

---

## Remote Repository

Caches external repositories.

Example:

```text id="m2m2i1"
nuget.org
docker.io
```

---

## Virtual Repository

Single URL combining Local + Remote.

Developers usually use this.

---

# Step 3 : Create Repository

For your API project we will use:

```text id="5f3p5q"
Generic Repository
```

because we are uploading:

```text id="v1x7s3"
zip file
```

---

Navigate:

```text id="ud6n6m"
Administration
      ↓
Repositories
      ↓
Create Repository
```

Choose:

```text id="l5b1rh"
Local
```

Then choose:

```text id="gt6mhm"
Generic
```

---

Repository Name:

```text id="bw0t2g"
dotnet-generic-local
```

Click:

```text id="pr0w5u"
Create
```

---

# Repository Structure

You will see:

```text id="vbg3v2"
dotnet-generic-local
```

Initially empty.

---

# Step 4 : Publish ASP.NET Project

On local machine:

```bash id="gjfij9"
dotnet publish ProductApi/ProductApi.csproj \
-c Release \
-o publish
```

Creates:

```text id="jovgrf"
publish/
│
├── ProductApi.dll
├── ProductApi.exe
├── appsettings.json
└── runtime files
```

---

# Step 5 : Compress Output

Windows:

```powershell id="4rvtdr"
Compress-Archive `
-Path publish\* `
-DestinationPath ProductApi-1.0.0.zip
```

Linux:

```bash id="wx13fi"
zip -r ProductApi-1.0.0.zip publish/
```

---

# Step 6 : Upload Through UI

Open:

```text id="1ck8d5"
Artifacts
      ↓
dotnet-generic-local
```

Click:

```text id="8h4lq8"
Deploy
```

Upload:

```text id="5m1c42"
ProductApi-1.0.0.zip
```

---

After upload:

```text id="mjlwmm"
dotnet-generic-local
│
└── ProductApi-1.0.0.zip
```

---

# Step 7 : Download Artifact

Click:

```text id="a2zwf5"
ProductApi-1.0.0.zip
```

You can download it again.

This proves Artifactory is working.

---

# Step 8 : Understand Build Promotion Concept

Later CI/CD becomes:

```text id="8fhk3e"
GitLab
   │
   ▼
Build
   │
   ▼
ProductApi-1.0.0.zip
   │
   ▼
Artifactory
```

Deployment servers then download artifacts from JFrog.

---

# Step 9 : Create NuGet Repository (Learning Purpose)

Now create another repository.

Go to:

```text id="1b9rza"
Administration
      ↓
Repositories
      ↓
Create
```

Choose:

```text id="e1up53"
NuGet
```

Repository Name:

```text id="9n1s4w"
nuget-local
```

---

This repository stores:

```text id="qfepot"
*.nupkg
```

---

# Step 10 : Create Remote Repository

Create:

```text id="hiz1z0"
NuGet Remote Repository
```

URL:

```text id="ry7g7c"
https://api.nuget.org
```

This acts as a cache.

---

# Step 11 : Create Virtual Repository

Create:

```text id="7qk90d"
nuget-virtual
```

Include:

```text id="k2c0pq"
nuget-local
nuget-remote
```

Developers then configure:

```text id="jlwm7f"
nuget-virtual
```

instead of nuget.org.

---

# Final Architecture

```text id="jlwm1o"
NuGet Client
      │
      ▼
nuget-virtual
      │
 ┌────┴────┐
 │         │
 ▼         ▼
Local    Remote
            │
            ▼
        nuget.org
```

---

# Recommended Learning Order

## Phase 1

Create Generic Repo.

Upload ZIP manually.

---

## Phase 2

Download manually.

---

## Phase 3

Create NuGet Repository.

---

## Phase 4

Create Remote Repository.

---

## Phase 5

Create Virtual Repository.

---

## Phase 6

Use CLI.

---

## Phase 7

Integrate GitLab.

---

# Topics for Your Course

1. Introduction to Artifactory
2. Repository Types
3. Generic Repository
4. NuGet Repository
5. Remote Repository
6. Virtual Repository
7. Manual Upload
8. Manual Download
9. Build Information
10. Artifacts vs Packages
11. Security Scanning
12. GitLab Integration

---

# For your current ASP.NET Core project, the immediate next steps are:

```text id="jlwmfo"
1. Create Generic Repository
2. Publish Project
3. Zip publish folder
4. Upload ZIP manually
5. Download ZIP and verify
```

Once this is working, we can move to:

```text id="njlwmc"
GitLab → JFrog Upload Automation
```
