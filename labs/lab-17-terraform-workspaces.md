# Lab 17 - Terraform Workspaces

## Objective

In this lab, you will learn how to use **Terraform Workspaces** to manage multiple environments such as **Development**, **Staging**, and **Production** using the same Terraform configuration.

Instead of creating separate Terraform projects for each environment, Workspaces allow you to maintain separate state files while sharing the same code.

---

## What You Will Build

- Development Workspace
- Staging Workspace
- Production Workspace
- Environment-specific EC2 Instances
- Separate Terraform State for Each Workspace

---

## What are Terraform Workspaces?

A **Terraform Workspace** is a feature that lets you manage multiple state files using the same Terraform configuration.

Each workspace has its own:

- Terraform State
- Infrastructure
- Resource Names

---

## Why Use Workspaces?

Without Workspaces:

- Multiple Terraform folders
- Duplicate code
- Difficult to manage

With Workspaces:

- Same Terraform code
- Separate state files
- Easy environment management
- Better organization

---

## Workspace Architecture

```text
Terraform Configuration
          │
          ▼
     Workspaces
          │
 ┌────────┼────────┐
 │        │        │
 ▼        ▼        ▼
dev    staging    prod
 │        │        │
 ▼        ▼        ▼
State    State    State
```

---

## Step 1 - Create the Lab File

```bash
touch labs/lab-17-terraform-workspaces.md
```

---

## Step 2 - Create the Project Directory

```bash
mkdir -p terraform/lab-17-workspaces

cd terraform/lab-17-workspaces
```

---

## Step 3 - Create Terraform Files

```bash
touch versions.tf
touch provider.tf
touch variables.tf
touch terraform.tfvars
touch main.tf
touch outputs.tf
```

---

## Verify the Structure

Run:

```bash
tree
```

Expected output:

```text
.
├── main.tf
├── outputs.tf
├── provider.tf
├── terraform.tfvars
├── variables.tf
└── versions.tf
```

---

## What You Will Learn

- Terraform Workspaces
- Workspace Commands
- Separate State Files
- Environment Isolation
- Workspace Variables
- Best Practices

---
# Step 1 - Create `versions.tf`

## Objective

Configure the Terraform version and AWS provider for the project.

---

## Open the File

```bash
nano versions.tf
```

---

## Add the Following Code

```hcl
terraform {

  required_version = ">= 1.5.0"

  required_providers {

    aws = {

      source  = "hashicorp/aws"

      version = "~> 6.0"

    }

  }

}
```

---

## Explanation

- `terraform {}` – Defines the Terraform project settings.
- `required_version` – Requires Terraform version **1.5.0 or later**.
- `required_providers` – Specifies the providers required by the project.
- `source = "hashicorp/aws"` – Uses the official AWS provider.
- `version = "~> 6.0"` – Uses AWS Provider version **6.x**.

---

## Initialize the Project

Run:

```bash
terraform fmt
terraform init
terraform validate
```
-------------
# Step 2 - Create `provider.tf`

## Objective

Configure the AWS provider for the Terraform project.

---

## Open the File

```bash
nano provider.tf
```

---

## Add the Following Code

```hcl
provider "aws" {

  region = var.aws_region

  default_tags {

    tags = {

      Project     = "terraform-zero-to-hero"

      Environment = terraform.workspace

      ManagedBy   = "Terraform"

    }

  }

}
```

---

## Explanation

- `provider "aws"` – Configures the AWS provider.
- `region = var.aws_region` – Uses the AWS Region from `terraform.tfvars`.
- `terraform.workspace` – Automatically sets the `Environment` tag to the active workspace (such as `dev`, `staging`, or `prod`).
- `default_tags` – Applies the tags to all supported AWS resources.

Example:

If the active workspace is:

```bash
terraform workspace select dev
```

Resources will be tagged as:

```text
Environment = dev
```

If you switch to:

```bash
terraform workspace select prod
```

Resources will be tagged as:

```text
Environment = prod
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---------------
# Step 3 - Create `variables.tf`

## Objective

Define the input variables used in the Terraform configuration.

---

## Open the File

```bash
nano variables.tf
```

---

## Add the Following Code

```hcl
variable "aws_region" {

  description = "AWS Region"

  type = string

}

variable "instance_type" {

  description = "EC2 Instance Type"

  type = string

}

