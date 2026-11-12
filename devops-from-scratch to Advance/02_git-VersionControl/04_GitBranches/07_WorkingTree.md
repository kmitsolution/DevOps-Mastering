# Git Worktree 

`git worktree` allows you to check out **multiple branches of the same repository into separate folders at the same time**.

Normally, one Git repository can have only one branch checked out in a working directory. With `git worktree`, you can work on several branches simultaneously without cloning the repository multiple times.

---

# Why Use `git worktree`?

Without `git worktree`:

* You have one folder.
* You switch between branches using `git switch`.
* Uncommitted changes may prevent switching.

With `git worktree`:

* Each branch gets its own directory.
* You can open multiple branches in different editor windows.
* No need for multiple full clones.

---

# Example Scenario

Repository branches:

* `main`
* `dev`
* `feature-login`

Main repository location:

```text
C:\Projects\kmit-project
```

Current branch:

```text
main
```

---

# Create a Worktree for `dev`

```bash
git worktree add kmit-project-dev dev
```

This creates:

```text
C:\Projects\kmit-project        (main)
C:\Projects\kmit-project-dev    (dev)
```

Both directories share the same `.git` object database, so only one copy of repository history is stored.

---

# Visual Representation

```text
Shared Git Repository Data (.git)
        │
        ├── kmit-project       → main
        └── kmit-project-dev   → dev
```

---

# Create a New Branch and Worktree Together

```bash
git worktree add -b feature-api kmit-project-api
```

This:

1. Creates a new branch named `feature-api`
2. Creates a new directory
3. Checks out the new branch there

---

# List Worktrees

```bash
git worktree list
```

Example output:

```text
C:/Projects/kmit-project        abc1234 [main]
C:/Projects/kmit-project-dev    def5678 [dev]
```

---

# Remove a Worktree

```bash
git worktree remove kmit-project-dev
```

This removes the additional working directory but leaves the branch and commits intact.

---

# Prune Stale Worktree Metadata

```bash
git worktree prune
```

Useful if a worktree directory was deleted manually.

---

# Real-World Use Cases

* Fix a production bug on `main` while continuing feature work on `dev`
* Compare branches side by side
* Run multiple test environments simultaneously
* Build and test release branches independently

---

# Practical Example

```bash
git clone https://github.com/YOUR_USERNAME/kmit-project.git
cd kmit-project

# Main branch in current directory
git branch

# Create a second directory for dev
git worktree add ../kmit-project-dev dev

# Open the new directory
cd kmit-project-dev
git branch
```

---

# Worktree vs Clone

| Feature                    | `git clone` | `git worktree` |
| -------------------------- | ----------- | -------------- |
| Separate directories       | Yes         | Yes            |
| Separate `.git` database   | Yes         | No (shared)    |
| Additional disk usage      | Higher      | Lower          |
| Requires downloading again | Yes         | No             |

---

# Important Rule

A branch can be checked out in only one worktree at a time. If `dev` is already checked out in another worktree, Git will prevent checking it out elsewhere until that worktree is removed or switched to another branch.

---

# Summary

```bash
git worktree add ../kmit-project-dev dev
```

This command creates a new directory where the `dev` branch is checked out, while sharing the same underlying repository data. It is an efficient way to work on multiple branches simultaneously without creating additional full clones.
