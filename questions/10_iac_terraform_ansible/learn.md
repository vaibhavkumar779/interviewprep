# Infrastructure as Code (Terraform & Ansible) — Deep-Dive Learning Guide

---

## 1. IaC Overview

```
┌─── Without IaC (manual) ────────────────────────────────────┐
│  Click through portal → inconsistent → no audit trail       │
│  "It works on my infra" → snowflake servers                 │
└──────────────────────────────────────────────────────────────┘

┌─── With IaC ────────────────────────────────────────────────┐
│  Code → version controlled → reviewed → tested → applied   │
│  Reproducible, consistent, auditable, self-documenting     │
└──────────────────────────────────────────────────────────────┘

┌─── Two Approaches ──────────────────────────────────────────┐
│                                                              │
│  Declarative (WHAT)              Imperative (HOW)           │
│  "I want 3 VMs"                  "Create VM1, Create VM2,   │
│  Tool figures out HOW             Create VM3"                │
│                                                              │
│  Terraform, CloudFormation       Ansible, Chef, Puppet      │
│  Kubernetes manifests            Shell scripts               │
│                                                              │
│  Idempotent by design            Must code idempotency      │
└──────────────────────────────────────────────────────────────┘
```

### Terraform vs Ansible — When to Use What

```
┌─── Terraform ──────────────┐    ┌─── Ansible ─────────────────┐
│  PROVISION infrastructure  │    │  CONFIGURE what's on infra   │
│                            │    │                               │
│  Create VMs, networks,     │    │  Install packages, copy      │
│  load balancers, databases,│    │  configs, start services,    │
│  DNS, storage accounts     │    │  deploy apps, patch OS       │
│                            │    │                               │
│  Declarative (desired state)│   │  Procedural (step by step)   │
│  State file tracks reality │    │  Agentless (SSH/WinRM)       │
│  HCL language              │    │  YAML playbooks              │
│  Plan → Apply workflow     │    │  Push-based execution        │
└────────────────────────────┘    └───────────────────────────────┘

Common pattern:  Terraform creates VMs → Ansible configures them
```

---

## 2. Terraform Architecture

```
┌─── Terraform Workflow ──────────────────────────────────────┐
│                                                              │
│  1. Write    →  .tf files (HCL configuration)               │
│  2. Init     →  terraform init (download providers)         │
│  3. Plan     →  terraform plan (preview changes)            │
│  4. Apply    →  terraform apply (execute changes)           │
│  5. Destroy  →  terraform destroy (tear down)               │
│                                                              │
└──────────────────────────────────────────────────────────────┘

┌─── How Terraform Works Internally ──────────────────────────┐
│                                                              │
│  .tf files (desired state)                                  │
│       │                                                      │
│       ▼                                                      │
│  Terraform Core                                              │
│       │                                                      │
│  ┌────┴──────┐                                              │
│  │State File │  ← records what Terraform has ACTUALLY created│
│  │(.tfstate) │  ← maps resources to real IDs                │
│  └────┬──────┘                                              │
│       │                                                      │
│       ▼  DIFF: desired vs actual                            │
│  ┌─────────────┐                                            │
│  │   Plan      │  ← "I need to create 2 VMs, delete 1 LB" │
│  └─────┬───────┘                                            │
│        │                                                     │
│        ▼  Apply                                              │
│  ┌─────────────┐                                            │
│  │  Providers  │  ← API calls to cloud                      │
│  │  (azurerm,  │                                            │
│  │   aws, gcp) │                                            │
│  └─────────────┘                                            │
└──────────────────────────────────────────────────────────────┘
```

### State File — Critical Concept

```
terraform.tfstate contains:
  - Every resource Terraform manages
  - Resource IDs (Azure resource ID, AWS ARN, etc.)
  - All attributes (IP, DNS name, etc.)
  - Dependencies between resources

NEVER:
  ❌ Edit state manually
  ❌ Store state in Git (contains secrets!)
  ❌ Run terraform from multiple machines with local state

ALWAYS:
  ✅ Use remote backend (Azure Blob, S3, Terraform Cloud)
  ✅ Enable state locking (prevent concurrent modifications)
  ✅ Enable encryption at rest
```

```hcl
# Remote backend configuration
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "tfstatestore"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

---

## 3. Terraform — HCL Deep Dive

```hcl
# ─── Provider ───
terraform {
  required_version = ">= 1.5"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"       # Any 3.x version
    }
  }
}

provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

# ─── Variables ───
variable "environment" {
  type        = string
  description = "Environment name"
  default     = "staging"
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

variable "vm_count" {
  type    = number
  default = 3
}

variable "tags" {
  type = map(string)
  default = {
    managed_by = "terraform"
    team       = "devops"
  }
}

# ─── Locals ───
locals {
  name_prefix = "${var.environment}-myapp"
  common_tags = merge(var.tags, {
    environment = var.environment
  })
}

# ─── Resource ───
resource "azurerm_resource_group" "main" {
  name     = "${local.name_prefix}-rg"
  location = "East US"
  tags     = local.common_tags
}

resource "azurerm_virtual_network" "main" {
  name                = "${local.name_prefix}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  tags                = local.common_tags
}

# ─── Count (create multiple) ───
resource "azurerm_linux_virtual_machine" "web" {
  count               = var.vm_count
  name                = "${local.name_prefix}-web-${count.index}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B2s"
  admin_username      = "azureuser"
  # ...
}

# ─── for_each (create from map/set) ───
variable "environments" {
  default = {
    dev     = { size = "Standard_B1s", count = 1 }
    staging = { size = "Standard_B2s", count = 2 }
    prod    = { size = "Standard_D4s_v3", count = 5 }
  }
}

resource "azurerm_linux_virtual_machine" "env" {
  for_each            = var.environments
  name                = "vm-${each.key}"
  size                = each.value.size
  # ...
}

# ─── Data source (read existing resource) ───
data "azurerm_key_vault" "existing" {
  name                = "mykeyvault"
  resource_group_name = "rg-shared"
}

# ─── Output ───
output "resource_group_id" {
  value       = azurerm_resource_group.main.id
  description = "The ID of the resource group"
}

output "vm_ips" {
  value = [for vm in azurerm_linux_virtual_machine.web : vm.private_ip_address]
}
```

---

## 4. Terraform Modules

```
modules/
├── networking/
│   ├── main.tf          # Resources: VNet, subnets, NSGs
│   ├── variables.tf     # Input variables
│   ├── outputs.tf       # Output values
│   └── README.md
├── compute/
│   ├── main.tf          # Resources: VMs, scale sets
│   ├── variables.tf
│   └── outputs.tf
└── database/
    ├── main.tf          # Resources: SQL server, DB
    ├── variables.tf
    └── outputs.tf

environments/
├── dev/
│   └── main.tf          # Uses modules with dev values
├── staging/
│   └── main.tf
└── prod/
    └── main.tf
```

```hcl
# environments/prod/main.tf
module "networking" {
  source         = "../../modules/networking"
  environment    = "prod"
  vnet_cidr      = "10.0.0.0/16"
  subnet_count   = 4
}

module "compute" {
  source         = "../../modules/compute"
  environment    = "prod"
  subnet_id      = module.networking.subnet_ids[0]   # Use output from networking
  vm_count       = 5
  vm_size        = "Standard_D4s_v3"
}
```

---

## 5. Terraform Key Commands

```bash
terraform init              # Download providers + modules, init backend
terraform plan              # Preview changes (dry run)
terraform plan -out=plan.tf # Save plan for exact apply
terraform apply             # Apply changes (with confirmation)
terraform apply plan.tf     # Apply saved plan (no confirmation)
terraform destroy           # Destroy ALL managed resources

terraform fmt               # Format .tf files
terraform validate          # Syntax validation
terraform state list        # List resources in state
terraform state show <res>  # Show resource details
terraform import <res> <id> # Import existing resource into state
terraform taint <res>       # Mark for recreation on next apply
terraform output            # Show all outputs

# ─── Workspaces (multiple environments, same code) ───
terraform workspace new staging
terraform workspace select prod
terraform workspace list
```

---

## 6. Ansible Architecture

```
┌─── Ansible Architecture ───────────────────────────────────┐
│                                                             │
│  Control Node (your laptop / CI agent)                      │
│  ┌────────────────────────────────────────────────────────┐│
│  │  ansible / ansible-playbook                            ││
│  │  ┌──────────┐  ┌───────────┐  ┌─────────────────────┐││
│  │  │Inventory │  │ Playbook  │  │ Roles / Collections │││
│  │  │(hosts)   │  │ (tasks)   │  │ (reusable units)    │││
│  │  └──────────┘  └───────────┘  └─────────────────────┘││
│  └──────────────────────────┬─────────────────────────────┘│
│                             │ SSH / WinRM (agentless!)      │
│                             │ (no agent to install!)        │
│  ┌──────────────────────────▼─────────────────────────────┐│
│  │  Managed Nodes (targets)                                ││
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐               ││
│  │  │ web1    │  │ web2    │  │ db1     │               ││
│  │  │ (Linux) │  │ (Linux) │  │ (Linux) │               ││
│  │  └─────────┘  └─────────┘  └─────────┘               ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘

Key difference from Terraform:
  - Agentless (uses SSH)
  - Push-based (you run it, it pushes config to nodes)
  - Procedural (tasks run in order)
  - Idempotent modules (most modules check before acting)
```

---

## 7. Ansible — Inventory

```ini
# inventory/hosts.ini

[webservers]
web1.example.com ansible_user=deploy
web2.example.com ansible_user=deploy
web3.example.com ansible_user=deploy

[databases]
db1.example.com ansible_user=admin ansible_port=2222

[staging:children]
webservers
databases

[staging:vars]
env=staging
log_level=debug

[production]
prod-web[1:5].example.com    # prod-web1 through prod-web5
```

```yaml
# Dynamic inventory (YAML format)
# inventory/hosts.yml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_user: deploy
          http_port: 8080
        web2.example.com:
          ansible_user: deploy
    databases:
      hosts:
        db1.example.com:
      vars:
        db_port: 5432
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

---

## 8. Ansible — Playbooks

```yaml
# deploy.yml
---
- name: Deploy web application
  hosts: webservers
  become: yes                    # Run as root (sudo)
  vars:
    app_version: "2.1.0"
    app_dir: /opt/myapp
    app_user: appuser

  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

  tasks:
    - name: Create app user
      user:
        name: "{{ app_user }}"
        system: yes
        shell: /usr/sbin/nologin

    - name: Install dependencies
      apt:
        name:
          - nginx
          - python3
          - python3-pip
        state: present

    - name: Create app directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ app_user }}"
        mode: '0755'

    - name: Copy application code
      copy:
        src: files/app/
        dest: "{{ app_dir }}/"
        owner: "{{ app_user }}"
      notify: Restart app          # Trigger handler on change

    - name: Deploy nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/myapp
      notify: Reload nginx

    - name: Enable nginx site
      file:
        src: /etc/nginx/sites-available/myapp
        dest: /etc/nginx/sites-enabled/myapp
        state: link
      notify: Reload nginx

    - name: Ensure app service is running
      systemd:
        name: myapp
        state: started
        enabled: yes

    - name: Wait for app to be healthy
      uri:
        url: "http://localhost:8080/health"
        status_code: 200
      register: health
      until: health.status == 200
      retries: 10
      delay: 5

  handlers:
    - name: Restart app
      systemd:
        name: myapp
        state: restarted

    - name: Reload nginx
      systemd:
        name: nginx
        state: reloaded

  post_tasks:
    - name: Send notification
      uri:
        url: "https://hooks.slack.com/services/xxx"
        method: POST
        body_format: json
        body:
          text: "Deployed v{{ app_version }} to {{ inventory_hostname }}"
```

---

## 9. Ansible — Roles

```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yml          # Main task list
    ├── handlers/
    │   └── main.yml          # Event handlers
    ├── templates/
    │   └── nginx.conf.j2     # Jinja2 templates
    ├── files/
    │   └── ssl-cert.pem      # Static files
    ├── vars/
    │   └── main.yml          # Role variables
    ├── defaults/
    │   └── main.yml          # Default values (lowest priority)
    ├── meta/
    │   └── main.yml          # Dependencies, metadata
    └── README.md
```

```yaml
# Using roles in playbook:
- hosts: webservers
  roles:
    - common                  # roles/common/
    - role: webserver         # roles/webserver/
      vars:
        http_port: 8080
    - role: monitoring
      when: enable_monitoring | bool
```

---

## 10. Ansible Key Modules

| Module | Purpose | Example |
|--------|---------|---------|
| `apt/yum/dnf` | Package management | `apt: name=nginx state=present` |
| `copy` | Copy files to remote | `copy: src=app.conf dest=/etc/` |
| `template` | Render Jinja2 template | `template: src=conf.j2 dest=/etc/` |
| `file` | File/directory permissions | `file: path=/opt state=directory` |
| `service/systemd` | Manage services | `systemd: name=nginx state=started` |
| `user/group` | User management | `user: name=deploy groups=docker` |
| `command/shell` | Run commands | `command: /opt/scripts/deploy.sh` |
| `git` | Clone/pull repos | `git: repo=URL dest=/opt/app` |
| `docker_container` | Manage containers | `docker_container: name=web image=nginx` |
| `uri` | HTTP requests | `uri: url=http://localhost/health` |
| `lineinfile` | Edit single line in file | `lineinfile: path=/etc/hosts line=...` |
| `cron` | Manage cron jobs | `cron: name=backup job=/opt/backup.sh` |

---

## 11. Jinja2 Templates

```jinja2
{# templates/nginx.conf.j2 #}
server {
    listen {{ http_port | default(80) }};
    server_name {{ server_name }};

    {% if ssl_enabled %}
    listen 443 ssl;
    ssl_certificate /etc/ssl/{{ domain }}.crt;
    ssl_certificate_key /etc/ssl/{{ domain }}.key;
    {% endif %}

    {% for upstream in app_servers %}
    upstream backend {
        server {{ upstream }}:{{ app_port }};
    }
    {% endfor %}

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 12. Ansible Key Commands

```bash
# Run playbook
ansible-playbook deploy.yml -i inventory/hosts.ini

# Limit to specific hosts
ansible-playbook deploy.yml -l web1.example.com

# Dry run (check mode)
ansible-playbook deploy.yml --check --diff

# Extra variables
ansible-playbook deploy.yml -e "app_version=2.1.0 env=prod"

# Vault (encrypt secrets)
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-playbook deploy.yml --ask-vault-pass
ansible-playbook deploy.yml --vault-password-file=.vault_pass

# Ad-hoc commands
ansible all -m ping                           # Test connectivity
ansible webservers -m shell -a "uptime"       # Run command
ansible webservers -m apt -a "name=nginx state=latest" -b  # Install
```

---

## 13. Terraform + Ansible Together

```
┌─── Common Pattern ──────────────────────────────────────────┐
│                                                              │
│  Step 1: Terraform provisions infrastructure                │
│    - VMs, networks, load balancers, DNS                     │
│    - Outputs: VM IPs, DNS names, resource IDs               │
│                                                              │
│  Step 2: Terraform generates Ansible inventory              │
│    - Dynamic inventory from terraform output                │
│                                                              │
│  Step 3: Ansible configures the VMs                         │
│    - Install packages, deploy apps, configure services      │
│                                                              │
│  Pipeline:                                                   │
│    terraform apply → generate inventory → ansible-playbook  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```hcl
# Terraform: output IPs for Ansible
resource "local_file" "ansible_inventory" {
  content = templatefile("inventory.tpl", {
    web_servers = azurerm_linux_virtual_machine.web[*].private_ip_address
    db_servers  = [azurerm_linux_virtual_machine.db.private_ip_address]
  })
  filename = "${path.module}/inventory.ini"
}
```

---

## 14. Terraform Lifecycle Meta-Arguments

```hcl
resource "azurerm_linux_virtual_machine" "web" {
  name = "web-server"
  # ...

  lifecycle {
    # Create new resource BEFORE destroying old one (zero downtime)
    create_before_destroy = true

    # Prevent accidental deletion (must remove this to destroy)
    prevent_destroy = true

    # Ignore changes to tags (managed externally, e.g. Azure Policy)
    ignore_changes = [tags, custom_data]

    # Replace when any of these change (force recreation)
    replace_triggered_by = [null_resource.trigger.id]
  }
}
```

---

## 15. Terraform Import & Moved Blocks

```hcl
# import block (Terraform 1.5+) — bring existing resources under management
import {
  to = azurerm_resource_group.main
  id = "/subscriptions/xxx/resourceGroups/my-rg"
}
# Run: terraform plan -generate-config-out=generated.tf
# Generates HCL config for the imported resource

# moved block (Terraform 1.1+) — rename/refactor without destroy+create
moved {
  from = azurerm_linux_virtual_machine.web
  to   = module.compute.azurerm_linux_virtual_machine.web
}
# Terraform moves state entry — no infrastructure change!
```

---

## 16. Terraform Provisioners (and Why to Avoid Them)

```hcl
resource "azurerm_linux_virtual_machine" "web" {
  # ...

  # Runs ONCE after resource creation
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx"
    ]
    connection {
      type = "ssh"
      user = "adminuser"
      host = self.public_ip_address
    }
  }

  # local-exec runs on the machine running Terraform
  provisioner "local-exec" {
    command = "ansible-playbook -i '${self.public_ip_address},' setup.yml"
  }
}

# ❌ Why to avoid provisioners:
#   - Not in state → Terraform can't track or update
#   - Not idempotent → may fail on re-apply
#   - Breaks declarative model
# ✅ Better alternatives:
#   - Use cloud-init/user_data for VM bootstrap
#   - Use Ansible for configuration management
#   - Use Packer to bake images with everything pre-installed
```

---

## 17. Ansible Roles

```bash
# Role = reusable, self-contained unit of automation
ansible-galaxy init myrole
# Creates structure:
# myrole/
# ├── defaults/main.yml     # Default variables (lowest priority)
# ├── files/                 # Static files to copy
# ├── handlers/main.yml     # Triggered actions (e.g., restart nginx)
# ├── meta/main.yml          # Dependencies, metadata
# ├── tasks/main.yml         # Main task list
# ├── templates/             # Jinja2 templates
# ├── tests/                 # Test playbook
# └── vars/main.yml          # Variables (higher priority than defaults)
```

```yaml
# roles/nginx/tasks/main.yml
- name: Install nginx
  apt:
    name: nginx
    state: present
  notify: restart nginx

- name: Deploy config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

# roles/nginx/handlers/main.yml
- name: restart nginx
  service:
    name: nginx
    state: restarted

# roles/nginx/defaults/main.yml
nginx_port: 80
nginx_worker_connections: 1024
```

```yaml
# Using roles in a playbook
- hosts: webservers
  become: yes
  roles:
    - common
    - nginx
    - { role: app, tags: ['app'], app_version: '2.1.0' }
```

---

## 18. Ansible Vault

```bash
# Encrypt sensitive data
ansible-vault create secrets.yml
# Editor opens → add secrets:
# db_password: SuperSecret123
# api_key: abcdef123456

# Encrypt existing file
ansible-vault encrypt vars/production.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Decrypt file
ansible-vault decrypt secrets.yml

# Encrypt single string (inline in YAML)
ansible-vault encrypt_string 'SuperSecret123' --name 'db_password'
# Output:
# db_password: !vault |
#   $ANSIBLE_VAULT;1.1;AES256
#   6562663366...

# Use in playbook
ansible-playbook deploy.yml --ask-vault-pass
ansible-playbook deploy.yml --vault-password-file=~/.vault_pass
```

---

## 19. Ansible Galaxy & Dynamic Inventory

```bash
# Galaxy — community role repository
ansible-galaxy install geerlingguy.docker
ansible-galaxy install -r requirements.yml

# requirements.yml
roles:
  - name: geerlingguy.docker
    version: 7.1.0
  - name: geerlingguy.certbot

collections:
  - name: community.docker
    version: 3.4.0
```

```yaml
# Dynamic Inventory — auto-discover hosts from cloud
# azure_rm.yml
plugin: azure.azcollection.azure_rm
auth_source: auto
include_vm_resource_groups:
  - production-rg
keyed_groups:
  - key: tags.role     # Group by Azure tag "role"
    prefix: tag
  - key: location      # Group by Azure region
    prefix: region

# Usage:
ansible-playbook -i azure_rm.yml deploy.yml
# Automatically discovers VMs, no static inventory file needed
```

---

## 20. Molecule Testing for Ansible Roles

```bash
# Molecule = test framework for Ansible roles
pip install molecule molecule-docker

# Initialize tests for a role
cd roles/nginx
molecule init scenario -d docker

# molecule/default/molecule.yml
driver:
  name: docker
platforms:
  - name: instance
    image: ubuntu:22.04
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible

# molecule/default/verify.yml
- name: Verify nginx
  hosts: all
  tasks:
    - name: Check nginx is installed
      command: nginx -v
      register: result
      failed_when: result.rc != 0

    - name: Check nginx is running
      service:
        name: nginx
        state: started
      check_mode: true

# Run tests:
molecule test
# Creates container → runs role → runs verify → destroys container
```
