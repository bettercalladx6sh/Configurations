# Ansible Complete Reference

A reusable beginner-to-intermediate Ansible reference covering:

- What Ansible is
- Ansible architecture
- Control node and managed nodes
- SSH authentication
- Inventory
- Inventory groups
- Variables
- Group variables
- Ad-hoc commands
- Modules
- Playbooks
- Plays
- Tasks
- Handlers
- Jinja2 templates
- Roles
- Check mode
- Diff mode
- Tags
- Limits
- Ansible Vault
- Useful commands
- Troubleshooting
- Recommended project structure
- Recommended workflow

---

# 1. What is Ansible?

**Ansible is an open-source automation and configuration-management tool.**

It allows you to automate tasks across multiple servers from a central machine.

For example, without Ansible:

```text
Your Laptop
    |
    +-- SSH → Server 1 → install package
    |
    +-- SSH → Server 2 → install package
    |
    +-- SSH → Server 3 → install package
    |
    +-- SSH → Server 4 → install package
```

With Ansible:

```text
                 Ansible Control Node
                         |
                         |
                One automation command
                         |
          +--------------+--------------+
          |              |              |
          ↓              ↓              ↓
       Server 1       Server 2       Server 3
```

You define the desired configuration once and Ansible applies it to the selected machines.

---

# 2. Why Use Ansible?

Ansible is useful because it provides:

- Automation
- Repeatability
- Consistency
- Configuration management
- Remote execution
- Infrastructure deployment
- Application deployment
- Server configuration
- Monitoring configuration
- Package management
- Service management

Instead of manually doing:

```text
SSH
Install
Configure
Restart
Verify
```

you can automate the entire process.

---

# 3. Basic Ansible Architecture

```text
                         CONTROL NODE
                       Your Linux machine
                              |
                              |
                             SSH
                              |
          +-------------------+-------------------+
          |                   |                   |
          ↓                   ↓                   ↓
     Managed Node 1      Managed Node 2      Managed Node 3
        Web Server          Web Server         Database
```

There are two important sides:

```text
Control Node
    ↓
Runs Ansible

Managed Nodes
    ↓
Machines Ansible manages
```

---

# 4. Control Node

The **Control Node** is the machine where Ansible is installed.

For example:

```text
Your Ubuntu Laptop
```

You run commands such as:

```bash
ansible
ansible-playbook
ansible-inventory
ansible-vault
ansible-doc
```

from the control node.

---

# 5. Managed Nodes

Managed nodes are the remote machines Ansible controls.

Examples:

```text
AWS EC2
GCP VM
Azure VM
On-premise Linux server
Virtual machine
```

For Linux systems, Ansible normally connects using SSH.

---

# 6. How SSH Fits Into Ansible

A simplified connection:

```text
Ansible Control Node
        |
        | SSH
        ↓
Ubuntu Server
```

For SSH key authentication:

```text
Control Node
     |
     | Private Key
     ↓
SSH Authentication
     |
     ↓
Remote Server
     |
     | checks
     ↓
authorized_keys
```

For example:

```bash
ssh -i ansible-key.pem ubuntu@SERVER_IP
```

The private key proves that you possess the private half of the configured SSH key pair.

Ansible uses SSH underneath for normal Linux administration.

---

# 7. Recommended Project Structure

For a small reusable project:

```text
ansible-project/
│
├── README.md
├── ansible.cfg
├── .gitignore
│
├── inventory/
│   ├── hosts.ini
│   │
│   └── group_vars/
│       └── all.yml
│
├── playbooks/
│   └── main.yml
│
├── templates/
│   └── example.conf.j2
│
└── files/
    └── example.txt
```

For a larger project:

```text
ansible-project/
│
├── README.md
├── ansible.cfg
├── .gitignore
│
├── inventory/
│   │
│   ├── production/
│   │   ├── hosts.ini
│   │   └── group_vars/
│   │       └── all.yml
│   │
│   └── staging/
│       ├── hosts.ini
│       └── group_vars/
│           └── all.yml
│
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
│   ├── databases.yml
│   └── monitoring.yml
│
├── roles/
│   │
│   ├── nginx/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   ├── files/
│   │   ├── vars/
│   │   │   └── main.yml
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   └── meta/
│   │       └── main.yml
│   │
│   └── monitoring/
│       └── ...
│
├── templates/
│   └── example.conf.j2
│
└── files/
    └── example.txt
```

