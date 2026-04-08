

# 🔁 What is a Pull Request (PR) in GitHub?

A **Pull Request (PR)** is a request to:
 Merge changes from one branch → into another branch (usually `main`)

### 📌 Simple Definition:

> “Hey team, I’ve made some changes in my branch. Please review and merge them.”

---

## 💡 Why Pull Requests are used?

* Code review before merging
* Team collaboration
* Track changes & discussions
* Prevent direct changes to `main`
* Handle merge conflicts safely

---

# 🧑‍💻 REAL EXAMPLE (YOUR SCENARIO)

* Dev1 → `dev1` branch
* Dev2 → `dev2` branch
* Both modify `index.js`
* Code owner reviews using PR

---

# 🚀 How to Create a Pull Request

## 🔹 Step 1: Push your branch

### Dev1:

```bash
git push origin dev1
```

### Dev2:

```bash
git push origin dev2
```

---

## 🔹 Step 2: Go to GitHub

1. Open your repo
2. Click **“Compare & pull request”** (shown after push)

OR

👉 Go to **Pull Requests tab → New Pull Request**

---

## 🔹 Step 3: Select branches

* **Base branch** → `main`
* **Compare branch** → `dev1` (or `dev2`)

---

## 🔹 Step 4: Add details

* Title: `Added feature from dev1`
* Description: Explain changes

---

## 🔹 Step 5: Create PR

👉 Click **Create Pull Request**

---

# 🔍 Code Review Process

Code owner can:

* Review code
* Add comments
* Request changes
* Approve PR

---

# 🔀 Merge Pull Request (NO CONFLICT CASE)

If no conflict:

👉 Click **Merge Pull Request**
👉 Click **Confirm Merge**

✔ Done

---

# ❌ Merge Conflict in Pull Request

When both dev1 & dev2 modify same lines:

👉 GitHub shows:

⚠️ **“This branch has conflicts that must be resolved”**

---

# 🛠️ How to Resolve Merge Conflict in PR

## 🔹 Option 1: Resolve on GitHub UI (Easy way)

1. Click **Resolve conflicts**
2. You will see:

```javascript
<<<<<<< HEAD
console.log("Code from Dev1");
=======
console.log("Code from Dev2");
>>>>>>> dev2
```

---

## 🔹 Edit the code

Example:

```javascript
console.log("Code from Dev1");
console.log("Code from Dev2");
```

---

## 🔹 Mark resolved

👉 Click **Mark as resolved**
👉 Click **Commit merge**

---

## 🔹 Merge PR

👉 Click **Merge Pull Request**

---

# 🛠️ Option 2: Resolve Conflict Locally (Advanced)

```bash
git checkout dev2
git pull origin main
```

👉 Fix conflict in `index.js`

```bash
git add index.js
git commit -m "Resolved conflict"
git push origin dev2
```

👉 PR will automatically update

---

# 🔄 After Merge (Important Step)

## Update branches

```bash
git checkout dev1
git pull origin main

git checkout dev2
git pull origin main
```

---

# 📊 COMPLETE FLOW

```
Dev1 → PR → main
Dev2 → PR → main (conflict occurs)
        ↓
Resolve conflict
        ↓
Merge successful
```

---

# ⚠️ KEY POINTS

* PR = Safe way to merge code
* Prevents direct changes to main
* Conflicts handled before merge
* Enables team collaboration

---

# 🧠 LEARNING OUTCOMES

✔ What is Pull Request
✔ How to create PR
✔ Code review workflow
✔ Conflict resolution in PR
✔ Industry best practices

---

