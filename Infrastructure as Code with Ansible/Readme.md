
---

# 🚀 Infrastructure as Code with Ansible

This project demonstrates how to use **Ansible** to provision and configure a basic web server infrastructure in a reproducible, automated, and idempotent way.

It sets up a 3-node architecture using **Ansible roles** for modularity:
- Two **web servers** running Nginx and PHP
- One **database server** running MySQL
- Common hardening and firewall rules across all nodes

---

## 📌 Project Goals

- Automate infrastructure setup for a LEMP-style stack (Linux, Nginx, MySQL, PHP)
- Apply basic security hardening (UFW, SSH, user management)
- Ensure configurations are repeatable and version-controlled
- Follow role-based Ansible best practices

---

## 🧰 Tech Stack

- ✅ Ansible
- ✅ Ubuntu 22.04 LTS (VirtualBox, Cloud, or VPS)
- ✅ Nginx + PHP-FPM
- ✅ MySQL Server
- ✅ UFW Firewall
- ✅ SSH User Hardening

---

## 📂 Directory Structure

```
ansible-infra/
├── inventory/         # Static inventory file
├── roles/             # Ansible roles (common, webserver, database)
├── playbooks/         # Main playbooks
├── group_vars/        # Group-specific variables
├── ansible.cfg        # Config file
└── README.md          # Project documentation
```

---

## ⚙️ What This Project Does

- Installs and configures **Nginx** and **PHP-FPM** on web nodes
- Installs and configures **MySQL** with secure settings
- Deploys a test `phpinfo()` page for validation
- Configures **UFW** rules to allow only required ports (22, 80, 443, 3306 internal)
- Creates a non-root **deploy user** with sudo privileges
- Disables root login over SSH
- Ensures all services are enabled and running on boot

---

## 🚀 How to Run

1. Clone the repo:
```bash
git clone https://github.com/yourusername/ansible-infra.git
cd ansible-infra
```

2. Update inventory with your server IPs under `inventory/hosts.ini`.

3. Run the playbook:
```bash
ansible-playbook -i inventory/hosts.ini playbooks/site.yml
```

---
