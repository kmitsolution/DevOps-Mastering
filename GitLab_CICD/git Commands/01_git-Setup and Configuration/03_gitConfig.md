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

## System-Level Git Configuration

### What is System-Level Git Config?

**System-level Git configuration** applies to **all users and all Git repositories on a machine**.
It defines **default settings** that affect every user unless they are overridden by **global** or **local** configuration.

This level is mainly used by **system administrators** to enforce organization-wide defaults.

---

## Configuration Levels Recap (Context)

Git has **three configuration levels**, evaluated in this priority order:

```
Local  →  Global  →  System
(highest)          (lowest)
```

➡ If the same setting exists at multiple levels:

* **Local** overrides Global and System
* **Global** overrides System
* **System** acts as the default baseline

---

## System-Level Config File Location

### Linux / macOS

```
/etc/gitconfig
```

### Windows

```
C:\Program Files\Git\etc\gitconfig
```

⚠️ Modifying system-level config usually requires **administrator/root privileges**.

---

## How to Set System-Level Configuration

Use the `--system` flag:

```
sudo git config --system key value
```

### Example

```
sudo git config --system core.editor vim
```

This sets **vim as the default editor** for all users on the system.

---

## How to View System-Level Configuration

```
git config --system --list
```

To see **where each config value comes from**:

```
git config --list --show-origin
```

---

## Common Use Cases for System-Level Config

### 1. Setting Default Editor for All Users

```
git config --system core.editor nano
```

📌 Ensures consistency across teams using shared machines.

---

### 2. Enforcing Line Ending Behavior

```
git config --system core.autocrlf input
```

📌 Prevents line-ending issues across operating systems.

---

### 3. Setting Default Branch Name

```
git config --system init.defaultBranch main
```

📌 Standardizes repository initialization across an organization.

---

## Scenario-Based Examples

### Scenario 1: Corporate Training Lab

**Situation:**
A company sets up a shared Linux server for Git training.

**Requirement:**

* Same editor
* Same default branch
* Same Git behavior for everyone

**Solution:**
Use system-level configuration so no individual setup is required.

✔ Result: Consistent environment for all users.

---

### Scenario 2: Enterprise Policy Enforcement

**Situation:**
An organization wants to enforce certain Git behaviors (e.g., line endings).

**Solution:**
Configure system-level defaults and allow developers to override locally if required.

---

## When NOT to Use System-Level Config

* Personal identity settings (name, email)
* Repo-specific behaviors
* Developer-specific preferences

These belong to **global** or **local** configuration instead.

---

## Comparison: System vs Global vs Local

| Feature      | System               | Global              | Local            |
| ------------ | -------------------- | ------------------- | ---------------- |
| Scope        | All users, all repos | One user, all repos | One repo         |
| File         | `/etc/gitconfig`     | `~/.gitconfig`      | `.git/config`    |
| Priority     | Lowest               | Medium              | Highest          |
| Typical User | Admin                | Developer           | Project-specific |

---

### One-Line Summary

> **System-level Git configuration defines default settings for all users and repositories on a machine and has the lowest priority among Git config levels.**

