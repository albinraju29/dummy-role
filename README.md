---

# Ansible Roles: Web Server & HTML App Deployment

This repository contains two powerful and reusable **Ansible roles**:

1. **Webserver Role** → Installs and manages **Nginx, Apache, PHP** dynamically depending on your configuration and target OS.
2. **Deploy HTML App Role** → Deploys a static HTML application to multiple servers, ensuring that the appropriate web service is running and serving your content.

Both roles can be used **individually** or **together** to create a full web stack automation pipeline.

---

## 🚀 Features

* Supports **Debian/Ubuntu** and **RHEL/CentOS/AlmaLinux/Rocky**
* Installs **nginx**, **apache2/httpd**, and **php** dynamically
* Starts, stops, enables, and restarts services via handlers
* Deploys **static HTML apps** to `/var/www/html` (default)
* Compatible with **AWS EC2**, **local VMs**, **bare metal servers**
* Works with **SSH keys** or **password-based authentication**
* Scalable across **dozens or hundreds of servers**

---

## 📂 Role Structure

### Webserver Role

```
roles/webserver/
├── defaults/
│   └── main.yml        # Default variables (nginx, apache2, php)
├── handlers/
│   └── main.yml        # Restart/start/stop services
├── tasks/
│   └── main.yml        # Install, configure, enable services
├── vars/
│   └── main.yml        # OS-specific package/service names
├── meta/
│   └── main.yml        # Galaxy metadata
├── README.md           # Documentation
```

### Deploy HTML App Role

```
roles/deploy_html_app/
├── defaults/
│   └── main.yml        # Default HTML source, destination, web package
├── files/
│   └── index.html      # Static HTML file to deploy
├── tasks/
│   └── main.yml        # Copy files, install package, start service
├── README.md           # Documentation
```

---

## ⚙️ Variables

### Webserver Role (`defaults/main.yml`)

```yaml
# List of packages to install
web_packages:
  - nginx
  - apache2
  - php
```

### Deploy HTML App Role (`defaults/main.yml`)

| Variable        | Default         | Description                                  |
| --------------- | --------------- | -------------------------------------------- |
| `html_src_file` | `index.html`    | HTML file path on the control node           |
| `html_dest_dir` | `/var/www/html` | Destination directory on target servers      |
| `web_package`   | `apache2`       | Web package (`apache2`, `httpd`, or `nginx`) |

---

## 📝 Tasks

### Webserver Role

```yaml
- name: Install packages (Debian)
  apt:
    name: "{{ item }}"
    state: present
    update_cache: yes
  loop: "{{ web_packages }}"
  when: ansible_os_family == "Debian"

- name: Install packages (RedHat)
  yum:
    name: "{{ item }}"
    state: present
  loop: "{{ web_packages }}"
  when: ansible_os_family == "RedHat"

- name: Enable and start services
  service:
    name: "{{ item }}"
    state: started
    enabled: yes
  loop: "{{ web_packages }}"
  notify: restart services
```

### Deploy HTML App Role

```yaml
- name: Install web package
  package:
    name: "{{ web_package }}"
    state: present

- name: Copy HTML file
  copy:
    src: "{{ html_src_file }}"
    dest: "{{ html_dest_dir }}/index.html"

- name: Ensure web service is running
  service:
    name: "{{ web_package }}"
    state: started
    enabled: yes
```

---

## 🔄 Handlers

### Webserver Role (`handlers/main.yml`)

```yaml
- name: restart services
  service:
    name: "{{ item }}"
    state: restarted
  loop: "{{ web_packages }}"

- name: stop services
  service:
    name: "{{ item }}"
    state: stopped
  loop: "{{ web_packages }}"

- name: start services
  service:
    name: "{{ item }}"
    state: started
  loop: "{{ web_packages }}"
```

---

## 🗂️ Inventory File (`inventory.ini`)

Example for AWS EC2 or local servers:

```ini
[webservers]
server1 ansible_host=13.203.201.253 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
server2 ansible_host=3.6.88.89 ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/id_rsa
server3 ansible_host=3.110.177.165 ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/id_rsa
```

---

## ▶️ Example Playbooks

### Using Webserver Role (Nginx + PHP)

```yaml
- hosts: webservers
  become: yes
  roles:
    - role: webserver
      vars:
        web_packages:
          - nginx
          - php
```

### Using Webserver Role (Apache + PHP)

```yaml
- hosts: webservers
  become: yes
  roles:
    - role: webserver
      vars:
        web_packages:
          - apache2   # or httpd on RHEL
          - php
```

### Using Deploy HTML App Role

```yaml
- hosts: webservers
  become: yes
  roles:
    - role: deploy_html_app
      vars:
        html_src_file: index.html
        html_dest_dir: /var/www/html
        web_package: apache2
```

---

## 🏃 Running the Playbook

Run against multiple servers:

```bash
ansible-playbook -i inventory.ini site.yml
```

If using password auth:

```bash
ansible-playbook -i inventory.ini site.yml --ask-pass --ask-become-pass
```

---

## 🌍 Example Workflow

1. Clone repo:

   ```bash
   git clone https://github.com/yourusername/deploy_html_app.git
   cd deploy_html_app
   ```

2. Update `inventory.ini` with your AWS EC2 or VM details

3. Place your HTML file in `roles/deploy_html_app/files/`

4. Run:

   ```bash
   ansible-playbook -i inventory.ini site.yml
   ```

5. Open in browser:

   ```
   http://<server-ip>/
   ```

---

## 📌 Notes

* Role does **not change ports** (extendable with templates/vhosts later).
* Both **nginx** and **apache** can be installed, but typically only one should serve traffic.
* Works with **AWS EC2**, **local VMs**, **on-prem servers**.
* Can be integrated with **load balancers** for HA setups.

---

## ✅ Benefits

* **Reusable**: Same role for multiple distros & services
* **Dynamic**: Install only the services you want
* **Scalable**: Manage fleets of servers easily
* **Beginner-friendly**: Works out-of-the-box with minimal config
* **Extensible**: Add SSL, vhosts, DB integration later

---

## 📚 Dependencies

* Ansible installed on the control machine
* SSH access to servers
* Supported package managers: `apt`, `yum`, `dnf`

---

# Uninstall Web Servers with Ansible

This playbook helps you **uninstall Nginx, Apache, and PHP** from your AWS (or any Linux) instances in one go.

## 📋 Prerequisites

* Ansible installed on your control machine.
* SSH access to your target instances.
* Inventory file (`hosts.ini`) with target server details.

Example `hosts.ini`:

```ini
[webservers]
your-ec2-instance-ip ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/your-key.pem
```

## ▶️ Usage

1. Save the playbook below as `uninstall.yml`.
2. Run the playbook with:

   ```bash
   ansible-playbook -i hosts.ini uninstall.yml
   ```

---

## 📝 Playbook: `uninstall.yml`

```yaml
---
- name: Uninstall web servers and PHP
  hosts: all
  become: yes
  tasks:

    - name: Remove nginx
      package:
        name: nginx
        state: absent

    - name: Remove apache2
      package:
        name: apache2
        state: absent

    - name: Remove php
      package:
        name: php
        state: absent
```

---

## ⚙️ What this does

* Removes **Nginx** (if installed).
* Removes **Apache2** (if installed).
* Removes **PHP** (if installed).

This ensures your server is **clean** from these web server stacks.

---


## 📜 License

BSD

---

## 👨‍💻 Author Information

Maintainer: **Albin Raju**
🎓 MCA Student | 🚀 DevOps & AI Enthusiast
🔗 [GitHub](https://github.com/albinraju29)

---


