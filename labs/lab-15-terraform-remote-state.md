# Lab 15 - Terraform Remote State (S3 + DynamoDB)

## Objective

In this lab, you will configure a **remote Terraform state** using an Amazon S3 bucket and enable **state locking** with a DynamoDB table.

This allows multiple team members to work safely on the same Terraform project.

---

## What You Will Build

- S3 Bucket for Terraform State
- DynamoDB Table for State Locking
- Terraform Backend Configuration
- Remote State Storage
- State Locking
- State Migration

---

## What is Terraform State?

Terraform stores information about your infrastructure in a **state file**.

Example:

```text
terraform.tfstate
```

The state file keeps track of:

- Resources created
- Resource IDs
- Dependencies
- Current infrastructure

Without the state file, Terraform cannot determine what already exists.

---

## Why Use Remote State?

Storing the state file on your local computer has several drawbacks:

- Only one person has the latest state.
- The state file can be lost.
- Teams cannot collaborate easily.

Using an S3 bucket allows the entire team to share the same Terraform state.

---

## Why Use DynamoDB?

If two users run:

```bash
terraform apply
```

at the same time, the state file could become corrupted.

A DynamoDB table provides **state locking**, ensuring only one user can modify the state at a time.

---

## Architecture

```text
             Terraform

                 |

         Remote Backend

         /             \

        /               \

   S3 Bucket      DynamoDB Table

(State Storage)   (State Locking)
```

---

## Step 1 - Create the Lab File

```bash
touch labs/lab-15-terraform-remote-state.md
```

---

## Step 2 - Create the Project Directory

```bash
mkdir -p terraform/lab-15-remote-state
cd terraform/lab-15-remote-state
```

---

## Step 3 - Create Terraform Files

```bash
touch versions.tf
touch provider.tf
touch variables.tf
touch terraform.tfvars
touch s3.tf
touch dynamodb.tf
touch backend.tf
touch outputs.tf
```

---

## Step 4 - Verify the Files

Run:

```bash
ls
```

Expected output:

```text
backend.tf
dynamodb.tf
outputs.tf
provider.tf
s3.tf
terraform.tfvars
variables.tf
versions.tf
```

---

## What You Will Learn

- Terraform State
- Remote Backend
- S3 Backend
- DynamoDB State Locking
- Backend Configuration
- State Migration
- Team Collaboration

---

## Lab Outcome

At the end of this lab, you will have:

- An S3 bucket storing the Terraform state file
- A DynamoDB table providing state locking
- A Terraform backend configured to use remote state
- A production-ready Terraform backend

---
# Step 1 - Create `versions.tf`

## Objective

Configure the Terraform version and AWS provider required for this project.

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
- `required_version` – Ensures Terraform version **1.5.0 or later** is used.
- `required_providers` – Specifies the providers required for the project.
- `source = "hashicorp/aws"` – Downloads the official AWS provider.
- `version = "~> 6.0"` – Uses AWS Provider **6.x** and prevents automatic upgrades to **7.x**.

---

## Initialize the Project

Run:

```bash
terraform fmt
terraform init
terraform validate
```

- `terraform fmt` – Formats Terraform files.
- `terraform init` – Initializes the project and downloads the AWS provider.
- `terraform validate` – Checks the configuration for syntax and validation errors.

---
# Step 2 - Create `provider.tf`

## Objective

Configure the AWS provider and define default tags for the resources created in this lab.

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

      Environment = "Lab-15"

      ManagedBy   = "Terraform"

    }

  }

}
```

---

## Explanation

- `provider "aws"` – Configures the AWS provider.
- `region = var.aws_region` – Uses the AWS Region defined in `terraform.tfvars`.
- `default_tags` – Automatically applies tags to supported AWS resources.
- `Project`, `Environment`, and `ManagedBy` help identify and organize resources in AWS.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```

- `terraform fmt` – Formats Terraform files.
- `terraform validate` – Checks the configuration for syntax and validation errors.

---
# Step 3 - Create `variables.tf`

## Objective

Define the input variables used to create the S3 bucket and DynamoDB table.

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

