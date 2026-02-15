
# 🧠 Scenario-Based Git & GitHub Quiz

---

## 1.

You modified a file but accidentally ran `git add .`. You want to unstage the file but keep changes. What should you do?

1. git reset --hard
2. git reset HEAD <file>
3. git revert <file>
4. git clean -fd

---

## 2.

You committed sensitive data (API key) and pushed to remote. You want to remove that commit from history completely. Which approach is correct?

1. git revert
2. git reset --soft
3. git rebase -i and force push
4. git pull

---

## 3.

Two developers modified the same line in a file. When merging, Git stops and shows conflict markers. What is your next step?

1. Run git push
2. Delete the file
3. Manually resolve conflict, then add and commit
4. Run git reset

---

## 4.

You want to see changes from remote repository but do NOT want to merge them yet. What should you use?

1. git pull
2. git fetch
3. git clone
4. git merge

---

## 5.

You are working on feature branch. Meanwhile main branch has new commits. You want clean linear history before merging. What should you use?

1. git merge main
2. git rebase main
3. git reset
4. git revert

---

## 6.

You accidentally committed 5 small commits. Before pushing, you want to combine them into one meaningful commit. What should you use?

1. git merge
2. git squash
3. git rebase -i
4. git reset --hard

---

## 7.

Your local branch is behind remote branch. Someone else pushed new commits. What is safest way to update your branch?

1. git push
2. git pull
3. git reset
4. git clean

---

## 8.

You cloned a repository and made changes. When pushing, you get error: "rejected – non-fast-forward". What is likely required?

1. git merge
2. git pull first
3. git clone again
4. git init

---

## 9.

You want to undo last commit but keep the changes staged. Which command?

1. git reset --soft HEAD~1
2. git reset --hard HEAD~1
3. git revert HEAD
4. git clean

---

## 10.

You want to undo last commit and delete changes permanently. What should you use?

1. git reset --soft
2. git revert
3. git reset --hard HEAD~1
4. git fetch

---

## 11.

Your team wants to mark version 1.0 of application for production release. What should you create?

1. Branch
2. Tag
3. Fork
4. Pull Request

---

## 12.

You want to contribute to someone else's repository but you don’t have write access. What is first step?

1. Clone repository
2. Fork repository
3. Create branch
4. Push directly

---

## 13.

You created a fork and pushed changes. Now you want maintainers to review your changes. What should you create?

1. Merge request locally
2. Pull Request
3. Tag
4. Rebase

---

## 14.

You accidentally merged wrong branch into main. The merge is already pushed. What is safest way to undo?

1. git reset --hard and force push
2. git revert merge_commit_hash
3. Delete repository
4. git clean

---

## 15.

You want to check which remote URL your repo is connected to. What command?

1. git branch
2. git remote -v
3. git status
4. git config

---

## 16.

You want to rename local branch from "dev" to "development". What should you do?

1. git branch -m development
2. git rename dev
3. git checkout development
4. git branch dev

---

## 17.

You want to delete a remote branch named feature1. Which command?

1. git branch -d feature1
2. git push origin --delete feature1
3. git delete feature1
4. git remove branch feature1

---

## 18.

You want to temporarily save uncommitted changes to switch branches. What should you use?

1. git commit
2. git stash
3. git reset
4. git merge

---

## 19.

You want to reapply stashed changes. What command?

1. git apply
2. git pop
3. git stash pop
4. git merge stash

---

## 20.

In Codespaces, you want your app running on port 5000 accessible in browser. What must you ensure?

1. App runs on localhost only
2. App runs on 0.0.0.0
3. Use git fetch
4. Create new branch

---

## 21.

You want to clone only a specific branch instead of full repo history. Which option helps?

1. git clone --branch <name>
2. git fetch --branch
3. git pull branch
4. git init branch

---

## 22.

You want to create a Pull Request using GitHub CLI. Which is correct?

1. git pr create
2. gh pr create
3. github pr create
4. gh merge

---

## 23.

Your team wants to avoid merge commits in main branch and maintain clean history. What strategy should they use?

1. Force push
2. Rebase before merge
3. Delete commits
4. Reset branch

---

## 24.

You want to see difference between two branches before merging. What should you use?

1. git status
2. git diff branch1 branch2
3. git merge
4. git log

---

## 25.

You pushed wrong commit to remote main branch and want to undo it safely without rewriting history. What should you use?

1. git reset --hard and force push
2. git revert commit_hash
3. Delete branch
4. git clean

---

# ✅ Answers

1. 2
2. 3
3. 3
4. 2
5. 2
6. 3
7. 2
8. 2
9. 1
10. 3
11. 2
12. 2
13. 2
14. 2
15. 2
16. 1
17. 2
18. 2
19. 3
20. 2
21. 1
22. 2
23. 2
24. 2
25. 2

---

