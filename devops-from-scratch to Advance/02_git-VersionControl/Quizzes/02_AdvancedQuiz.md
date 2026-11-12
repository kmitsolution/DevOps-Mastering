# 🚀 Advanced DevOps Git Troubleshooting Quiz

---

## 1.

Your CI pipeline fails after `git pull` with:
`fatal: refusing to merge unrelated histories`

What is the likely cause?

1. Remote branch deleted
2. Local repo initialized separately
3. SSH key expired
4. Tag conflict

---

## 2.

You force pushed after a rebase and now teammates cannot push. Why?

1. Their Git version outdated
2. History was rewritten
3. Branch was deleted
4. Tags were modified

---

## 3.

Pipeline fails with:
`detached HEAD state`
What does this mean?

1. Repo corrupted
2. Not on any branch
3. Merge conflict
4. No remote configured

---

## 4.

After `git rebase main`, multiple conflicts appear across commits. Best practice?

1. Abort rebase and merge instead
2. Resolve conflicts commit-by-commit and continue
3. Reset branch
4. Delete branch

---

## 5.

You accidentally ran `git reset --hard` and lost local commits not pushed. What is best recovery option?

1. git clone again
2. git fetch
3. git reflog
4. git pull

---

## 6.

Your deployment pipeline clones repo but only latest commit appears (no history). Why?

1. Shallow clone used
2. Branch deleted
3. Repo corrupted
4. Detached HEAD

---

## 7.

After merging feature branch, production bug appears. You must undo merge safely without rewriting history. What do you do?

1. git reset --hard
2. git revert -m 1 <merge_commit>
3. git clean
4. git fetch

---

## 8.

You changed remote URL but push still goes to old repo. Why?

1. Branch protected
2. Multiple remotes exist
3. Git cache issue
4. Repo locked

---

## 9.

CI pipeline says:
`error: failed to push some refs`
Most common cause?

1. Local ahead of remote
2. Remote ahead of local
3. Detached HEAD
4. Missing tags

---

## 10.

After rebasing and pushing, Pull Request shows all commits again. Why?

1. Base branch changed
2. History rewritten
3. Fork corrupted
4. Branch renamed

---

## 11.

You want to see which commit deleted a specific file. Which command helps?

1. git status
2. git log -- file
3. git blame
4. git branch

---

## 12.

You want to ensure CI always builds clean environment without cached Git metadata. What is best practice?

1. Use git reset
2. Use fresh clone each time
3. Delete .git folder
4. Use git merge

---

## 13.

Two developers rebased same branch and both force pushed. History now inconsistent. What is safest recovery?

1. Delete repo
2. Reset to known good commit
3. Merge both histories
4. Pull again

---

## 14.

Your GitHub Actions workflow needs full commit history but checkout step only fetches 1 commit. Fix?

1. Add fetch-depth: 0
2. Use git fetch
3. Delete action
4. Use git reset

---

## 15.

You accidentally committed secrets and pushed. Repo is public. First immediate step?

1. Delete commit
2. Rotate credentials
3. Reset branch
4. Delete fork

---

## 16.

While merging, you see:
`fatal: Not possible to fast-forward, aborting.`
Meaning?

1. Branch ahead
2. Requires merge commit
3. Repo corrupted
4. Tag mismatch

---

## 17.

Production deployment uses tag `v1.2`. Someone retags different commit with same tag and pushes with force. Risk?

1. Nothing changes
2. Tag history mismatch
3. Branch deleted
4. Repo corrupted

---

## 18.

You want to squash commits before merging PR on GitHub. Which option?

1. Rebase locally
2. Use "Squash and Merge"
3. Reset branch
4. Delete branch

---

## 19.

Pipeline fails because submodules not cloned. Solution?

1. git pull
2. git clone --recursive
3. git reset
4. git merge

---

## 20.

You accidentally pushed to main branch directly but branch protection is enabled. What happens?

1. Push rejected
2. Repo deleted
3. Merge forced
4. Tag created

---

## 21.

CI job checks out specific commit hash and not branch. What state is repo in?

1. Merge state
2. Detached HEAD
3. Conflict state
4. Reset state

---

## 22.

You want to compare local branch with remote branch without merging. What should you use?

1. git status
2. git diff origin/main
3. git merge
4. git clone

---

## 23.

Pipeline shows inconsistent build because developers rebased shared branch frequently. Root cause?

1. Branch protection
2. Rewriting shared history
3. Merge strategy
4. Shallow clone

---

## 24.

You want to recover deleted branch that was pushed earlier. Best chance?

1. git branch new
2. git reflog
3. git status
4. git clone

---

## 25.

In large monorepo, clone is slow. What is best optimization?

1. git pull
2. Shallow clone with depth
3. git merge
4. Delete history

---

# ✅ Answers

1. 2
2. 2
3. 2
4. 2
5. 3
6. 1
7. 2
8. 2
9. 2
10. 2
11. 2
12. 2
13. 2
14. 1
15. 2
16. 2
17. 2
18. 2
19. 2
20. 1
21. 2
22. 2
23. 2
24. 2
25. 2

---