variable "bucket_name" {

  description = "S3 Bucket Name for Terraform State"

  type = string

}

variable "dynamodb_table_name" {

  description = "DynamoDB Table Name for State Locking"

  type = string

}
```

---

## Explanation

- `aws_region` – AWS Region where resources will be created.
- `bucket_name` – Name of the S3 bucket that stores the Terraform state file.
- `dynamodb_table_name` – Name of the DynamoDB table used for Terraform state locking.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```

- `terraform fmt` – Formats Terraform files.
- `terraform validate` – Checks the configuration for syntax and validation errors.

---
# Step 4 - Create `terraform.tfvars`

## Objective

Provide values for the variables declared in `variables.tf`.

---

## Open the File

```bash
nano terraform.tfvars
```

---

## Add the Following Code

```hcl
aws_region = "ap-south-1"

bucket_name = "terraform-zero-to-hero-state-12345"

dynamodb_table_name = "terraform-state-lock"
```

> **Note:** S3 bucket names must be globally unique. If the bucket name already exists, choose a different name (for example, add your initials or a random number).

---

## Explanation

- `aws_region` – AWS Region where the S3 bucket and DynamoDB table will be created.
- `bucket_name` – Unique S3 bucket name used to store the Terraform state file.
- `dynamodb_table_name` – Name of the DynamoDB table used for state locking.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```

- `terraform fmt` – Formats Terraform files.
- `terraform validate` – Checks the configuration for syntax and validation errors.

---
# Step 5 - Create `s3.tf`

## Objective

Create an S3 bucket to store the Terraform state file remotely.

This allows multiple users to share the same Terraform state.

---

## What is an S3 Backend?

An **S3 Backend** stores the Terraform state file in an Amazon S3 bucket instead of on your local machine.

Benefits:

- Centralized state storage
- Team collaboration
- State backup and durability
- Accessible from multiple computers

---

## Open the File

```bash
nano s3.tf
```

---

## Add the Following Code

```hcl
resource "aws_s3_bucket" "terraform_state" {

  bucket = var.bucket_name

  tags = {

    Name = "terraform-state-bucket"

  }

}

resource "aws_s3_bucket_versioning" "terraform_state" {

  bucket = aws_s3_bucket.terraform_state.id

  versioning_configuration {

    status = "Enabled"

  }

}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {

  bucket = aws_s3_bucket.terraform_state.id

  rule {

    apply_server_side_encryption_by_default {

      sse_algorithm = "AES256"

    }

  }

}
```

---

## Explanation

### S3 Bucket

```hcl
resource "aws_s3_bucket" "terraform_state"
```

Creates an S3 bucket to store the Terraform state file.

---

### Bucket Name

```hcl
bucket = var.bucket_name
```

Uses the unique bucket name defined in `terraform.tfvars`.

---

### Versioning

```hcl
status = "Enabled"
```

Enables versioning for the S3 bucket.

Benefits:

- Keeps previous versions of the state file.
- Allows recovery if the state file is accidentally deleted or overwritten.

---

### Server-Side Encryption

```hcl
sse_algorithm = "AES256"
```

Encrypts the Terraform state file using Amazon S3 managed encryption (AES-256).

This helps protect sensitive information stored in the state file.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```

- `terraform fmt` – Formats Terraform files.
- `terraform validate` – Checks the configuration for syntax and validation errors.
- `terraform plan` – Shows the resources Terraform will create.

---
# Step 6 - Create `dynamodb.tf`

## Objective

Create a DynamoDB table to provide state locking for Terraform.

State locking prevents multiple users from modifying the same Terraform state file at the same time.

---

## What is State Locking?

When someone runs:

```bash
terraform apply
```

Terraform locks the state file before making changes.

This prevents another user from running Terraform commands that could corrupt the state.

---

## Open the File

```bash
nano dynamodb.tf
```

---

## Add the Following Code

```hcl
resource "aws_dynamodb_table" "terraform_lock" {

  name = var.dynamodb_table_name

  billing_mode = "PAY_PER_REQUEST"

  hash_key = "LockID"

  attribute {

    name = "LockID"

    type = "S"

  }

  tags = {

    Name = "terraform-state-lock"

  }

}
```