---

# 8. What Each Directory Does

## `README.md`

Documentation for the project.

It explains:

```text
What the project does
How to install dependencies
How to configure inventory
How to run it
How to troubleshoot it
```

---

## `ansible.cfg`

Project-level Ansible configuration.

Example:

```ini
[defaults]
inventory = ./inventory/hosts.ini
remote_user = ubuntu
host_key_checking = False
retry_files_enabled = False
```

For production, consider keeping host key checking enabled.

---

## `inventory/`

Contains information about the machines Ansible manages.

```text
inventory/
└── hosts.ini
```

---

## `group_vars/`

Contains variables for inventory groups.

Example:

```text
inventory/
└── group_vars/
    └── all.yml
```

---

## `playbooks/`

Contains your automation playbooks.

Example:

```text
playbooks/
├── main.yml
├── webservers.yml
└── monitoring.yml
```

---

## `templates/`

Contains Jinja2 templates.

Example:

```text
templates/
├── nginx.conf.j2
├── application.conf.j2
└── ops-agent-config.yaml.j2
```

---

## `files/`

Contains static files that need to be copied to remote machines.

Example:

```text
files/
└── script.sh
```

---

## `roles/`

Roles provide a structured way of organizing large Ansible projects.

More information is covered later.

---

# 9. Inventory

An **inventory tells Ansible which machines it should manage.**

Example:

```ini
[webservers]
web01 ansible_host=192.168.1.10
web02 ansible_host=192.168.1.11

[dbservers]
db01 ansible_host=192.168.1.20
```

Here:

```text
webservers
    ├── web01
    └── web02

dbservers
    └── db01
```

---

# 10. Host

A **host** is one managed machine in the inventory.

Example:

```ini
[webservers]
web01
web02
```

Here:

```text
web01 = host
web02 = host
```

---

# 11. Host Group

A **host group** is a collection of hosts.

```ini
[webservers]
web01
web02
web03
```

Now you can target all three with:

```bash
ansible webservers -m ping
```

---

# 12. Inventory With SSH Information

Example:

```ini
[webservers]
web01 ansible_host=3.110.154.122
web02 ansible_host=3.110.154.123

[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/ansible-key.pem
```

Then:

```bash
ansible webservers -m ping
```

---

# 13. Better SSH Key Practice

Instead of storing the private key path in inventory, you can specify it during execution:

```bash
ansible all \
  -m ping \
  --private-key ~/.ssh/ansible-key.pem
```

Playbook:

```bash
ansible-playbook \
  playbooks/main.yml \
  --private-key ~/.ssh/ansible-key.pem
```

Never commit:

```text
*.pem
id_rsa
id_ed25519
```

to Git.

---

# 14. `group_vars`

Variables can be placed in:

```text
inventory/group_vars/
```

Example:

```text
inventory/
└── group_vars/
    └── all.yml
```

`all.yml`:

```yaml
---
app_name: my_application
app_environment: production

custom_log_path: /var/log/syslog

config_directory: /etc/my_application
config_file: /etc/my_application/application.conf

service_name: my_application
```

These variables can be used by the playbook and templates.

---

# 15. What is a Variable?

A variable stores a value that can be reused.

Instead of:

```yaml
port: 8080
```

in multiple places, define:

```yaml
app_port: 8080
```

and use:

```yaml
port: "{{ app_port }}"
```

This makes automation reusable.

---

# 16. What is a Module?

An **Ansible module is a reusable unit of functionality that performs an operation on a managed node.**

Common modules:

```text
ping
command
shell
file
copy
template
service
package
apt
user
debug
setup
```

Example:

```bash
ansible all -m ping
```

Here:

```text
-m
 ↓
module

ping
 ↓
module name
```

---

# 17. What is an Ad-hoc Command?

An **ad-hoc command is a one-time Ansible command used to perform a quick operation without creating a playbook.**

Think:

```text
"I need to quickly check or change something."
```

Example:

```bash
ansible all -m ping
```

Or:

```bash
ansible all -m command -a "uptime"
```

Ad-hoc commands are useful for:

- Troubleshooting
- Testing connectivity
- Checking server information
- Quick administration
- One-time operations

---

# 18. Ad-hoc Command Syntax

General syntax:

```bash
ansible <host-pattern> -m <module> -a "<arguments>"
```

