# Terraform command and HCL reference

> Companion to [terraform-for-aem.md](terraform-for-aem.md). Every Terraform CLI command, the HCL language (blocks, types, expressions, functions, meta-arguments), state operations, files you will see in a repository, supporting tools, and the commands to verify DNS/CDN/TLS changes around AEM.

---

## Contents

1. [Installing and getting help](#1-installing-and-getting-help)
2. [Core workflow commands](#2-core-workflow-commands)
3. [Plan and apply options](#3-plan-and-apply-options)
4. [Inspecting: output, show, graph, console](#4-inspecting-output-show-graph-console)
5. [State commands](#5-state-commands)
6. [Import, moved, removed](#6-import-moved-removed)
7. [Workspaces](#7-workspaces)
8. [Providers and the lock file](#8-providers-and-the-lock-file)
9. [Environment variables and logging](#9-environment-variables-and-logging)
10. [HCL syntax](#10-hcl-syntax)
11. [Types, expressions and functions](#11-types-expressions-and-functions)
12. [Meta-arguments and lifecycle](#12-meta-arguments-and-lifecycle)
13. [Variables: ways to set them and precedence](#13-variables-ways-to-set-them-and-precedence)
14. [Files in a Terraform repository](#14-files-in-a-terraform-repository)
15. [Supporting tools](#15-supporting-tools)
16. [Verification commands for AEM-related changes](#16-verification-commands-for-aem-related-changes)
17. [One-page cheat sheet](#17-one-page-cheat-sheet)

---

## 1. Installing and getting help

```bash
terraform version                 # CLI + provider versions
terraform -help                   # all commands
terraform plan -help              # options of one command
terraform -install-autocomplete   # shell completion
```
Windows: `winget install Hashicorp.Terraform` (or `choco install terraform`). Linux/macOS: HashiCorp package repo or `brew install terraform`. Version managers such as `tfenv` keep several versions side by side.

---

## 2. Core workflow commands

| Command | Does |
|---------|------|
| `terraform init` | Download providers and modules, configure backend; run first and after changing providers/backend/modules |
| `terraform init -upgrade` | Upgrade providers/modules within version constraints |
| `terraform init -reconfigure` | Re-initialise backend ignoring saved config |
| `terraform init -migrate-state` | Move state to a new backend |
| `terraform init -backend-config=prod.backend.hcl` | Supply backend settings from a file (per environment) |
| `terraform fmt` / `fmt -recursive` / `fmt -check` | Format code / all folders / fail if unformatted (CI) |
| `terraform validate` | Syntax and internal consistency (no API calls) |
| `terraform plan` | Show proposed changes |
| `terraform apply` | Plan + ask for confirmation + apply |
| `terraform destroy` | Destroy everything in this state (same as `apply -destroy`) |

---

## 3. Plan and apply options

```bash
terraform plan -out=tfplan                     # save the exact plan
terraform apply tfplan                         # apply exactly that plan (no new prompt)
terraform plan -var-file=prod.tfvars
terraform plan -var='environment=prod'
terraform plan -target=aws_route53_record.site # limit to one resource (emergency use only)
terraform plan -replace=aws_instance.agent[0]  # force recreation (replaces old "taint")
terraform plan -refresh-only                   # only detect drift; apply -refresh-only accepts it into state
terraform plan -refresh=false                  # skip reading real objects (faster, riskier)
terraform plan -destroy                        # preview a destroy
terraform plan -detailed-exitcode              # exit 0 = no changes, 2 = changes, 1 = error (CI drift checks)
terraform plan -input=false -no-color -lock-timeout=5m   # CI-friendly
terraform apply -auto-approve                  # no prompt (CI after a human approved the plan)
terraform apply -parallelism=5                 # fewer concurrent API calls (rate limits)
```
Plan symbols: `+` create · `~` update in place · `-` destroy · `-/+` destroy then create (replace) · `+/-` create then destroy · `<=` read data source.

---

## 4. Inspecting: output, show, graph, console

```bash
terraform output                     # all outputs
terraform output -raw site_fqdn      # one value, no quotes (for scripts)
terraform output -json
terraform show                       # current state, human readable
terraform show tfplan                # a saved plan
terraform show -json tfplan > plan.json   # machine readable (policy checks)
terraform graph | dot -Tpng > graph.png   # dependency graph (needs Graphviz)
terraform console                    # interactive: try expressions and functions
  > cidrsubnet("10.0.0.0/16", 8, 2)
  > upper(var.environment)
terraform providers                  # providers required by the configuration
terraform providers schema -json     # every resource attribute of installed providers
```

---

## 5. State commands

```bash
terraform state list                                 # every resource address in state
terraform state list module.dns                      # inside a module
terraform state show 'aws_route53_record.aem["www.example.com"]'
terraform state mv aws_instance.old aws_instance.new # rename without destroy (prefer moved {} block)
terraform state rm aws_route53_record.legacy         # forget an object (it keeps existing in the real world)
terraform state pull > backup.tfstate                # download remote state
terraform state push backup.tfstate                  # upload (dangerous; last resort)
terraform force-unlock <LOCK_ID>                     # clear a stale lock after confirming no run is active
```
Resource addresses: `type.name`, `type.name[0]` (count), `type.name["key"]` (for_each), `module.dns.type.name`, `data.type.name`. Quote addresses with brackets in the shell.

---

## 6. Import, moved, removed

**Bring an existing object under Terraform** (e.g. a DNS record created by hand during a go-live):
```bash
terraform import aws_route53_record.site Z123456ABC_www.example.com_CNAME
```
or declaratively (Terraform 1.5+):
```hcl
import {
  to = aws_route53_record.site
  id = "Z123456ABC_www.example.com_CNAME"
}
```
```bash
terraform plan -generate-config-out=generated.tf   # writes resource blocks for import blocks
```

**Rename without destroying**:
```hcl
moved {
  from = aws_route53_record.www
  to   = aws_route53_record.aem["www.example.com"]
}
```

**Stop managing without destroying** (1.7+):
```hcl
removed {
  from = aws_route53_record.legacy
  lifecycle { destroy = false }
}
```

---

## 7. Workspaces

```bash
terraform workspace list
terraform workspace new stage
terraform workspace select prod
terraform workspace show
terraform workspace delete stage
```
In code: `terraform.workspace` gives the current name. Each workspace has its own state in the same backend. Many teams prefer separate folders/backends per environment for production isolation.

---

## 8. Providers and the lock file

```hcl
terraform {
  required_version = ">= 1.6, < 2.0"
  required_providers {
    aws        = { source = "hashicorp/aws",        version = "~> 5.0" }   # >=5.0, <6.0
    cloudflare = { source = "cloudflare/cloudflare", version = "~> 4.0" }
    fastly     = { source = "fastly/fastly",         version = ">= 5.0" }
    github     = { source = "integrations/github",   version = "~> 6.0" }
  }
}
provider "aws" { region = "eu-west-1" }
provider "aws" {                 # second configuration via alias
  alias  = "us"
  region = "us-east-1"
}
resource "aws_acm_certificate" "cdn" {
  provider    = aws.us           # use the alias
  domain_name = "www.example.com"
  validation_method = "DNS"
}
```
- `.terraform.lock.hcl` pins exact provider versions and checksums: **commit it**.
- `.terraform/` folder holds downloaded providers: **do not commit**.
- `terraform providers lock -platform=linux_amd64 -platform=windows_amd64` adds checksums for other OSs (CI on Linux, laptop on Windows).
- `terraform providers mirror ./mirror` for offline/air-gapped environments.

Version constraints: `= 1.2.0` exact · `>= 1.2` · `~> 1.2` (≥1.2, <2.0) · `~> 1.2.3` (≥1.2.3, <1.3.0).

---

## 9. Environment variables and logging

| Variable | Use |
|----------|-----|
| `TF_VAR_environment=prod` | Set input variable `environment` |
| `TF_LOG=TRACE|DEBUG|INFO|WARN|ERROR` | Logging level |
| `TF_LOG_PATH=./tf.log` | Write log to a file |
| `TF_LOG_PROVIDER=DEBUG` | Only provider logs |
| `TF_CLI_ARGS_plan="-input=false"` | Default args for a command |
| `TF_INPUT=0` | Never prompt (CI) |
| `TF_IN_AUTOMATION=1` | Shorter output for CI |
| `TF_DATA_DIR` | Other location for `.terraform` |
| `TF_PLUGIN_CACHE_DIR` | Shared provider cache |
| Provider credentials | `AWS_PROFILE` / `AWS_ACCESS_KEY_ID`…, `ARM_CLIENT_ID`/`ARM_SUBSCRIPTION_ID`… (Azure), `CLOUDFLARE_API_TOKEN`, `FASTLY_API_KEY`, `GITHUB_TOKEN` |

PowerShell: `$env:TF_LOG="DEBUG"; terraform plan`. Bash: `TF_LOG=DEBUG terraform plan`.

---

## 10. HCL syntax

```hcl
# Comments: #, //, /* ... */

# Block: TYPE "LABEL" "LABEL" { arguments and nested blocks }
resource "aws_security_group" "author" {
  name   = "author-${var.environment}"      # argument with interpolation
  vpc_id = var.vpc_id

  ingress {                                  # nested block
    description = "Authors from offices"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = var.office_cidrs
  }

  dynamic "ingress" {                        # generate nested blocks from a collection
    for_each = var.extra_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["10.0.0.0/16"]
    }
  }

  tags = local.tags
}

variable "office_cidrs" {
  type        = list(string)
  description = "Office egress ranges"
  default     = []
  sensitive   = false
  nullable    = false
  validation {
    condition     = alltrue([for c in var.office_cidrs : can(cidrhost(c, 0))])
    error_message = "Every entry must be a valid CIDR, e.g. 203.0.113.0/24."
  }
}

output "author_sg_id" {
  value       = aws_security_group.author.id
  description = "Security group of author"
  sensitive   = false
}

check "site_responds" {                      # continuous validation (1.5+)
  data "http" "home" { url = "https://www.example.com/" }
  assert {
    condition     = data.http.home.status_code == 200
    error_message = "Home page did not return 200."
  }
}
```

Heredoc strings:
```hcl
user_data = <<-EOT
  #!/bin/bash
  echo "env=${var.environment}" > /etc/app.env
EOT
```

---

## 11. Types, expressions and functions

**Types**: `string`, `number`, `bool`, `list(T)`, `set(T)`, `map(T)`, `object({name=string, port=number})`, `tuple([string, number])`, `any`, `optional(T, default)` inside objects.

**References**: `var.x` · `local.x` · `resource_type.name.attr` · `data.type.name.attr` · `module.name.output` · `each.key` / `each.value` · `count.index` · `path.module` / `path.root` · `terraform.workspace` · `self` (in provisioners).

**Expressions**:
```hcl
condition ? a : b                                         # conditional
[for h in var.hostnames : lower(h)]                       # list comprehension
{for k, v in var.sites : k => v.domain if v.enabled}      # map comprehension with filter
aws_instance.agent[*].private_ip                          # splat
"${var.env}-site"                                         # interpolation
%{ if var.env == "prod" }live%{ else }test%{ endif }      # template directive
```

**Functions you will use most**:
| Group | Functions |
|-------|-----------|
| String | `format`, `join`, `split`, `lower`, `upper`, `replace`, `trimspace`, `substr`, `regex`, `startswith`, `endswith` |
| Collection | `length`, `concat`, `merge`, `flatten`, `distinct`, `contains`, `keys`, `values`, `lookup`, `element`, `zipmap`, `toset`, `tolist`, `tomap`, `setunion`, `coalesce` |
| Network | `cidrsubnet`, `cidrhost`, `cidrnetmask`, `cidrsubnets` |
| Encoding | `jsonencode`, `jsondecode`, `yamlencode`, `yamldecode`, `base64encode`, `base64decode` |
| Files | `file`, `templatefile`, `fileexists`, `filebase64` |
| Type/errors | `can`, `try`, `type`, `tostring`, `tonumber` |
| Crypto/time | `sha256`, `md5`, `uuid`, `timestamp`, `formatdate` |

Example: render a Dispatcher-like config or a policy file from a template:
```hcl
resource "local_file" "vhost" {
  filename = "${path.module}/out/site.vhost"
  content  = templatefile("${path.module}/templates/site.vhost.tftpl", { hosts = var.hostnames })
}
```

---

## 12. Meta-arguments and lifecycle

```hcl
resource "aws_route53_record" "aem" {
  for_each = toset(var.hostnames)          # one per hostname; address uses the key
  # count = 3                              # alternative: index-based (avoid for things that get removed from the middle)
  provider   = aws
  depends_on = [aws_route53_zone.main]

  lifecycle {
    prevent_destroy       = true           # plan fails if this would be destroyed (protect prod DNS)
    create_before_destroy = true           # replace without gap
    ignore_changes        = [ttl]          # someone else manages this attribute
    replace_triggered_by  = [terraform_data.cert_version]
    precondition {
      condition     = can(regex("\\.example\\.com$", each.key))
      error_message = "Only example.com hostnames."
    }
  }
  zone_id = aws_route53_zone.main.zone_id
  name    = each.key
  type    = "CNAME"
  ttl     = 300
  records = ["cdn.adobeaemcloud.com"]
}
```
`count` vs `for_each`: with `count`, removing item 0 shifts every index and recreates resources; `for_each` keys are stable. Prefer `for_each` for named things like hostnames or CIDR entries.

Provisioners (`local-exec`, `remote-exec`) exist but are a last resort; prefer images/config tools.

---

## 13. Variables: ways to set them and precedence

Later wins:
1. `default` in the `variable` block
2. Environment variables `TF_VAR_name`
3. `terraform.tfvars` / `terraform.tfvars.json`
4. `*.auto.tfvars` (alphabetical)
5. `-var-file=...` and `-var=...` on the command line (in the order given)

```hcl
# prod.tfvars
environment  = "prod"
hostnames    = ["www.example.com", "www.example.co.uk"]
office_cidrs = ["203.0.113.0/24", "198.51.100.0/25"]
```
Secrets: mark variables `sensitive = true`, pass them from the CI credentials store or a vault (`data "vault_generic_secret"`, AWS Secrets Manager data sources), never in committed tfvars.

---

## 14. Files in a Terraform repository

| File | Purpose | Commit? |
|------|---------|---------|
| `main.tf` | Resources | Yes |
| `variables.tf` | Variable declarations | Yes |
| `outputs.tf` | Outputs | Yes |
| `providers.tf` / `versions.tf` | Provider and version constraints | Yes |
| `backend.tf` / `*.backend.hcl` | Backend config | Yes (no secrets) |
| `*.tfvars` | Values per environment | Yes, if no secrets |
| `.terraform.lock.hcl` | Provider version lock | **Yes** |
| `.terraform/` | Downloaded providers/modules | **No** |
| `terraform.tfstate`, `*.tfstate.backup` | Local state | **No** (use remote backend) |
| `tfplan` | Saved plan | No (may contain secrets) |
| `crash.log` | Crash report | No |

`.gitignore`:
```
.terraform/
*.tfstate
*.tfstate.*
tfplan
crash.log
*.auto.tfvars.secret
```

---

## 15. Supporting tools

| Tool | Command | Purpose |
|------|---------|---------|
| **tflint** | `tflint --init && tflint` | Lint: deprecated syntax, provider-specific mistakes |
| **checkov** / **tfsec** (now in Trivy) | `checkov -d .` / `trivy config .` | Security scan: open security groups, unencrypted storage |
| **terraform-docs** | `terraform-docs markdown table . > README.md` | Generate module docs |
| **infracost** | `infracost breakdown --path .` | Cost estimate of a plan |
| **pre-commit** | `pre-commit run -a` | Run fmt/validate/tflint before each commit |
| **Terragrunt** | `terragrunt run-all plan` | Keep backend/env config DRY across many folders |
| **OPA / Sentinel** | `conftest test plan.json` | Policy as code on `show -json` output |
| **Packer** | `packer build aem.pkr.hcl` | Build VM images (e.g. AEM 6.5 or Jenkins agent images) |
| **Ansible** | `ansible-playbook site.yml` | Configure servers after Terraform creates them |

---

## 16. Verification commands for AEM-related changes

After applying DNS, CDN, certificate or firewall changes, verify from the outside, the way you would in an incident (Module 1):
```bash
dig www.example.com CNAME +short            # expect cdn.adobeaemcloud.com
dig _aemverification.www.example.com TXT +short
dig @8.8.8.8 www.example.com +short          # public resolver (propagation)
nslookup www.example.com                     # Windows equivalent
curl -sIL https://www.example.com/           # status, redirects, cache headers
curl -sI https://www.example.com/ -H "Host: www.example.com" --resolve www.example.com:443:<ip>
openssl s_client -connect www.example.com:443 -servername www.example.com </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
curl -s https://ifconfig.me                  # your public egress IP (is it in the CIDR list?)
python -c "import ipaddress;print(ipaddress.ip_address('203.0.113.45') in ipaddress.ip_network('203.0.113.0/24'))"
```
PowerShell equivalents: `Resolve-DnsName www.example.com -Type CNAME`, `Invoke-WebRequest -Method Head https://www.example.com`, `Test-NetConnection www.example.com -Port 443`.

---

## 17. One-page cheat sheet

```text
SETUP      init [-upgrade|-reconfigure|-migrate-state|-backend-config=f] · fmt -recursive · validate
PLAN       plan [-out=f] [-var-file=f] [-var k=v] [-target=a] [-replace=a] [-refresh-only] [-destroy] [-detailed-exitcode]
APPLY      apply [planfile] [-auto-approve] · destroy
INSPECT    output [-raw|-json] · show [planfile] [-json] · graph · console · providers
STATE      state list|show|mv|rm|pull|push · force-unlock ID · import ADDR ID
CODE BLKS  terraform · provider · variable · locals · data · resource · module · output · import · moved · removed · check
META       count · for_each · depends_on · provider · lifecycle{prevent_destroy, create_before_destroy, ignore_changes, replace_triggered_by, precondition}
REFS       var. · local. · type.name.attr · data. · module. · each.key/value · count.index · path.module
VARS       default < TF_VAR_ < terraform.tfvars < *.auto.tfvars < -var-file/-var
DEBUG      TF_LOG=DEBUG · TF_LOG_PATH=f · plan -refresh-only (drift) · state show ADDR
COMMIT     *.tf, *.tfvars (no secrets), .terraform.lock.hcl     NEVER: .terraform/, *.tfstate, tfplan
VERIFY     dig/nslookup · curl -sIL · openssl s_client · Test-NetConnection
```
