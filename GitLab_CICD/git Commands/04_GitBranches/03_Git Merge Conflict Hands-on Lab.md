#  Git Merge Conflict Hands-on Lab (Correct Workflow)

## Objective

Understand how merge conflict occurs when:

* Code owner first pulls **dev1 & dev2 branches locally**
* Then merges them into `main`

---

# 🧩 SCENARIO

| Role       | Branch | Change           |
| ---------- | ------ | ---------------- |
| Dev1       | dev1   | Updates index.js |
| Dev2       | dev2   | Updates index.js |
| Code Owner | main   | Merges both      |

👉 Both modify the **same file → conflict expected**

---

# 🧑‍💻 STEP 1: DEV1 CHANGES

```bash
git checkout dev1
```

Edit `index.js`:

```javascript
console.log("Code from Dev1");
```

```bash
git add index.js
git commit -m "Dev1 updated index.js"
git push origin dev1
```

---

# 🧑‍💻 STEP 2: DEV2 CHANGES

```bash
git checkout dev2
```

Edit `index.js`:

```javascript
console.log("Code from Dev2");
```

```bash
git add index.js
git commit -m "Dev2 updated index.js"
git push origin dev2
```

---

# 👨‍💼 STEP 3: CODE OWNER SETUP

## Clone repository

```bash
git clone https://github.com/kmitcourses/first-demo-repo.git
cd first-demo-repo
```

---

# 🔄 STEP 4: PULL BOTH BRANCHES LOCALLY

## Pull dev1 branch

```bash
git checkout dev1
git pull origin dev1
```

---

## Pull dev2 branch

```bash
git checkout dev2
git pull origin dev2
```

👉 Now code owner has both branches updated locally

---

# 🔀 STEP 5: MERGE INTO MAIN

## Switch to main

```bash
git checkout main
git pull origin main
```

---

## Merge dev1 (✅ SUCCESS)

```bash
git merge dev1
```

✔ No conflict

---

## Merge dev2 (❌ CONFLICT)

```bash
git merge dev2
```

👉 Conflict occurs:

```bash
CONFLICT (content): Merge conflict in index.js
Automatic merge failed; fix conflicts and then commit the result.
```

---

# 🔍 STEP 6: CHECK CONFLICT

Open `index.js`:

```javascript
<<<<<<< HEAD
console.log("Code from Dev1");
=======
console.log("Code from Dev2");
>>>>>>> dev2
```

---

# 🛠️ STEP 7: RESOLVE CONFLICT

👉 Code owner decides final version

Example:

```javascript
console.log("Code from Dev1");
console.log("Code from Dev2");
```

---

## Mark resolved

```bash
git add index.js
git commit -m "Resolved merge conflict between dev1 and dev2"
```

---

## Push main branch

```bash
git push origin main
```

---

# 🔄 STEP 8: UPDATE DEV BRANCHES

## Update dev1

```bash
git checkout dev1
git merge main
git push origin dev1
```

---

## Update dev2

```bash
git checkout dev2
git merge main
git push origin dev2
```

---

# 📂 FINAL RESULT

```
main
 ├── index.js (merged content)

dev1 (synced)
dev2 (synced)
```

---

# ⚠️ KEY POINTS

* Code owner **pulls branches first**
* Merge happens **locally in main**
* Conflict occurs when merging second branch
* Manual resolution is required

---

# 🧠 LEARNING OUTCOMES

✔ Correct merge workflow
✔ Why conflict occurs only on second merge
✔ Local branch handling before merge
✔ Real industry practice

---

# 🚀 PRO TIP

👉 Always merge one branch at a time
👉 Resolve conflicts immediately
👉 Prefer Pull Requests for better control

---