Example:

```bash
ansible webservers -m command -a "uptime"
```

Breakdown:

```text
ansible
   ↓
Ansible CLI

webservers
   ↓
Target hosts

-m command
   ↓
Module

-a "uptime"
   ↓
Module arguments
```

---

# 19. Ad-hoc: Ping

```bash
ansible all -m ping
```

Tests whether Ansible can connect and successfully execute its ping module.

---

# 20. Ad-hoc: Uptime

```bash
ansible all -m command -a "uptime"
```

---

# 21. Ad-hoc: Hostname

```bash
ansible all -m command -a "hostname"
```

---

# 22. Ad-hoc: Disk Space

```bash
ansible all -m command -a "df -h"
```

---

# 23. Ad-hoc: Memory

```bash
ansible all -m command -a "free -h"
```

---

# 24. `command` vs `shell`

## command

```bash
ansible all -m command -a "uptime"
```

The `command` module executes commands directly.

It does not process shell operators such as:

```text
|
>
<
&&
||
```

---

## shell

```bash
ansible all -m shell -a "ps aux | grep nginx"
```

Use `shell` when you specifically need shell features.

Prefer `command` when shell functionality isn't required.

---

# 25. Ad-hoc: Become Root

Use:

```bash
ansible all -m command -a "whoami" --become
```

Short form:

```bash
ansible all -m command -a "whoami" -b
```

`--become` generally means Ansible will use privilege escalation, commonly through `sudo`.

---

# 26. Ad-hoc: Install Package

Ubuntu/Debian:

```bash
ansible webservers \
  -m ansible.builtin.apt \
  -a "name=nginx state=present" \
  --become
```

---

# 27. Ad-hoc: Start Service

```bash
ansible webservers \
  -m ansible.builtin.service \
  -a "name=nginx state=started" \
  --become
```

---

# 28. Ad-hoc: Create Directory

```bash
ansible all \
  -m ansible.builtin.file \
  -a "path=/opt/myapp state=directory mode=0755" \
  --become
```

---

# 29. Ad-hoc: Copy File

```bash
ansible all \
  -m ansible.builtin.copy \
  -a "src=./test.txt dest=/tmp/test.txt"
```

---

# 30. What is a Playbook?

A **playbook is a YAML file containing one or more plays that describe automation tasks.**

Instead of manually running:

```text
Install
Configure
Copy
Restart
Verify
```

you define the process in YAML.

Example:

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
```

Run:

```bash
ansible-playbook playbooks/main.yml
```

---

# 31. Ad-hoc vs Playbook

The easiest way to remember:

```text
Ad-hoc
    ↓
Quick / one-time

Playbook
    ↓
Reusable / repeatable
```

### Ad-hoc

```bash
ansible all -m command -a "uptime"
```

### Playbook

```yaml
- name: Configure servers
  hosts: all

  tasks: ...
```

Use playbooks when you want:

- Repeatability
- Version control
- Documentation
- Complex automation
- Multiple tasks
- Variables
- Templates
- Handlers
- Conditions
- Loops

---

# 32. What is a Play?

A **play connects a group of hosts to a set of tasks.**

Example:

```yaml
- name: Configure web servers
  hosts: webservers

  tasks: ...
```

Think:

```text
Play
 |
 +-- Target hosts
 |
 +-- Tasks
```

A playbook can contain multiple plays.

---

# 33. What is a Task?

A task is one individual operation.

Example:

```yaml
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    state: present
```

A play can contain:

```text
Task 1 → Install nginx
Task 2 → Create directory
Task 3 → Deploy configuration
Task 4 → Start service
```

---

# 34. What is Idempotency?

**Idempotency means that repeatedly applying the same automation should result in the same desired state rather than repeatedly making unnecessary changes.**

Example:

```yaml
- name: Ensure nginx is installed
  ansible.builtin.apt:
    name: nginx
    state: present
```

First run:

```text
nginx missing
    ↓
Install nginx
    ↓
CHANGED
```

Second run:

```text
nginx already installed
    ↓
Nothing needs changing
    ↓
