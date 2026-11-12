# Git Fetch, Merge, and Pull Explained

These three commands are closely related and are fundamental to synchronizing your local repository with a remote repository such as [GitHub](https://github.com?utm_source=chatgpt.com).

---

# 1. `git fetch`

```bash
git fetch origin
```

`git fetch` downloads new commits, branches, and tags from the remote repository, but it does **not** modify your current branch or working files.

### Example

Before fetch:

```text
Local main:   A → B
origin/main:  A → B
Remote main:  A → B → C → D
```

After `git fetch origin`:

```text
Local main:   A → B
origin/main:  A → B → C → D
Remote main:  A → B → C → D
```

Your working branch is unchanged, but Git updates the remote-tracking branch `origin/main`.

### Useful Commands

```bash
git fetch origin
git log --oneline main..origin/main
git diff main origin/main
```

---

# 2. `git merge`

```bash
git merge origin/main
```

`git merge` combines changes from another branch into your current branch.

### Example

After fetching, if you are on `main`:

```bash
git merge origin/main
```

Before merge:

```text
main:         A → B
origin/main:  A → B → C → D
```

After merge (fast-forward case):

```text
main:         A → B → C → D
```

---

# 3. `git pull`

```bash
git pull origin main
```

`git pull` is a shortcut for:

```bash
git fetch origin
git merge origin/main
```

It downloads changes and immediately merges them into your current branch.

---

# Comparison Table

| Command     | Downloads Changes | Updates Working Branch |
| ----------- | ----------------- | ---------------------- |
| `git fetch` | Yes               | No                     |
| `git merge` | No                | Yes                    |
| `git pull`  | Yes               | Yes                    |

---

# Recommended Workflow

Many developers prefer:

```bash
git fetch origin
git log --oneline main..origin/main
git merge origin/main
```

This allows them to inspect incoming changes before applying them.

---

# Pull with Rebase

```bash
git pull --rebase
```

Equivalent to:

```bash
git fetch
git rebase
```

This keeps history more linear by replaying local commits on top of the updated remote branch.

# `git pull --rebase` Example

`git pull --rebase` is used when:

1. Someone has pushed new commits to the remote branch.
2. You have local commits that have not been pushed yet.
3. You want to place your local commits on top of the latest remote changes, creating a clean, linear history.

---

# Initial Situation

Both your local branch and the remote branch start at commit `A`.

```text
A
```

---

# Remote Changes

Another developer pushes commit `B` to [GitHub](https://github.com?utm_source=chatgpt.com).

```text
A → B
```

---

# Your Local Changes

You create a local commit `C`.

```text
A → C
```

---

# Before Running `git pull --rebase`

```text
Remote main (origin/main): A → B
Local main:                A → C
```

The histories have diverged.

---

# What Happens with `git pull`

```bash
git pull
```

Equivalent to fetch + merge. Git creates a merge commit.

```text
A → B → M
 \     /
  C ---
```

`M` is the merge commit.

---

# What Happens with `git pull --rebase`

```bash
git pull --rebase
```

Equivalent to:

```bash
git fetch origin
git rebase origin/main
```

Git performs these steps:

1. Temporarily removes your local commit `C`.
2. Updates your branch to `B`.
3. Replays your changes as a new commit (`C'`) on top of `B`.

Final history:

```text
A → B → C'
```

---

# Why the Commit Name Changes

`C'` represents the same code changes as `C`, but it is a newly created commit with a different commit ID.

---

# Practical Demonstration

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/kmit-project.git
cd kmit-project

# Make a local change
echo "Local change" >> app.js
git add .
git commit -m "My local update"

# Incorporate remote updates and replay your commit
git pull --rebase
```

---

# Example Output

```text
First, rewinding head to replay your work on top of it...
Applying: My local update
```

---

# If a Conflict Occurs

Resolve the conflicted files, then run:

```bash
git add .
git rebase --continue
```

To cancel the rebase:

```bash
git rebase --abort
```

---

# Comparison

## `git pull` (Merge)

```text
A → B → M
 \     /
  C ---
```

## `git pull --rebase`

```text
A → B → C'
```

---

# Benefits of `git pull --rebase`

* Produces a cleaner, linear history
* Avoids unnecessary merge commits
* Makes `git log` easier to read
* Common in collaborative workflows

---

# Configure Git to Use Rebase Automatically

```bash
git config --global pull.rebase true
```

After this setting, a plain `git pull` behaves like `git pull --rebase`.

---

# Summary

```bash
git pull --rebase
```

* Fetches the latest remote commits
* Replays your local commits on top of them
* Creates a linear commit history
* Rewrites your local commit IDs

This command is especially useful when you have local commits that are not yet pushed and you want to integrate upstream changes cleanly.


---

# Visual Example

Initial state:

```text
Remote main: A → B → C
Local main:  A → B
```

### After `git fetch`

```text
origin/main: A → B → C
Local main:  A → B
```

### After `git merge origin/main`

```text
Local main:  A → B → C
```

### After `git pull`

```text
Local main:  A → B → C
```

---

# Real-World Usage

In collaborative development, teammates push changes to GitHub. You run `git fetch` or `git pull` to obtain those updates before continuing your work or resolving conflicts.

---

# Summary

| Command                 | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `git fetch origin`      | Download updates without modifying your current branch  |
| `git merge origin/main` | Integrate downloaded updates into the current branch    |
| `git pull origin main`  | Fetch and merge in one step                             |
| `git pull --rebase`     | Fetch and rebase local commits on top of remote updates |

---

# Most Common Commands

```bash
git fetch origin
git merge origin/main
```

or simply:

```bash
git pull
```

Use `git fetch` when you want to review incoming changes first, and `git pull` when you want Git to update your branch immediately.
