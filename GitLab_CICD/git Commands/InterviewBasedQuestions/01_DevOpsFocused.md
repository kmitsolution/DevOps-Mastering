
# 🚀 Git Interview Mock Round (DevOps Focused)

---

## 🟢 Round 1 – Fundamentals (Warm-up)

### 1️⃣ What is the difference between Git and GitHub?

**Expected Answer:**

* Git is a distributed version control system.
* GitHub is a cloud-based hosting platform for Git repositories with collaboration features (PRs, Actions, Issues).

---

### 2️⃣ Explain the Git architecture.

**Expected Points:**

* Working directory
* Staging area (index)
* Local repository (.git folder)
* Remote repository

---

### 3️⃣ What happens internally when you run:

```
git add file.txt
git commit -m "message"
```

**Expected Answer:**

* `git add` → moves file to staging area
* `git commit` → creates snapshot in local repo with SHA hash

---

## 🟡 Round 2 – Branching & Merging

---

### 4️⃣ What is the difference between merge and rebase?

**Expected Answer:**

* Merge preserves history with merge commit.
* Rebase rewrites history for linear structure.
* Rebase should not be used on shared branches.

---

### 5️⃣ Scenario:

You rebased a shared branch and force pushed. Now teammates cannot push. What happened?

**Expected Answer:**

* History rewritten.
* Their local branch history no longer matches remote.
* They must rebase or reset.

---

### 6️⃣ How do you safely undo a merge commit already pushed to production?

**Expected Command:**

```
git revert -m 1 <merge_commit_hash>
```

Reason:

* Creates new commit.
* Does not rewrite history.

---

## 🔵 Round 3 – Reset, Revert & Recovery

---

### 7️⃣ Difference between:

```
git reset --soft
git reset --hard
git revert
```

**Expected Answer:**

| Command | Effect                              |
| ------- | ----------------------------------- |
| --soft  | Moves HEAD, keeps changes staged    |
| --hard  | Deletes commits and changes         |
| revert  | Creates new commit undoing previous |

---

### 8️⃣ Scenario:

You accidentally deleted local commits using `git reset --hard`. How can you recover?

**Expected Answer:**

```
git reflog
```

Then reset to commit hash.

---

## 🟣 Round 4 – CI/CD & DevOps Integration

---

### 9️⃣ Why should CI pipelines avoid using `git pull`?

**Expected Answer:**

* `git pull` can merge unexpectedly.
* Pipelines should use clean clone for deterministic builds.

---

### 🔟 Why do GitHub Actions sometimes use:

```yaml
fetch-depth: 0
```

**Expected Answer:**

* To fetch full commit history.
* Needed for tags, versioning, changelog generation.

---

### 1️⃣1️⃣ Scenario:

Pipeline shows:

```
detached HEAD
```

What does it mean?

**Expected Answer:**

* Checked out specific commit, not branch.
* Common in CI systems.

---

## 🟠 Round 5 – Conflict Handling

---

### 1️⃣2️⃣ How do you resolve merge conflicts?

**Expected Steps:**

1. Open conflicted file
2. Edit manually
3. Remove conflict markers
4. `git add`
5. `git commit`

---

### 1️⃣3️⃣ How do you abort a rebase in middle of conflict?

```
git rebase --abort
```

---

## 🔴 Round 6 – Advanced DevOps Scenarios

---

### 1️⃣4️⃣ Scenario:

Production branch is protected. How do you ensure no one pushes directly?

**Expected Answer:**

* Enable branch protection rules.
* Require PR approval.

---

### 1️⃣5️⃣ What is a shallow clone? Why use it in DevOps?

**Answer:**

* `git clone --depth 1`
* Faster CI builds
* Reduces network load

---

### 1️⃣6️⃣ What is Git squash and when do you use it?

**Answer:**

* Combines multiple commits into one.
* Used before merging PR for clean history.

---

### 1️⃣7️⃣ Scenario:

Your repo contains large binary files and is growing huge. What should you use?

**Expected Answer:**

* Git LFS (Large File Storage)

---

### 1️⃣8️⃣ How do you delete a remote branch?

```
git push origin --delete branch_name
```

---

### 1️⃣9️⃣ Difference between fork and clone?

**Answer:**

* Fork → copy on GitHub account
* Clone → local copy

---

### 2️⃣0️⃣ What is GitHub CLI used for?

**Answer:**

* Manage PRs, issues, workflows from terminal
  Example:

```
gh pr create
```

---

# 🎯 Rapid Fire (Interview Speed Round)

21. What is HEAD?
22. What is origin?
23. How do you list remotes?
24. How do you see last 5 commits?
25. What is fast-forward merge?

---

# ✅ Model Quick Answers

21. Pointer to current branch/commit
22. Default remote name
23. `git remote -v`
24. `git log -5`
25. Merge without creating new merge commit

---

# 🧠 Real Interview Trick Question

**Q:** When should you NEVER use rebase?

**Answer:**
On shared/public branches already pushed.

---


