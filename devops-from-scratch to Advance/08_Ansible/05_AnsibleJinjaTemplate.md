
---

# 📘 Ansible Jinja2 Templates

---

# 🚀 1. What is Jinja2 in Ansible?

**Jinja2** is a templating engine used in Ansible to create **dynamic configuration files**.

👉 It allows you to:

* Insert variables
* Use conditions
* Use loops
* Generate configs dynamically

---

## 🧠 Why Jinja2?

Instead of writing static files like:

```ini
server_name myapp.com;
```

👉 You can write dynamic:

```jinja
server_name {{ domain_name }};
```

---

# ⚙️ 2. Jinja2 Syntax Basics

| Feature   | Syntax                   |
| --------- | ------------------------ |
| Variable  | `{{ variable }}`         |
| Condition | `{% if condition %}`     |
| Loop      | `{% for item in list %}` |
| Comment   | `{# comment #}`          |

---

# 📁 3. Template File Example (`.j2`)

👉 File: `nginx.conf.j2`

```jinja
server {
    listen 80;
    server_name {{ domain_name }};

    location / {
        proxy_pass http://{{ backend_server }};
    }
}
```

---

# 📦 4. Playbook Using Template Module

```yaml
---
- name: Deploy nginx config using template
  hosts: webservers
  vars:
    domain_name: myapp.com
    backend_server: 127.0.0.1:8080

  tasks:
    - name: Copy template
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
```

---

# 🔁 5. Example with Variables

## Template (`file.j2`)

```jinja
Hello {{ name }}
Welcome to {{ company }}
```

## Playbook

```yaml
vars:
  name: John
  company: Google
```

👉 Output:

```
Hello John
Welcome to Google
```

---

# 🔀 6. Conditional Statements in Jinja

## Template

```jinja
{% if ansible_facts['os_family'] == "Debian" %}
Package Manager: APT
{% else %}
Package Manager: YUM
{% endif %}
```

---

# 🔁 7. Loops in Jinja

## Template

```jinja
Users:
{% for user in users %}
- {{ user }}
{% endfor %}
```

## Playbook

```yaml
vars:
  users:
    - Alice
    - Bob
    - Charlie
```

👉 Output:

```
Users:
- Alice
- Bob
- Charlie
```

---

# 📄 8. Real-Time Example (Hosts File)

## Template (`hosts.j2`)

```jinja
127.0.0.1 localhost

{% for host in groups['webservers'] %}
{{ host }} webserver
{% endfor %}
```

---

# 📦 Playbook

```yaml
- name: Update hosts file
  hosts: all
  tasks:
    - name: Copy hosts template
      template:
        src: hosts.j2
        dest: /etc/hosts
```

---

# 🧪 9. Using Filters in Jinja

👉 Filters modify variables

## Example:

```jinja
{{ name | upper }}
{{ name | lower }}
{{ list | length }}
```

---

## Example Template

```jinja
User: {{ name | upper }}
Total Users: {{ users | length }}
```

---

# 🔐 10. Default Values

```jinja
{{ username | default('admin') }}
```

👉 If `username` is not defined → uses `admin`

---

# 🧠 11. Best Practices

* Keep templates in `templates/` folder
* Use meaningful variable names
* Avoid hardcoding values
* Use filters for formatting
* Use conditions for flexibility

---

# ⚠️ Common Mistakes

❌ Using `{{ }}` inside logic blocks
✔ Use `{% %}` for logic

❌ Hardcoding values
✔ Use variables

---

# 🧩 Folder Structure (Best Practice)

```
project/
│
├── playbook.yml
├── templates/
│   └── nginx.conf.j2
```

---

# 🚀 Real DevOps Use Cases

* Nginx / Apache configs
* Application config files
* Kubernetes YAML generation
* Environment-specific configs

---

# 🧠 Final Summary

| Concept         | Purpose            |
| --------------- | ------------------ |
| Jinja2          | Dynamic templating |
| {{ }}           | Variables          |
| {% %}           | Logic              |
| template module | Apply template     |
| filters         | Modify values      |

---

