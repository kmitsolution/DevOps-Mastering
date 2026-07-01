# Git Diff Command

`git diff` is used to compare changes in Git. It shows exactly what has changed line by line.

---

# What `git diff` Can Compare

1. Working Directory ↔ Staging Area
2. Staging Area ↔ Last Commit (`HEAD`)
3. Commit ↔ Commit
4. Branch ↔ Branch
5. File ↔ File at different revisions

---

# Sample Repository

```bash
echo "Hello" > app.txt
git add app.txt
git commit -m "Initial commit"

echo "World" >> app.txt
```

Current `app.txt`:

```text
Hello
World
```

Last committed version:

```text
Hello
```

---

# 1. `git diff`

Shows **unstaged changes** (Working Directory vs Staging Area).

```bash
git diff
```

Output:

```diff
 Hello
+World
```

Meaning:

* `+` = added line
* `-` = removed line

---

# 2. `git diff --staged`

(or `git diff --cached`)

Shows **staged changes** (Staging Area vs `HEAD`).

```bash
git add app.txt
git diff --staged
```

Output:

```diff
 Hello
+World
```

---

# 3. `git diff HEAD`

Shows **all changes** since the last commit (both staged and unstaged).

```bash
git diff HEAD
```

---

# 4. `git diff HEAD~1`

Compare the current working tree with the previous commit.

```bash
git diff HEAD~1
```

---

# 5. `git diff HEAD~1 HEAD`

Compare two commits.

```bash
git diff HEAD~1 HEAD
```

---

# 6. `git diff <commit1> <commit2>`

```bash
git diff a1b2c3d d4e5f6g
```

Shows differences between two specific commits.

---

# 7. `git diff branch1 branch2`

Compare two branches.

```bash
git diff main feature
```

---

# 8. `git diff --name-only`

Show only file names that changed.

```bash
git diff --name-only HEAD~1 HEAD
```

---

# 9. `git diff --stat`

Show a summary.

```bash
git diff --stat HEAD~1 HEAD
```

Example:

```text
app.txt | 2 ++
1 file changed, 2 insertions(+)
```

---

# 10. `git diff <file>`

Compare changes in one file.

```bash
git diff app.txt
```

---

# 11. `git diff --word-diff`

Highlight changes word by word.

```bash
git diff --word-diff
```

---

# 12. `git diff --color-words`

Display word-level changes with color.

```bash
git diff --color-words
```

---

# Real-World Examples

## Check Before Staging

```bash
git diff
```

## Review What Will Be Committed

```bash
git diff --staged
```

## Compare Two Releases

```bash
git diff v1.0 v2.0
```

## See Changed Files Only

```bash
git diff --name-only main feature
```

---

# Visual Representation

```text
Working Directory  ---- git diff ----> Staging Area
Staging Area       ---- git diff --staged ----> HEAD
HEAD               ---- git diff HEAD ----> Working Directory + Staging
```

---

# Common Commands Summary

```bash
git diff                    # Unstaged changes
git diff --staged           # Staged changes
git diff HEAD               # All changes since last commit
git diff HEAD~1 HEAD        # Compare two commits
git diff main feature       # Compare branches
git diff --name-only        # Show file names only
git diff --stat             # Show summary
```

---

# Memory Trick

* `git diff` → "What changed but not staged?"
* `git diff --staged` → "What will be committed?"
* `git diff HEAD` → "Everything changed since last commit"
