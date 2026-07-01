# Git `cherry-pick` Command 

The `git cherry-pick` command allows you to **copy one or more specific commits from one branch and apply them to another branch**.

Instead of merging an entire branch, you select only the commits you want.

---

# Real-Life Example

Suppose you have two branches:

* `main`
* `dev`

On `dev`, three commits were created:

```text
A → B → C → D
```

Where:

* `B` = Added login feature
* `C` = Fixed typo
* `D` = Added payment integration

You want to copy only commit `D` to `main`.

---

# Step 1: Find the Commit Hash

On the source branch (`dev`):

```bash
git log --oneline
```

Example output:

```text
a1b2c3d Added payment integration
e4f5g6h Fixed typo
i7j8k9l Added login feature
```

---

# Step 2: Switch to the Target Branch

```bash
git switch main
```

---

# Step 3: Cherry-Pick the Commit

```bash
git cherry-pick a1b2c3d
```

Git applies the changes from commit `a1b2c3d` to `main` and creates a new commit.

---

# Result

Before:

```text
main: A
dev:  A → B → C → D
```

After:

```text
main: A → D'
dev:  A → B → C → D
```

`D'` contains the same code changes as `D`, but it is a new commit with a different hash.

---

# Cherry-Pick Multiple Commits

## Specific Commits

```bash
git cherry-pick a1b2c3d e4f5g6h
```

## Range of Commits

```bash
git cherry-pick B^..D
```

This includes commits `B`, `C`, and `D`.

---

# Real-World Use Cases

* Apply a hotfix from `dev` to `main`
* Copy a bug fix to a release branch
* Selectively move features between branches
* Avoid merging unrelated work

---

# Conflict Handling

If conflicts occur:

1. Resolve the conflicted files.
2. Stage the resolved files.

```bash
git add .
```

3. Continue:

```bash
git cherry-pick --continue
```

To cancel:

```bash
git cherry-pick --abort
```

---

# Common Options

| Command                           | Purpose                                 |
| --------------------------------- | --------------------------------------- |
| `git cherry-pick <hash>`          | Apply one commit                        |
| `git cherry-pick <hash1> <hash2>` | Apply multiple commits                  |
| `git cherry-pick --continue`      | Continue after resolving conflicts      |
| `git cherry-pick --abort`         | Cancel the operation                    |
| `git cherry-pick -n <hash>`       | Apply changes without creating a commit |

---

# Practical Example

```bash
git switch dev
git log --oneline

git switch main
git cherry-pick a1b2c3d
git push origin main
```

---

# Difference Between Merge and Cherry-Pick

| Merge                                             | Cherry-Pick                         |
| ------------------------------------------------- | ----------------------------------- |
| Combines all relevant commits from another branch | Copies only selected commits        |
| Preserves full branch history                     | Creates new commits with new hashes |
| Best for integrating complete branches            | Best for targeted fixes             |

---

# Typical Hotfix Workflow

1. Fix a bug on `dev`.
2. Identify the fix commit.
3. Switch to `main`.
4. Run `git cherry-pick <commit-hash>`.
5. Push `main`.

---

# Summary

```bash
git cherry-pick <commit-hash>
```

This command copies the changes introduced by a specific commit and applies them to your current branch, creating a new commit. It is ideal when you want only selected changes rather than merging an entire branch.
