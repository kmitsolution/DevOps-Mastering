# Git Reset (`--soft`, `--mixed`, and `--hard`) 

`git reset` is used to move the current branch (`HEAD`) to a specific commit and optionally update:

1. **HEAD** (current commit pointer)
2. **Staging Area (Index)**
3. **Working Directory (Actual Files)**

---

# Git Areas Overview

```text
Working Directory  → Your actual files
Staging Area       → Files prepared for commit
Repository (HEAD)  → Last committed snapshot
```

---

# Sample Commit History

```text
A --- B --- C (HEAD -> master)
```

Suppose:

* `A` = Initial commit
* `B` = Added login
* `C` = Added dashboard

If we run:

```bash
git reset <option> B
```

We move `HEAD` from `C` back to `B`.

---

# 1. `git reset --soft`

## What Happens?

* HEAD moves to target commit
* Staging area is preserved
* Working directory is preserved

## Diagram

```text
HEAD -> B
Staging Area: Changes from C are staged
Working Directory: Changes from C are present
```

## Example

```bash
git reset --soft HEAD~1
```

### Before

```text
A --- B --- C (HEAD)
```

### After

```text
A --- B (HEAD)
```

Changes introduced by commit `C` are still staged.

## Check Status

```bash
git status
```

Output:

```text
Changes to be committed:
    modified: app.txt
```

## Use Case

Useful when:

* You want to undo the last commit
* Keep all changes staged
* Recommit with a new message

```bash
git commit -m "Better commit message"
```

---

# 2. `git reset --mixed` (Default)

## What Happens?

* HEAD moves to target commit
* Staging area is cleared
* Working directory is preserved

## Diagram

```text
HEAD -> B
Staging Area: Empty
Working Directory: Changes from C are present but unstaged
```

## Example

```bash
git reset --mixed HEAD~1
```

or simply:

```bash
git reset HEAD~1
```

## Check Status

```text
Changes not staged for commit:
    modified: app.txt
```

## Use Case

Useful when:

* You want to undo a commit
* Review and selectively stage files again

```bash
git add app.txt
git commit -m "Revised commit"
```

---

# 3. `git reset --hard`

## What Happens?

* HEAD moves to target commit
* Staging area is reset
* Working directory is reset

## Diagram

```text
HEAD -> B
Staging Area: Empty
Working Directory: Changes from C are deleted
```

## Example

```bash
git reset --hard HEAD~1
```

## Check Status

```text
nothing to commit, working tree clean
```

## Use Case

Useful when:

* You want to permanently discard commits and local changes

> ⚠️ Warning: Uncommitted changes will be lost.

---

# Complete Practical Example

```bash
echo "Version 1" > app.txt
git add .
git commit -m "Commit A"

echo "Version 2" >> app.txt
git commit -am "Commit B"

echo "Version 3" >> app.txt
git commit -am "Commit C"
```

History:

```text
A --- B --- C (HEAD)
```

---

## Soft Reset

```bash
git reset --soft HEAD~1
```

* Commit C removed
* Changes from C staged

---

## Mixed Reset

```bash
git reset --mixed HEAD~1
```

* Commit C removed
* Changes from C unstaged

---

## Hard Reset

```bash
git reset --hard HEAD~1
```

* Commit C removed
* Changes from C deleted

---

# Comparison Table

| Option    | Move HEAD | Keep Staged Changes | Keep File Changes | Deletes Changes |
| --------- | --------- | ------------------- | ----------------- | --------------- |
| `--soft`  | Yes       | Yes                 | Yes               | No              |
| `--mixed` | Yes       | No                  | Yes               | No              |
| `--hard`  | Yes       | No                  | No                | Yes             |

---

# Memory Trick

* **Soft** → "Only uncommit"
* **Mixed** → "Uncommit and unstage"
* **Hard** → "Erase everything"

---

# Common Commands

```bash
git reset --soft HEAD~1     # Undo last commit, keep staged
git reset --mixed HEAD~1    # Undo last commit, keep files
git reset --hard HEAD~1     # Undo last commit, delete files
git reset --hard origin/main # Match remote branch exactly
```

---

# Real-World Scenarios

### Wrong Commit Message

```bash
git reset --soft HEAD~1
git commit -m "Correct message"
```

### Forgot to Review Files

```bash
git reset HEAD~1
```

### Discard All Local Changes

```bash
git reset --hard HEAD
```

### Synchronize with Remote

```bash
git fetch origin
git reset --hard origin/main
```

---

# Visual Summary

```text
Commit C
   |
   v

--soft
  Commit removed, changes staged

--mixed
  Commit removed, changes unstaged

--hard
  Commit removed, changes deleted
```
