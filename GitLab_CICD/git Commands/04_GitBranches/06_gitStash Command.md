## Git Stash Command

`git stash` is used to **temporarily save your uncommitted changes** (both staged and unstaged) so that you can switch branches or pull new changes without committing your work.

Think of it as putting your current work into a temporary storage area and restoring it later.

---

# Why Use Git Stash?

Suppose you are working on a feature and suddenly need to:

* Fix a production bug
* Switch to another branch
* Pull the latest changes
* Test something quickly

But your current changes are not ready to commit.

In this case, use `git stash`.

---

# Basic Workflow

```bash
# Save current changes
git stash

# View all stashes
git stash list

# Restore latest stash and remove it from stash list
git stash pop

# Restore latest stash but keep it in stash list
git stash apply

# Delete a specific stash
git stash drop stash@{0}

# Delete all stashes
git stash clear
```

---

# Real-Time Example

## Step 1: Modify a File

```bash
echo "New feature code" >> app.js
git status
```

Output:

```bash
modified: app.js
```

---

## Step 2: Save Changes Temporarily

```bash
git stash
```

Output:

```bash
Saved working directory and index state WIP on main: abc123 Initial commit
```

Now your working directory becomes clean.

```bash
git status
```

Output:

```bash
nothing to commit, working tree clean
```

---

## Step 3: Switch to Another Branch

```bash
git checkout hotfix
```

Make your urgent changes and commit them.

---

## Step 4: Return to Original Branch

```bash
git checkout main
```

---

## Step 5: Restore Stashed Changes

```bash
git stash pop
```

Your previous work is restored.

---

# Git Stash List

```bash
git stash list
```

Example Output:

```bash
stash@{0}: WIP on main: abc123 Initial commit
stash@{1}: WIP on feature-login: def456 Added login page
```

---

# Create a Named Stash

```bash
git stash push -m "Work on login feature"
```

This makes it easier to identify later.

---

# Restore a Specific Stash

```bash
git stash apply stash@{1}
```

---

# Stash Including Untracked Files

```bash
git stash -u
```

or

```bash
git stash --include-untracked
```

---

# Stash Everything Including Ignored Files

```bash
git stash -a
```

or

```bash
git stash --all
```

---

# Show What Is Inside a Stash

```bash
git stash show
```

Detailed diff:

```bash
git stash show -p
```

---

# Difference Between Apply and Pop

| Command           | Restores Changes | Removes Stash |
| ----------------- | ---------------- | ------------- |
| `git stash apply` | Yes              | No            |
| `git stash pop`   | Yes              | Yes           |

---

# Practical DevOps Use Cases

* Temporarily save changes to Ansible playbooks
* Switch branches to fix urgent CI/CD pipeline issues
* Store Kubernetes YAML modifications
* Save Terraform changes before pulling updates

---

# Most Common Commands

```bash
git stash
git stash push -m "my changes"
git stash list
git stash show -p stash@{0}
git stash apply stash@{0}
git stash pop
git stash drop stash@{0}
git stash clear
git stash -u
```

---

# Summary

`git stash` is one of the most useful Git commands for temporarily saving incomplete work. It allows you to clean your working directory, switch contexts, and resume your changes later without creating unnecessary commits.