---

## Explanation

### DynamoDB Table

```hcl
resource "aws_dynamodb_table" "terraform_lock"
```

Creates a DynamoDB table for Terraform state locking.

---

### Table Name

```hcl
name = var.dynamodb_table_name
```

Uses the table name defined in `terraform.tfvars`.

---

### Billing Mode

```hcl
billing_mode = "PAY_PER_REQUEST"
```

Charges only for the read and write requests you use.

No capacity planning is required.

---

### Partition Key

```hcl
hash_key = "LockID"
```

Defines the partition key used by Terraform to store lock information.

---

### Attribute

```hcl
attribute {

  name = "LockID"

  type = "S"

}
```

Creates a string attribute named `LockID`, which Terraform uses to identify and manage state locks.

---

## How State Locking Works

```text
User A

↓

terraform apply

↓

State Locked

↓

Infrastructure Updated

↓

State Unlocked
```

If another user runs:

```bash
terraform apply
```

while the state is locked, Terraform waits or returns a lock error until the lock is released.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```

- `terraform fmt` – Formats Terraform files.
- `terraform validate` – Checks the configuration for syntax and validation errors.
- `terraform plan` – Shows the resources Terraform will create.

---
# Step 7 - Create `backend.tf`

## Objective

Configure Terraform to store the state file in the S3 bucket and use the DynamoDB table for state locking.

---

## What is a Backend?

A **backend** tells Terraform where to store the state file.

Instead of storing `terraform.tfstate` locally, Terraform stores it in an S3 bucket.

Benefits:

- Shared state
- Team collaboration
- State locking
- Secure storage

---

## Open the File

```bash
nano backend.tf
```

---

## Add the Following Code

```hcl
terraform {

  backend "s3" {

    bucket = "terraform-zero-to-hero-state-12345"

    key = "lab15/terraform.tfstate"

    region = "ap-south-1"

    dynamodb_table = "terraform-state-lock"

    encrypt = true

  }

}
```

> **Note:** Replace the bucket name with the one you created in `terraform.tfvars`.

---

## Explanation

### Backend

```hcl
backend "s3"
```

Configures Terraform to use Amazon S3 as the remote backend.

---

### Bucket

```hcl
bucket = "terraform-zero-to-hero-state-12345"
```

Specifies the S3 bucket where the Terraform state file will be stored.

---

### Key

```hcl
key = "lab15/terraform.tfstate"
```

Defines the path and filename of the state file inside the S3 bucket.

Example:

```text
terraform-zero-to-hero-state-12345
└── lab15
    └── terraform.tfstate
```

---

### Region

```hcl
region = "ap-south-1"
```

Specifies the AWS Region where the S3 bucket is located.

---

### DynamoDB Table

```hcl
dynamodb_table = "terraform-state-lock"
```

Uses the DynamoDB table for state locking.

Only one user can update the state file at a time.

---

### Encryption

```hcl
encrypt = true
```

Encrypts the Terraform state file before storing it in Amazon S3.

---

## Initialize the Backend

Since the backend configuration has changed, reinitialize Terraform:

```bash
terraform init
```

Terraform prompts you to migrate the existing local state.

Type:

```text
yes
```

Terraform copies the local `terraform.tfstate` file to the S3 bucket.

---

## Verify the Backend

Run:

```bash
terraform state list
```

Terraform now reads the state from the remote S3 backend.

---

## Validate the Configuration

Run:

```bash
terraform fmt

terraform validate
```

- `terraform fmt` – Formats Terraform files.
- `terraform validate` – Checks the configuration for syntax and validation errors.

------
# Step 8 - Create `outputs.tf`

## Objective

Display useful information about the Terraform remote backend after the infrastructure is created.

Outputs make it easy to verify the S3 bucket and DynamoDB table without opening the AWS Console.

---

## Open the File

```bash
nano outputs.tf
```

---

## Add the Following Code

```hcl
output "terraform_state_bucket" {

  description = "Terraform State S3 Bucket"

  value = aws_s3_bucket.terraform_state.bucket

}