OK
```

This is an important concept in Ansible.

---

# 35. What is a Template?

A template is a file containing variables that Ansible renders before putting it on the remote machine.

Example:

```jinja2
application={{ app_name }}
environment={{ app_environment }}
port={{ app_port }}
```

Variables:

```yaml
app_name: myapp
app_environment: production
app_port: 8080
```

Generated file:

```text
application=myapp
environment=production
port=8080
```

Template files normally end with:

```text
.j2
```

---

# 36. What is Jinja2?

**Jinja2 is the templating language used by Ansible.**

Common syntax:

```jinja2
{{ variable }}
```

Example:

```jinja2
{{ app_name }}
```

Conditional:

```jinja2
{% if app_environment == "production" %}
production_mode=true
{% endif %}
```

Loop:

```jinja2
{% for server in servers %}
server={{ server }}
{% endfor %}
```

---

# 37. Template Module

The template module renders a Jinja2 template and copies the resulting file to the managed node.

Example:

```yaml
- name: Deploy configuration
  ansible.builtin.template:
    src: example.conf.j2
    dest: /etc/myapp/example.conf
```

Flow:

```text
example.conf.j2
       |
       ↓
Ansible renders variables
       |
       ↓
example.conf
       |
       ↓
Remote Server
```

---

# 38. Example Template

`templates/example.conf.j2`

```jinja2
# Managed by Ansible.
# Do not edit manually.

application:
  name: "{{ app_name }}"
  environment: "{{ app_environment }}"

logging:
  path: "{{ custom_log_path }}"

server:
  hostname: "{{ inventory_hostname }}"
```

---

# 39. What is a Handler?

A handler is a task that runs when another task notifies it.

Example:

```yaml
- name: Deploy configuration
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx
```

Handler:

```yaml
handlers:
  - name: Restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
```

Flow:

```text
Configuration changes
        ↓
Task becomes CHANGED
        ↓
notify
        ↓
Handler runs
        ↓
Service restarts
```

If the configuration does not change:

```text
No change
   ↓
No notification
   ↓
No restart
```

---

# 40. Complete Playbook Example

`playbooks/main.yml`

```yaml
---
- name: Configure Linux Servers
  hosts: all
  become: true

  vars:
    config_template: ../templates/example.conf.j2

  tasks:
    - name: Ensure configuration directory exists
      ansible.builtin.file:
        path: "{{ config_directory }}"
        state: directory
        owner: root
        group: root
        mode: "0755"

    - name: Deploy configuration
      ansible.builtin.template:
        src: "{{ config_template }}"
        dest: "{{ config_file }}"
        owner: root
        group: root
        mode: "0644"
      notify: Restart application

    - name: Display configuration information
      ansible.builtin.debug:
        msg:
          - "Application: {{ app_name }}"
          - "Environment: {{ app_environment }}"
          - "Configuration: {{ config_file }}"

  handlers:
    - name: Restart application
      ansible.builtin.service:
        name: "{{ service_name }}"
        state: restarted
```

---

# 41. Running a Playbook

Basic:

```bash
ansible-playbook playbooks/main.yml
```

With inventory:

```bash
ansible-playbook \
  -i inventory/hosts.ini \
  playbooks/main.yml
```

With SSH key:

```bash
ansible-playbook \
  -i inventory/hosts.ini \
  playbooks/main.yml \
  --private-key ~/.ssh/ansible-key.pem
```

---

# 42. Specify SSH User

```bash
ansible-playbook \
  playbooks/main.yml \
  -u ubuntu
```

With key:

```bash
ansible-playbook \
  playbooks/main.yml \
  -u ubuntu \
  --private-key ~/.ssh/ansible-key.pem
```

---

# 43. Check SSH Connectivity First

Before running a playbook:

```bash
ansible all -m ping
```

If using a key:

```bash
ansible all \
  -m ping \
  --private-key ~/.ssh/ansible-key.pem
```

Always verify connectivity before troubleshooting the playbook itself.

---

# 44. What is Check Mode?

Check mode is a **dry run**.

It shows what Ansible would change without applying the changes.

```bash
ansible-playbook playbooks/main.yml --check
```

Think:

```text
Normal
    ↓
Make changes

Check mode
    ↓
Predict changes
```

---

# 45. What is Diff Mode?

Diff mode shows configuration differences where supported.

```bash
ansible-playbook \
  playbooks/main.yml \
  --check \
  --diff
```

Useful for configuration files.

---

# 46. What is `--limit`?

`--limit` restricts execution to selected hosts.

One server:

```bash
ansible-playbook playbooks/main.yml \
  --limit web01
```

One group:

```bash
ansible-playbook playbooks/main.yml \
  --limit webservers
