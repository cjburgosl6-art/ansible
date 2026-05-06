Ansible TP
📌 Project overview
This project demonstrates the use of Ansible to automate the configuration of two Debian 12 servers from a single control machine.
The goal is to apply infrastructure-as-code principles such as idempotence, templating, roles, and secrets management.

🖥️ Infrastructure

Control node: macOS (Ansible ≥ 2.15)
Managed nodes:

web1 — Debian 12 (HTTP 80 / HTTPS 443)
web2 — Debian 12 (HTTP 8080 / HTTPS 444)


Virtualization: Proxmox
Network: private network
Access:

SSH key-based authentication
User with passwordless sudo on both servers

📁 Project structure

ansible-tp/
├── ansible.cfg
├── inventory/
│   └── hosts.ini
├── playbooks/
│   ├── bootstrap.yml
│   └── site.yml
├── roles/
│   └── webserver/
│       ├── tasks/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       ├── templates/
│       │   ├── index.html.j2
│       │   └── vhost.conf.j2
│       └── defaults/
│           └── main.yml
├── vault/
│   └── secrets.yml
└── README.md

🟦 Step 1 — Inventory and connectivity
The inventory defines the two managed servers in a group called web.

inventory/hosts.ini
[web]
web1 ansible_host=192.168.1.204 http_port=80   https_port=443
web2 ansible_host=192.168.1.205 http_port=8080 https_port=444

[all:vars]
ansible_user=cristian
ansible_become=true

Connectivity is verified using:
ansible web -m ping

✅ Both hosts respond successfully.

🟦 Step 2 — Bootstrap playbook and idempotence
The bootstrap playbook prepares the servers:

Updates APT cache
Installs base packages (curl, htop, vim)
Creates a deploy user with sudo privileges
Deploys a custom /etc/motd

playbooks/bootstrap.yml
The playbook is executed twice:
ansible-playbook playbooks/bootstrap.yml
ansible-playbook playbooks/bootstrap.ymlMostrar más líneas

On the second run, all tasks return ok.

✅ What is idempotence?

Idempotence means that a playbook can be executed multiple times without changing the system if it is already in the desired state.
This is crucial to ensure reliability, safety, and predictability in automation.

🟦 Step 3 — Variables, templates and handlers (Nginx)
Nginx is deployed using:

Host-specific variables for ports
Jinja2 templates for configuration and HTML
A handler that reloads Nginx only when needed

Template: dynamic homepage
Each server displays its own system information:

Hostname
IP address
Operating system
RAM

Handler
A handler reloads Nginx only if a configuration file changes.
Difference between a task and a handler

Task: executed every time the playbook runs
Handler: executed only when notified by a changed task


🟦 Step 4 — Refactoring into a role
The Nginx logic is refactored into a role named webserver.
Advantages of using roles:

Better code organization
Reusability
Easier maintenance
Clear separation of concerns

The main playbook becomes minimal:
playbooks/site.yml
- hosts: web
  roles:
    - webserver

🟦 Step 5 — Secrets management with Ansible Vault
Sensitive data is stored securely using Ansible Vault.

Encrypted file
ansible-vault create vault/secrets.yml

Content:
db_password: fakepassword
api_token: faketokenMostrar

🔐 For this project, the Vault password used during execution was vault123.
This password is never stored in the repository.

Secure deployment
Secrets are written to /etc/myapp.env with strict permissions:

Mode 0600
no_log: true to prevent leaks in console output

Why is no_log important?
Without no_log, sensitive information could appear in:

Ansible console output
Logs
CI/CD pipelines

Using no_log prevents accidental exposure of secrets.

📦 Version control
The project is versioned using Git.
Included in Git:

Playbooks
Roles
Templates
README

Excluded from Git:

Private SSH keys
Vault passwords
Sensitive runtime data


❓ Final questions
Difference between Ansible and Puppet/Chef
Ansible is agentless and uses SSH, while Puppet and Chef require agents installed on managed nodes.

Why is Ansible agentless?
Because it relies on SSH and existing system tools, reducing complexity and attack surface.

When to use a playbook vs a role?
Playbook: orchestration
Role: reusable and modular logic

How would you manage more than 1000 servers?
By using inventories, dynamic inventory sources, parallelism, and delegation tools.

✅ Conclusion
This project demonstrates:

Infrastructure as Code
Idempotent automation
Modular design with roles
Secure secret handling

All requirements of the assignment are fulfilled.