variable "key_name" {

  description = "AWS EC2 Key Pair Name"

  type = string

}
```

---

## Explanation

- `aws_region` – AWS Region where the resources will be created.
- `instance_type` – EC2 instance type to launch.
- `key_name` – Existing AWS EC2 Key Pair for SSH access.

These values are provided in the `terraform.tfvars` file.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---
# Step 4 - Create `terraform.tfvars`

## Objective

Assign values to the variables defined in `variables.tf`.

---

## Open the File

```bash
nano terraform.tfvars
```

---

## Add the Following Code

```hcl
aws_region = "ap-south-1"

instance_type = "t2.micro"

key_name = "terraform-key"
```

---

## Explanation

- `aws_region` – AWS Region where the EC2 instance will be created.
- `instance_type` – EC2 instance type used in all workspaces.
- `key_name` – Existing AWS EC2 Key Pair used for SSH access.

The same `terraform.tfvars` file is shared by all workspaces. Each workspace keeps its own Terraform state while using the same configuration.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---
# Step 5 - Create `main.tf`

## Objective

Create an EC2 instance and use the active Terraform workspace to generate a unique name for each environment.

---

## Open the File

```bash
nano main.tf
```

---

## Add the Following Code

```hcl
data "aws_ami" "amazon_linux" {

  most_recent = true

  owners = ["amazon"]

  filter {

    name = "name"

    values = ["al2023-ami-*-x86_64"]

  }

}

resource "aws_instance" "web" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = var.instance_type

  key_name = var.key_name

  tags = {

    Name = "web-${terraform.workspace}"

  }

}
```

---

## Explanation

### Data Source

```hcl
data "aws_ami" "amazon_linux"
```

Finds the latest Amazon Linux 2023 AMI automatically.

---

### EC2 Instance

```hcl
resource "aws_instance" "web"
```

Creates an EC2 instance.

---

### Workspace Name

```hcl
Name = "web-${terraform.workspace}"
```

Uses the active workspace name to create a unique EC2 name.

Examples:

If the current workspace is:

```bash
terraform workspace select dev
```

The EC2 instance name becomes:

```text
web-dev
```

If the workspace is:

```bash
terraform workspace select staging
```

The EC2 instance name becomes:

```text
web-staging
```

If the workspace is:

```bash
terraform workspace select prod
```

The EC2 instance name becomes:

```text
web-prod
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```
---
# Step 6 - Create `outputs.tf`

## Objective

Display useful information after the infrastructure is created for each Terraform workspace.

---

## Open the File

```bash
nano outputs.tf
```

---

## Add the Following Code

```hcl
output "workspace_name" {

  description = "Current Terraform Workspace"

  value = terraform.workspace

}

output "instance_id" {

  description = "EC2 Instance ID"

  value = aws_instance.web.id

}

output "public_ip" {

  description = "EC2 Public IP"

  value = aws_instance.web.public_ip

}
```

---

## Explanation

### Workspace Name

```hcl
output "workspace_name"
```

Displays the active Terraform workspace.

Example:

```text
workspace_name = "dev"
```

---

### EC2 Instance ID

```hcl
output "instance_id"
```

Displays the ID of the EC2 instance created in the current workspace.

Example:

```text
instance_id = "i-0123456789abcdef0"
```

---

### Public IP

```hcl
output "public_ip"
```

Displays the public IP address of the EC2 instance.

Example:

```text
public_ip = "13.233.120.45"
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```

---
# Step 7 - Create Terraform Workspaces

## Objective

Create separate workspaces for **Development**, **Staging**, and **Production** environments.

Each workspace maintains its own Terraform state while using the same Terraform configuration.

---

## View the Current Workspace

Run:

```bash
terraform workspace show
```

Expected output:

```text
default
```

---

## List Existing Workspaces

Run:

```bash
terraform workspace list
```

Expected output:

```text
* default
```

The `*` indicates the active workspace.

---

## Create the Development Workspace

Run:

```bash
terraform workspace new dev
```

Expected output:

```text
Created and switched to workspace "dev"!
```

Verify:

```bash
terraform workspace show
```

Output:

```text
dev
```

---

## Create the Staging Workspace

Run:

```bash
terraform workspace new staging
```

---

## Create the Production Workspace

Run:

```bash
terraform workspace new prod
```

---

## List All Workspaces

Run:

```bash
terraform workspace list
```

Expected output:

```text
default

dev

prod

staging
```

The active workspace is marked with `*`.

Example:

```text
default