```

Multiple hosts:

```bash
ansible-playbook playbooks/main.yml \
  --limit "web01,web02"
```

Very useful when testing changes.

---

# 47. What are Tags?

Tags allow you to execute only selected tasks.

Example:

```yaml
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    state: present
  tags:
    - packages

- name: Configure nginx
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  tags:
    - config
```

Run only config:

```bash
ansible-playbook playbooks/main.yml --tags config
```

Run only packages:

```bash
ansible-playbook playbooks/main.yml --tags packages
```

Skip:

```bash
ansible-playbook playbooks/main.yml --skip-tags packages
```

---

# 48. What are Extra Variables?

Extra variables allow you to provide values when running a playbook.

Example:

```yaml
- name: Show environment
  ansible.builtin.debug:
    msg: "Environment: {{ environment }}"
```

Run:

```bash
ansible-playbook playbooks/main.yml \
  -e "environment=production"
```

Multiple:

```bash
ansible-playbook playbooks/main.yml \
  -e "environment=production version=2.0"
```

---

# 49. Syntax Check

Before running:

```bash
ansible-playbook playbooks/main.yml --syntax-check
```

This checks the playbook syntax.

---

# 50. List Tasks

See what tasks exist:

```bash
ansible-playbook playbooks/main.yml --list-tasks
```

---

# 51. List Hosts

```bash
ansible-playbook playbooks/main.yml --list-hosts
```

---

# 52. List Tags

```bash
ansible-playbook playbooks/main.yml --list-tags
```

---

# 53. Verbose Mode

Normal:

```bash
ansible-playbook playbooks/main.yml
```

Verbose:

```bash
ansible-playbook playbooks/main.yml -v
```

More verbose:

```bash
ansible-playbook playbooks/main.yml -vv
```

Very verbose:

```bash
ansible-playbook playbooks/main.yml -vvv
```

Extremely verbose:

```bash
ansible-playbook playbooks/main.yml -vvvv
```

Use high verbosity carefully because connection/debug output can contain sensitive information.

---

# 54. What is `ansible-inventory`?

`ansible-inventory` lets you inspect your inventory.

Graph:

```bash
ansible-inventory \
  -i inventory/hosts.ini \
  --graph
```

List:

```bash
ansible-inventory \
  -i inventory/hosts.ini \
  --list
```

---

# 55. What is `ansible-doc`?

`ansible-doc` displays documentation for modules.

Example:

```bash
ansible-doc ansible.builtin.file
```

Another:

```bash
ansible-doc ansible.builtin.template
```

This is extremely useful when learning a new module.

---

# 56. What is `ansible-config`?

Used to inspect Ansible configuration.

```bash
ansible-config dump
```

List configuration options:

```bash
ansible-config list
```

---

# 57. What are Ansible Facts?

Facts are information Ansible gathers about managed machines.

Example:

```bash
ansible all -m ansible.builtin.setup
```

Facts can contain information such as:

```text
Hostname
Operating system
OS version
Architecture
Memory
CPU
Network interfaces
IP addresses
```

Example inside a playbook:

```yaml
- name: Show operating system
  ansible.builtin.debug:
    msg: "{{ ansible_distribution }}"
```

---

# 58. Useful Facts

Examples:

```text
ansible_hostname
ansible_distribution
ansible_distribution_version
ansible_architecture
ansible_processor_vcpus
ansible_memtotal_mb
```

---

# 59. What is Ansible Vault?

**Ansible Vault is used to encrypt sensitive data used by Ansible.**

Examples:

```text
Passwords
API keys
Credentials
Secrets
Private configuration
```

Create:

```bash
ansible-vault create secrets.yml
```

Edit:

```bash
ansible-vault edit secrets.yml
```

View:

```bash
ansible-vault view secrets.yml
```

Encrypt existing file:

```bash
ansible-vault encrypt secrets.yml
```

Decrypt:

```bash
ansible-vault decrypt secrets.yml
```

Run:

```bash
ansible-playbook playbooks/main.yml \
  --ask-vault-pass
```

Never put secrets directly into playbooks or Git.

---

# 60. What is a Role?

A **role is a standardized way of organizing reusable Ansible automation.**

Instead of putting everything into one giant playbook:

```text
main.yml
  ├── install
  ├── configure
  ├── templates
  ├── handlers
  └── variables
