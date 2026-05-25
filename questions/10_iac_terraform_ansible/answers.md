# Terraform & Ansible — COMPREHENSIVE ANSWERS (All 70+ Questions)

---

## Terraform Basics

**1. Terraform? Infrastructure as Code?**

Terraform is HashiCorp's tool for provisioning infrastructure using **declarative** configuration files (HCL). IaC = managing infrastructure through version-controlled code instead of manual clicks.

```
┌─── Without IaC ──────────────────┬─── With IaC (Terraform) ──────────┐
│                                   │                                    │
│  Click in Azure portal            │  Write HCL code                   │
│  Manual, error-prone              │  Version-controlled (Git)          │
│  "Who changed this?"              │  PR review + audit trail           │
│  Impossible to reproduce          │  Reproducible across envs          │
│  No rollback                      │  Git revert = rollback             │
│  Tribal knowledge                 │  Self-documenting                  │
└───────────────────────────────────┴────────────────────────────────────┘
```

---

**2. Terraform vs Ansible?**

```
┌─── Terraform ──────────────────────┬─── Ansible ─────────────────────┐
│ "Create the infrastructure"        │ "Configure the infrastructure"  │
│                                    │                                  │
│  Provisions: VMs, networks, K8s,  │  Configures: install packages,  │
│  databases, load balancers         │  deploy apps, edit config files  │
│                                    │                                  │
│  Declarative (desired state)       │  Procedural (step-by-step)      │
│  State file tracks everything      │  Stateless (idempotent tasks)   │
│  HCL language                      │  YAML playbooks                  │
│  Providers (API plugins)           │  Agentless (SSH/WinRM)          │
│                                    │                                  │
│  Day 0: Build infrastructure      │  Day 1+: Manage software         │
└────────────────────────────────────┴──────────────────────────────────┘

Best practice: Use BOTH together
  Terraform creates VM → Ansible configures it
```

| Feature | Terraform | Ansible |
|---------|-----------|---------|
| Purpose | Infrastructure provisioning | Configuration management |
| Approach | Declarative | Procedural (imperative) |
| State | State file (tracks resources) | Stateless |
| Language | HCL | YAML |
| Agent | Provider plugins (API calls) | Agentless (SSH) |
| Idempotent | Yes (by design) | Yes (if modules used correctly) |
| Best for | Cloud resources | Server configuration |

---

**3. Declarative vs imperative?**

```
Declarative (Terraform):             Imperative (Shell script):
┌─────────────────────────┐          ┌─────────────────────────┐
│ "I want 3 VMs with      │          │ "Create VM1"            │
│  4GB RAM in East US"    │          │ "Create VM2"            │
│                          │          │ "Create VM3"            │
│ Terraform figures out    │          │ "Set each to 4GB"       │
│ HOW to get there.        │          │ "Place in East US"      │
│                          │          │                         │
│ Already have 2 VMs?      │          │ Already have 2 VMs?     │
│ → creates only 1 more    │          │ → script creates 3 MORE │
│                          │          │   (total 5!) ❌         │
│ Idempotent ✅            │          │ Not idempotent ❌       │
└─────────────────────────┘          └─────────────────────────┘
```

---

**4. Terraform providers? Name 5.**

Plugins that interact with cloud/service APIs:

```
Terraform Core ──► Provider ──► Cloud API
                   (plugin)

Common providers:
  azurerm     → Azure
  aws         → AWS
  google      → GCP
  kubernetes  → K8s
  helm        → Helm charts
  github      → GitHub repos/actions
  docker      → Docker containers
  random      → Random values
  null        → Null operations (triggers)
  local       → Local files
```

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"        # >= 3.0, < 4.0 (pessimistic constraint)
    }
  }
}

provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}
```

---

**5. Terraform workflow?**

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Write Code ──► terraform init ──► terraform plan            │
│                    │                     │                    │
│              Download providers    Preview changes            │
│              Init backend          + (add), ~ (change),      │
│              Download modules      - (destroy)               │
│                                         │                    │
│                                  terraform apply             │
│                                    │                         │
│                              Execute changes                 │
│                              Update state file               │
│                                    │                         │
│                              terraform destroy               │
│                                    │                         │
│                              Tear down everything            │
│                                                              │
│  Also:                                                       │
│  terraform fmt       → format code                           │
│  terraform validate  → check syntax                          │
│  terraform console   → interactive expression testing        │
│  terraform graph     → dependency visualization              │
│  terraform output    → show output values                    │
└──────────────────────────────────────────────────────────────┘
```

```bash
terraform init          # Download providers + modules, init backend
terraform fmt           # Auto-format HCL files
terraform validate      # Syntax + logic validation
terraform plan          # Preview changes (dry run)
terraform plan -out=plan.tfplan   # Save plan to file
terraform apply plan.tfplan       # Apply EXACT saved plan
terraform apply -auto-approve     # Skip confirmation (CI/CD only)
terraform destroy       # Tear down everything
terraform console       # Interactive HCL expression tester
```

---

**6-9. Init, Plan, Apply, Destroy — Deep details:**

| Command | Purpose | When to Run |
|---------|---------|-------------|
| `init` | Download providers, init backend, download modules | New project, new provider, backend change |
| `plan` | Preview changes without modifying anything | Before every apply |
| `apply` | Execute the plan, create/modify/delete resources | After reviewing plan |
| `destroy` | Delete ALL managed resources | Decommissioning |

```
terraform plan output:
  + azurerm_resource_group.rg       ← Will CREATE
  ~ azurerm_virtual_machine.vm     ← Will MODIFY in-place
  - azurerm_storage_account.old    ← Will DESTROY
  -/+ azurerm_vm.web               ← Will DESTROY and RECREATE
                                      (forced replacement)
```

---

**10. HCL (HashiCorp Configuration Language)?**

