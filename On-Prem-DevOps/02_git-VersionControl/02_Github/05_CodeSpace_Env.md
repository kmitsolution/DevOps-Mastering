
# ✅ 1️⃣ Important: Restart Required

After creating a new Codespaces secret:

👉 You must **restart or rebuild the Codespace**

Otherwise the variable won’t appear.

---

# ✅ 2️⃣ Access Secret in Terminal

Inside Codespace terminal:

```bash
echo $PASSWORD
```

If configured correctly, it will print:

```
your_secret_value
```

If it prints nothing:

* Restart Codespace
* Or run: **Codespaces → Rebuild Container**

---

# ✅ 3️⃣ Access Secret in Node.js Application

In your `index.js`:

```javascript
const password = process.env.PASSWORD;

console.log("Secret value is:", password);
```

Or safer way:

```javascript
const password = process.env.PASSWORD || "Not Set";

console.log("Password:", password);
```

Run:

```bash
node index.js
```

---

# ✅ 4️⃣ Example: Use Secret in Express App

```javascript
const express = require('express');
const app = express();

const PASSWORD = process.env.PASSWORD;

app.get('/', (req, res) => {
  res.send(`Secret is: ${PASSWORD}`);
});

app.listen(3000, '0.0.0.0', () => {
  console.log('Server running');
});
```

---

# 🔐 How Codespaces Secrets Work

When you create a secret under:

```
Settings → Secrets and variables → Codespaces
```

GitHub injects it as an **environment variable inside the container**.

So:

```
Secret Name = PASSWORD
Environment Variable = PASSWORD
```

---

# ❗ Common Problems

### Problem 1: Secret Not Showing

Fix:

* Stop Codespace
* Start again
* Or Rebuild Container

---

### Problem 2: Wrong Secret Scope

Make sure you created it under:

* Repository → Codespaces secrets
  OR
* Account-level Codespaces secrets

---

# 🧪 How to Verify All Environment Variables

Run:

```bash
printenv | grep PASSWORD
```

If it shows:

```
PASSWORD=your_value
```

It is working.

---

# 🚨 Security Warning

Never log secrets in real production apps like:

```javascript
console.log(process.env.PASSWORD);
```

This is only for learning.

---

# 🧠 Real DevOps Use Case

Instead of storing:

```
DB_PASSWORD=admin123
```

inside `.env` file,

You store it in Codespaces secret.

Developers can:

* Use the app
* Access DB
  But never see the actual password in repo.

---

# 🎯 Final Answer

> Codespaces secrets are automatically injected as environment variables inside the container and can be accessed using `process.env.SECRET_NAME` in your application.

