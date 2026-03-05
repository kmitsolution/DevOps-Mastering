## Git Branch – What is it and Why Do We Need It?

### 1️⃣ What is a Git Branch?

A **Git branch** is a **separate line of development** in a repository. It allows multiple developers to work on different features or fixes **without affecting the main codebase**.

By default, every repository starts with a **main branch** (or **master** in older repositories). Developers create new branches to work independently and later merge the changes into the main branch.

👉 In simple terms:
A **branch is like a copy of the project where developers can safely make changes.**

---

### 2️⃣ Why Do We Need Branching?

Branching is important because it allows:

* Multiple developers to work on the **same project simultaneously**
* Safe development without breaking the **main production code**
* Better **code management and collaboration**
* Easy **feature development, bug fixing, and experimentation**

Without branches, every developer would push code directly to **main**, which could break the project.

---
<img width="500" height="536" alt="image" src="https://github.com/user-attachments/assets/5e661d49-9a5b-487b-8d24-63b3283c9bf8" />


### 3️⃣ Example Project: `kmitcourses`

Suppose we have a project called:

```
kmitcourses
```

Two developers join the project:

```
Dev1
Dev2
```

Instead of both developers working directly on the **main branch**, we create separate branches for them.

```
main
 ├── dev1-branch
 └── dev2-branch
```

Each developer works in their own branch.

---

### 4️⃣ Workflow of Branching

Step 1: Create project repository

```
kmitcourses
```

Step 2: Default branch created

```
main
```

Step 3: Create developer branches

```
git branch dev1
git branch dev2
```

Step 4: Developers switch to their branches

```
git checkout dev1
git checkout dev2
```

Step 5: Developers push their code to remote branches

```
git push origin dev1
git push origin dev2
```

Step 6: Code owner reviews and merges

```
dev1  → main
dev2  → main
```

---

### 5️⃣ Diagram of Git Branching Workflow

```
                    GitHub Repository
                        kmitcourses
                              │
                              │
                            main
                              │
               ┌──────────────┴──────────────┐
               │                             │
          dev1 branch                   dev2 branch
            (Dev1)                        (Dev2)
               │                             │
        Dev1 pushes code              Dev2 pushes code
               │                             │
               └──────────────┬──────────────┘
                              │
                        Code Owner
                       Reviews Code
                              │
                         Merge to
                            main
```

---

### 6️⃣ Real Development Flow

```
Developer → Create Branch
Developer → Write Code
Developer → Commit Changes
Developer → Push to GitHub
Code Owner → Review Code
Code Owner → Merge to Main
```

---

### 7️⃣ Common Git Branch Commands

Create branch

```
git branch dev1
```

Switch branch

```
git checkout dev1
```

Create and switch

```
git checkout -b dev1
```

Push branch to GitHub

```
git push origin dev1
```

Merge branch

```
git merge dev1
```

---

### 8️⃣ Benefits of Git Branching

✔ Parallel development
✔ Safe experimentation
✔ Easy collaboration
✔ Organized codebase
✔ Better version control

---

### 9️⃣ Simple Real-Life Analogy

Think of **branching like a Google Docs file**.

* Original document = **main branch**
* Each person makes a **copy to edit** = **branch**
* After editing, changes are **merged back to the original document**

---

✅ **Summary**

| Concept          | Meaning                            |
| ---------------- | ---------------------------------- |
| Branch           | Separate development line          |
| Main             | Stable production code             |
| Developer Branch | Used for feature development       |
| Merge            | Combining branch changes into main |

---