```hcl
# ─── Resource Block ───
resource "azurerm_resource_group" "example" {   # resource "TYPE" "NAME"
  name     = "my-rg"                            # argument
  location = "East US"                          # argument

  tags = {                                      # map argument
    Environment = var.environment               # variable reference
    ManagedBy   = "terraform"
  }
}

# ─── Reference Other Resources ───
resource "azurerm_virtual_network" "vnet" {
  name                = "my-vnet"
  resource_group_name = azurerm_resource_group.example.name   # ← reference!
  location            = azurerm_resource_group.example.location
  address_space       = ["10.0.0.0/16"]
}
# Terraform automatically knows: create RG first, then VNet (implicit dependency)
```

---

## Terraform State — Deep Dive

**11. What is Terraform state?**

JSON file that maps your HCL config to real-world resources. Terraform's **source of truth**.

```
┌─── Your Code (main.tf) ──────────┐
│ resource "azurerm_rg" "rg" {     │
│   name = "my-rg"                 │
│ }                                 │
└────────────┬──────────────────────┘
             │ maps to
┌────────────▼──────────────────────┐
│ State (terraform.tfstate)         │
│ {                                 │
│   "azurerm_rg.rg": {            │
│     "id": "/subscriptions/.../rg"│
│     "name": "my-rg"             │
│     "location": "eastus"        │
│   }                              │
│ }                                 │
└────────────┬──────────────────────┘
             │ represents
┌────────────▼──────────────────────┐
│ Real Infrastructure (Azure)       │
│ Resource Group: my-rg             │
│ Location: East US                 │
│ ID: /subscriptions/.../rg         │
└───────────────────────────────────┘
```

Default: `terraform.tfstate` in current directory. **Never manage this manually.**

---

**12. Why NOT store state in Git?**

```
❌ Security:  Contains secrets in plaintext (DB passwords, keys)
❌ Concurrency: Two people apply simultaneously → state corruption
❌ Locking:  Git has no lock mechanism
❌ Size:     State grows large over time
❌ History:  Secrets visible in entire git history
```

---

**13. Remote state backends?**

```hcl
# Azure Storage (most common for Azure teams)
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# AWS S3 + DynamoDB (locking)
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

# Terraform Cloud / HCP
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "production"
    }
  }
}
```

---

**14. State locking?**

```
Developer A: terraform apply          Developer B: terraform apply
      │                                     │
      ▼                                     ▼
  Lock state ✅                         Lock state ❌ BLOCKED!
  Make changes                          "Error: state locked by A"
  Update state                          Wait...
  Release lock ✅
                                        Lock state ✅ (now available)
                                        Make changes
                                        Release lock ✅
```

| Backend | Locking Mechanism |
|---------|-------------------|
| Azure Storage | Blob lease |
| AWS S3 | DynamoDB table |
| Terraform Cloud | Built-in |
| Local | `.terraform.tfstate.lock.info` file |

Force unlock (dangerous): `terraform force-unlock <LOCK_ID>`

---

**15-16. State commands?**

```bash
# ─── Inspect ───
terraform state list                           # All resources in state
terraform state show azurerm_resource_group.rg # Details of one resource
terraform show                                 # Entire state (formatted)

# ─── Modify ───
terraform state mv old_name new_name           # Rename in state (refactoring)
terraform state rm azurerm_vm.old              # Remove from state (don't destroy)
terraform state pull                           # Download remote state to stdout
terraform state push                           # Upload local state to remote

# ─── Common scenario: Refactoring ───
# Renamed resource in code from "web" to "app":
terraform state mv azurerm_vm.web azurerm_vm.app
# Without this → Terraform would destroy "web" and create "app" (data loss!)
```

---

**17. Import existing resources?**

Two methods:

```hcl
# ─── Method 1: CLI (classic) ───
# Step 1: Write the matching config
resource "azurerm_resource_group" "imported" {
  name     = "existing-rg"
  location = "East US"
}
# Step 2: Import
# terraform import azurerm_resource_group.imported /subscriptions/.../resourceGroups/existing-rg
# Step 3: Run terraform plan to verify (should show no changes)

# ─── Method 2: import block (Terraform 1.5+, recommended) ───
import {
  to = azurerm_resource_group.imported
  id = "/subscriptions/.../resourceGroups/existing-rg"
}

resource "azurerm_resource_group" "imported" {
  name     = "existing-rg"
  location = "East US"
}
# Run: terraform plan → shows import action
# Run: terraform apply → imports into state
# Then remove the import block
```

---

**18. Lose state file?**

```
Terraform loses ALL knowledge of managed resources.
Resources still exist in cloud — but Terraform can't manage them.

Recovery options:
  1. Restore from backup (always backup state!) ← best
  2. terraform import each resource manually ← painful
  3. Recreate infrastructure (if disposable) ← last resort

Prevention:
  - Remote backend with versioning enabled
  - Azure Storage: enable blob versioning / soft delete
  - S3: enable versioning on bucket
  - Regular backups: terraform state pull > backup.json
```

---

**19. State file encryption?**

| Backend | Encryption |
|---------|-----------|
| Azure Storage | Encrypted at rest (SSE) by default + HTTPS in transit |
| AWS S3 | Enable SSE-S3 or SSE-KMS + enforce HTTPS |
| Terraform Cloud | Encrypted at rest and in transit |
| Local | **NOT encrypted!** Never use for sensitive infra |

---

## Resources, Data Sources & Dependencies

**20. Resource block — Azure VM example:**

```hcl
resource "azurerm_linux_virtual_machine" "web" {
  name                = "web-vm"
  resource_group_name = azurerm_resource_group.rg.name     # implicit dependency
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

  tags = local.common_tags
}
```

---

**21. Data source?**

Read-only query to fetch info about **existing** resources (not managed by this Terraform):

```hcl
# Query existing resource group (managed elsewhere)
data "azurerm_resource_group" "existing" {
  name = "shared-services-rg"
}

# Use it
resource "azurerm_virtual_network" "vnet" {
  resource_group_name = data.azurerm_resource_group.existing.name
  location            = data.azurerm_resource_group.existing.location
  # ...
}

# Other common data sources:
data "azurerm_client_config" "current" {}        # Current Azure credentials
data "azurerm_subscription" "current" {}         # Current subscription
data "azurerm_key_vault_secret" "db_pass" {      # Read secret from Key Vault
  name         = "db-password"
  key_vault_id = azurerm_key_vault.kv.id
}
```

