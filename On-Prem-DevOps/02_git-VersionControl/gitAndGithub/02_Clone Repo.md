## Clone an Entire Repository

To copy a complete repository from <b> GitHub </b> to your local machine:

```bash
git clone https://github.com/YOUR_USERNAME/kmit-project.git
git branch -a
git fetch origin # to get the update from remote repository like if a new branch and you want the update
```

This command:

1. Creates a folder named `kmit-project`
2. Downloads all commits, branches, and tags
3. Checks out the default branch (usually `main`)

---

## Clone into a Specific Folder Name

```bash
git clone https://github.com/YOUR_USERNAME/kmit-project.git myproject
```

This creates a local folder named `myproject`.

---

## Clone a Specific Branch Only

If you want to clone and immediately check out a particular branch, for example `development`:

```bash
git clone -b development https://github.com/YOUR_USERNAME/kmit-project.git
git branch -a # it will show all the branches and we can switch or checkout to a particular branch
```

* `-b` means `--branch`
* Git checks out the `development` branch after cloning

---

## Clone Only One Branch (Shallow Branch Clone)

To download only one branch instead of all branches:

```bash
git clone --single-branch -b development https://github.com/YOUR_USERNAME/kmit-project.git
git branch -a # it will show only dev branch
```

This is useful when:

* You only need one branch
* You want to reduce download size
* You are demonstrating branch-specific cloning

---

## Clone Repository and Then Switch to Another Branch

```bash
git clone https://github.com/YOUR_USERNAME/kmit-project.git
cd kmit-project
git checkout development
```

---

## Modern Alternative to `git checkout`

```bash
git switch development
```

---

## List All Available Branches

```bash
git branch -a
```

Example output:

```text
* main
  remotes/origin/main
  remotes/origin/development
  remotes/origin/feature-login
```

---

## Clone from a Specific Repository Example

```bash
git clone https://github.com/kmitsolution/kmit-project.git
```

Clone the `development` branch directly:

```bash
git clone -b development https://github.com/kmitsolution/kmit-project.git
```

---

## Summary

| Command                                            | Description                           |
| -------------------------------------------------- | ------------------------------------- |
| `git clone <repo-url>`                             | Clone the full repository             |
| `git clone -b <branch> <repo-url>`                 | Clone and check out a specific branch |
| `git clone --single-branch -b <branch> <repo-url>` | Download only one branch              |
| `git branch -a`                                    | Show local and remote branches        |
| `git switch <branch>`                              | Switch to another branch              |

---
## What Is `--depth` in `git clone`?

The `--depth` option creates a **shallow clone**, which means Git downloads only a limited number of recent commits instead of the entire repository history.

---

# Basic Syntax

```bash
git clone --depth <number> <repository-url>
```

Example:

```bash
git clone --depth 1 https://github.com/YOUR_USERNAME/kmit-project.git
```

---

# What `--depth 1` Does

Git downloads:

* The latest commit
* The current working files
* Minimal history

Git does **not** download older commits.

---

# Example Repository History

```text
A → B → C → D → E (latest)
```

---

## Normal Clone

```bash
git clone https://github.com/YOUR_USERNAME/kmit-project.git
```

Downloads:

```text
A → B → C → D → E
```

---

## Shallow Clone with `--depth 1`

```bash
git clone --depth 1 https://github.com/YOUR_USERNAME/kmit-project.git
```

Downloads only:

```text
E
```

---

## Shallow Clone with `--depth 3`

```bash
git clone --depth 3 https://github.com/YOUR_USERNAME/kmit-project.git
```

Downloads:

```text
C → D → E
```

---

# Why Use `--depth`?

### Faster Cloning

Large repositories can contain thousands of commits. Shallow cloning reduces download time.

### Reduced Bandwidth

Only the required history is transferred.

### Smaller Disk Usage

Less commit data is stored locally.

### Common in CI/CD

Tools such as [Jenkins](https://www.jenkins.io?utm_source=chatgpt.com), [GitHub Actions](https://github.com/features/actions?utm_source=chatgpt.com), and [Azure DevOps](https://azure.microsoft.com/products/devops?utm_source=chatgpt.com) often use shallow clones because build servers typically need only the latest code.

---

# Combining with Branch Cloning

```bash
git clone --depth 1 --single-branch -b development https://github.com/YOUR_USERNAME/kmit-project.git
```

This command:

1. Clones only the `development` branch
2. Downloads only the latest commit
3. Minimizes clone size and time

---

# Limitations of Shallow Clones

Some operations may be restricted:

* Viewing complete history
* Running certain comparisons
* Rebasing across missing commits
* Generating release notes from older commits

---

# Convert to a Full Clone Later

```bash
git fetch --unshallow
```

This downloads the remaining history.

---

# Check Whether the Repository Is Shallow

```bash
git rev-parse --is-shallow-repository
```

Output:

```text
true
```

or

```text
false
```

---

# Real-World Example

A repository contains 20,000 commits.

| Clone Type   | Commits Downloaded |
| ------------ | ------------------ |
| Normal clone | 20,000             |
| `--depth 1`  | 1                  |
| `--depth 10` | 10                 |

---

# Summary

* `--depth N` downloads only the most recent `N` commits.
* `--depth 1` is the most common shallow clone.
* Useful for CI/CD, automation, and large repositories.
* Can be combined with `-b` and `--single-branch`.
* Full history can be retrieved later with `git fetch --unshallow`.

---

# Most Common CI/CD Command

```bash
git clone --depth 1 --single-branch -b main https://github.com/YOUR_USERNAME/kmit-project.git
```

This downloads only the latest commit from the `main` branch, making cloning fast and efficient.



