# Git Merge vs Git Rebase Explained

Both `git merge` and `git rebase` are used to integrate changes from one branch into another, but they work differently.

---

# Example Setup

```text id="g7xw3n"
main:    A --- B --- C
                  \
feature:           D --- E
```

* `main` has commits `A`, `B`, `C`
* `feature` branched from `C`
* `feature` has commits `D`, `E`

Now suppose `main` receives more commits:

```text id="e2dz6k"
main:    A --- B --- C --- F --- G
                  \
feature:           D --- E
```

---

# 1. Git Merge

## Command

```bash id="q53llp"
git checkout feature
git merge main
```

## What Happens?

Git combines the histories and creates a **merge commit** if necessary.

## Result

```text id="wh2mgd"
main:    A --- B --- C --- F --- G
                  \             \
feature:           D --- E ------ M
```

* `M` is a new merge commit.
* History preserves both branches exactly as they happened.

---

# 2. Git Rebase

## Command

```bash id="0rwjlwm"
git checkout feature
git rebase main
```

## What Happens?

Git:

1. Temporarily removes commits `D` and `E`.
2. Moves `feature` to `G`.
3. Replays `D` and `E` on top of `G`.

## Result

```text id="r2o2m7"
main:    A --- B --- C --- F --- G
                                \
feature:                         D' --- E'
```

* `D'` and `E'` are new commits with new hashes.
* No merge commit is created.

---

# Why Rebase Does Not Create a Merge Commit

Rebase rewrites history to make it look as though your work started from the latest commit.

Instead of combining two branch tips, it simply replays your commits one by one.

Since the result is a straight line, no merge commit is needed.

---

# Important Clarification

Although rebase does not create a merge commit, it **does create new commit objects** (`D'`, `E'`) because the parent commit changes.

The old commits `D` and `E` become unreachable and are eventually cleaned up by Git.

---

# Visual Comparison

## Merge

```text id="f6x2k5"
A --- B --- C --- F --- G
          \             \
           D --- E ------ M
```

## Rebase

```text id="r0zjv8"
A --- B --- C --- F --- G --- D' --- E'
```

---

# When Merge Creates No New Commit

If the target branch has not diverged, Git can perform a **fast-forward merge**:

```text id="a1s0cc"
main:    A --- B
                  \
feature:           C --- D
```

```bash id="g11o8k"
git checkout main
git merge feature
```

Result:

```text id="wrnyup"
main: A --- B --- C --- D
```

No merge commit is created because Git only moves the branch pointer forward.

---

# Real-World Usage

## Use Merge When

* You want to preserve branch history.
* You want to see where branches were integrated.
* Working on shared branches.

## Use Rebase When

* You want a clean, linear history.
* You are cleaning up local feature branches.
* You have not yet shared the branch widely.

---

# Direct Summary

* **Merge** combines two branch histories and may create a merge commit.
* **Rebase** rewrites your commits onto a new base and creates new versions of those commits.
* **Rebase does not create a merge commit because the final history is linear rather than branched.**