---

**22. Dependencies — implicit vs explicit:**

```hcl
# ─── Implicit (Terraform auto-detects via references) ───
resource "azurerm_resource_group" "rg" {
  name     = "my-rg"
  location = "East US"
}

resource "azurerm_virtual_network" "vnet" {
  resource_group_name = azurerm_resource_group.rg.name   # ← reference = dependency
  # Terraform knows: create RG FIRST, then VNet
}

# ─── Explicit (when Terraform CAN'T auto-detect) ───
resource "azurerm_kubernetes_cluster" "aks" {
  # ...
  depends_on = [azurerm_role_assignment.acr_pull]
  # AKS needs the role assignment, but there's no direct reference
}
```

```
Dependency Graph:
  RG ──► VNet ──► Subnet ──► NIC ──► VM
                     │
                     └──► NSG

  terraform graph | dot -Tpng > graph.png   # Visualize
```

---

## Lifecycle — Deep Dive

**23. Lifecycle block — ALL options explained:**

```hcl
resource "azurerm_linux_virtual_machine" "web" {
  name = "web-vm"
  # ...

  lifecycle {
    # ─── create_before_destroy ───
    # Create the replacement BEFORE destroying the old one
    # Critical for: zero-downtime replacements (LBs, DNS)
    # Without: old destroyed first → gap in service!
    create_before_destroy = true

    # ─── prevent_destroy ───
    # Terraform will ERROR if you try to destroy this resource
    # Use for: databases, storage accounts, critical infra
    # Must be removed from config to actually destroy
    prevent_destroy = true

    # ─── ignore_changes ───
    # Don't track changes to specific attributes
    # Use for: tags managed by Azure Policy, autoscaling changes,
    #          fields modified outside Terraform
    ignore_changes = [
      tags,                    # Tags managed by policy
      tags["LastModified"],    # Specific tag
    ]

    # ignore ALL changes (manage creation only):
    # ignore_changes = all

    # ─── replace_triggered_by ───
    # Force replacement when another resource changes
    # Use for: recreate VM when user-data script changes
    replace_triggered_by = [
      null_resource.user_data_trigger.id
    ]

    # ─── precondition (Terraform 1.2+) ───
    # Validate BEFORE creating/updating resource
    precondition {
      condition     = var.environment != "prod" || var.vm_size != "Standard_B1s"
      error_message = "Production VMs must not use B1s (too small)."
    }

    # ─── postcondition (Terraform 1.2+) ───
    # Validate AFTER resource is created
    postcondition {
      condition     = self.public_ip_address != ""
      error_message = "VM must have a public IP assigned."
    }
  }
}
```

### Lifecycle use cases summary:

```
┌─── Lifecycle Option ──────────┬─── When to Use ──────────────────────┐
│                                │                                      │
│ create_before_destroy = true  │ Load balancers, DNS records,         │
│                                │ anything that must not have downtime │
│                                │                                      │
│ prevent_destroy = true        │ Databases, storage accounts,         │
│                                │ anything with irreplaceable data     │
│                                │                                      │
│ ignore_changes = [tags]       │ Tags managed by Azure Policy,        │
│                                │ autoscaled fields, external changes  │
│                                │                                      │
│ ignore_changes = all          │ "Create once, never update" resources│
│                                │                                      │
│ replace_triggered_by          │ Force recreate when dependency       │
│                                │ changes (user-data, init script)     │
│                                │                                      │
│ precondition                  │ Validate inputs before apply         │
│                                │ "Don't let bad configs through"      │
│                                │                                      │
│ postcondition                 │ Validate outputs after creation      │
│                                │ "Ensure resource was set up right"   │
└────────────────────────────────┴──────────────────────────────────────┘
```

---

## Variables, Outputs, Locals — Deep Dive

**24. Variable types:**

```hcl
variable "name"        { type = string }
variable "count"       { type = number }
variable "enabled"     { type = bool }
variable "tags"        { type = map(string) }
variable "subnets"     { type = list(string) }
variable "ports"       { type = set(number) }

# Complex types
variable "vm_config" {
  type = object({
    name    = string
    size    = string
    count   = number
    enabled = bool
    tags    = map(string)
  })
}

variable "servers" {
  type = list(object({
    name = string
    ip   = string
    role = string
  }))
}

# Tuple (fixed-length, mixed types)
variable "example" {
  type = tuple([string, number, bool])
}
```

---

**25. Variable validation (Terraform 0.13+):**

```hcl
variable "environment" {
  type        = string
  description = "Deployment environment"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "vm_size" {
  type = string

  validation {
    condition     = can(regex("^Standard_", var.vm_size))
    error_message = "VM size must start with 'Standard_'."
  }
}

variable "cidr_block" {
  type = string

  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid CIDR block (e.g., 10.0.0.0/16)."
  }
}
```

---

**26. Pass variables — priority order (lowest → highest):**

```
┌─── Priority (low → high) ────────────────────────────────────┐
│                                                               │
│  1. default value in variable block (lowest)                 │
│  2. terraform.tfvars (auto-loaded)                           │
│  3. *.auto.tfvars (auto-loaded, alphabetical)                │
│  4. -var-file="prod.tfvars" (CLI)                            │
│  5. -var="env=prod" (CLI)                                    │
│  6. TF_VAR_env=prod (environment variable) (highest)         │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

```bash
# CLI flag
terraform apply -var="environment=prod"

# Variable file
terraform apply -var-file="prod.tfvars"

# Environment variable
export TF_VAR_environment=prod

# terraform.tfvars (auto-loaded — no flag needed)
environment = "prod"
region      = "eastus"

