# Lab 09 - Terraform State Management

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 09 |
| Lab Name | Terraform State Management |
| Difficulty | Intermediate |
| Duration | 2 Hours |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will understand:

- What Terraform State is
- Why Terraform needs state
- terraform.tfstate file
- State commands
- State locking
- Remote state
- Importing existing resources
- Production state management

---

# What is Terraform State?

Terraform State is a file where Terraform stores information about the infrastructure it manages.

The file name:

```
terraform.tfstate
```

It contains:

- Resource IDs
- Resource attributes
- Dependencies
- Metadata

---

# Simple Example

You write:

```hcl
resource "aws_instance" "web" {

  instance_type = "t2.micro"

}
```

Terraform creates:

```
EC2 Instance

ID:

i-0123456789
```

Terraform stores:

```
aws_instance.web

↓

i-0123456789
```

inside:

```
terraform.tfstate
```

---

# Why Does Terraform Need State?

Without state Terraform would not know:

- What resources it created
- What already exists
- What changed
- What needs updating

---

# Real World Example

First Apply:

Terraform creates:

```
EC2 Instance

i-12345
```

State:

```
EC2 = i-12345
```

Later you change:

```hcl
instance_type = "t3.micro"
```

Terraform checks:

```
Configuration

vs

State

vs

AWS
```

Then decides:

```
Modify existing EC2
```

It does not create a new one.

---

# Terraform State Flow

```
Terraform Code

        |

        ▼

terraform plan

        |

        ▼

Compare

        |

        ▼

terraform.tfstate

        |

        ▼

AWS Infrastructure
```

---

# Folder Structure

Create:

```
terraform/
└── lab-09/
    ├── versions.tf
    ├── provider.tf
    ├── variables.tf
    ├── terraform.tfvars
    ├── main.tf
    ├── outputs.tf
    ├── backend.tf
    └── README.md
```

---

# Step 1 - Create Lab Folder

Go to project:

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create:

```bash
mkdir -p terraform/lab-09

mkdir -p images/lab-09

touch labs/lab-09-terraform-state-management.md
```

Enter:

```bash
cd terraform/lab-09
```

---

# Step 2 - Copy Previous Files

Copy:

```bash
cp ../lab-08/versions.tf .
cp ../lab-08/provider.tf .
cp ../lab-08/variables.tf .
cp ../lab-08/terraform.tfvars .
cp ../lab-08/locals.tf .
cp ../lab-08/data.tf .
cp ../lab-08/outputs.tf .
```

Create:

```bash
touch main.tf

touch backend.tf

touch README.md
```

---

# Part 1 - Understanding terraform.tfstate

After running:

```bash
terraform apply
```

Terraform creates:

```
terraform.tfstate
```

Example structure:

```json
{
 "resources": [

  {
   "type": "aws_instance",
   "name": "web"
  }

 ]
}
```

The real file is much larger.

---

# Important Rule

Never manually edit:

```
terraform.tfstate
```

Terraform manages it.

---

# Step 3 - Create Simple Resource

Open:

```
main.tf
```

Add:

```hcl
resource "aws_s3_bucket" "state_test" {

  bucket = "${var.project_name}-state-test"

  tags = {

    Name = "state-test"

  }

}
```

---

# Step 4 - Add Output

Open:

```
outputs.tf
```

Add:

```hcl
output "bucket_id" {

  value = aws_s3_bucket.state_test.id

}
```

---

# Step 5 - Initialize

Run:

```bash
terraform init
```

---

# Step 6 - Plan

Run:

```bash
terraform plan
```

Terraform shows:

```
+ create aws_s3_bucket.state_test
```

---

# Step 7 - Apply

Run:

```bash
terraform apply
```

Type:

```
yes
```

Terraform creates:

```
S3 Bucket
```

---

# Step 8 - Check State File

Run:

```bash
ls
```

You should see:

```
terraform.tfstate
```

---

# Step 9 - View State

Run:

```bash
terraform show
```

You will see:

```
aws_s3_bucket.state_test
```

---

# Step 10 - List Resources In State

Run:

```bash
terraform state list
```

Output:

```
aws_s3_bucket.state_test
```

---

# Step 11 - Show Specific Resource

Run:

```bash
terraform state show aws_s3_bucket.state_test
```

You will see:

```
bucket name

ARN

region

tags
```

---

# Important Terraform State Commands

## List Resources

```bash
terraform state list
```

---

## Show Resource

```bash
terraform state show RESOURCE_NAME
```

---

## Remove Resource From State

```bash
terraform state rm RESOURCE_NAME
```

