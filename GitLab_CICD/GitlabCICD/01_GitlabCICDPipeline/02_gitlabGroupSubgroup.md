## What are GitLab Groups?

A **Group** in GitLab is a logical container used to organize multiple projects and users.

Think of it like this:

```text
Company
 ├── Development Team (Group)
 │     ├── Project A
 │     ├── Project B
 │     └── Project C
```

### Example

Suppose your company is **KMIT Solutions**.

You can create a group:

```text
kmit-solutions
```

Inside the group, you can create projects:

```text
devops-course
azure-course
gitlab-course
terraform-course
```

This makes management easier.

---

# Why do we need Groups?

Without groups:

```text
raman/devops-course
raman/azure-course
raman/gitlab-course
```

Everything is under your personal account.

Problems:

* Difficult to manage permissions
* Cannot easily share administration
* Difficult to organize projects
* Difficult to apply common settings

With Groups:

```text
kmit-solutions/devops-course
kmit-solutions/gitlab-course
```

Everything is centrally managed.

---

# Benefits of Groups

### 1. User Management

You can add members once.

Example:

```text
DevOps Team
├── Raman (Owner)
├── John (Maintainer)
├── Smith (Developer)
```

All projects automatically inherit permissions.

---

### 2. Common CI/CD Variables

You can define:

```text
DOCKER_USER
DOCKER_PASSWORD
AWS_ACCESS_KEY
```

at group level.

Every project can use them.

---

### 3. Shared Runners

Instead of configuring runners for every project.

One runner can serve all projects.

---

### 4. Security Policies

You can enforce:

* Approval Rules
* Protected Branches
* SAST Policies
* Compliance Pipelines

for all projects.

---

# What are Subgroups?

Subgroups are groups inside another group.

Example:

```text
KMIT-Solutions
│
├── Development
│      ├── Banking-App
│      ├── HR-App
│
├── Training
│      ├── GitLab-Course
│      ├── Azure-Course
│
├── DevOps-Team
       ├── Terraform
       ├── Kubernetes
```

This provides hierarchical organization.

---

# Why do we need Subgroups?

Large organizations may have:

* Multiple departments
* Multiple teams
* Multiple business units

Subgroups help separate responsibilities.

---

## Example

```text
TCS
│
├── BFSI
├── Telecom
├── Healthcare
├── Retail
```

Each can have its own projects.

---

# What is Namespace?

Namespace is the unique path where a project exists.

Example:

```text
https://gitlab.com/kmit-solutions/gitlab-course
```

Namespace:

```text
kmit-solutions
```

If subgroup:

```text
https://gitlab.com/kmit-solutions/training/gitlab-course
```

Namespace:

```text
kmit-solutions/training
```

---

## Purpose of Namespace

Namespace determines:

1. Ownership
2. Permissions
3. URL structure
4. Storage location
5. CI/CD inheritance

---

# What is Slug?

Slug is a URL-friendly version of the name.

Example:

Group Name:

```text
KMIT Solutions Services
```

Slug becomes:

```text
kmit-solutions-services
```

URL:

```text
https://gitlab.com/kmit-solutions-services
```

---

## Another Example

Project Name:

```text
GitLab CI/CD Course
```

Slug:

```text
gitlab-ci-cd-course
```

Final URL:

```text
https://gitlab.com/kmit-solutions/gitlab-ci-cd-course
```

---

# Why do we need Slugs?

Because URLs cannot properly handle:

* Spaces
* Special characters
* Duplicate names

Slug provides:

* Unique URL
* SEO-friendly URL
* Easier API access
* Easier cloning

---

Example:

```bash
git clone https://gitlab.com/kmit-solutions/gitlab-course.git
```

---

# Project Creation Inside Group

## Step 1

Go to:

```text
Groups
```

---

## Step 2

Select:

```text
KMIT-Solutions
```

---

## Step 3

Click:

```text
New Project
```

---

## Step 4

Choose:

```text
Create Blank Project
```

---

## Step 5

Enter:

### Project Name

```text
gitlab-course
```

### Project Slug

```text
gitlab-course
```

### Namespace

```text
kmit-solutions
```

---

## Step 6

Choose Visibility:

* Private
* Internal
* Public

---

## Step 7

Initialize:

```text
README.md
.gitignore
License
```

---

## Step 8

Click:

```text
Create Project
```

---

# Final Structure

```text
KMIT-Solutions
│
├── Training (Subgroup)
│      ├── GitLab-Course
│      ├── Azure-Course
│
├── DevOps-Team
│      ├── Terraform
│      ├── Kubernetes
│
└── Applications
       ├── Banking-App
       └── HR-App
```

---

# Real Industry Example

```text
Infosys
│
├── Banking
│      ├── Customer-Service
│      ├── Payment-Service
│
├── Insurance
│      ├── Policy-Service
│      ├── Claims-Service
│
├── Platform-Team
       ├── CI-CD
       ├── Terraform
       ├── Monitoring
```

This hierarchy provides:

✅ Centralized Administration
✅ Better Security
✅ Easier Permission Management
✅ Shared CI/CD Resources
✅ Better Organization
✅ Compliance and Governance

This is why almost every enterprise GitLab implementation uses **Groups → Subgroups → Projects** hierarchy instead of keeping projects under personal namespaces.
