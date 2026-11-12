# 1. Git Revert

`git revert` **does not delete any commit**. Instead, it creates a **new commit** that applies the opposite changes of a previous commit.

## Example History

```text id="hnsw3y"
A --- B --- C --- D (HEAD)
```

Suppose commit `C` introduced a bug.

## Command

```bash id="p6hupz"
git revert <commit-C-hash>
```

## Result

```text id="48h7r4"
A --- B --- C --- D --- E (HEAD)
                      ^
                      E = Revert of C
```

* Commits `A`, `B`, `C`, and `D` remain in history.
* New commit `E` reverses the changes made by `C`.

---

# Practical Example

## Create Commits

```bash id="h4fqlt"
echo "Line 1" > app.txt
git add .
git commit -m "Initial commit"

echo "Buggy code" >> app.txt
git commit -am "Added buggy code"
```

## Revert the Buggy Commit

```bash id="cx5gbn"
git log --oneline
git revert HEAD
```

Git creates a new commit such as:

```text id="d4v9zx"
Revert "Added buggy code"
```

The file `app.txt` no longer contains `Buggy code`.

---

# 2. Git Reset

`git reset` **moves the branch pointer** to an earlier commit.

## Example

```text id="2jlfvs"
A --- B --- C --- D (HEAD)
```

```bash id="s6mxgx"
git reset --hard B
```

## Result

```text id="6m4l2q"
A --- B (HEAD)
```

Commits `C` and `D` are no longer part of the current branch.

---

# Key Difference

| Feature                      | `git revert` | `git reset`                       |
| ---------------------------- | ------------ | --------------------------------- |
| Removes commits from history | No           | Yes (from current branch history) |
| Creates a new commit         | Yes          | No                                |
| Safe after pushing           | Yes          | Usually No (unless coordinated)   |
| Preserves history            | Yes          | No                                |
| Best for shared branches     | Yes          | No                                |

---

# Real-World Scenario

## Shared Branch (`main`)

You pushed commit `C`, and it caused a production issue.

### Safe Approach

```bash id="e7h5pq"
git revert <commit-C-hash>
git push
```

Everyone keeps the same history, plus a new revert commit.

---

## Local Branch Only

You made three commits and want to discard the last two.

```bash id="gvr4qb"
git reset --hard HEAD~2
```

The last two commits are removed from the current branch.

---

# Reverting a Specific Commit in the Middle

History:

```text id="g85zfu"
A --- B --- C --- D --- E (HEAD)
```

To undo only commit `C`:

```bash id="9rggto"
git revert <hash-of-C>
```

Result:

```text id="ryy2zr"
A --- B --- C --- D --- E --- F
                          ^
                          F = Revert of C
```

---

# Useful Revert Options

## Revert Without Auto Commit

```bash id="3xfr7v"
git revert --no-commit <commit-hash>
```

Applies the reverse changes to your working directory and staging area, allowing you to review or combine them before committing.

## Revert Multiple Commits

```bash id="0kqqf5"
git revert HEAD~2..HEAD
```

Creates revert commits for each commit in the specified range.

---

# When to Use Which

### Use `git revert` When:

* The commit has already been pushed.
* You want a complete audit trail.
* You are working on a shared branch.

### Use `git reset` When:

* The commits are local only.
* You want to rewrite history.
* You want to remove commits entirely from the branch.

---

# Direct Summary

* **`git revert`** = "Undo the changes by adding a new commit."
* **`git reset`** = "Move the branch pointer backward, optionally discarding commits and changes."

---

# Memory Trick

* **Revert** → Reverse
* **Reset** → Rewind

---

# Most Common Commands

```bash id="a2hffp"
git revert HEAD
git revert <commit-hash>

git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```