# *.auto.tfvars (also auto-loaded)
# prod.auto.tfvars
```

---

**27. `terraform.tfvars` vs `variables.tf`?**

```
variables.tf:                        terraform.tfvars:
┌────────────────────────────┐      ┌────────────────────────────┐
│ DECLARES variables          │      │ ASSIGNS values             │
│ (name, type, description,  │      │                            │
│  default, validation)       │      │ environment = "prod"       │
│                              │      │ region      = "eastus"    │
│ variable "environment" {    │      │ vm_size     = "Standard_D2"│
│   type    = string          │      │                            │
│   default = "dev"           │      │ Like function parameters   │
│ }                            │      │ vs function arguments     │
└────────────────────────────┘      └────────────────────────────┘
```

---

**28. Outputs and locals:**

```hcl
# ─── Outputs: export values after apply ───
output "aks_cluster_name" {
  value       = azurerm_kubernetes_cluster.aks.name
  description = "The AKS cluster name"
}

output "db_connection_string" {
  value     = azurerm_postgresql_server.db.connection_string
  sensitive = true       # Hidden in console output
}

# Access from CLI:
# terraform output aks_cluster_name
# terraform output -json    # All outputs as JSON

# ─── Locals: computed internal values (not settable from outside) ───
locals {
  name_prefix = "${var.project}-${var.environment}"
  common_tags = {
    Environment = var.environment
    Project     = var.project
    ManagedBy   = "terraform"
    CreatedAt   = timestamp()
  }
  is_production = var.environment == "prod"
}

resource "azurerm_resource_group" "rg" {
  name     = "${local.name_prefix}-rg"
  location = var.location
  tags     = local.common_tags
}
```

---

## Modules — Deep Dive

**29. What is a module?**

Reusable, encapsulated group of resources. Like a **function** in programming.

```
Root Module (main.tf):
┌─────────────────────────────────────────┐
│                                          │
│  module "networking" {                  │
│    source = "./modules/networking"      │
│    cidr   = "10.0.0.0/16"             │
│  }                                      │
│                                          │
│  module "app" {                         │
│    source  = "./modules/web-app"        │
│    subnet  = module.networking.subnet_id│
│    env     = var.environment            │
│  }                                      │
│                                          │
└─────────────────────────────────────────┘
         │                    │
         ▼                    ▼
  modules/networking/    modules/web-app/
  ├── main.tf            ├── main.tf
  ├── variables.tf       ├── variables.tf
  └── outputs.tf         └── outputs.tf
```

---

**30. Module from Terraform Registry:**

```hcl
module "aks" {
  source  = "Azure/aks/azurerm"
  version = "~> 7.0"                    # Pessimistic constraint

  resource_group_name = "my-rg"
  cluster_name        = "my-aks"
  node_count          = 3
}

# Version constraints:
# "= 7.0.0"   Exact version
# ">= 7.0"    Minimum version
# "~> 7.0"    >= 7.0, < 8.0 (recommended)
# ">= 7.0, < 8.0"  Range
```

---

## Advanced Terraform

**31. `count` vs `for_each`?**

```hcl
# ─── count: by index (0, 1, 2...) ───
resource "azurerm_resource_group" "rg" {
  count    = 3
  name     = "rg-${count.index}"    # rg-0, rg-1, rg-2
  location = "East US"
}
# Problem: remove rg-1 → rg-2 becomes rg-1 (index shift!)
# Terraform destroys and recreates rg-2 ❌

# ─── for_each: by key (stable identity) ───
resource "azurerm_resource_group" "rg" {
  for_each = toset(["dev", "staging", "prod"])
  name     = "rg-${each.key}"       # rg-dev, rg-staging, rg-prod
  location = "East US"
}
# Remove "staging" → only staging destroyed. dev and prod untouched ✅

# ─── for_each with map ───
variable "resource_groups" {
  default = {
    dev  = "East US"
    prod = "West US"
  }
}
resource "azurerm_resource_group" "rg" {
  for_each = var.resource_groups
  name     = "rg-${each.key}"
  location = each.value
}
```

**Rule: Always prefer `for_each` over `count`** unless creating identical resources.

---

**32. Dynamic blocks:**

```hcl
variable "nsg_rules" {
  default = [
    { name = "ssh",   port = 22,  priority = 100 },
    { name = "http",  port = 80,  priority = 200 },
    { name = "https", port = 443, priority = 300 },
  ]
}

resource "azurerm_network_security_group" "nsg" {
  name                = "web-nsg"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location

  dynamic "security_rule" {
    for_each = var.nsg_rules
    content {
      name                       = security_rule.value.name
      priority                   = security_rule.value.priority
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = security_rule.value.port
      source_address_prefix      = "*"
      destination_address_prefix = "*"
    }
  }
}
```

---

**33. `moved` block (Terraform 1.1+) — resource refactoring:**

```hcl
# Renamed resource from "web" to "app" in code
# Without moved block: Terraform destroys "web" and creates "app" (data loss!)
# With moved block: Terraform moves state automatically

moved {
  from = azurerm_linux_virtual_machine.web
  to   = azurerm_linux_virtual_machine.app
}

# Or moved to a module:
moved {
  from = azurerm_linux_virtual_machine.web
  to   = module.compute.azurerm_linux_virtual_machine.app
}

# After apply succeeds, you can remove the moved block
```

Better than `terraform state mv` because:
- Tracked in code (reviewable in PR)
- Works for the whole team (not just your local state)
- Can be planned and applied like normal changes

---

**34. `terraform_data` resource (replaces `null_resource`):**

```hcl
# ─── null_resource (legacy) ───
resource "null_resource" "trigger" {
  triggers = {
    script_hash = filemd5("scripts/init.sh")
  }
  provisioner "local-exec" {
    command = "bash scripts/init.sh"
  }
}

# ─── terraform_data (Terraform 1.4+, recommended) ───
resource "terraform_data" "trigger" {
  triggers_replace = [
    filemd5("scripts/init.sh")
  ]
  provisioner "local-exec" {
    command = "bash scripts/init.sh"
  }
}

