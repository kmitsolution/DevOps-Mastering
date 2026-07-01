# Git Squash Explained

Git does not have a standalone command named `git squash`, but **squashing** is a common Git operation that combines multiple commits into a single commit.

The two most common ways to squash commits are:

1. **Interactive Rebase (`git rebase -i`)** – the most common method
2. **`git merge --squash`** – squash an entire branch before merging

---

# 1. Squashing Commits with Interactive Rebase

## Example History

```text id="9v6q1z"
A --- B --- C --- D (HEAD)
```

Suppose you want to combine commits `B`, `C`, and `D` into one commit.

---

## Command

```bash id="ztfkmb"
git rebase -i HEAD~3
```

Git opens an editor:

```text id="kjm4f0"
pick abc123 Commit B
pick def456 Commit C
pick ghi789 Commit D
```

Change it to:

```text id="m70uwr"
pick abc123 Commit B
squash def456 Commit C
squash ghi789 Commit D
```

You can also use the short form:

```text id="qdfv5m"
pick abc123 Commit B
s def456 Commit C
s ghi789 Commit D
```

---

## Result

```text id="jlwmj6"
A --- B' (HEAD)
```

* `B'` is a new commit containing all changes from `B`, `C`, and `D`.
* Git lets you edit the final commit message.

---

# 2. Squashing a Branch with `git merge --squash`

## Example

```text id="0i2ycg"
main:    A --- B
               \
feature:        C --- D --- E
```

## Command

```bash id="lm98jw"
git checkout main
git merge --squash feature
git commit -m "Add complete feature"
```

## Result

```text id="c3llkv"
main: A --- B --- F
```

* All changes from `feature` are combined into a single commit `F`.
* The individual commits `C`, `D`, and `E` are not added to `main`.

---

# Practical Example

```bash id="jj8wl4"
git log --oneline
```

Output:

```text id="lmdg6q"
c3 Added line 3
b2 Added line 2
a1 Added line 1
```

Squash the last three commits:

```bash id="2ltmhn"
git rebase -i HEAD~3
```

Change:

```text id="cxt1jr"
pick a1 Added line 1
pick b2 Added line 2
pick c3 Added line 3
```

To:

```text id="6p09uy"
pick a1 Added line 1
squash b2 Added line 2
squash c3 Added line 3
```

Save and exit.

---

# Why Squash Commits?

* Clean up many small commits
* Remove "WIP" or "fix typo" commits
* Present one logical change
* Keep project history easier to read

---

# Common Workflow

```bash id="tmfdhz"
git add .
git commit -m "WIP"
git commit -m "Fix typo"
git commit -m "Final adjustment"

git rebase -i HEAD~3
# squash commits into one
```

---

# Direct Summary

> **Squashing combines multiple commits into one commit.**
> The most common approach is `git rebase -i HEAD~N` and changing subsequent commits from `pick` to `squash`. This creates a cleaner, more readable commit history.