output "terraform_lock_table" {

  description = "Terraform State Lock Table"

  value = aws_dynamodb_table.terraform_lock.name

}

output "bucket_region" {

  description = "AWS Region"

  value = var.aws_region

}
```

---

## Explanation

### Terraform State Bucket

```hcl
output "terraform_state_bucket"
```

Displays the S3 bucket name where the Terraform state file is stored.

Example:

```text
terraform_state_bucket = "terraform-zero-to-hero-state-12345"
```

---

### DynamoDB Lock Table

```hcl
output "terraform_lock_table"
```

Displays the DynamoDB table used for Terraform state locking.

Example:

```text
terraform_lock_table = "terraform-state-lock"
```

---

### AWS Region

```hcl
output "bucket_region"
```

Displays the AWS Region where the backend resources were created.

Example:

```text
bucket_region = "ap-south-1"
```

---

## Deploy the Infrastructure

Run:

```bash
terraform fmt
terraform validate
terraform plan
terraform apply
```

Type:

```text
yes
```

Wait until you see:

```text
Apply complete!
```

---

## Verify the Outputs

Run:

```bash
terraform output
```

Example:

```text
terraform_state_bucket = "terraform-zero-to-hero-state-12345"

terraform_lock_table = "terraform-state-lock"

bucket_region = "ap-south-1"
```

---

## Verify in AWS Console

Confirm the following resources were created:

- S3 Bucket
- Bucket Versioning Enabled
- Server-Side Encryption Enabled
- DynamoDB Table
- Terraform State File (after backend migration)

---

## Summary

You created:

- S3 Bucket
- Bucket Versioning
- Server-Side Encryption
- DynamoDB Table
- Remote Backend Configuration
- Terraform Outputs

You also learned:

- Terraform Remote State
- S3 Backend
- State Locking
- Backend Migration
- Team Collaboration

---
# Lab Cleanup

## Step 1 - Destroy AWS Resources

Run:

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

> **Note:** Before running `terraform destroy`, temporarily comment out or remove the `backend "s3"` block in `backend.tf` if needed, or ensure Terraform is initialized with the remote backend. Terraform must be able to access the remote state to destroy the resources.

---

## Step 2 - Verify AWS Console

Confirm these resources have been deleted:

- S3 Bucket
- DynamoDB Table

> **Note:** If the S3 bucket is not empty, Terraform cannot delete it. Empty the bucket first or configure the bucket to allow force deletion for lab purposes.

---

## Step 3 - Remove Local Terraform Files

Run:

```bash
rm -rf .terraform

rm -f .terraform.lock.hcl

rm -f terraform.tfstate

rm -f terraform.tfstate.backup

rm -f crash.log
```

---

## Step 4 - Verify Cleanup

Run:

```bash
ls -la
```

You should see files similar to:

```text
backend.tf
dynamodb.tf
outputs.tf
provider.tf
s3.tf
terraform.tfvars
variables.tf
versions.tf
```

---

## Step 5 - Check Git Status

Run:

```bash
git status
```

Review the changes before committing.

---

## Step 6 - Commit Changes

```bash
git add .

git commit -m "Complete Lab 15 Terraform Remote State"
```

---

## Step 7 - Push to GitHub

```bash
git push origin main
```

---

# Lab Completed

You learned:

- Terraform State
- Remote Backend
- Amazon S3 Backend
- DynamoDB State Locking
- Backend Migration
- State Encryption
- State Versioning
- Team Collaboration
- Infrastructure Cleanup

---

# Congratulations!

You have now completed the core Terraform topics:

- ✅ Terraform Basics
- ✅ Variables
- ✅ Outputs
- ✅ Locals
- ✅ Data Sources
- ✅ Expressions
- ✅ Meta Arguments
- ✅ State Management
- ✅ AWS EC2
- ✅ AWS Networking
- ✅ NAT Gateway
- ✅ Application Load Balancer
- ✅ Auto Scaling Group
- ✅ Remote State (S3 + DynamoDB)