Important:

This removes Terraform management.

It does NOT delete AWS resource.

---

## Move Resource In State

```bash
terraform state mv
```

Used when changing resource names.

---

# Real Production Problem

Imagine:

Engineer A:

```
terraform apply
```

Engineer B:

```
terraform apply
```

Both modify state at the same time.

Problem:

```
State corruption
```

Solution:

```
Remote State + State Locking
```

---

# Local State vs Remote State

## Local State

Stored:

```
terraform.tfstate
```

on your laptop.

Problems:

- Hard to share
- No locking
- Risk of losing file

---

## Remote State

Stored:

```
AWS S3
```

Benefits:

- Team sharing
- Backup
- Locking
- Security

---

# Production Architecture

```
Developer A

       |

       ▼

Terraform


       |

       ▼

S3 Remote State

       |

       ▼

DynamoDB Lock


       |

       ▼

AWS Resources
```

---

# Summary

You learned:

- Terraform State
- terraform.tfstate
- State commands
- Local state
- Why remote state is needed

---

# End of Part 1

Next Part:

# Part 2 - Remote Backend with AWS S3

You will learn:

- Create S3 backend
- Enable state locking
- Configure backend.tf
- Team collaboration workflow
- Production Terraform state architecture

---

# Part 2 - Terraform Remote Backend with AWS S3

Now we will move Terraform State from your laptop to AWS.

---

# Why Remote State?

Local state:

```
Developer Laptop

terraform.tfstate
```

Problems:

- Only one person can use it
- File can be deleted accidentally
- Difficult for teams
- No locking

---

# Remote State

Production setup:

```
Developer

     |

     ▼

Terraform

     |

     ▼

S3 Bucket

     |

     ▼

terraform.tfstate
```

Now everyone uses the same state.

---

# What is State Locking?

Imagine:

Engineer A runs:

```
terraform apply
```

at the same time:

Engineer B runs:

```
terraform apply
```

Both try to modify state.

This can corrupt the state.

State locking prevents this.

---

# State Locking Architecture

```
Terraform

    |

    ▼

DynamoDB Lock Table

    |

    ▼

S3 State File
```

Before changing infrastructure:

Terraform asks:

"Is anyone using the state?"

If yes:

```
Wait
```

If no:

```
Lock state

Apply changes
```

---

# AWS Services Used

We use:

## S3

Stores:

```
terraform.tfstate
```

---

## DynamoDB

Stores:

```
State Lock Information
```

---

# Step 1 - Create Backend Folder

Create:

```
terraform/
└── backend/
    ├── main.tf
    ├── provider.tf
    └── outputs.tf
```

---

# Step 2 - Create Backend Directory

Run:

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create:

```bash
mkdir -p terraform/backend
```

Go inside:

```bash
cd terraform/backend
```

---

# Step 3 - Create Provider

Create:

```
provider.tf
```

Add:

```hcl
terraform {

  required_providers {

    aws = {

      source = "hashicorp/aws"

      version = "~> 6.0"

    }

  }

}


provider "aws" {

  region = "ap-south-1"

}
```

---

# Step 4 - Create S3 Backend Bucket

Create:

```
main.tf
```

Add:

```hcl
resource "aws_s3_bucket" "terraform_state" {

  bucket = "terraform-zero-to-hero-state-bucket"


  lifecycle {

    prevent_destroy = true

  }


  versioning {

    enabled = true

  }


  tags = {

    Name = "Terraform State"

    ManagedBy = "Terraform"

  }

}
```

---

# Understanding This

## Versioning

```hcl
versioning {

enabled = true

}
```

Keeps previous versions of state.

Example:

```
terraform.tfstate

version 1

version 2

version 3
```

If something goes wrong, you can recover.

---

## prevent_destroy

```hcl
prevent_destroy = true
```

Protects the state bucket.

Nobody can accidentally delete it.

---

# Step 5 - Create DynamoDB Lock Table

Append to:

```
main.tf
```

```hcl
resource "aws_dynamodb_table" "terraform_lock" {

  name = "terraform-state-lock"

  billing_mode = "PAY_PER_REQUEST"


  hash_key = "LockID"


  attribute {

    name = "LockID"

    type = "S"

  }


  tags = {

    Name = "Terraform Lock"

  }

}
```

---

# Step 6 - Initialize Backend Project

Run:

```bash
terraform init
```

---

# Step 7 - Validate

Run:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```

---

# Step 8 - Plan

Run:

```bash
terraform plan
```

You should see:

```
aws_s3_bucket.terraform_state

