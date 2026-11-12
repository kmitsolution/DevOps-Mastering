---

# 🔰 1. Introduction to Ansible Modules

## ✅ What are Ansible Modules?

**Modules** are reusable units of code that Ansible executes on managed nodes.

👉 They perform specific tasks like:

* Installing packages
* Managing services
* Copying files
* Running commands

---

## 🔧 Commonly Used Modules

| Module  | Purpose                 |
| ------- | ----------------------- |
| ping    | Check connectivity      |
| shell   | Run Linux commands      |
| apt/yum | Package management      |
| service | Manage services         |
| copy    | Copy files              |
| file    | Manage file permissions |

---

# ⚡ 2. What are Ad-hoc Commands?

## ✅ Definition

**Ad-hoc commands** are one-line commands used to perform quick tasks without writing a playbook.

👉 Used when:

* Task is simple
* No loops/conditions required
* Quick testing or execution

---

## 📌 Syntax

```bash
ansible <host-pattern> -m <module> -a "<arguments>"
```

---

# 🧱 3. Prerequisites

* Ansible installed on **Master Node**
* Root access on all nodes (optional but used here)
* SSH connectivity between nodes
* SSH keys configured:

  * Generate on master: `ssh-keygen`
  * Copy to agents: `/root/.ssh/authorized_keys`

---

# 🖥️ 4. Environment Setup

## 📍 Servers

| Role               | IP            |
| ------------------ | ------------- |
| Master (DB Server) | 192.168.33.10 |
| Webserver1         | 192.168.33.11 |
| Webserver2         | 192.168.33.12 |

---

# 📁 5. Inventory (Hosts File Setup)

Edit file:

```bash
/etc/ansible/hosts
```

## 🔹 Configuration

```ini
[dbservers]
192.168.33.10

[webservers]
192.168.33.11
192.168.33.12
```

---

# ⚙️ 6. Ansible Configuration File

File location:

```bash
/etc/ansible/ansible.cfg
```

## 🔹 Update:

```ini
[defaults]
inventory = /etc/ansible/hosts
```

👉 This tells Ansible where your hosts file is located.

---

# 🧪 7. Ad-hoc Commands (Hands-on)

---

## 🔹 Challenge 1: Ping Module

### 🎯 Tasks:

* Check all webservers
* Check all servers
* Check specific server
* Use custom inventory file

### ✅ Commands:

```bash
ansible webservers -m ping
ansible all -m ping
ansible 192.168.33.11 -m ping
ansible -i /home/vagrant/mydir/hosts all -m ping
```

---

## 🔹 Challenge 2: Shell Module

👉 Used to run Linux commands

### 🎯 Tasks:

* Install Apache
* Uninstall Apache
* Create file
* Change permissions
* Copy file

### ✅ Commands:

```bash
ansible webservers -m shell -a "apt update && apt install apache2 -y"
ansible webservers -m shell -a "apt purge apache2 -y"

ansible webservers,dbservers -m shell -a "touch /tmp/1.txt"
ansible webservers,dbservers -m shell -a "chmod 644 /tmp/1.txt"
ansible webservers,dbservers -m shell -a "cp /tmp/1.txt /tmp/2.txt"
```

---

## 🔹 Challenge 3: apt Module

👉 Preferred over shell for package management

### 🎯 Tasks:

* Install Apache
* Remove Apache

### ✅ Commands:

```bash
ansible webservers -m apt -a "name=apache2 state=present"
ansible webservers -m apt -a "name=apache2 state=absent purge=yes"
```

---

## 🔹 Challenge 4: Service Module

👉 Manage services

### 🎯 Tasks:

* Start service
* Stop service
* Restart service

### ✅ Commands:

```bash
ansible webservers -m service -a "name=apache2 state=started"
ansible webservers -m service -a "name=apache2 state=stopped"
ansible webservers -m service -a "name=apache2 state=restarted"
```

---

## 🔹 Challenge 5: Copy Module

👉 Copy files or create content

### 🎯 Tasks:

* Copy file
* Rename while copying
* Create file with content

### ✅ Commands:

```bash
ansible webservers -m copy -a "src=/tmp/status.txt dest=/tmp/status.txt"
ansible webservers -m copy -a "src=/tmp/status.txt dest=/tmp/newstatus.txt"
ansible webservers -m copy -a "content='Hello World' dest=/tmp/4.txt"
```

---

# 🧠 Key Takeaways

* **Modules** = actual work units
* **Ad-hoc commands** = quick execution method
* Prefer modules (apt, service) over shell when possible
* Inventory file organizes your infrastructure
* SSH key setup is mandatory for smooth automation

---