```

you can organize it:

```text
roles/
└── nginx/
    ├── tasks/
    ├── handlers/
    ├── templates/
    ├── files/
    ├── vars/
    ├── defaults/
    └── meta/
```

Roles become especially useful as projects grow.

---

# 61. Role Directory Structure

Example:

```text
roles/
└── nginx/
    │
    ├── tasks/
    │   └── main.yml
    │
    ├── handlers/
    │   └── main.yml
    │
    ├── templates/
    │   └── nginx.conf.j2
    │
    ├── files/
    │   └── example.txt
    │
    ├── vars/
    │   └── main.yml
    │
    ├── defaults/
    │   └── main.yml
    │
    └── meta/
        └── main.yml
```

---

# 62. Role Components

```text
tasks/
    What should Ansible do?

handlers/
    What should happen after a change?

templates/
    Dynamic configuration files

files/
    Static files

defaults/
    Default variables

vars/
    Role variables

meta/
    Role metadata and dependencies
```

---

# 63. Simple Role Example

`roles/nginx/tasks/main.yml`

```yaml
---
- name: Install nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: true

- name: Start nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

Playbook:

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true

  roles:
    - nginx
```

---

# 64. What is `.gitignore`?

Use `.gitignore` to prevent sensitive or unnecessary files from being committed.

Example:

```gitignore
*.pem
*.key
id_rsa
id_ed25519

*.retry

.env

__pycache__/
*.pyc

.vscode/
.idea/
```

Never commit private SSH keys.

---

# 65. Complete Basic Project

```text
ansible-project/
│
├── README.md
│
├── ansible.cfg
│
├── .gitignore
│
├── inventory/
│   │
│   ├── hosts.ini
│   │
│   └── group_vars/
│       └── all.yml
│
├── playbooks/
│   └── main.yml
│
├── templates/
│   └── example.conf.j2
│
└── files/
    └── example.txt
```

---

# 66. Complete Example Inventory

`inventory/hosts.ini`

```ini
[webservers]
web01 ansible_host=192.168.1.10
web02 ansible_host=192.168.1.11

[dbservers]
db01 ansible_host=192.168.1.20

[all:vars]
ansible_user=ubuntu
```

---

# 67. Complete Example Variables

`inventory/group_vars/all.yml`

```yaml
---
app_name: my_application
app_environment: production
app_port: 8080

custom_log_path: /var/log/syslog

config_directory: /etc/my_application
config_file: /etc/my_application/application.conf

service_name: my_application
```

---

# 68. Complete Example Template

`templates/example.conf.j2`

```jinja2
# Managed by Ansible.
# Do not edit manually.

application:
  name: "{{ app_name }}"
  environment: "{{ app_environment }}"
  port: {{ app_port }}

logging:
  path: "{{ custom_log_path }}"

server:
  hostname: "{{ inventory_hostname }}"
```

---

# 69. Complete Example Playbook

`playbooks/main.yml`

```yaml
---
- name: Configure Linux Servers
  hosts: all
  become: true

  tasks:
    - name: Ensure configuration directory exists
      ansible.builtin.file:
        path: "{{ config_directory }}"
        state: directory
        owner: root
        group: root
        mode: "0755"

    - name: Deploy configuration
      ansible.builtin.template:
        src: ../templates/example.conf.j2
        dest: "{{ config_file }}"
        owner: root
        group: root
        mode: "0644"

    - name: Display server information
      ansible.builtin.debug:
        msg:
          - "Server: {{ inventory_hostname }}"
          - "Application: {{ app_name }}"
          - "Environment: {{ app_environment }}"
```

---

# 70. First-Time Setup

From the project directory:

```bash
cd ansible-project
```

Check Ansible:

```bash
ansible --version
```

Check inventory:

```bash
ansible-inventory -i inventory/hosts.ini --graph
```

Test SSH:

```bash
ansible all \
  -m ping \
  --private-key ~/.ssh/ansible-key.pem
```

Validate playbook:

```bash
ansible-playbook \
  playbooks/main.yml \
  --syntax-check
```

Dry run:

```bash
ansible-playbook \
  playbooks/main.yml \
  --check \
  --diff
```

Run:

```bash
ansible-playbook \
  playbooks/main.yml \
  --private-key ~/.ssh/ansible-key.pem
