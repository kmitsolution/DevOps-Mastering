## Git Configuration Files – Local and Global

Git uses **configuration files** to store **user information, preferences, and behavior settings**.
These settings control how Git works on your system and on individual repositories.

---

## What is Git Configuration?

**Git configuration** defines:

* Who you are (name, email)
* How Git behaves (editor, aliases, colors, line endings, etc.)
* Scope of settings (system-wide, user-wide, or repository-specific)

Git supports **three levels** of configuration:

1. System
2. Global
3. Local

👉 In practice, **Global and Local** are the most commonly used.

---

## Configuration Priority (Important)

When the same setting exists in multiple places, Git follows this order:

```
Local  →  Global  →  System
(highest)          (lowest)
```

So, **local config overrides global config**.

---

## 1. Global Git Configuration

### What is Global Configuration?

* Applies to **all Git repositories** for a specific user
* Stored in the user’s home directory
* Used for **default identity and preferences**

### Configuration File Location

* Linux / macOS: `~/.gitconfig`
* Windows: `C:\Users\<username>\.gitconfig`

### How to Set Global Configuration

```
git config --global user.name "John Doe"
git config --global user.email "john.doe@example.com"
```

### How to View Global Configuration

```
git config --global --list
```

---

### Scenario-Based Example (Global)

**Scenario:**
You are a developer working on **multiple projects**, and you want the **same name and email** for all repositories.

**Solution:**
Set global configuration once:

```
git config --global user.name "John Doe"
git config --global user.email "john.doe@company.com"
```

✔ Now every repository automatically uses this identity.

---

## 2. Local Git Configuration

### What is Local Configuration?

* Applies to **only one specific repository**
* Stored inside the repository itself
* Overrides global settings

### Configuration File Location

```
<repo>/.git/config
```

### How to Set Local Configuration

(Inside the repository directory)

```
git config user.name "Project Bot"
git config user.email "bot@project.com"
```

### How to View Local Configuration

```
git config --local --list
```

---

### Scenario-Based Example (Local)

**Scenario:**
You work on:

* Personal GitHub projects
* Office/company project

Your company requires commits to use your **official company email**, not your personal one.

**Solution:**

1. Set personal email globally

```
git config --global user.email "personal@gmail.com"
```

2. Override email only for the company repository

```
cd company-project
git config user.email "john.doe@company.com"
```

✔ Result:

* Personal repos → personal email
* Company repo → company email

---

## Another Practical Scenario (Local)

**Scenario:**
A repository requires:

* Different merge behavior
* Custom editor
* Special hooks or aliases

**Solution:**
Configure locally so it affects **only that repo**, not others.

Example:

```
git config core.editor "vim"
```

This setting applies **only to the current repository**.

---

## Comparison: Local vs Global Configuration

| Feature       | Global Config         | Local Config            |
| ------------- | --------------------- | ----------------------- |
| Scope         | All repositories      | Single repository       |
| File Location | `~/.gitconfig`        | `.git/config`           |
| Priority      | Lower                 | Higher                  |
| Common Use    | Name, email, defaults | Repo-specific overrides |

---

## Useful Commands Summary

| Purpose           | Command                           |
| ----------------- | --------------------------------- |
| Set global config | `git config --global key value`   |
| Set local config  | `git config key value`            |
| View all configs  | `git config --list`               |
| View file origin  | `git config --list --show-origin` |

---

### One-Line Summary

> **Global Git configuration applies to all repositories for a user, while local configuration applies to a single repository and overrides global settings when needed.**
