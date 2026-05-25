# Terraform & Ansible - COMPREHENSIVE ANSWERS (All 70 Questions)

---

## Terraform Basics

**1. Terraform? Infrastructure as Code?**
Terraform: HashiCorp tool for provisioning infrastructure using declarative configuration files. IaC: Managing infrastructure through code instead of manual clicks. Benefits: version control, reproducibility, automation, review process.

**2. Terraform vs Ansible?**
| Terraform | Ansible |
|---|---|
| Infrastructure provisioning | Configuration management |
| Declarative (desired state) | Procedural (step-by-step) |
| State file tracks resources | Stateless (idempotent tasks) |
| Create VMs, networks, K8s clusters | Install software, configure servers |
| HCL language | YAML playbooks |
| Best for: cloud resources | Best for: server configuration |

**3. Declarative vs imperative?**
- **Declarative** (Terraform): "I want 3 VMs with 4GB RAM" — you describe desired state, tool figures out how.
- **Imperative** (scripts): "Create VM1, then VM2, then VM3" — you describe exact steps.

**4. Terraform providers? Name 5.**
Plugins that interact with APIs:
1. **azurerm** — Azure
2. **aws** — AWS
3. **google** — GCP
4. **kubernetes** — K8s
5. **helm** — Helm charts
6. **github** — GitHub
7. **docker** — Docker

**5. Terraform workflow?**
```bash
terraform init      # Download providers, initialize backend
terraform plan      # Preview changes (dry run)
terraform apply     # Apply changes to infrastructure
terraform destroy   # Tear down everything
```

**6. `terraform init`?**
Downloads provider plugins, initializes backend (state storage), downloads modules. Run after: new project, adding providers, changing backend.

**7. `terraform plan`?**
Shows what Terraform will do without making changes. Outputs: resources to add (+), change (~), destroy (-). **Always review plan before apply.**

**8. `terraform apply`? `-auto-approve`?**
Applies the planned changes. By default asks for confirmation. `-auto-approve` skips confirmation — use in CI/CD only, never manually.

**9. `terraform destroy`?**
Destroys ALL resources managed by current configuration. Dangerous. Ask for confirmation. Use `-target` to destroy specific resources.

**10. HCL?**
HashiCorp Configuration Language. Terraform's native config format:
```hcl
resource "azurerm_resource_group" "example" {
  name     = "my-rg"
  location = "East US"
}
```

---

## Terraform State

**11. Terraform state? Default location?**
JSON file tracking all managed resources. Maps config to real infrastructure. Default: `terraform.tfstate` in current directory.

**12. Why NOT store state in Git?**
1. Contains sensitive data (passwords, keys) in plaintext
2. Concurrent access causes conflicts/corruption
3. No locking mechanism
4. State file grows large