```

---

# 71. Recommended Ansible Workflow

Use this workflow whenever creating or modifying automation:

```text
                START
                  |
                  ↓
          Check inventory
                  |
                  ↓
       ansible-inventory --graph
                  |
                  ↓
            Test connectivity
                  |
                  ↓
          ansible all -m ping
                  |
                  ↓
          Write / modify playbook
                  |
                  ↓
             Syntax check
                  |
                  ↓
       ansible-playbook --syntax-check
                  |
                  ↓
              Dry run
                  |
                  ↓
       ansible-playbook --check --diff
                  |
                  ↓
             Review changes
                  |
                  ↓
          Run the playbook
                  |
                  ↓
              Verify
                  |
                  ↓
                DONE
```

---

# 72. Most Important Commands

## Check Ansible

```bash
ansible --version
```

## Test connectivity

```bash
ansible all -m ping
```

## Check inventory

```bash
ansible-inventory --graph
```

## List hosts

```bash
ansible all --list-hosts
```

## Run ad-hoc command

```bash
ansible all -m command -a "uptime"
```

## Run playbook

```bash
ansible-playbook playbooks/main.yml
```

## Syntax check

```bash
ansible-playbook playbooks/main.yml --syntax-check
```

## Dry run

```bash
ansible-playbook playbooks/main.yml --check
```

## Dry run with diff

```bash
ansible-playbook playbooks/main.yml --check --diff
```

## Limit to host

```bash
ansible-playbook playbooks/main.yml --limit web01
```

## Limit to group

```bash
ansible-playbook playbooks/main.yml --limit webservers
```

## Run tag

```bash
ansible-playbook playbooks/main.yml --tags config
```

## Verbose

```bash
ansible-playbook playbooks/main.yml -vvv
```

## Module documentation

```bash
ansible-doc ansible.builtin.template
```

## Inspect configuration

```bash
ansible-config dump
```

## Vault

```bash
ansible-vault create secrets.yml
```

---

# 73. Ad-hoc Command Cheat Sheet

```bash
# Connectivity
ansible all -m ping

# Hostname
ansible all -m command -a "hostname"

# Uptime
ansible all -m command -a "uptime"

# Disk
ansible all -m command -a "df -h"

# Memory
ansible all -m command -a "free -h"

# Kernel
ansible all -m command -a "uname -a"

# Process
ansible all -m shell -a "ps aux"

# Install nginx
ansible webservers -m apt \
  -a "name=nginx state=present" \
  --become

# Start nginx
ansible webservers -m service \
  -a "name=nginx state=started" \
  --become

# Create directory
ansible all -m file \
  -a "path=/opt/app state=directory" \
  --become

# Copy file
ansible all -m copy \
  -a "src=./test.txt dest=/tmp/test.txt"
```

---

# 74. Playbook Execution Cheat Sheet

```bash
# Basic
ansible-playbook playbooks/main.yml

# Specific inventory
ansible-playbook \
  -i inventory/hosts.ini \
  playbooks/main.yml

# SSH key
ansible-playbook \
  playbooks/main.yml \
  --private-key ~/.ssh/ansible-key.pem

# Specific user
ansible-playbook \
  playbooks/main.yml \
  -u ubuntu

# Check mode
ansible-playbook \
  playbooks/main.yml \
  --check

# Check + diff
ansible-playbook \
  playbooks/main.yml \
  --check \
  --diff

# Specific host
ansible-playbook \
  playbooks/main.yml \
  --limit web01

# Specific group
ansible-playbook \
  playbooks/main.yml \
  --limit webservers

# Specific tag
ansible-playbook \
  playbooks/main.yml \
  --tags config

# Extra variables
ansible-playbook \
  playbooks/main.yml \
  -e "environment=production"

# Verbose
ansible-playbook \
  playbooks/main.yml \
  -vvv
```

---

# 75. Troubleshooting Checklist

If Ansible fails, don't immediately modify the playbook.

Check things in this order:

```text
1. Is the server reachable?
        ↓
2. Does SSH work?
        ↓
3. Is the correct user being used?
        ↓
4. Is the correct private key being used?
        ↓
5. Does ansible all -m ping work?
        ↓
6. Is the inventory correct?
        ↓
7. Is the playbook syntax valid?
        ↓
8. Does the user have sudo privileges?
        ↓
9. Is the module being used correctly?
        ↓
