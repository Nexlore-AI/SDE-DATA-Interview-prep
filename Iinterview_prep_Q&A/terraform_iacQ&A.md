# Terraform & Infrastructure as Code (IaC) — Interview Q&A

---

## 1. What is Infrastructure as Code (IaC)? Why is it important?

IaC is managing and provisioning infrastructure through **code/configuration files** instead of manual processes.

**Benefits:**
- **Version control:** Track infrastructure changes in Git
- **Reproducibility:** Same code = same infrastructure every time
- **Automation:** No manual clicking in cloud consoles
- **Peer review:** PRs for infrastructure changes
- **Documentation:** Code IS the documentation of your infra
- **Disaster recovery:** Recreate entire infrastructure from code

**Tools:** Terraform, Pulumi, CloudFormation (AWS), ARM/Bicep (Azure), Deployment Manager (GCP).

---

## 2. What is Terraform and how does it differ from CloudFormation?

| Feature | Terraform | CloudFormation |
|---------|-----------|----------------|
| Provider | Multi-cloud (AWS, GCP, Azure, etc.) | AWS only |
| Language | HCL (HashiCorp Configuration Language) | JSON / YAML |
| State | Managed externally (S3, Terraform Cloud) | Managed by AWS automatically |
| Community | Huge ecosystem of providers/modules | AWS-only ecosystem |
| Drift detection | `terraform plan` shows drift | Stack drift detection |

**Terraform** is preferred for multi-cloud or cloud-agnostic setups. **CloudFormation** is fine if you're AWS-only and want tighter integration.

---

## 3. Explain the Terraform workflow.

```
terraform init → terraform plan → terraform apply → terraform destroy
```

1. **`terraform init`** — Downloads providers and modules, initializes backend
2. **`terraform plan`** — Shows what changes will be made (dry run). Creates execution plan.
3. **`terraform apply`** — Executes the plan. Creates/modifies/destroys resources.
4. **`terraform destroy`** — Tears down all managed resources.

---

## 4. What is Terraform state? Why is it important?

Terraform state (`terraform.tfstate`) is a **JSON file that maps your config to real-world resources**.

**Why it matters:**
- Tracks which resources Terraform manages
- Stores resource attributes (IDs, IPs, etc.)
- Determines what changes are needed on `terraform plan`
- Without state, Terraform would try to create everything from scratch

**Best practices:**
- **Remote state:** Store in S3 + DynamoDB (locking), Terraform Cloud, or GCS
- **Never** commit `terraform.tfstate` to Git (contains secrets)
- **State locking:** Prevents concurrent modifications (DynamoDB lock table for S3 backend)

---

## 5. What are Terraform providers and modules?

**Provider:** Plugin that interacts with a cloud/service API.
```hcl
provider "aws" {
  region = "us-east-1"
}
```

**Module:** Reusable, encapsulated bundle of Terraform config.
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"
  azs  = ["us-east-1a", "us-east-1b"]
}
```

**Modules are like functions** — parameterized and reusable. Use them to avoid duplication.

---

## 6. What is the difference between `terraform plan` output: create, update, destroy?

```
+ resource "aws_instance" "web"     # CREATE (new resource)
~ resource "aws_instance" "web"     # UPDATE in-place (modify existing)
-/+ resource "aws_instance" "web"   # DESTROY and RECREATE (forced replacement)
- resource "aws_instance" "web"     # DESTROY (remove resource)
```

**Forced replacement** happens when you change an attribute that can't be updated in-place (e.g., AMI of an EC2 instance).

---

## 7. What are Terraform variables and outputs?

**Variables (inputs):**
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

**Outputs:**
```hcl
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

**Variable precedence (highest to lowest):**
1. `-var` flag on CLI
2. `*.auto.tfvars` files
3. `terraform.tfvars` file
4. Environment variables (`TF_VAR_name`)
5. Default value in variable block

---

## 8. What is a Terraform backend?

The backend defines **where Terraform stores state** and **how operations are executed**.

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

**Common backends:** local (default), S3, GCS, Azure Blob, Terraform Cloud.

---

## 9. What is `terraform import` and when would you use it?

`terraform import` brings **existing resources** into Terraform management.

```bash
terraform import aws_instance.web i-1234567890abcdef0
```

**Use case:** You created resources manually (console/CLI) and now want to manage them with Terraform.

**Limitation:** Only imports state. You must write the corresponding HCL config manually.

---

## 10. What is the difference between Terraform and Ansible?

| Feature | Terraform | Ansible |
|---------|-----------|---------|
| Purpose | Infrastructure provisioning | Configuration management |
| Approach | Declarative ("I want 3 servers") | Procedural/Declarative ("Install nginx, start service") |
| State | Maintains state file | Stateless (idempotent tasks) |
| Best for | Creating cloud resources | Configuring servers, deploying apps |
| Language | HCL | YAML (playbooks) |

**Common pattern:** Terraform provisions infra → Ansible configures it.

---

## 11. What are Terraform workspaces?

Workspaces allow you to manage **multiple environments** (dev, staging, prod) with the same code but separate state.

```bash
terraform workspace new dev
terraform workspace new prod
terraform workspace select dev
terraform workspace list
```

```hcl
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "prod" ? "m5.large" : "t3.micro"
}
```

**Alternative approach:** Many teams prefer separate directories or separate state files per environment instead of workspaces for better isolation.

---

## 12. What is `terraform taint` and `terraform untaint`?

- **`terraform taint`** marks a resource for **destruction and recreation** on next apply.
- **`terraform untaint`** removes the taint.

```bash
terraform taint aws_instance.web    # Will be destroyed + recreated on next apply
terraform untaint aws_instance.web  # Cancel the taint
```

**Use case:** When a resource is in a bad state (e.g., failed provisioner) and you want to force recreation.

**Note:** In newer Terraform, use `terraform apply -replace=aws_instance.web` instead.

---

## 13. How do you handle secrets in Terraform?

- **Never** hardcode secrets in `.tf` files
- Use `variable` blocks and pass via environment variables or `terraform.tfvars` (gitignored)
- Use **data sources** to fetch from secret managers:
```hcl
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db-password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```
- Mark outputs as `sensitive = true` to prevent display in logs
- Enable **state encryption** (S3 backend with `encrypt = true`)

---

## 14. What are Terraform data sources?

Data sources let you **read information** from existing resources (not managed by Terraform):

```hcl
data "aws_ami" "latest_ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-*"]
  }
}

resource "aws_instance" "web" {
  ami = data.aws_ami.latest_ubuntu.id
}
```

**Use case:** Look up AMI IDs, VPC IDs, existing security groups, etc.

---

## 15. What is a Terraform provisioner? Why are they discouraged?

Provisioners execute **scripts on resources** after creation:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t3.micro"

  provisioner "remote-exec" {
    inline = ["sudo apt update", "sudo apt install -y nginx"]
  }
}
```

**Types:** `local-exec` (runs on your machine), `remote-exec` (runs on the resource), `file` (copies files).

**Why discouraged:**
- Not tracked in state → drift
- Terraform can't detect if the script failed partially
- Better alternatives: cloud-init (user_data), Ansible, Packer (bake AMIs)
