# Terraform Notes

## Types of IaC Tools
- **Configuration Management**: Ansible, Puppet, Chef…  
- **Provisioning Tools**: Terraform, CloudFormation…  
- **Server Templating**: Docker, Packer, Vagrant…  

---

## Terraform Basics
- Built using **Go** language.  
- Uses **HCL (HashiCorp Configuration Language)**.  
- Declarative: the code defines the **desired state** of the infrastructure.  

### Terraform Workflow
1. **terraform init** – Initialize providers and backend.  
2. **terraform plan** – Preview execution plan.  
3. **terraform apply** – Apply changes to reach desired state.  

### Terraform State
- Ensures infrastructure always matches the declared configuration.  
- **Resource**: An object Terraform manages.  

---

## Example: AWS S3 Bucket
```hcl
resource "aws_s3_bucket" "example" {
  bucket = "webserver-bucket-org-2207"
  acl    = "private"
}
```

### Steps
1. Write configuration file (`.tf`).  
2. Run `terraform init`.  
3. Run `terraform plan`.  
4. Run `terraform apply`.  
5. Use `terraform show` to view details.  

> 📌 Resource types can always be found in the [Terraform Registry](https://registry.terraform.io/browse/providers).  

### Updating & Deleting
- To update: run `terraform plan` → `terraform apply`.  
- To delete: run `terraform destroy`.  

---

## Variables
```hcl
variable "bucket" {
  default     = "webserver-bucket-org-2207"
  type        = string   # optional: string, number, bool, list, map, set, object, tuple, any
  description = "S3 bucket name"
}

variable "acl" {
  default = "private"
}

resource "aws_s3_bucket" "example" {
  bucket = var.bucket
  acl    = var.acl
}
```

### Ways to Provide Variables
1. **Interactive Mode**: leave variable empty → prompt on `apply`.  
2. **Command Line Flags**:  
   ```bash
   terraform apply -var "bucket=my-bucket" -var "acl=private"
   ```
3. **Environment Variables**:  
   ```bash
   export TF_VAR_bucket="my-bucket"
   export TF_VAR_acl="private"
   terraform apply
   ```
4. **Variable Definition Files** (`.tfvars`):  
   ```hcl
   bucket = "my-bucket"
   acl    = "private"
   ```
   Run: `terraform apply -var-file=variables.tfvars`  

📌 Auto-loaded filenames: `terraform.tfvars`, `terraform.tfvars.json`, `*.auto.tfvars`, `*.auto.tfvars.json`.  

### Variable Priority Order
1. CLI flags (`-var`, `-var-file`)  
2. `*.auto.tfvars` (alphabetical)  
3. `terraform.tfvars`  
4. Environment variables  

---

## Resource Attribute Reference
- Syntax: `${resource_type.resource_name.attribute}`  

```hcl
output "bucket_name" {
  value = aws_s3_bucket.example.bucket
}
```
Command: `terraform output`  

---

## Terraform State (`terraform.tfstate`)
- Stores **current state** of infrastructure.  
- Contains **sensitive data** → should NOT be stored in GitHub.  

### Purpose of State
- **Tracking Metadata**: keeps resource IDs and dependencies.  
- **Performance**: speeds up planning.  
- **Collaboration**: shared remote state for teams.  

### Remote State
- Store state remotely in **S3, Terraform Cloud, Consul, etc.**  

---

## Common Terraform Commands
- `terraform validate` → check syntax only.  
- `terraform fmt` → format code.  
- `terraform show` → show current state.  
- `terraform providers` → list used providers.  
- `terraform output` → print output variables.  
- `terraform refresh` → sync state with real infra.  
- `terraform graph` → visualize dependencies (export DOT file).  

---

## Mutable vs Immutable
- **Mutable**: updates resource in place.  
- **Immutable**: destroys and recreates resource.  

### Lifecycle Rules
- `create_before_destroy`: new resource before destroying old one.  
- `prevent_destroy`: protect resource from deletion.  
- `ignore_changes`: ignore attribute changes.  

Example:
```hcl
resource "aws_s3_bucket" "example" {
  bucket = "webserver-bucket-org-2207"
  acl    = "private"

  lifecycle {
    prevent_destroy = true
  }
}
```

---

## Data Sources vs Resources
- **Resource** (`resource`) → Creates, updates, deletes infra.  
- **Data Source** (`data`) → Reads existing infra only.  

Example:
```hcl
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
}
```

---

## Meta-Arguments
- **count** → create multiple instances.  
- **for_each** → loop over maps/sets.  
- **depends_on** → enforce dependency order.  

Example:
```hcl
resource "aws_instance" "example" {
  count = 3
  ami   = "ami-123456"
  type  = "t2.micro"
}
```

---

## Version Constraints
```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 1.2"
    }
  }
}
```

---

## Providers
- Providers manage interaction with specific platforms (AWS, GCP, Azure, Kubernetes, etc.).  
- Credentials are stored in `~/.aws/config` or `~/.aws/credentials`.  

Example:
```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_iam_user" "admin_user" {
  name = "lucy"
  tags = {
    Description = "Technical Team Leader"
  }
}
```

### AWS Credentials File
`~/.aws/credentials`
```
[default]
aws_access_key_id     = AKIAI44QH8DHBEXAMPLE
aws_secret_access_key = je7MtGbClwBF/2tk/h3yCo8n...
```

---

## Documentation
- Official Docs: [https://developer.hashicorp.com/terraform](https://developer.hashicorp.com/terraform)  
- Provider Registry: [https://registry.terraform.io](https://registry.terraform.io)  
