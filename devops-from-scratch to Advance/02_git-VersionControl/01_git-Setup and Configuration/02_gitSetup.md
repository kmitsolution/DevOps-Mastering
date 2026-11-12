## What is Git?

**Git** is a **distributed version control system (DVCS)** used to track changes in source code during software development.
It allows multiple developers to work on the same project efficiently, supports branching and merging, and keeps a complete history of changes.

Git is designed to be:

* Fast
* Scalable
* Reliable
* Suitable for both small and large projects

---

## Who Developed Git?

Git was **developed by Linus Torvalds in 2005** to support the development of the Linux kernel.
The main goals were **speed, data integrity, and support for distributed workflows**.

---

## Official Git Website

👉 **[https://git-scm.com](https://git-scm.com)**

This is the **official and authoritative website** for:

* Git downloads
* Installation instructions
* Official documentation

All installation steps below are **referenced only from the git-scm website**.

---

## How to Install Git

---

## 1. Install Git on Windows

### Steps (as per git-scm.com)

1. Go to **[https://git-scm.com](https://git-scm.com)**
2. Click **Download for Windows**
3. Run the downloaded `.exe` installer
4. Follow the setup wizard:

   * Select default options (recommended)
   * Choose editor (default is fine)
   * Choose PATH environment option
5. Complete installation

### Verify Installation

```
git --version
```

---

## 2. Install Git on Linux

Git installation depends on the Linux distribution.

### Debian / Ubuntu

```
sudo apt update
sudo apt install git
```

### Fedora

```
sudo dnf install git
```

### RHEL / CentOS

```
sudo yum install git
```

### Verify Installation

```
git --version
```

> These commands are listed and referenced directly on **git-scm.com → Downloads → Linux**

---

## 3. Install Git on macOS

### Method 1: Using Installer (from git-scm)

1. Go to **[https://git-scm.com](https://git-scm.com)**
2. Click **Download for macOS**
3. Download the `.pkg` installer
4. Run the installer and follow the instructions

### Method 2: Using Xcode Command Line Tools

```
git --version
```

macOS will prompt to install Command Line Tools if Git is not installed.

### Verify Installation

```
git --version
```

---

## Summary Table

| OS      | Installation Method              |
| ------- | -------------------------------- |
| Windows | Git installer from git-scm.com   |
| Linux   | Package manager (apt, yum, dnf)  |
| macOS   | Git installer or Xcode CLI tools |

---

### One-Line Summary

> **Git is a distributed version control system developed by Linus Torvalds, officially available at git-scm.com, and installable on Windows, Linux, and macOS using platform-specific methods.**