# Use case: store and track arbitrary values, trigger replacements
resource "terraform_data" "version" {
  input = var.app_version    # stored in state
}
# Access: terraform_data.version.output
```

---

**35. Terraform workspaces:**

```bash
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod
terraform workspace select dev
terraform workspace list
terraform workspace show         # Current workspace
```

```hcl
# Use workspace name in config:
resource "azurerm_resource_group" "rg" {
  name = "rg-${terraform.workspace}"    # rg-dev, rg-prod
}

locals {
  vm_size = terraform.workspace == "prod" ? "Standard_D4s_v3" : "Standard_B2s"
}
```

```
Workspace Pros:                    Workspace Cons:
✅ Same code, different state      ❌ Easy to apply to wrong workspace
✅ Quick env switching              ❌ Same backend for all workspaces
✅ Built-in                        ❌ No visual separation

Alternative: Separate directories per environment (more explicit)
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
└── modules/          ← shared modules
```

---

**36. Provider versioning and constraints:**

```hcl
terraform {
  required_version = ">= 1.5.0"        # Terraform CLI version

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.80"               # >= 3.80, < 4.0
    }
    random = {
      source  = "hashicorp/random"
      version = ">= 3.0"
    }
  }
}

# .terraform.lock.hcl (auto-generated, commit to git!)
# Locks exact provider versions for reproducible builds
# Like package-lock.json for Terraform
```

---

**37. Handle Terraform drift:**

```
Drift = someone manually changes infrastructure outside Terraform

  Terraform State          Real Infrastructure
  ┌──────────────┐         ┌──────────────┐
  │ VM size: D2  │         │ VM size: D4  │ ← someone changed manually!
  │ Tags: {a:1}  │         │ Tags: {a:1,  │
  │              │         │        b:2}  │ ← Azure Policy added tag
  └──────────────┘         └──────────────┘

Detection:
  terraform plan → shows differences
  "VM size will be changed from D4 to D2"

Resolution:
  Option 1: terraform apply → forces infra back to desired state
  Option 2: Update code to match reality, then apply
  Option 3: terraform state refresh → update state only

Prevention:
  - Use CI/CD pipeline (no manual changes)
  - Restrict Azure portal write access
  - Regular drift detection (scheduled terraform plan)
  - Use lifecycle { ignore_changes = [tags] } for policy-managed fields
```

---

**38. `terraform apply -replace` (replaces taint):**

```bash
# Force destroy + recreate of specific resource
terraform apply -replace="azurerm_linux_virtual_machine.web"

# Why? VM is in a bad state, need fresh instance
# Terraform marks it for replacement in the plan

# Old way (deprecated):
# terraform taint azurerm_linux_virtual_machine.web
# terraform untaint azurerm_linux_virtual_machine.web
```

---

**39. Backend migration:**

```hcl
# Currently: local backend → want to move to Azure Storage

# Step 1: Add backend config
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.tfstate"
  }
}

# Step 2: Run terraform init
# Terraform detects backend change:
# "Do you want to copy existing state to the new backend?"
# → yes

# State is migrated. Local tfstate file can be deleted.
```

---

**40. Multiple environments approach comparison:**

| Approach | Code Duplication | State Separation | Risk Level |
|----------|-----------------|------------------|------------|
| **Workspaces** | None (same code) | Same backend, different state | Medium (wrong workspace) |
| **Directories** | Some (can use modules) | Separate backends | Low (explicit) |
| **Terragrunt** | None (DRY) | Separate state + backend | Low |
| **tfvars per env** | None | ⚠️ Same state! | High |

---

## Provisioners & When to Avoid

**41. Provisioners — why discouraged?**

```
Provisioners execute scripts on resources. Avoid because:
  ❌ Not declarative (can't detect drift)
  ❌ Poor error handling
  ❌ Run only on create (not on update)
  ❌ Tightly couple Terraform to configuration

Better alternatives:
  ✅ cloud-init / user_data for VM bootstrap
  ✅ Ansible for configuration management
  ✅ Packer for golden images
  ✅ Kubernetes for container config
```

```hcl
# If you MUST use provisioners:
resource "azurerm_linux_virtual_machine" "web" {
  # ...

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install -y nginx",
    ]
    connection {
      type        = "ssh"
      user        = "adminuser"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip_address
    }
  }

  provisioner "local-exec" {
    command = "echo ${self.public_ip_address} >> ip_list.txt"
  }

  # Runs on destroy:
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'VM destroyed' >> destroy.log"
  }
}
```

---

## Sensitive Data & Security

**42. Sensitive variables:**

```hcl
variable "db_password" {
  type      = string
  sensitive = true       # Hidden in plan/apply output
}

output "connection_string" {
  value     = "postgresql://user:${var.db_password}@db:5432"
  sensitive = true       # Must mark output sensitive too
}

# ⚠️ Still stored in state file in plaintext!
# → Encrypt your state backend
# → Use Azure Key Vault / AWS Secrets Manager data sources
# → Never commit .tfvars with secrets to Git
```

---

## Terraform in CI/CD

**43. CI/CD pipeline for Terraform:**

```
┌─── PR Created ───────────────────────────────────────────────┐
│                                                               │
│  terraform fmt -check        → formatting OK?                │
│  terraform init              → download providers            │
│  terraform validate          → syntax check                  │
│  tfsec / checkov             → security scanning             │
│  terraform plan -out=plan    → preview changes               │
│  Post plan as PR comment     → team reviews                  │
│                                                               │
└───────────────────────────────────────────────────────────────┘
                    │
                    ▼ PR Merged to main
