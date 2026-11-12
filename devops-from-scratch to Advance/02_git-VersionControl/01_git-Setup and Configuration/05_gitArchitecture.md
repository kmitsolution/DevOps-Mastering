## Git Architecture – Working Tree, Staging Area, and Repository

**Git architecture** explains how Git internally manages files and tracks changes using **three main areas**:

1. Working Tree
2. Staging Area
3. Repository

Understanding this flow is essential to use Git effectively.

---

## 1. Working Tree (Working Directory)

### What is the Working Tree?

The **working tree** is the **actual directory on your local machine** where you create, edit, and delete files.

* It contains the **current version of files**
* Files here can be modified, new, or deleted
* Git watches this area for changes

### File States in Working Tree

* **Untracked** – New files not yet added to Git
* **Modified** – Tracked files that have been changed

### Example

You edit a file:

```
index.html
```

At this point, the change exists **only in the working tree**.

---

## 2. Staging Area (Index)

### What is the Staging Area?

The **staging area** (also called the **index**) is an **intermediate area** where you prepare changes before committing them.

* It allows you to **select specific changes** for the next commit
* Acts as a preview of what will go into the repository

### Why Staging Area is Important

* Partial commits are possible
* Better control over commits
* Cleaner commit history

### Example

Add a file to staging:

```
git add index.html
```

Now the file moves from **working tree → staging area**.

---

## 3. Repository (Local Repository)

### What is the Repository?

The **repository** is where Git **permanently stores committed snapshots** of the project.

* Contains full project history
* Stored in the `.git` directory
* Each commit has a unique hash (SHA)

### Example

Commit staged changes:

```
git commit -m "Add homepage file"
```

Now the file moves from **staging area → repository**.

---

## Git Architecture Flow (Text Diagram)



```
Working Tree
   │
   │  git add
   ▼
Staging Area
   │
   │  git commit
   ▼
Repository
```

---

## Summary Table

| Area         | Purpose        | Commands     |
| ------------ | -------------- | ------------ |
| Working Tree | Edit files     | edit, delete |
| Staging Area | Prepare commit | git add      |
| Repository   | Store history  | git commit   |

---

## Why This Architecture Matters

* Enables selective commits
* Improves collaboration
* Ensures reliable version tracking
* Supports powerful branching and merging

---

### One-Line Summary

> **Git architecture consists of the working tree for edits, the staging area for preparing changes, and the repository for permanently storing version history.**

