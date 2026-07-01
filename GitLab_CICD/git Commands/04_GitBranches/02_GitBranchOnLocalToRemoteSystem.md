

# 🧪 Git Branching Hands-on Lab (Node.js Repo)

Repository:
🔗 [https://github.com/kmitcourses/first-demo-repo.git](https://github.com/kmitcourses/first-demo-repo.git)

Branches used:

* `dev1` → Developer 1 (RHEL)
* `dev2` → Developer 2 (Ubuntu)
* `main` → Code owner

---

# 🧩 LAB SETUP OVERVIEW

| Role       | Machine | Branch | Auth Method   |
| ---------- | ------- | ------ | ------------- |
| Dev1       | RHEL    | dev1   | HTTPS (Token) |
| Dev2       | Ubuntu  | dev2   | SSH           |
| Code Owner | Any     | main   | HTTPS/SSH     |

---

# 🔧 STEP 1: Create Personal Access Token (Dev1 - HTTPS)

On GitHub:

1. Go to **Settings → Developer Settings → Personal Access Tokens**
2. Generate token with:

   * `repo` access
3. Copy the token

👉 This will be used instead of password.

---

# 🧑‍💻 STEP 2: DEV1 (RHEL MACHINE)

## Clone only dev1 branch

```bash
git clone -b dev1 https://github.com/kmitcourses/first-demo-repo.git
cd first-demo-repo
```

## Create new file

```bash
touch dev1.js
```

Add sample Node.js code:

```javascript
console.log("Hello from Dev1");
```

## Stage & Commit

```bash
git add dev1.js
git commit -m "Added dev1.js file"
```

## Push to dev1 branch

```bash
git push origin dev1
```

👉 Enter:

* Username
* Personal Access Token (instead of password)

---

# 🧑‍💻 STEP 3: DEV2 (UBUNTU MACHINE - SSH)

## Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

## Copy SSH key

```bash
cat ~/.ssh/id_ed25519.pub
```

👉 Add this key in:
GitHub → Settings → SSH Keys

---

## Clone dev2 branch using SSH

```bash
git clone -b dev2 git@github.com:kmitcourses/first-demo-repo.git
cd first-demo-repo
```

## Create new file

```bash
touch dev2.js
```

Add code:

```javascript
console.log("Hello from Dev2");
```

## Stage & Commit

```bash
git add dev2.js
git commit -m "Added dev2.js file"
```

## Push to dev2 branch

```bash
git push origin dev2
```

---

# 👨‍💼 STEP 4: CODE OWNER REVIEW & MERGE

## Clone repo (main branch)

```bash
git clone https://github.com/kmitcourses/first-demo-repo.git
cd first-demo-repo
```

## Fetch all branches

```bash
git fetch --all
```

---

## Checkout main

```bash
git checkout main
```

---

## Merge dev1

```bash
git merge origin/dev1
```

## Merge dev2

```bash
git merge origin/dev2
```

👉 Resolve conflicts if any.

---

## Push updated main

```bash
git push origin main
```

---

# 🔄 STEP 5: SYNC BACK TO DEV BRANCHES

## Update dev1 branch

```bash
git checkout dev1
git merge main
git push origin dev1
```

---

## Update dev2 branch

```bash
git checkout dev2
git merge main
git push origin dev2
```

---

# 📂 FINAL STRUCTURE

After merge, repo contains:

```
first-demo-repo/
├── dev1.js
├── dev2.js
└── (existing files)
```

---

# ✅ LEARNING OUTCOMES

By completing this lab, you practiced:

* Branch-based development
* HTTPS vs SSH authentication
* Multi-developer workflow
* Code review & merge by owner
* Syncing branches after merge

---