┌─── Main Branch ─────────────────────────────────────────────┐
│                                                               │
│  terraform init                                              │
│  terraform plan -out=plan                                    │
│  terraform apply plan        → apply approved plan           │
│                                                               │
│  Only CI/CD applies. Devs never run apply locally.           │
└───────────────────────────────────────────────────────────────┘
```

---

# ANSIBLE — Deep Dive

---

## Ansible Basics

**44. What is Ansible?**

Agentless automation tool for configuration management, application deployment, and orchestration.

```
┌─── Control Node ────────┐          ┌─── Managed Nodes ──────┐
│                          │          │                         │
│  ansible-playbook        │   SSH    │  Node 1 (web server)   │
│  site.yml            ────┼────────► │  Node 2 (db server)    │
│                          │          │  Node 3 (cache server)  │
│  Inventory (hosts)       │   No     │                         │
│  Playbooks (YAML)        │  agent   │  Only needs: Python     │
│  Roles                   │ needed!  │  + SSH access            │
│  ansible.cfg             │          │                         │
└──────────────────────────┘          └─────────────────────────┘
```

---

**45. Agentless — how does it work?**

```
1. Ansible reads playbook + inventory
2. Generates Python scripts for each task
3. Connects via SSH (Linux) or WinRM (Windows)
4. Copies Python scripts to target
5. Executes scripts on target
6. Captures output
7. Returns results to control node
8. Cleans up temporary files
```

No daemon, no agent installation, no PKI infrastructure. Just SSH.

---

**46. Inventory — static vs dynamic:**

```ini
# ─── Static Inventory (hosts.ini) ───
[webservers]
web1.example.com ansible_host=10.0.1.5
web2.example.com ansible_host=10.0.1.6

[databases]
db1.example.com ansible_host=10.0.2.5

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa

# Group of groups
[production:children]
webservers
databases
```

```bash
# ─── Dynamic Inventory (cloud) ───
# Azure:
ansible-inventory -i azure_rm.yml --list

# AWS:
ansible-inventory -i aws_ec2.yml --list

# Queries cloud API for current instances
# No manual IP maintenance!
```

---

**47. Playbook structure:**

```yaml
---
# Playbook = list of plays
- name: Configure web servers           # Play 1
  hosts: webservers                      # Target group
  become: yes                            # Run as root (sudo)
  vars:
    app_port: 8080                       # Play-level variables

  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

  roles:
    - nginx                              # Include role
    - app-deploy

  tasks:
    - name: Ensure app is running
      service:
        name: myapp
        state: started
        enabled: yes

  post_tasks:
    - name: Verify deployment
      uri:
        url: "http://localhost:{{ app_port }}/health"
        status_code: 200

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted

- name: Configure databases              # Play 2
  hosts: databases
  become: yes
  tasks:
    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present
```

```
Execution order within a play:
  1. pre_tasks
  2. roles
  3. tasks
  4. post_tasks
  5. handlers (triggered by notify)
```

---

**48. 10 essential Ansible modules:**

| Module | Purpose | Example |
|--------|---------|---------|
| `apt` / `yum` | Package management | `apt: name=nginx state=present` |
| `copy` | Copy files to target | `copy: src=file.conf dest=/etc/app/` |
| `template` | Jinja2 template rendering | `template: src=config.j2 dest=/etc/app/config` |
| `service` / `systemd` | Manage services | `service: name=nginx state=restarted` |
| `file` | File/directory permissions | `file: path=/data mode=0755 state=directory` |
| `user` | User management | `user: name=deploy groups=sudo` |
| `command` / `shell` | Run commands | `command: /usr/bin/check.sh` |
| `docker_container` | Manage Docker | `docker_container: name=app image=myapp:v1` |
| `lineinfile` | Edit lines in files | `lineinfile: path=/etc/hosts line="10.0.1.5 web1"` |
| `uri` | HTTP requests | `uri: url=http://localhost/health status_code=200` |

---

**49. Handlers — why they exist:**

```yaml
tasks:
- name: Update nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx                # Trigger handler

- name: Update SSL cert
  copy:
    src: cert.pem
    dest: /etc/ssl/cert.pem
  notify: restart nginx                # Same handler triggered

handlers:
- name: restart nginx
  service:
    name: nginx
    state: restarted
  # Runs ONCE at end of play, even if notified multiple times!
  # Prevents unnecessary restarts
```

---

**50. `copy` vs `template`?**

```
copy:                                template:
┌───────────────────────────┐       ┌───────────────────────────┐
│ Copies file AS-IS          │       │ Processes Jinja2 FIRST    │
│ No variable substitution  │       │ Then copies result         │
│                            │       │                            │
│ Use: binary files, static │       │ Use: config files that     │
│ configs that don't change  │       │ need dynamic values        │
└───────────────────────────┘       └───────────────────────────┘

# template: config.j2
server_name: {{ server_name }}
port: {{ app_port | default(8080) }}
{% if enable_ssl %}
ssl_certificate: /etc/ssl/cert.pem
{% endif %}
{% for host in db_hosts %}
  - {{ host }}:5432
{% endfor %}
```

---

## Ansible Advanced

**51. Roles — directory structure:**

```
roles/
└── webserver/
    ├── tasks/main.yml        # Main task list
    ├── handlers/main.yml     # Handlers
    ├── templates/            # Jinja2 templates
    │   └── nginx.conf.j2
    ├── files/                # Static files
    │   └── index.html
    ├── vars/main.yml         # Role variables (high priority)
    ├── defaults/main.yml     # Default variables (low priority, overridable)
    ├── meta/main.yml         # Role dependencies
    └── README.md
```

```yaml
# Use in playbook:
- hosts: webservers
  roles:
    - webserver
    - { role: monitoring, tags: monitoring }
```

---

**52. Ansible Galaxy:**

```bash
ansible-galaxy install geerlingguy.docker       # Install community role
ansible-galaxy init myrole                       # Create role scaffold
ansible-galaxy collection install community.docker  # Install collection
ansible-galaxy list                              # List installed roles

# requirements.yml (version-controlled dependencies)
roles:
  - name: geerlingguy.docker
    version: "6.0.0"
collections:
  - name: community.docker
    version: ">=3.0"

ansible-galaxy install -r requirements.yml
```

---

**53. Variable precedence (low → high):**