10. Run with -vvv
```

---

# 76. SSH Troubleshooting

Test SSH manually:

```bash
ssh -i ~/.ssh/ansible-key.pem ubuntu@SERVER_IP
```

If this doesn't work, fix SSH first.

Check key permissions:

```bash
chmod 400 ~/.ssh/ansible-key.pem
```

Test Ansible:

```bash
ansible all \
  -m ping \
  --private-key ~/.ssh/ansible-key.pem \
  -u ubuntu
```

---

# 77. Common Permission Problem

Error:

```text
Permission denied
```

Possible causes:

```text
Wrong username
Wrong private key
Wrong public key on server
Incorrect authorized_keys
Incorrect key permissions
SSH configuration issue
```

Remember:

```text
SSH problem
    ↓
Fix SSH

Ansible problem
    ↓
Fix Ansible
```

Don't try to debug both simultaneously.

---

# 78. Common `sudo` Problem

If you get permission errors while performing administrative operations, try:

```bash
--become
```

or:

```bash
-b
```

Example:

```bash
ansible-playbook playbooks/main.yml --become
```

Your remote user must have appropriate privilege-escalation permissions.

---

# 79. Understanding Ansible Output

Typical output:

```text
PLAY [Configure Linux Servers] **********************

TASK [Ensure directory exists] ***********************
changed: [web01]
ok: [web02]

TASK [Deploy configuration] **************************
changed: [web01]
ok: [web02]

PLAY RECAP *******************************************
web01 : ok=2 changed=2 failed=0
web02 : ok=2 changed=0 failed=0
```

Important states:

```text
ok
    Task ran successfully and no change was required.

changed
    Task ran successfully and changed something.

failed
    Task failed.

skipped
    Task was intentionally skipped.

unreachable
    Ansible could not connect to the host.
```

---

# 80. The Most Important Mental Model

Remember Ansible like this:

```text
                 ANSIBLE
                    |
          +---------+---------+
          |                   |
      INVENTORY            PLAYBOOK
          |                   |
       WHERE?              WHAT?
          |                   |
     Which servers?       Which tasks?
                              |
                    +---------+---------+
                    |         |         |
                 Modules   Variables  Templates
                    |
                 Actions
                    |
                Managed Node
```

And:

```text
Inventory
    ↓
WHERE?

Playbook
    ↓
WHAT?

Task
    ↓
ONE ACTION

Module
    ↓
HOW?

Variable
    ↓
WHAT VALUE?

Template
    ↓
WHAT CONFIGURATION?

Handler
    ↓
WHAT TO DO AFTER CHANGE?

Role
    ↓
HOW TO ORGANIZE EVERYTHING?
```

---

# 81. Final Beginner Mental Model

If you remember only this, remember:

```text
Ansible
    |
    +── Inventory
    |      └── Servers
    |
    +── Playbook
    |      └── Plays
    |             └── Tasks
    |                    └── Modules
    |
    +── Variables
    |
    +── Templates
    |
    +── Handlers
    |
    +── Roles
    |
    └── SSH
           └── Connection to Linux servers
```

### Ad-hoc

```text
"I need to quickly do something."
```

```bash
ansible all -m ping
```

### Playbook

```text
"I want reusable automation."
```

```bash
ansible-playbook playbooks/main.yml
```

### Template

```text
"I need to generate configuration dynamically."
```

```jinja2
{{ variable }}
```

### Handler

```text
"The configuration changed, so restart the service."
```

```yaml
notify: Restart service
```

### Role

```text
"My project is getting large, so organize the automation."
```

```text
roles/
└── nginx/
```

### Vault

```text
"I need to safely store secrets."
```

```bash
ansible-vault create secrets.yml
```

---

# 82. Golden Workflow

For most real-world Ansible work:

```bash
# 1. Check Ansible
ansible --version

# 2. Check inventory
ansible-inventory -i inventory/hosts.ini --graph

# 3. Test SSH / connectivity
ansible all -m ping

# 4. Validate playbook
ansible-playbook playbooks/main.yml --syntax-check

# 5. Dry run
ansible-playbook playbooks/main.yml --check --diff

# 6. Test on one server
ansible-playbook playbooks/main.yml --limit web01

# 7. Run against required hosts
ansible-playbook playbooks/main.yml

# 8. Verify
ansible all -m command -a "uptime"
```

This is a good default workflow to follow whenever you build or modify an Ansible automation project.