**13. Remote state? 3 backends.**
Store state remotely for team collaboration:
1. **Azure Storage Account** (blob container)
2. **AWS S3** (+ DynamoDB for locking)
3. **Terraform Cloud** (HCP)
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.tfstate"
  }
}
```

**14. State locking?**
Prevents concurrent modifications. When someone runs `apply`, state is locked. Others must wait. Azure: uses blob lease. AWS: uses DynamoDB table.

**15. `terraform state list`? `terraform state show`?**
```bash
terraform state list                          # List all resources in state
terraform state show azurerm_resource_group.rg # Show details of one resource
```

**16. Move resource in state?**
```bash
terraform state mv azurerm_virtual_machine.old azurerm_virtual_machine.new
# Renames resource in state (after refactoring code)
# Prevents destroy + recreate
```

**17. Import existing resources?**
```bash
terraform import azurerm_resource_group.rg /subscriptions/xxx/resourceGroups/my-rg
# Imports existing resource into state
# Must write matching config first
```

**18. Lose state file?**
Terraform loses track of all resources. Options:
1. Restore from backup (always backup state!)
2. `terraform import` each resource manually
3. Resources still exist but Terraform can't manage them

**19. State file encryption?**
- Azure Storage: encrypted at rest by default
- S3: enable server-side encryption (SSE-S3 or SSE-KMS)
- Terraform Cloud: encrypted at rest and in transit

---

## Resources & Modules

**20. Terraform resource? Azure VM example.**
```hcl
resource "azurerm_linux_virtual_machine" "example" {
  name                = "my-vm"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  size                = "Standard_DS1_v2"
  admin_username      = "adminuser"

  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  network_interface_ids = [azurerm_network_interface.nic.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
}
```

**21. Data source?**
Read-only query to fetch info about existing resources (not managed by Terraform):
```hcl
data "azurerm_resource_group" "existing" {
  name = "existing-rg"
}

# Use: data.azurerm_resource_group.existing.location
```

**22. Module? Why use?**
Reusable, encapsulated group of resources. Benefits: DRY (don't repeat), consistency, abstraction, versioning. Like a function in programming.

**23. Module directory structure?**
```
modules/
└── web-app/
    ├── main.tf           # Resource definitions
    ├── variables.tf      # Input variables
    ├── outputs.tf        # Output values
    └── README.md
```

**24. Module from Terraform Registry?**
```hcl
module "aks" {
  source  = "Azure/aks/azurerm"
  version = "7.0.0"

  resource_group_name = "my-rg"
  cluster_name        = "my-aks"
  # ...
}
```

**25. Input variables, output values, locals?**
```hcl
# variables.tf (inputs)
variable "environment" {
  type        = string
  description = "Environment name"
  default     = "dev"
}

# locals (computed values, internal)
locals {
  name_prefix = "${var.project}-${var.environment}"
}

# outputs.tf (exports)
output "cluster_id" {
  value = azurerm_kubernetes_cluster.aks.id
}
```

**26. Variable types?**
```hcl
variable "name" { type = string }
variable "count" { type = number }
variable "enabled" { type = bool }
variable "tags" { type = map(string) }
variable "subnets" { type = list(string) }
variable "config" {
  type = object({
    name    = string
    size    = number
    enabled = bool
  })
}
```

**27. Pass variables?**
```bash
# CLI
terraform apply -var="environment=prod"

# tfvars file
terraform apply -var-file="prod.tfvars"

# Environment variable
export TF_VAR_environment=prod

# Default value in variable block
variable "environment" { default = "dev" }

# terraform.tfvars (auto-loaded)
environment = "prod"
```

**28. `terraform.tfvars` vs `variables.tf`?**
- `variables.tf`: **Declares** variables (name, type, description, default)
- `terraform.tfvars`: **Assigns** values to variables. Auto-loaded.

**29. Output values?**
Export values after apply. Use for: displaying info, passing between modules, remote state data sharing.
```hcl
output "aks_cluster_name" {
  value       = azurerm_kubernetes_cluster.aks.name
  description = "The AKS cluster name"
}
```

**30. `depends_on`?**
Explicit dependency when Terraform can't infer it:
```hcl
resource "azurerm_kubernetes_cluster" "aks" {
  # ...
  depends_on = [azurerm_role_assignment.acr_pull]
  # AKS needs the role assignment first
}
```

---

## Advanced Terraform

**31. Provisioners? Why discouraged?**
Execute scripts on resources after creation. Discouraged because: not declarative, can't detect drift, error handling is poor. Use instead: cloud-init, Ansible, Packer.
```hcl
provisioner "remote-exec" {
  inline = ["sudo apt update"]
}
```

**32. `count` vs `for_each`?**
```hcl
# count: create N identical resources (by index)
resource "azurerm_resource_group" "rg" {
  count    = 3
  name     = "rg-${count.index}"
  location = "East US"
}

# for_each: create resources from a map/set (by key)
resource "azurerm_resource_group" "rg" {
  for_each = toset(["dev", "staging", "prod"])
  name     = "rg-${each.key}"
  location = "East US"
}
```
**Prefer `for_each`**: removing middle item in `count` shifts all indexes.

**33. Dynamic blocks?**
Generate repeated nested blocks:
```hcl
dynamic "ingress" {
  for_each = var.ingress_rules
  content {
    from_port   = ingress.value.port
    to_port     = ingress.value.port
    protocol    = "tcp"
    cidr_blocks = ingress.value.cidrs
  }
}
```

**34. Terraform workspace?**
Isolated state within same config. Use for: dev/staging/prod with same code.
```bash
terraform workspace new dev
terraform workspace new prod
terraform workspace select dev
terraform workspace list
```
Alternative: separate directories per environment (more common in practice).

**35. Sensitive variables?**
```hcl
variable "db_password" {
  type      = string
  sensitive = true    # Hidden in plan/apply output
}
```
Still stored in state file. Encrypt state backend.

**36. Lifecycle block?**
```hcl
lifecycle {
  create_before_destroy = true    # Create replacement before destroying old
  prevent_destroy       = true    # Prevent accidental deletion
  ignore_changes        = [tags]  # Don't track tag changes
}
```

**37. Handle Terraform drift?**
Drift = someone changes infrastructure outside Terraform.
1. `terraform plan` detects drift (shows changes to bring back to desired state)
2. `terraform apply` fixes drift
3. `terraform refresh` updates state to match reality (without changing infra)
4. Prevent: restrict manual access, use CI/CD only

**38. `terraform taint`? Replacement?**
`terraform taint` (deprecated) → use `terraform apply -replace`:
```bash
terraform apply -replace="azurerm_linux_virtual_machine.vm"
# Forces destroy + recreate of specific resource
```

**39. `terraform refresh`?**
Updates state file to match actual infrastructure. Doesn't change resources. Now built into `plan` and `apply` automatically.

**40. Multiple environments?**
| Approach | Pros | Cons |
|---|---|---|
| **Workspaces** | Same code, different state | Implicit, easy to confuse |
| **Directories** | Explicit separation | Code duplication |
| **Terragrunt** | DRY + directories | Extra tool |
| **tfvars per env** | One codebase, different values | Shared state risk |

---

## Ansible Basics

**41. Ansible? Use cases?**
Agentless automation tool for: configuration management, application deployment, orchestration, provisioning. Push-based over SSH.

**42. Ansible vs Terraform?**
- **Terraform**: Creates infrastructure (VMs, networks, K8s). Declarative. State-based.
- **Ansible**: Configures infrastructure (install packages, deploy apps). Procedural. Stateless.
- Often used together: Terraform creates, Ansible configures.

**43. Agentless? How connects?**
No software needed on target machines. Connects via: SSH (Linux), WinRM (Windows). Push model: control node pushes to targets.

**44. Inventory?**
List of target hosts:
```ini
# Static inventory
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com

[all:vars]
ansible_user=admin
```
**Dynamic inventory**: Script/plugin that queries cloud API (AWS, Azure) for hosts.

**45. Playbook? Play?**
```yaml
# Playbook: YAML file with one or more plays
---
- name: Configure web servers     # Play 1
  hosts: webservers
  become: yes
  tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present

- name: Configure databases       # Play 2
  hosts: databases
  tasks:
  - name: Install PostgreSQL
    apt:
      name: postgresql
      state: present
```

**46. Task? Module?**
- **Task**: Single action (install package, copy file, start service)
- **Module**: Plugin that performs the action (apt, copy, service, template)

**47. 10 Ansible modules?**
1. `apt`/`yum` — Package management
2. `copy` — Copy files
3. `template` — Jinja2 template rendering
4. `service`/`systemd` — Manage services
5. `file` — File/directory permissions
6. `user` — User management
7. `command`/`shell` — Run commands
8. `docker_container` — Manage Docker
9. `git` — Git operations
10. `lineinfile` — Edit lines in files

**48. `ansible.cfg`?**
Configuration file. Priority: env var → ./ansible.cfg → ~/.ansible.cfg → /etc/ansible/ansible.cfg
```ini
[defaults]
inventory = ./inventory
remote_user = admin
host_key_checking = False
retry_files_enabled = False
```

**49. Handlers?**
Tasks triggered by `notify`. Run once at end of play, even if notified multiple times:
```yaml
tasks:
- name: Update nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

handlers:
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

**50. `copy` vs `template`?**
- `copy`: Copies file as-is from control node to target
- `template`: Processes Jinja2 templates (variable substitution) before copying
```yaml
- template:
    src: config.j2
    dest: /etc/app/config.yml
# config.j2: server_name: {{ server_name }}
```

---

## Ansible Advanced

**51. Roles? Directory structure?**
Reusable, organized collection of tasks, variables, templates:
```
roles/
└── webserver/
    ├── tasks/main.yml        # Tasks
    ├── handlers/main.yml     # Handlers
    ├── templates/            # Jinja2 templates
    ├── files/                # Static files
    ├── vars/main.yml         # Variables
    ├── defaults/main.yml     # Default variables (lowest priority)
    ├── meta/main.yml         # Dependencies
    └── README.md
```

**52. Ansible Galaxy?**
Community hub for sharing roles:
```bash
ansible-galaxy install geerlingguy.docker    # Install role
ansible-galaxy init myrole                    # Create role scaffold
ansible-galaxy collection install community.docker
```

**53. Variable precedence?**
(Low → High): role defaults → inventory vars → playbook vars → role vars → extra vars (`-e`)
```bash
ansible-playbook site.yml -e "env=production"  # Highest priority
```

**54. Jinja2 templating?**
```jinja2
# config.j2
server_name: {{ server_name }}
port: {{ app_port | default(8080) }}
{% if enable_ssl %}
ssl_certificate: /etc/ssl/cert.pem
{% endif %}
{% for host in db_hosts %}
  - {{ host }}:5432
{% endfor %}
```

**55. Facts? Custom facts?**
Auto-gathered system info (OS, IP, CPU, memory):
```yaml
- debug:
    msg: "OS: {{ ansible_os_family }}, IP: {{ ansible_default_ipv4.address }}"
```
Custom facts: place files in `/etc/ansible/facts.d/` on target.

**56. Conditionals?**
```yaml
- name: Install on Debian
  apt:
    name: nginx
  when: ansible_os_family == "Debian"

- name: Install on RedHat
  yum:
    name: nginx
  when: ansible_os_family == "RedHat"

- name: Only if variable is defined
  debug:
    msg: "{{ custom_var }}"
  when: custom_var is defined
```

**57. Loops?**
```yaml
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - vim

- name: Create users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: 'alice', groups: 'developers' }
    - { name: 'bob', groups: 'admins' }
```

**58. Ansible Vault?**
Encrypt sensitive data:
```bash
ansible-vault create secrets.yml       # Create encrypted file
ansible-vault edit secrets.yml         # Edit
ansible-vault encrypt vars.yml         # Encrypt existing file
ansible-playbook site.yml --ask-vault-pass    # Run with vault
ansible-playbook site.yml --vault-password-file=~/.vault_pass  # Auto
```

**59. Tags?**
Run specific tasks:
```yaml
- name: Install nginx
  apt: name=nginx
  tags: [install, nginx]

- name: Configure nginx
  template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
  tags: [configure, nginx]
```
```bash
ansible-playbook site.yml --tags "configure"
ansible-playbook site.yml --skip-tags "install"
```

**60. Test Ansible playbooks?**
- **Molecule**: Testing framework. Creates test instances, runs playbook, verifies.
- **ansible-lint**: Checks for best practices and errors.
- `--check` mode: Dry run (no changes).
- `--diff` mode: Shows file changes.
```bash
ansible-lint site.yml
ansible-playbook site.yml --check --diff
molecule test
```

---

## Interview-Style

**61. Terraform: VPC + Subnet + SG + EC2?**
```hcl
provider "aws" { region = "us-east-1" }

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags       = { Name = "main-vpc" }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

resource "aws_security_group" "web" {
  vpc_id = aws_vpc.main.id
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t3.micro"
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]
  tags                   = { Name = "web-server" }
}
```

**62. Terraform: Resource Group + AKS on Azure?**
```hcl
resource "azurerm_resource_group" "rg" {
  name     = "aks-rg"
  location = "East US"
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = "my-aks"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "myaks"

  default_node_pool {
    name       = "default"
    node_count = 3
    vm_size    = "Standard_DS2_v2"
  }

  identity {
    type = "SystemAssigned"
  }
}

output "kube_config" {
  value     = azurerm_kubernetes_cluster.aks.kube_config_raw
  sensitive = true
}
```

**63. Reusable Terraform module?**
```hcl
# modules/web-app/main.tf
variable "name" { type = string }
variable "environment" { type = string }
variable "instance_type" { type = string; default = "t3.micro" }

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  tags          = { Name = "${var.name}-${var.environment}" }
}

output "public_ip" { value = aws_instance.web.public_ip }

# Usage:
module "staging" {
  source       = "./modules/web-app"
  name         = "myapp"
  environment  = "staging"
}
```

**64. Ansible: Install Docker, pull image, run container?**
```yaml
---
- name: Deploy Docker container
  hosts: servers
  become: yes
  tasks:
  - name: Install Docker dependencies
    apt:
      name: [apt-transport-https, ca-certificates, curl]
      state: present

  - name: Install Docker
    apt:
      name: docker.io
      state: present

  - name: Start Docker service
    service:
      name: docker
      state: started
      enabled: yes

  - name: Pull application image
    docker_image:
      name: myapp
      tag: latest
      source: pull

  - name: Run application container
    docker_container:
      name: myapp
      image: myapp:latest
      ports:
        - "8080:8080"
      restart_policy: always
```

**65. Ansible: Prometheus + Grafana setup?**
```yaml
---
- name: Setup Monitoring Stack
  hosts: monitoring
  become: yes
  roles:
    - prometheus
    - grafana
  tasks:
  - name: Create Prometheus config
    template:
      src: prometheus.yml.j2
      dest: /etc/prometheus/prometheus.yml
    notify: restart prometheus

  - name: Run Prometheus container
    docker_container:
      name: prometheus
      image: prom/prometheus:latest
      ports: ["9090:9090"]
      volumes:
        - /etc/prometheus:/etc/prometheus

  - name: Run Grafana container
    docker_container:
      name: grafana
      image: grafana/grafana:latest
      ports: ["3000:3000"]
      env:
        GF_SECURITY_ADMIN_PASSWORD: "{{ grafana_password }}"
```

**66. State management in team of 10?**
1. Remote backend (Azure Storage / S3) with state locking
2. Separate state files per environment (dev.tfstate, prod.tfstate)
3. CI/CD pipeline runs plan/apply (not individual devs)
4. PR-based workflow: plan in PR, apply on merge
5. Access controls on state storage
6. Regular state backups

**67. `terraform apply` fails halfway?**
State is partially updated. Resources created before failure ARE in state.
1. Fix the error in config
2. Run `terraform plan` to see current state
3. Run `terraform apply` again — Terraform picks up where it left off
4. It won't re-create already-created resources
5. If state is corrupted: `terraform state rm` problematic resource

**68. Blue-green deployment with Terraform?**
```hcl
# Two identical environments
variable "active" { default = "blue" }

module "blue" {
  source = "./modules/app"
  name   = "blue"
}

module "green" {
  source = "./modules/app"
  name   = "green"
}

# Switch LB to point to active environment
resource "aws_lb_target_group_attachment" "active" {
  target_group_arn = aws_lb_target_group.main.arn
  target_id        = var.active == "blue" ? module.blue.instance_id : module.green.instance_id
}
# Deploy new version to inactive → test → switch active
```

**69. IaC setup at your organization?**
"We use Terraform for Azure infrastructure (AKS, networking, Key Vault, ACR). State stored in Azure Storage with locking. Modules for reusable components. PR-based workflow: plan runs on PR, apply on merge to main. Separate workspaces for dev/staging/prod. Pipeline runs tfsec for security scanning. Ansible used for VM configuration where needed."

**70. Test IaC before production?**
1. `terraform plan` review
2. `terraform validate` syntax check
3. `tfsec` / `checkov` security scanning
4. `terratest` (Go-based integration tests)
5. Apply to dev/staging first
6. PR review by team
7. `terraform plan` output in PR comments
8. Canary deployments for critical infrastructure changes
