Let’s build this **from scratch → practical DevOps level understanding** so you can actually start using Ansible confidently.

---

# 🚀 What is Ansible?

**Ansible** is an automation tool used for:

* Configuration management (install packages, setup servers)
* Application deployment
* Orchestration (multi-server workflows)

👉 It uses **YAML playbooks** and runs tasks over **SSH**

---

# 🤖 Why is Ansible called *Agentless*?

👉 Because **no software (agent) is required on target machines**

### 🔧 How it works:

* Ansible connects using **SSH**
* Executes commands remotely
* No daemon/agent needed on remote servers

---

## ✅ Comparison

| Tool    | Agent Required?  |
| ------- | ---------------- |
| Ansible | ❌ No (Agentless) |
| Puppet  | ✅ Yes            |
| Chef    | ✅ Yes            |

👉 This makes Ansible:

* Easy to set up
* Lightweight
* Secure (uses SSH)

---

# 🖥️ Install Latest Ansible on Ubuntu

### 🔹 Step 1: Update system

```bash
sudo apt update
```

---

### 🔹 Step 2: Install dependencies

```bash
sudo apt install -y software-properties-common
```

---

### 🔹 Step 3: Add official Ansible PPA

```bash
sudo add-apt-repository --yes --update ppa:ansible/ansible
```

---

### 🔹 Step 4: Install Ansible

```bash
sudo apt install -y ansible
```

---

### 🔹 Step 5: Verify installation

```bash
ansible --version
```

---

# 🔐 Setup SSH Key Authentication (VERY IMPORTANT)

Ansible works best with **passwordless SSH**

---

## 🔹 Step 1: Generate SSH key (on control node)

```bash
ssh-keygen -t rsa -b 4096
```

Press Enter → no passphrase (for learning)

---

## 🔹 Step 2: Copy key to remote machine

```bash
ssh-copy-id user@remote_ip
```

👉 Example:

```bash
ssh-copy-id ubuntu@192.168.1.10
```

---

## 🔹 Step 3: Test connection

```bash
ssh ubuntu@192.168.1.10
```

👉 Should login without password ✅

---

# 📁 Inventory (Hosts) File

👉 This tells Ansible **which servers to manage**

---

## 🔹 Basic Inventory Example

```ini
[web]
192.168.1.10
192.168.1.11

[db]
192.168.1.20
```

---

## 🔹 With username

```ini
[web]
192.168.1.10 ansible_user=ubuntu
192.168.1.11 ansible_user=ubuntu
```

---

## 🔹 With SSH key explicitly

```ini
[web]
192.168.1.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

# 🧪 Test Inventory

```bash
ansible all -i inventory -m ping
```

👉 Output:

```
192.168.1.10 | SUCCESS => {"ping": "pong"}
```

---

# ⚙️ What is `ansible.cfg`?

👉 This is Ansible’s **main configuration file**

It controls:

* Inventory location
* SSH settings
* Default user
* Key file
* Timeout

---

## 🔹 Example `ansible.cfg`

```ini
[defaults]
inventory = ./inventory
remote_user = ubuntu
host_key_checking = False
private_key_file = ~/.ssh/id_rsa
```

---

## 🔹 What each line means:

| Setting           | Meaning            |
| ----------------- | ------------------ |
| inventory         | Default hosts file |
| remote_user       | Default SSH user   |
| host_key_checking | Skip SSH prompt    |
| private_key_file  | SSH key path       |

---

## 🔹 Where Ansible looks for config

Priority order:

1. `ANSIBLE_CONFIG` env variable
2. `./ansible.cfg` (current directory) ✅
3. `~/.ansible.cfg`
4. `/etc/ansible/ansible.cfg`

---

# 🧪 First Real Command

```bash
ansible web -i inventory -m ping
```

👉 Meaning:

* `web` → group
* `-i` → inventory file
* `-m ping` → test module

---

# 🧠 Full Flow Summary

1. Install Ansible
2. Setup SSH keys
3. Create inventory
4. Configure ansible.cfg
5. Run commands/playbooks

---

# ⚡ Pro Tips (Important)

* Always use SSH keys (not passwords)
* Keep inventory organized (groups)
* Use `ansible.cfg` to avoid repeating flags
* Start with ad-hoc commands → then playbooks

---

