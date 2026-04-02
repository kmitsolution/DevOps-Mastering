Here is your **complete, well-structured Ansible Playbook Documentation** based on your notes . This is written in a clean, interview-ready + practical format.

---

# 📘 Ansible Playbook Documentation

---

# 🚀 1. What is an Ansible Playbook?

An **Ansible Playbook** is a YAML file that defines a set of tasks to be executed on managed nodes.

👉 It is used for:

* Automation
* Configuration management
* Application deployment

---

## 📌 Key Components of a Playbook

```yaml
---
- name: Play name
  hosts: target_group
  tasks:
    - name: Task name
      module: arguments
```

---

# 🧪 2. Basic Playbook Example

## ✅ Example 1: Print "Hello World"

```yaml
---
- name: play for running shell commands
  hosts: dbservers
  tasks:
    - name: Executing command module
      command: echo "Hello World"
```

### ▶️ Run:

```bash
ansible-playbook hello.yaml
```

---

# 🔄 3. Register & Debug (Capture Output)

## ✅ Example 2

```yaml
---
- name: play for running shell commands
  hosts: dbservers
  tasks:
    - name: Executing echo command
      command: echo "Hello World"
      register: output

    - name: Print the output
      debug:
        msg: "{{ output.stdout }}"
```

👉 `register` stores output
👉 `debug` prints it

---

# 📖 4. Read File Using Playbook

## ✅ Example 3

```yaml
---
- name: play for running shell commands
  hosts: dbservers
  tasks:
    - name: Read file
      command: cat /tmp/status.txt
      register: output

    - name: Print output
      debug:
        msg: "{{ output.stdout }}"
```

---

# 🔁 5. Multiple Plays in One Playbook

## ✅ Example 4

```yaml
---
- name: First play
  hosts: dbservers
  tasks:
    - name: create file
      command: touch /tmp/local.txt

- name: Second play
  hosts: webservers
  tasks:
    - name: create file
      command: touch /tmp/webserver.txt
```

---

# 🧩 6. Multiple Tasks in One Play

## ✅ Example 5

```yaml
---
- name: multiple tasks example
  hosts: dbservers
  tasks:
    - name: create file
      command: touch /tmp/myfile.txt

    - name: add content
      copy:
        content: "Hello World"
        dest: /tmp/myfile.txt

    - name: read file
      command: cat /tmp/myfile.txt
      register: output

    - name: print content
      debug:
        var: output.stdout
```

---

# 🔤 7. Variables in Playbooks

## ✅ Example 6

```yaml
---
- name: variable example
  hosts: dbservers
  vars:
    data: "Hello World!"

  tasks:
    - name: create file
      command: touch /tmp/myfile.txt

    - name: add content
      copy:
        content: "{{ data }}"
        dest: /tmp/myfile.txt
```

---

## 🔹 Runtime Variables (Prompt)

## ✅ Example 7

```yaml
---
- name: variable prompt example
  hosts: dbservers
  vars_prompt:
    - name: data
      prompt: Enter the value

  tasks:
    - name: add content
      copy:
        content: "{{ data }}"
        dest: /tmp/myfile.txt
```

---

# 🧪 8. Dry Run (Check Mode)

👉 Simulate execution without making changes

## ✅ Example 8

```bash
ansible-playbook dryrun.yaml --check
```

---

## 🔹 Task-level Check Mode

```yaml
check_mode: yes
```

---

# 📜 9. Logging in Ansible

👉 Enable in config:

```ini
log_path = /var/log/ansible.log
```

👉 Stores execution logs

---

# 🔒 10. Hide Sensitive Data

```yaml
no_log: true
```

👉 Prevents logging sensitive info

---

# ⚠️ 11. Error Handling

## ✅ Example 12

```yaml
- hosts: webservers
  tasks:
    - name: valid command
      command: date

    - name: invalid command
      command: date1
      ignore_errors: true

    - name: next task still runs
      command: ls
```

---

# 🧠 12. Facts & Magic Variables

👉 Collect system info

```bash
ansible dbservers -m setup
```

## ✅ Example

```yaml
- name: facts example
  hosts: webservers
  tasks:
    - name: print facts
      debug:
        msg: "{{ ansible_facts['os_family'] }}"
```

---

# 🔀 13. Conditional Statements

## ✅ Example

```yaml
when: ansible_facts['os_family'] == "RedHat"
```

---

## 🔹 Operators

* `==` equal
* `!=` not equal
* `and`
* `or`

---

# 🔧 14. User Module Example

```yaml
- name: create user
  user:
    name: John
  when: ansible_facts['os_family'] == "Debian"
```

---

# 🔁 15. Loops in Playbooks

## ✅ Example (with_items)

```yaml
- name: loop example
  debug:
    msg: "{{ item }}"
  with_items:
    - 1
    - 2
    - 3
```

---

## ✅ Example (loop)

```yaml
- name: create directories
  command: mkdir "{{ item }}"
  loop:
    - /tmp/1
    - /tmp/2
```

---

# 🏷️ 16. Tags in Playbooks

👉 Run specific tasks only

```yaml
- name: start apache
  service:
    name: apache2
    state: started
  tags: startservice
```

### ▶️ Run:

```bash
ansible-playbook tags.yaml --tags startservice
```

---

# 🧠 Final Summary

| Concept  | Purpose             |
| -------- | ------------------- |
| Playbook | Automation script   |
| Task     | Single action       |
| Module   | Work unit           |
| Register | Store output        |
| Debug    | Print output        |
| Vars     | Dynamic values      |
| When     | Conditional logic   |
| Loop     | Repeat tasks        |
| Tags     | Selective execution |

---

