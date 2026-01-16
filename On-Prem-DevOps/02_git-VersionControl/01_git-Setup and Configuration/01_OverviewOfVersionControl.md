## What is Version Control?

**Version Control** is a system that helps track **changes made to files over time**.
It allows multiple people to work on the same project, keeps a **history of changes**, and enables users to **recover previous versions** if something goes wrong.

### Why Version Control is Important

* Tracks who changed what and when
* Prevents loss of code
* Supports teamwork and collaboration
* Helps manage different versions of software

---

## Types of Version Control Systems

There are **three main types** of version control systems:

1. Local Version Control
2. Centralized Version Control
3. Distributed Version Control

---

## 1. Local Version Control System (LVCS)

### Explanation

In **Local Version Control**, versions of files are stored **only on a local machine**.
Changes are tracked using simple mechanisms like copying files into different folders or using local version databases.

![Image](https://git-scm.com/book/ms/v2/images/local.png)


### Example

* Manual file copies
* RCS (Revision Control System)

### How It Works

* Developer works alone
* Versions are saved locally
* No collaboration support

### Advantages

* Simple and easy to use
* No network required
* Fast performance

### Disadvantages

* No team collaboration
* High risk of data loss
* No central backup
* Not suitable for large projects

### Tools

* RCS
* Local file backups (manual)

---

## 2. Centralized Version Control System (CVCS)

### Explanation

In **Centralized Version Control**, there is **one central server** that stores all versions of files.
Developers connect to this server to **checkout**, **update**, and **commit** changes.

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AGgaGcwh5L246YcU5NVDA5A.png)


### Example

* SVN (Subversion)
* CVS
* Perforce

### How It Works

* Single central repository
* All users depend on the server
* Requires network connection

### Advantages

* Centralized control and management
* Easy to track changes
* Better suited for teams than local VCS

### Disadvantages

* Single point of failure (server down = no work)
* Requires constant network access
* Slower compared to local operations

### Tools

* SVN (Subversion)
* CVS
* Perforce

---

## 3. Distributed Version Control System (DVCS)

### Explanation

In **Distributed Version Control**, **every developer has a complete copy** of the repository, including full history.
Developers can work **offline** and synchronize changes later.

![Image](https://www.edureka.co/blog/wp-content/uploads/2016/11/Distributed-Version-Control-System-Workflow-What-Is-Git-Edureka.png)


### Example

* Git
* Mercurial
* Bazaar

### How It Works

* Each user has a local repository
* Changes are committed locally first
* Code is shared via push and pull

### Advantages

* No single point of failure
* Offline work possible
* Faster operations
* Strong support for collaboration and branching

### Disadvantages

* Initial learning curve
* Repository size can be large
* More complex than CVCS

### Tools

* Git
* Mercurial
* Bazaar

---

## Comparison Summary

| Feature             | Local VCS     | Centralized VCS | Distributed VCS |
| ------------------- | ------------- | --------------- | --------------- |
| Repository Location | Local machine | Central server  | Local + remote  |
| Collaboration       | No            | Yes             | Yes (best)      |
| Offline Work        | Yes           | No              | Yes             |
| Risk of Data Loss   | High          | Medium          | Very Low        |
| Scalability         | Low           | Medium          | High            |

---

### One-Line Summary

> **Version control systems manage file changes, with Local VCS for single users, Centralized VCS for server-based teams, and Distributed VCS for fully collaborative and resilient development.**