aws_dynamodb_table.terraform_lock
```

---

# Step 9 - Apply

Run:

```bash
terraform apply
```

Type:

```
yes
```

Terraform creates:

```
S3 Bucket

DynamoDB Table
```

---

# Verify AWS Console

## S3

Go:

```
AWS Console

↓

S3

↓

Buckets
```

You should see:

```
terraform-zero-to-hero-state-bucket
```

---

## DynamoDB

Go:

```
AWS Console

↓

DynamoDB

↓

Tables
```

You should see:

```
terraform-state-lock
```

---

# Step 10 - Configure Backend

Now go back to:

```
terraform/lab-09
```

Open:

```
backend.tf
```

Add:

```hcl
terraform {

  backend "s3" {

    bucket = "terraform-zero-to-hero-state-bucket"

    key = "lab-09/terraform.tfstate"

    region = "ap-south-1"

    dynamodb_table = "terraform-state-lock"

    encrypt = true

  }

}
```

---

# Understanding Backend Configuration

## bucket

Where state is stored:

```
S3 Bucket
```

---

## key

State file location:

```
lab-09/terraform.tfstate
```

---

## region

AWS Region:

```
ap-south-1
```

---

## dynamodb_table

Locking table:

```
terraform-state-lock
```

---

## encrypt

Encrypts state:

```
true
```

---

# Step 11 - Initialize Remote Backend

Inside:

```
terraform/lab-09
```

Run:

```bash
terraform init
```

Terraform asks:

```
Do you want to migrate existing state?
```

Answer:

```
yes
```

---

# What Happens Now?

Before:

```
Laptop

terraform.tfstate
```

After:

```
AWS S3

terraform.tfstate
```

---

# Step 12 - Verify State Location

Run:

```bash
terraform state list
```

You should still see resources.

But now state comes from S3.

---

# Production Workflow

Team members:

```
Developer A

      |

      ▼

Terraform Apply


Developer B

      |

      ▼

Terraform Plan
```

Both use:

```
Same S3 State
```

DynamoDB prevents conflicts.

---

# Best Practices

Always:

- Enable S3 versioning
- Enable encryption
- Use state locking
- Restrict S3 permissions
- Protect state bucket
- Separate state files per environment

Example:

```
dev/terraform.tfstate

test/terraform.tfstate

prod/terraform.tfstate
```

---

# Summary

You learned:

- Remote State
- S3 Backend
- DynamoDB Locking
- State Migration
- Team Terraform Workflow
- Production State Architecture

---

# End of Part 2

Next Part:

# Part 3 - Terraform Import and Advanced State Commands

You will learn:

- Import existing AWS resources
- Terraform refresh
- State recovery
- State troubleshooting
- Interview questions

---

# Part 3 - Terraform Import and Advanced State Commands

---

# What is Terraform Import?

Terraform Import allows you to bring existing AWS resources into Terraform state.

Important:

Import does NOT create resources.

It only tells Terraform:

"This resource already exists. Start managing it."

---

# Example Scenario

Before Terraform:

```
AWS Console

↓

EC2 Instance Created Manually

↓

Instance ID:

i-0123456789
```

Now you want Terraform management.

Process:

```
Existing EC2

      |

      ▼

terraform import

      |

      ▼

terraform.tfstate

      |

      ▼

Terraform manages it
```

---

# Important Concept

Terraform has two parts:

## Configuration

Your code:

```
main.tf
```

Example:

```hcl
resource "aws_instance" "web" {

}
```

---

## State

Terraform memory:

```
terraform.tfstate
```

Import connects:

```
AWS Resource

+

Terraform State
```

---

# Import Syntax

Basic syntax:

```bash
terraform import RESOURCE_ADDRESS RESOURCE_ID
```

Example:

```bash
terraform import aws_instance.web i-0123456789
```

---

# Lab Scenario

We will import an existing S3 bucket.

Flow:

```
Create S3 Bucket Manually

        |

        ▼

Write Terraform Resource

        |

        ▼

Import Bucket

        |

        ▼

Terraform Manages It
```

---

# Step 1 - Create S3 Bucket Manually

Go to:

```
AWS Console

↓

S3

↓

