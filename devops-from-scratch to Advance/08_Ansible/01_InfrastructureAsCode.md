---

# 🏗️ What is Infrastructure?

**Infrastructure** in IT means all the resources needed to run applications:

* Servers (physical or cloud like AWS EC2)
* Networks (IP, DNS, load balancers)
* Storage (databases, disks)
* OS & middleware (Linux, Docker, etc.)

👉 Example:
If you run a website, your infrastructure includes:

* A server (EC2 VM)
* Nginx web server
* Database (MySQL)
* Network rules

---

# ⚙️ What is Infrastructure as Code (IaC)?

**Infrastructure as Code (IaC)** means:
👉 Managing infrastructure using **code instead of manual setup**

Instead of logging into servers and configuring things manually, you write code like:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

---

## 🔥 Why IaC is important

* ✅ Automation (no manual work)
* ✅ Consistency (same setup every time)
* ✅ Version control (Git)
* ✅ Faster deployments

---

## 🧰 Popular IaC Tools

| Tool      | Type                        |
| --------- | --------------------------- |
| Ansible   | Configuration Management    |
| Terraform | Infrastructure Provisioning |
| Chef      | Configuration Management    |
| Puppet    | Configuration Management    |

---

# 🚀 What is Ansible?

**Ansible** is an IaC tool used for:

* Configuration management
* Application deployment
* Automation

👉 Best part:

* No agent required
* Uses SSH
* Simple YAML syntax

---

# 🔁 Push vs Pull Model

This is VERY important for understanding Ansible.

---

## 📤 Push-Based Model

👉 Control node pushes configuration to servers

### 🔧 How it works:

* You run command from your machine
* It connects to servers via SSH
* Executes tasks

### ✅ Example: Ansible

```bash
ansible all -m ping
```

👉 Ansible sends instructions to all servers

---

### ✔️ Characteristics:

* No agent needed
* Central control
* Easy to start

---

## 📥 Pull-Based Model

👉 Servers pull configuration from a central server

### 🔧 How it works:

* Agent runs on each server
* It periodically checks for updates

### ✅ Examples:

* Puppet
* Chef

---

### ✔️ Characteristics:

* Requires agent installation
* Good for large-scale infra
* Continuous enforcement

---

# ⚖️ Push vs Pull Comparison

| Feature   | Push Model (Ansible) | Pull Model (Puppet/Chef) |
| --------- | -------------------- | ------------------------ |
| Control   | Central              | Distributed              |
| Agent     | Not required         | Required                 |
| Execution | On demand            | Periodic                 |
| Example   | Ansible              | Puppet                   |

---

# 🧠 Simple Analogy

* **Push model** → Teacher gives homework directly
* **Pull model** → Students check portal for homework

---

# 🧪 Real Example Scenario

### 🎯 Goal: Install Nginx on 5 servers

### Push (Ansible):

```bash
ansible webservers -m apt -a "name=nginx state=present"
```

👉 You trigger it manually

---

### Pull (Puppet):

* Each server runs agent every 30 mins
* Automatically installs nginx if missing

---

# 🚀 How to Start Learning Ansible

### Step 1: Basics

* YAML syntax
* Inventory file
* Ad-hoc commands

### Step 2: Core Concepts

* Playbooks
* Modules (apt, yum, service)
* Variables

### Step 3: Advanced

* Roles
* Templates (Jinja2)
* Handlers

---

# 📚 Minimal Starter Example

### Inventory file:

```
[web]
192.168.1.10
```

### Playbook:

```yaml
- hosts: web
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

Run:

```bash
ansible-playbook install.yml
```

---

