#  Step 1: Create a New Repository (Not Empty)

Your repo must contain at least:

```
README.md
```

Push it to GitHub.

Then:

👉 Click **Code → Codespaces → Create Codespace**

---

#  Step 2: Initialize Node.js Project Inside Codespace

Open terminal in Codespace and run:

```bash
npm init -y
```

This creates:

```
package.json
```

---

#  Step 3: Install Express (Simple Web Server)

```bash
npm install express
```

---

#  Step 4: Create `index.js`

Create a file:

```
index.js
```

Add this code:

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Hello from GitHub Codespaces 🚀');
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

⚠️ Important:
We used:

```javascript
'0.0.0.0'
```

This is REQUIRED in Codespaces to expose the server properly.

---

#  Step 5: Start the Server

Run:

```bash
node index.js
```

You will see:

```
Server running on port 3000
```

---

#  Step 6: Access Using Browser URL

When your app starts on port 3000:

Codespaces automatically detects the port.

You will see a popup:

```
Port 3000 is running → Open in Browser
```

Click it.

OR

Go to:

```
Ports Tab (Bottom Panel)
```

You will see:

```
3000 → Forwarded → Public/Private
```

Click the URL.

---

#  Your Codespace URL Will Look Like:

```
https://<your-codespace-name>-3000.app.github.dev
```

Example:

```
https://curly-space-abc123-3000.app.github.dev
```

Now your Node app is accessible in browser 🎉

---

# Optional: Make Port Public

In the **Ports tab**:

* Right click port 3000
* Select **Port Visibility → Public**

Now anyone with the URL can access it.

---

# Recommended devcontainer.json for This Setup

```json
{
  "name": "Node.js Dev Environment",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "forwardPorts": [3000],
  "postCreateCommand": "npm install"
}
```

---

# Common Mistakes (Very Important)

### ❌ Using localhost instead of 0.0.0.0

Wrong:

```javascript
app.listen(3000);
```

Correct:

```javascript
app.listen(3000, '0.0.0.0');
```

---

### ❌ Not forwarding port

Add:

```json
"forwardPorts": [3000]
```

---

### ❌ Not starting server

Codespace won’t show URL until server is running.

---

# Final Project Structure

```
my-node-project/
├── .devcontainer/
│   └── devcontainer.json
├── node_modules/
├── package.json
├── package-lock.json
└── index.js
```

---

# One-Line Summary

> In GitHub Codespaces, create a Node.js app, listen on `0.0.0.0`, start the server, and access it via the forwarded port URL provided automatically.