```
┌─── Lowest Priority ──────────────────────────────────────────┐
│  1. Role defaults (defaults/main.yml)                        │
│  2. Inventory vars                                            │
│  3. Inventory group_vars                                      │
│  4. Inventory host_vars                                       │
│  5. Playbook group_vars                                       │
│  6. Playbook host_vars                                        │
│  7. Play vars                                                 │
│  8. Play vars_prompt                                          │
│  9. Play vars_files                                           │
│  10. Role vars (vars/main.yml)                                │
│  11. Block vars                                               │
│  12. Task vars                                                │
│  13. set_fact / registered vars                               │
│  14. Extra vars (-e) ← ALWAYS WINS                           │
└─── Highest Priority ─────────────────────────────────────────┘
```

```bash
ansible-playbook site.yml -e "environment=prod"   # Highest priority
```

---

**54. Ansible Vault — encrypt secrets:**

```bash
# Create encrypted file
ansible-vault create secrets.yml

# Encrypt existing file
ansible-vault encrypt vars/prod.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Decrypt
ansible-vault decrypt secrets.yml

# View without decrypting
ansible-vault view secrets.yml

# Run playbook with vault
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file=~/.vault_pass

# Encrypt single string (for inline use)
ansible-vault encrypt_string 'my_secret' --name 'db_password'
```

```yaml
# In vars file:
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  6538373038...encrypted...
```

---

**55. Conditionals, loops, blocks:**

```yaml
# ─── Conditionals ───
- name: Install on Debian
  apt: name=nginx
  when: ansible_os_family == "Debian"

- name: Only in production
  template: src=prod.conf.j2 dest=/etc/app/config
  when: environment == "production"

# ─── Loops ───
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

# ─── Blocks (group tasks + error handling) ───
- block:
    - name: Install app
      apt: name=myapp
    - name: Start app
      service: name=myapp state=started
  rescue:
    - name: Rollback on failure
      shell: /opt/rollback.sh
  always:
    - name: Send notification
      mail:
        to: team@example.com
        subject: "Deploy {{ 'succeeded' if not ansible_failed_task else 'failed' }}"
```

---

**56. Testing Ansible:**

```bash
# Lint (check best practices)
ansible-lint site.yml

# Syntax check
ansible-playbook site.yml --syntax-check

# Dry run (no changes)
ansible-playbook site.yml --check

# Show file differences
ansible-playbook site.yml --check --diff

# Molecule (full test framework)
molecule init scenario -d docker       # Create test scenario
molecule test                           # Full test cycle:
  # → create → converge → verify → destroy

# Step debugging
ansible-playbook site.yml --step       # Prompt before each task
```

---

**57. Tags — run specific tasks:**

```yaml
- name: Install nginx
  apt: name=nginx
  tags: [install, nginx]

- name: Configure nginx
  template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
  tags: [configure, nginx]

- name: Deploy app
  copy: src=app/ dest=/var/www/app/
  tags: [deploy]
```

```bash
ansible-playbook site.yml --tags "deploy"          # Only deploy
ansible-playbook site.yml --tags "install,configure"
ansible-playbook site.yml --skip-tags "install"    # Everything except install
ansible-playbook site.yml --list-tags              # Show all tags
```

---

## Interview-Style

**58. Terraform: Complete Azure AKS setup:**

```hcl
terraform {
  required_version = ">= 1.5"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.80"
    }
  }
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "aks.tfstate"
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "rg" {
  name     = "${var.project}-${var.environment}-rg"
  location = var.location
  tags     = local.common_tags

  lifecycle {
    prevent_destroy = true       # Don't accidentally delete RG
  }
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = "${var.project}-${var.environment}-aks"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "${var.project}-${var.environment}"

  default_node_pool {
    name       = "default"
    node_count = var.environment == "prod" ? 3 : 1
    vm_size    = var.environment == "prod" ? "Standard_D4s_v3" : "Standard_B2s"
  }

  identity {
    type = "SystemAssigned"
  }

  tags = local.common_tags

  lifecycle {
    ignore_changes = [
      default_node_pool[0].node_count,   # Managed by HPA/cluster autoscaler
    ]
  }
}

output "kube_config" {
  value     = azurerm_kubernetes_cluster.aks.kube_config_raw
  sensitive = true
}

output "cluster_name" {
  value = azurerm_kubernetes_cluster.aks.name
}
```

---

**59. Reusable Terraform module with validation:**

```hcl
# ─── modules/web-app/variables.tf ───
variable "name" {
  type = string
  validation {
    condition     = length(var.name) >= 3 && length(var.name) <= 24
    error_message = "Name must be 3-24 characters."
  }
}

variable "environment" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

variable "instance_type" {
  type    = string
  default = "Standard_B2s"
}

# ─── modules/web-app/main.tf ───
resource "azurerm_linux_virtual_machine" "web" {
  name                = "${var.name}-${var.environment}"
  resource_group_name = var.resource_group_name
  location            = var.location
  size                = var.instance_type
  # ...

  lifecycle {
    create_before_destroy = true
    precondition {
      condition     = var.environment != "prod" || var.instance_type != "Standard_B2s"
      error_message = "Production must not use B2s instances."
    }
  }
}

# ─── Usage ───
module "web_staging" {
  source       = "./modules/web-app"
  name         = "myapp"
  environment  = "staging"
}

module "web_prod" {
  source        = "./modules/web-app"
  name          = "myapp"
  environment   = "prod"
  instance_type = "Standard_D4s_v3"
}
```

---

**60. Ansible: Install Docker + deploy container:**

```yaml
---
- name: Deploy Docker Container
  hosts: servers
  become: yes
  vars:
    app_image: myapp:latest
    app_port: 8080

  tasks:
  - name: Install Docker prerequisites
    apt:
      name:
        - apt-transport-https
        - ca-certificates
        - curl
        - software-properties-common
      state: present
      update_cache: yes

  - name: Install Docker
    apt:
      name: docker.io
      state: present

  - name: Start and enable Docker
    service:
      name: docker
      state: started
      enabled: yes

  - name: Pull application image
    docker_image:
      name: "{{ app_image }}"
      source: pull

  - name: Run application container
    docker_container:
      name: myapp
      image: "{{ app_image }}"
      ports:
        - "{{ app_port }}:{{ app_port }}"
      restart_policy: always
      env:
        APP_ENV: "{{ environment }}"
    notify: verify deployment

  handlers:
  - name: verify deployment
    uri:
      url: "http://localhost:{{ app_port }}/health"
      status_code: 200
    retries: 5
    delay: 5
```

