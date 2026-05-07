📘 Ansible TP – Web & Database Automation
📌 Project Overview
This project demonstrates the use of Ansible to automate the configuration of a small
multi‑tier infrastructure using Infrastructure as Code principles.

The main objectives are:
Idempotent configuration management
Use of inventories, playbooks, roles, templates, handlers
Secure secret handling with Ansible Vault
Clear separation of responsibilities (web / database)


🖥️ Infrastructure

Control Node
macOS / Linux / WSL
Ansible ≥ 2.15

Managed Nodes
web1 – Debian 12 (HTTP 80 / HTTPS 443)
web2 – Debian 12 (HTTP 8080 / HTTPS 444)
db1 – CentOS (PostgreSQL)

Network & Access
Private network between all hosts
SSH key‑based authentication
User with passwordless sudo on all managed nodes


📁 Project Structure
ansible-tp/
├─ansible.cfg
├─inventory/
│   └─hosts.ini
├─playbooks/
│   ├─bootstrap.yml
│   ├─site.yml
│   └─extras/
│       └─nginx_block_rescue.yml
├─roles/
│   ├─webserver/
│   │   ├─tasks/
│   │   │   └─main.yml
│   │   ├─handlers/
│   │   │   └─main.yml
│   │   ├─templates/
│   │   │   ├─index.html.j2
│   │   │   └─vhost.conf.j2
│   │   └─defaults/
│   │       └ main.yml
│   └─db/
│       └─tasks/
│           └─main.yml
├─vault/
│   └─secrets.yml
└─README.md


🟦 Step 1 – Inventory and Connectivity

Inventory
INI[web]web1 ansible_host=192.168.1.204 http_port=80   https_port=443web2 ansible_host=192.168.1.205 http_port=8080 https_port=444[db]db1 ansible_host=192.168.1.206[all:vars]ansible_user=cristianansible_become=trueansible_python_interpreter=/usr/bin/python3Mostrar más líneas
Connectivity is verified using:
Shellansible web -m pingansible db  -m pingMostrar más líneas
✅ All hosts respond successfully.


🟦 Step 2 – Bootstrap Playbook

The bootstrap playbook prepares all servers with common requirements:
Update APT cache (idempotent)
Install base packages (curl, htop, vim) using a loop
Create a deploy user with sudo privileges
Deploy a custom /etc/motd

The playbook is executed twice to validate idempotence:
ansible-playbook playbooks/bootstrap.yml
ansible-playbook playbooks/bootstrap.yml

✅ Idempotence
Idempotence means that running the same playbook multiple times does not change the system if it is already in the desired state.
This guarantees safe, predictable automation.


🟦 Step 3 – Web Server Configuration (Nginx)

The webserver role handles all web‑related configuration:
Installation of Nginx
Removal of default configuration
Deployment of a custom virtual host
Host‑specific ports via variables
Dynamic HTML page generated with Jinja2 templates
Use of handlers to reload Nginx only when required

Dynamic Homepage
Each server displays its own system information:
Hostname
IP address
Operating system
Total RAM

All values are retrieved using ansible_facts.


🟦 Step 4 – Handlers

Handlers are used to manage non‑idempotent actions safely.
Example:
notify: reload nginx
- name: reload nginx
  service:
    name: nginx
    state: reloaded

Difference Between a Task and a Handler
Task: executed every time the playbook runs
Handler: executed only if notified by a changed task

This mechanism preserves idempotence while allowing service reloads when needed.


🟦 Step 5 – Database Role (Extra)

A separate role db is used to manage the database host.

Role Responsibility
Install PostgreSQL on db1
Demonstrate multi‑tier infrastructure management

Example task:
- name: Install PostgreSQL
  dnf:
    name: postgresql-server
    state: present

This shows clear separation between web and database responsibilities.


🟦 Step 6 – Error Handling with block / rescue / always (Extra)

To demonstrate error handling, a separate playbook is used:
Textplaybooks/extras/nginx_block_rescue.yml

This playbook wraps a risky operation (service reload) inside a:
block – normal execution
rescue – fallback action
always – cleanup / logging

This example is intentionally isolated so it does not affect the idempotent main workflow.


🟦 Step 7 – Secrets Management with Ansible Vault

Sensitive variables are stored securely using Ansible Vault.
ansible-vault create vault/secrets.yml

Content:
db_password: fakepassword
api_token: faketoken

Secrets are deployed to /etc/myapp.env with strict permissions:
Mode 0600
no_log: true to avoid leaking sensitive data

🔐 For this project, the Vault password used during execution was vault123.
The password itself is never stored in the repository.

Why no_log Is Important
Without no_log, secrets could appear in:
Console output
Logs
CI/CD pipelines

Using no_log prevents accidental exposure.

📦 Version Control
The project is versioned using Git.

Included
Playbooks
Roles
Templates
Inventory
README

Excluded
Private SSH keys
Vault passwords
Decrypted secrets

Encrypted Vault files are safe to commit.


❓ Final Questions

Difference between Ansible and Puppet / Chef
Ansible is agentless and uses SSH, while Puppet and Chef require agents installed on managed nodes.

Why is Ansible agentless?
Because it relies on existing system tools (SSH, Python), reducing complexity and attack surface.

When to use a playbook vs a role?
Playbook: orchestration
Role: reusable, modular logic

How would you version this project?
Using Git, excluding sensitive data and managing secrets with Ansible Vault.

Limits of Ansible at scale
For very large infrastructures (>1000 hosts), performance and orchestration complexity may require additional tooling or tuning.


✅ Conclusion

This project demonstrates:
Idempotent automation
Clean role‑based architecture
Safe handling of secrets
Separation of concerns
Realistic infrastructure management

All requirements of the assignment are fulfilled.