Create Bucket
```

Example name:

```
terraform-import-demo-bucket
```

Create it.

Do not add Terraform tags.

---

# Step 2 - Create Terraform Resource

Inside:

```
terraform/lab-09
```

Open:

```
main.tf
```

Add:

```hcl
resource "aws_s3_bucket" "import_bucket" {

  bucket = "terraform-import-demo-bucket"

}
```

---

# Step 3 - Initialize

Run:

```bash
terraform init
```

---

# Step 4 - Check Plan

Run:

```bash
terraform plan
```

Terraform shows:

```
+ create aws_s3_bucket.import_bucket
```

Why?

Because Terraform does not know the bucket already exists.

---

# Step 5 - Import Existing Bucket

Run:

```bash
terraform import aws_s3_bucket.import_bucket terraform-import-demo-bucket
```

Expected:

```
Import successful!
```

---

# Step 6 - Check State

Run:

```bash
terraform state list
```

Output:

```
aws_s3_bucket.import_bucket
```

Now Terraform knows about the bucket.

---

# Step 7 - Run Plan Again

Run:

```bash
terraform plan
```

Now Terraform compares:

```
Terraform Code

        VS

AWS Resource

        VS

State
```

Expected:

```
No changes.
```

---

# Why No Changes?

Because:

```
AWS Bucket

=

Terraform Configuration

=

Terraform State
```

Everything matches.

---

# Advanced State Commands

---

# 1. terraform state list

Shows all managed resources.

Command:

```bash
terraform state list
```

Example:

```
aws_instance.web

aws_s3_bucket.logs

aws_vpc.main
```

---

# 2. terraform state show

Shows resource details.

Command:

```bash
terraform state show aws_s3_bucket.import_bucket
```

Shows:

```
Bucket Name

Region

ARN

Tags
```

---

# 3. terraform state rm

Removes resource from Terraform management.

Example:

```bash
terraform state rm aws_s3_bucket.import_bucket
```

Important:

It does NOT delete AWS resource.

It only removes Terraform tracking.

Before:

```
Terraform

↓

AWS Bucket
```

After:

```
AWS Bucket

(no Terraform control)
```

---

# Real Use Case

A company wants:

```
Stop managing old resource
```

but keep it running.

Use:

```bash
terraform state rm
```

---

# 4. terraform state mv

Used when renaming resources.

Example:

Old:

```hcl
aws_instance.web
```

New:

```hcl
aws_instance.application
```

Without moving state:

Terraform thinks:

```
Old resource deleted

New resource created
```

Risk:

```
Unnecessary replacement
```

Solution:

```bash
terraform state mv \
aws_instance.web \
aws_instance.application
```

---

# 5. terraform refresh

Refresh updates state with real AWS information.

Command:

```bash
terraform refresh
```

Example:

AWS changed:

```
Instance tags
```

Terraform updates:

```
terraform.tfstate
```

---

# State Troubleshooting Workflow

When Terraform behaves unexpectedly:

Step 1:

Check resources:

```bash
terraform state list
```

---

Step 2:

Inspect resource:

```bash
terraform state show RESOURCE
```

---

Step 3:

Compare:

```bash
terraform plan
```

---

Step 4:

Import missing resources:

```bash
terraform import
```

---

# Production State Rules

Never:

❌ Edit terraform.tfstate manually

❌ Delete state file

❌ Share state file through email/chat

❌ Store production state locally

---

Always:

✅ Use remote backend

✅ Enable encryption

✅ Enable locking

✅ Backup state

✅ Control access using IAM

---

# Interview Questions

## Beginner

1. What is Terraform state?
2. Why does Terraform need state?
3. Where is state stored by default?
4. What does terraform import do?

---

## Intermediate

5. Does terraform import create resources?
6. Difference between terraform state rm and destroy?
7. Why use remote state?
8. What is state locking?

---

## Advanced

9. How do you manage existing AWS resources with Terraform?
10. What happens if terraform.tfstate is deleted?
11. How do teams share Terraform state?
12. How do you recover from state corruption?

---

# Lab Verification Checklist

Verify:

✅ Imported existing resource  
✅ State list works  
✅ State show works  
✅ Understand state rm  
✅ Understand state mv  
✅ Understand refresh  
✅ Understand production workflow  

---

# Summary

You completed Lab 09:

✅ Terraform State  
✅ Local State  
✅ Remote Backend  
✅ S3 Backend  
✅ DynamoDB Locking  
✅ Import Existing Resources  
✅ Advanced State Commands  

---

# Git Save

Run:

```bash
git status

git add .

git commit -m "Complete Lab 09 - Terraform State Management"

git push origin main
```

---

# Next Lab

# Lab 10 - Terraform AWS EC2 Production Project

You will combine everything:

```
Variables

+

Locals

+

Data Sources

+

Security Groups

+

EC2

+

User Data

+

Outputs

+

Remote State

+

Modules
```

This will become your first complete Terraform AWS DevOps project.