---

**61. State management in a team of 10:**

```
┌─── Team Terraform Workflow ──────────────────────────────────┐
│                                                               │
│  1. Remote backend (Azure Storage) with state locking         │
│  2. Separate state per environment:                          │
│     dev.tfstate, staging.tfstate, prod.tfstate                │
│  3. CI/CD pipeline runs plan/apply (NOT individual devs)     │
│  4. PR-based workflow:                                        │
│     - Developer creates PR with Terraform changes            │
│     - CI runs: fmt, validate, tfsec, plan                    │
│     - Plan output posted as PR comment                       │
│     - Team reviews plan                                       │
│     - Merge → CI runs apply                                  │
│  5. Access controls on state storage (read-only for devs)    │
│  6. State versioning enabled (blob versioning / S3 versioning)│
│  7. Never run terraform apply locally                        │
│                                                               │
│  Key rule: Git is the source of truth.                       │
│  State is an implementation detail.                          │
└───────────────────────────────────────────────────────────────┘
```

---

**62. `terraform apply` fails halfway:**

```
What happens:
  Resources created BEFORE the error → in state ✅
  Resource that failed → NOT in state (or partially)
  Resources AFTER the error → not created

Recovery:
  1. Fix the error in your code
  2. Run terraform plan → see current state
  3. Run terraform apply → Terraform picks up where it left off
     (won't re-create already-created resources)
  4. If state is corrupted:
     terraform state rm <problematic_resource>
     terraform import <resource> <id>
```

---

**63. IaC security scanning tools:**

```bash
# tfsec (Terraform-specific)
tfsec .                              # Scan current directory
tfsec --minimum-severity HIGH        # Only high+ findings

# checkov (multi-framework: Terraform, CloudFormation, K8s, Docker)
checkov -d .                         # Scan directory
checkov -f main.tf                   # Scan single file

# terrascan (OPA-based policies)
terrascan scan -t azure

# In CI/CD:
tfsec . --soft-fail                  # Warn but don't fail build
checkov -d . --soft-fail
```

---

**64. Testing IaC before production:**

```
┌─── IaC Testing Pyramid ─────────────────────────────────────┐
│                                                               │
│  ┌───────────────────┐                                       │
│  │ Integration Tests │  terratest (Go) — creates real infra  │
│  │ (expensive, slow) │  and validates it, then destroys       │
│  ├───────────────────┤                                       │
│  │ Policy Tests      │  tfsec, checkov, OPA/Conftest         │
│  │ (fast, automated) │  security + compliance checks         │
│  ├───────────────────┤                                       │
│  │ Unit Tests        │  terraform validate, fmt -check       │
│  │ (instant)         │  terraform plan (review output)       │
│  └───────────────────┘                                       │
│                                                               │
│  CI/CD Pipeline:                                              │
│  1. terraform fmt -check     → formatting                    │
│  2. terraform validate       → syntax                        │
│  3. tfsec / checkov          → security scanning             │
│  4. terraform plan           → preview (review in PR)        │
│  5. Apply to dev first       → integration test              │
│  6. Apply to staging         → validate                      │
│  7. Apply to prod            → with approval gate            │
└───────────────────────────────────────────────────────────────┘
```

---

**65. Terraform + Ansible together:**

```
┌─── Day 0: Terraform ─────────────────────────────────────────┐
│  Create: Resource Group, VNet, Subnet, NSG, VM, AKS, ACR    │
│  Output: VM IPs, AKS credentials, ACR URL                   │
└──────────────────────┬────────────────────────────────────────┘
                       │
                       ▼ Pass outputs to Ansible
┌─── Day 1+: Ansible ─────────────────────────────────────────┐
│  Configure: install packages, deploy apps, manage configs    │
│  Use Terraform output as inventory / variables               │
└──────────────────────────────────────────────────────────────┘

# Terraform output → Ansible inventory:
terraform output -json > tf_outputs.json
# Ansible reads tf_outputs.json as vars
```

---

**66. Common Terraform anti-patterns:**

```
❌ Using count when for_each is better (index shift problem)
❌ Hardcoding values instead of variables
❌ Giant monolithic root module (split into modules!)
❌ Not using remote backend (local state = team disaster)
❌ Using provisioners when cloud-init/Ansible is better
❌ Not pinning provider versions (breaks on upgrade)
❌ Putting secrets in terraform.tfvars in Git
❌ Using 'latest' for image versions (non-reproducible)
❌ Not using lifecycle blocks for critical resources
❌ Running terraform apply locally instead of CI/CD
```

---

**67. Terraform commands cheat sheet:**

```bash
# ─── Core ───
terraform init                    # Initialize
terraform plan                    # Preview
terraform apply                   # Apply
terraform destroy                 # Tear down

# ─── State ───
terraform state list              # List resources
terraform state show <resource>   # Show one resource
terraform state mv old new        # Rename in state
terraform state rm <resource>     # Remove from state
terraform state pull              # Download remote state

# ─── Operations ───
terraform import <res> <id>       # Import existing resource
terraform apply -replace="<res>"  # Force replace resource
terraform refresh                 # Sync state with reality
terraform output                  # Show output values
terraform console                 # Interactive expression tester
terraform graph                   # Generate dependency graph

# ─── Validation ───
terraform fmt                     # Format code
terraform fmt -check              # Check formatting (CI)
terraform validate                # Validate syntax

# ─── Workspace ───
terraform workspace list          # List workspaces
terraform workspace new dev       # Create workspace
terraform workspace select prod   # Switch workspace
```
