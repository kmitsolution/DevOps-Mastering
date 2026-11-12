# GitHub CLI (`gh`) 

```text
kmit-project
```

This first part focuses on:

1. What GitHub CLI is
2. Installation
3. Authentication
4. Creating your first GitHub repository
5. Cloning repositories
6. Opening repositories from terminal

Do not try to learn everything at once. GitHub CLI becomes easy when practiced step by step.

---

# Step 1 — What is GitHub CLI?

GitHub CLI is an official tool from [GitHub CLI](https://cli.github.com?utm_source=chatgpt.com).

Command name:

```bash
gh
```

It allows you to work with [GitHub](https://github.com?utm_source=chatgpt.com) directly from the terminal instead of opening the browser repeatedly.

---

# Without GitHub CLI

Normally you:

1. Open GitHub website
2. Click "New Repository"
3. Enter repository name
4. Create repository
5. Copy URL
6. Open terminal
7. Add remote
8. Push code

---

# With GitHub CLI

Everything can be done from terminal:

```bash
gh repo create
```

Much faster.

---

# Step 2 — Install GitHub CLI

Download from:

[GitHub CLI Downloads](https://cli.github.com?utm_source=chatgpt.com)

---

# Windows Installation

Using Winget:

```bash
winget install --id GitHub.cli
```

OR using Chocolatey:

```bash
choco install gh
```

---

# Verify Installation

```bash
gh --version
```

Example output:

```text
gh version 2.45.0
```

Now GitHub CLI is installed successfully.

---

# Step 3 — Authenticate with GitHub

This is the most important beginner step.

Run:

```bash
gh auth login
```

---

# What Happens Next?

GitHub CLI asks several questions.

---

## Question 1

```text
What account do you want to log into?
```

Choose:

```text
GitHub.com
```

---

## Question 2

```text
What is your preferred protocol?
```

Choose:

```text
HTTPS
```

(Beginner friendly)

---

## Question 3

```text
Authenticate Git with your GitHub credentials?
```

Choose:

```text
Yes
```

---

## Question 4

```text
How would you like to authenticate?
```

Choose:

```text
Login with a web browser
```

---

# Browser Authentication

GitHub CLI now opens your default browser.

If your default browser is [Microsoft Edge](https://www.microsoft.com/edge?utm_source=chatgpt.com), Edge opens automatically.

You will see:

```text
First copy your one-time code:
ABCD-1234
Press Enter to open github.com in your browser...
```

---

# Authorize GitHub CLI

1. Login to GitHub
2. Paste the one-time code
3. Click **Authorize GitHub CLI**

Done.

---

# Verify Authentication

```bash
gh auth status
```

Example:

```text
Logged in to github.com as RamanSharma
```

Now your terminal is connected to GitHub.

---

# Step 4 — Create Your First Local Project

Create a simple Node.js project.

```bash
mkdir kmit-project
cd kmit-project

git init
npm init -y
```

Create application:

```bash
echo 'console.log("Welcome to KMIT Project");' > app.js
```

Commit the code:

```bash
git add .
git commit -m "Initial commit"
```

---

# Step 5 — Create GitHub Repository Using GitHub CLI

Normally this requires browser work.

But now:

```bash
gh repo create
```

---

# GitHub CLI Asks Questions

---

## Repository Name

```text
What would you like to call your repository?
```

Enter:

```text
kmit-project
```

---

## Visibility

```text
Public or Private?
```

Choose:

```text
Public
```

---

## Push Existing Repository?

Choose:

```text
Push an existing local repository to GitHub
```

---

# What GitHub CLI Does Automatically

It:

1. Creates repository on GitHub
2. Adds remote origin
3. Pushes code

Equivalent manual commands would have been:

```bash
git remote add origin <url>
git push -u origin main
```

But `gh` handles everything automatically.

---

# Verify Repository

Open in browser:

```bash
gh repo view --web
```

This opens your repository directly in the browser.

---

# Step 6 — Clone Repository Using GitHub CLI

Suppose another repository exists.

Clone it:

```bash
gh repo clone kmitsolution/kmit-project
```

Equivalent to:

```bash
git clone https://github.com/kmitsolution/kmit-project.git
```

---

# Step 7 — View Repository Information

Inside a repository:

```bash
gh repo view
```

Shows:

* Repository name
* Description
* Visibility
* URL

---

# Step 8 — List Your Repositories

```bash
gh repo list
```

Example:

```text
RamanSharma/kmit-project
RamanSharma/devops-demo
```

---

# Beginner Commands Learned So Far

| Command              | Purpose                      |
| -------------------- | ---------------------------- |
| `gh auth login`      | Connect terminal with GitHub |
| `gh auth status`     | Verify login                 |
| `gh repo create`     | Create repository            |
| `gh repo clone`      | Clone repository             |
| `gh repo view`       | View repository              |
| `gh repo view --web` | Open repository in browser   |
| `gh repo list`       | List repositories            |

---

# Complete Beginner Example

```bash
# Install GitHub CLI

gh auth login

mkdir kmit-project
cd kmit-project

git init
npm init -y

echo 'console.log("Welcome");' > app.js

git add .
git commit -m "Initial commit"

gh repo create

gh repo view --web
```

---

# What You Learned in This Part

You now understand:

* What GitHub CLI is
* How to install it
* How authentication works
* How browser login works
* How to create repositories
* How to clone repositories
* How GitHub CLI simplifies GitHub operations

---

# More Practical GitHub CLI (`gh`) Examples for Beginners

Now that you understand the basics, let’s practice with real-world examples using your `kmit-project`.

We will focus on simple daily tasks developers perform using GitHub CLI.

---

# Example 1 — Create Repository in One Command

Suppose your local project already exists.

```bash id="8zj8ax"
cd kmit-project
```

Create GitHub repository and push immediately:

```bash id="4k4ypn"
gh repo create kmit-project --public --source=. --remote=origin --push
```

---

# What This Command Does

| Option            | Meaning                      |
| ----------------- | ---------------------------- |
| `--public`        | Public repository            |
| `--source=.`      | Current folder is the source |
| `--remote=origin` | Remote name becomes `origin` |
| `--push`          | Push code automatically      |

---

# Example 2 — Create a Private Repository

```bash id="2q4w2n"
gh repo create secret-project --private
```

Useful for:

* Company projects
* Personal code
* Confidential work

---

# Example 3 — Clone Repository

```bash id="17t5pm"
gh repo clone kmitsolution/kmit-project
```

Equivalent to:

```bash id="6ayf7u"
git clone https://github.com/kmitsolution/kmit-project.git
```

---

# Example 4 — Open Repository in Browser

Inside repository:

```bash id="f2r78l"
gh repo view --web
```

Very useful when you want to quickly open GitHub from terminal.

---

# Example 5 — Create Feature Branch and Push

```bash id="0l83l5"
git switch -c feature/login
```

Add code:

```bash id="mfv8ok"
echo 'console.log("Login Module");' >> app.js
```

Commit:

```bash id="ypg2yz"
git add .
git commit -m "Added login feature"
```

Push branch:

```bash id="8t3egv"
git push -u origin feature/login
```

---

# Example 6 — Create Pull Request Using GitHub CLI

Instead of opening browser:

```bash id="jlwmh5"
gh pr create
```

GitHub CLI asks:

```text id="gb3b4j"
Title?
Description?
Base branch?
```

Example:

```text id="rnzjlwm"
Title: Add login feature
Description: Added authentication module
Base branch: main
```

---

# What Happens?

GitHub automatically creates a Pull Request.

---

# Example 7 — List Pull Requests

```bash id="f8q3yz"
gh pr list
```

Example output:

```text id="g0ekv6"
#12 Add login feature
#13 Add payment integration
```

---

# Example 8 — View Pull Request

```bash id="2pqf1r"
gh pr view 12
```

Shows:

* Title
* Description
* Changed files
* Review status

---

# Example 9 — Checkout Pull Request Locally

Suppose another developer created PR #15.

```bash id="wnjlwm"
gh pr checkout 15
```

GitHub CLI:

1. Fetches PR branch
2. Creates local branch
3. Switches automatically

Very useful for testing others’ code.

---

# Example 10 — Merge Pull Request

```bash id="xch6n7"
gh pr merge 12
```

GitHub asks merge type:

* Merge commit
* Squash
* Rebase

---

# Example 11 — Create GitHub Issue

```bash id="xkl6kl"
gh issue create --title "Payment timeout bug" --body "UPI transactions failing after 30 seconds"
```

---

# Example 12 — List Issues

```bash id="drkpr5"
gh issue list
```

Example:

```text id="71j6n3"
#21 Payment timeout bug
#22 Dashboard UI issue
```

---

# Example 13 — Comment on an Issue

```bash id="0vljlm"
gh issue comment 21 --body "Working on the fix"
```

---

# Example 14 — Close an Issue

```bash id="f6dj7q"
gh issue close 21
```

---

# Example 15 — Create Release

Suppose version 1.0 is ready.

```bash id="f4l2qo"
gh release create v1.0 --title "Version 1.0" --notes "First stable release"
```

This creates a GitHub Release automatically.

---


# Most Important Beginner Commands

| Command              | Purpose           |
| -------------------- | ----------------- |
| `gh auth login`      | Login to GitHub   |
| `gh repo create`     | Create repository |
| `gh repo clone`      | Clone repository  |
| `gh repo view --web` | Open repository   |
| `gh pr create`       | Create PR         |
| `gh pr list`         | View PRs          |
| `gh issue create`    | Create issue      |
| `gh issue list`      | View issues       |
| `gh release create`  | Create release    |

---

