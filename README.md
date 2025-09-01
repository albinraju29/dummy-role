# Role: deploy\_html\_app

## Description

This Ansible role automates the deployment of a simple HTML application across multiple servers.
It installs required packages (Apache/Nginx depending on configuration), copies an HTML file from the control machine to the target servers, and ensures the web service is running.

This role can be used with an inventory file to target multiple servers at once (for example, for load balancing with AWS EC2 instances).

---

## Requirements

* Ansible installed on the control node
* SSH access to target servers (with key or password)
* Git installed (if you are cloning from a GitHub repository instead of copying static files)

---

## Role Variables

Variables can be defined in `defaults/main.yml` or directly in the playbook.

| Variable        | Default         | Description                                                         |
| --------------- | --------------- | ------------------------------------------------------------------- |
| `html_src_file` | `index.html`    | Path to the HTML file on the control node                           |
| `html_dest_dir` | `/var/www/html` | Destination directory on target servers                             |
| `web_package`   | `apache2`       | Web server package (e.g., `apache2` for Ubuntu, `httpd` for CentOS) |

---

## Inventory File (`inventory.ini`)

The **inventory.ini** defines the target hosts (servers) where the playbook will run. Example:

```ini
[webservers]
server1 ansible_host=192.168.1.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
server2 ansible_host=192.168.1.11 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Here:

* `webservers` → group name of servers
* `ansible_host` → IP or DNS of server
* `ansible_user` → SSH user
* `ansible_ssh_private_key_file` → SSH private key path (or you can use password auth)

---

## Example Playbook (`site.yml`)

```yaml
- name: Deploy HTML application
  hosts: webservers
  become: yes

  roles:
    - deploy_html_app
```

---

## Running the Playbook

Run the playbook with:

```bash
ansible-playbook -i inventory.ini site.yml
```

If using password-based login instead of SSH keys:

```bash
ansible-playbook -i inventory.ini site.yml --ask-pass --ask-become-pass
```

---

## Example Workflow

1. Clone the repo:

   ```bash
   git clone https://github.com/yourusername/deploy_html_app.git
   cd deploy_html_app
   ```

2. Update `inventory.ini` with your server details

3. Place your `index.html` in the role’s `files/` directory

4. Run the playbook

5. Access your app in the browser:

   ```
   http://<server-ip>/
   ```

---

## Dependencies

None (but ensure SSH connectivity and package manager availability).

---

## License

BSD

---

## Author Information

* Maintainer: **Albin Raju**
* MCA Student | DevOps & AI Enthusiast
* [GitHub](https://github.com/albinraju29)

---