* dev

prod

staging
```

---

## Switch Between Workspaces

Switch to the **Development** workspace:

```bash
terraform workspace select dev
```

Switch to the **Staging** workspace:

```bash
terraform workspace select staging
```

Switch to the **Production** workspace:

```bash
terraform workspace select prod
```

---

## Verify the Active Workspace

Run:

```bash
terraform workspace show
```

Example:

```text
prod
```

---

## Summary

You created:

- Development Workspace
- Staging Workspace
- Production Workspace

You also learned how to:

- View the current workspace
- List all workspaces
- Create new workspaces
- Switch between workspaces

---
# Step 8 - Deploy Infrastructure in Each Workspace

## Objective

Deploy the same Terraform configuration to multiple environments using different workspaces.

Each workspace creates its own infrastructure and maintains a separate Terraform state file.

---

## Deploy to the Development Environment

Select the workspace:

```bash
terraform workspace select dev
```

Verify:

```bash
terraform workspace show
```

Run:

```bash
terraform plan
```

Deploy:

```bash
terraform apply
```

Type:

```text
yes
```

Expected EC2 Name:

```text
web-dev
```

---

## Deploy to the Staging Environment

Switch workspace:

```bash
terraform workspace select staging
```

Run:

```bash
terraform plan

terraform apply
```

Type:

```text
yes
```

Expected EC2 Name:

```text
web-staging
```

---

## Deploy to the Production Environment

Switch workspace:

```bash
terraform workspace select prod
```

Run:

```bash
terraform plan

terraform apply
```

Type:

```text
yes
```

Expected EC2 Name:

```text
web-prod
```

---

## Verify the Outputs

Run:

```bash
terraform output
```

Example:

```text
workspace_name = "prod"

instance_id = "i-0123456789abcdef0"

public_ip = "13.233.120.45"
```

---

## Verify in AWS Console

You should see three EC2 instances:

```text
web-dev

web-staging

web-prod
```

Each instance belongs to a different Terraform workspace.

---

## Summary

You deployed:

- Development Environment
- Staging Environment
- Production Environment

Each workspace:

- Uses the same Terraform code
- Has its own Terraform state
- Creates separate AWS resources

---
# Lab Cleanup

## Objective

Destroy the infrastructure created in each Terraform workspace and remove local Terraform files.

---

## Step 1 - Destroy the Development Environment

Select the workspace:

```bash
terraform workspace select dev
```

Destroy the resources:

```bash
terraform destroy
```

Type:

```text
yes
```

Wait until you see:

```text
Destroy complete!
```

---

## Step 2 - Destroy the Staging Environment

Switch to the staging workspace:

```bash
terraform workspace select staging
```

Run:

```bash
terraform destroy
```

Type:

```text
yes
```

---

## Step 3 - Destroy the Production Environment

Switch to the production workspace:

```bash
terraform workspace select prod
```

Run:

```bash
terraform destroy
```

Type:

```text
yes
```

---

## Step 4 - Switch to the Default Workspace

```bash
terraform workspace select default
```

---

## Step 5 - Delete the Workspaces

Delete the Development workspace:

```bash
terraform workspace delete dev
```

Delete the Staging workspace:

```bash
terraform workspace delete staging
```

Delete the Production workspace:

```bash
terraform workspace delete prod
```

> **Note:** A workspace must be empty before it can be deleted.

---

## Step 6 - Verify Remaining Workspaces

Run:

```bash
terraform workspace list
```

Expected output:

```text
* default
```

---

## Step 7 - Remove Local Terraform Files

Run:

```bash
rm -rf .terraform

rm -f .terraform.lock.hcl

rm -f terraform.tfstate

rm -f terraform.tfstate.backup

rm -f crash.log
```

---

## Step 8 - Check Git Status

```bash
git status
```

---

## Step 9 - Commit Changes

```bash
git add .

git commit -m "Complete Lab 17 Terraform Workspaces"
```

---

## Step 10 - Push to GitHub

```bash
git push origin main
```

---

# Lab Completed

You learned:

- Terraform Workspaces
- Workspace Commands
- Environment Isolation
- Separate State Files
- Workspace Variables
- Deploying Multiple Environments
- Infrastructure Cleanup

---

# Congratulations!

You have successfully completed **Lab 17 – Terraform Workspaces**.

You now know how to manage multiple environments such as **Development**, **Staging**, and **Production** using a single Terraform configuration.

