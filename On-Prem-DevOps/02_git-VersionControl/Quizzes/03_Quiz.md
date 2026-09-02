# Git & GitHub – MCQ Test

### Topics: Branch, Stash, Revert & Reset

**1. Which command creates a new Git branch and switches to it immediately?**

A. `git branch feature`
B. `git checkout feature`
C. `git checkout -b feature`
D. `git switch branch feature`

---

**2. Which command is used to list all local branches in Git?**

A. `git branch`
B. `git list branch`
C. `git branches`
D. `git show branches`

---

**3. You are working on a feature but need to switch branches without committing your incomplete changes. What should you use?**

A. `git reset`
B. `git stash`
C. `git revert`
D. `git clone`

---

**4. Which command saves your current uncommitted changes temporarily?**

A. `git stash`
B. `git save`
C. `git commit --stash`
D. `git temporary`

---

**5. Which command restores the most recently stashed changes?**

A. `git stash restore`
B. `git stash apply`
C. `git stash get`
D. `git restore stash`

---

**6. What is the main purpose of `git revert`?**

A. Deletes a branch
B. Removes commits permanently from history
C. Creates a new commit that undoes the changes of an earlier commit
D. Deletes all uncommitted changes

---

**7. What happens when you run `git reset --soft HEAD~1`?**

A. The previous commit is removed and changes remain staged
B. The previous commit and all changes are deleted
C. The working directory is deleted
D. A new revert commit is created

---

**8. Which reset option removes the commit and unstages the changes but keeps the changes in the working directory?**

A. `git reset --soft`
B. `git reset --mixed`
C. `git reset --hard`
D. `git reset --delete`

---

**9. What is the major danger of `git reset --hard`?**

A. It creates duplicate commits
B. It automatically pushes changes to GitHub
C. It can permanently discard uncommitted changes
D. It creates a new branch

---

**10. Your team has already pushed commit `abc123` to GitHub. You want to undo its changes while keeping the shared history safe. Which command is generally preferred?**

A. `git reset --hard abc123`
B. `git revert abc123`
C. `git stash abc123`
D. `git branch -d abc123`

---

## Answer Key

| Question | Answer |
| -------- | ------ |
| 1        | **C**  |
| 2        | **A**  |
| 3        | **B**  |
| 4        | **A**  |
| 5        | **B**  |
| 6        | **C**  |
| 7        | **A**  |
| 8        | **B**  |
| 9        | **C**  |
| 10       | **B**  |
