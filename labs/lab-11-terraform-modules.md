# Lab 11 - Terraform Modules

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 11 |
| Lab Name | Terraform Modules |
| Difficulty | Intermediate |
| Duration | 3 Hours |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will understand:

✅ What Terraform Modules are  
✅ Why companies use modules  
✅ Module folder structure  
✅ Creating reusable modules  
✅ Passing variables to modules  
✅ Using module outputs  
✅ Calling modules from root configuration  

---

# What Are Terraform Modules?

A Terraform Module is a collection of Terraform files grouped together.

Example:

Without modules:

```
main.tf

variables.tf

outputs.tf

security.tf

network.tf
```

Everything is in one folder.

Problem:

As projects grow:

```
1000 lines

5000 lines

10000 lines
```

becomes difficult to manage.

---

# With Modules

We separate infrastructure:

```
terraform-project

|

├── modules

│

├── ec2

│

├── security-group

│

└── vpc


|

└── environments

    |

    ├── dev

    |

    └── prod
```

---

# Real Company Example

A company has:

Development:

```
EC2

Small instance
```

Production:

```
EC2

Large instance
```

Both use the same:

```
EC2 Module
```

Only values change.

---

# Module Architecture

```
Root Module

      |

      ▼

Calls Child Module

      |

      ▼

Creates AWS Resources
```

---

# Terraform Module Types

## 1. Root Module

The directory where you run:

```bash
terraform apply
```

Example:

```
terraform/lab-11
```

---

## 2. Child Module

Reusable component.

Example:

```
modules/ec2
```

---

# Lab Folder Structure

Create:

```
terraform/
└── lab-11-modules/

    ├── main.tf

    ├── variables.tf

    ├── outputs.tf


    └── modules/

        └── ec2/

            ├── main.tf

            ├── variables.tf

            └── outputs.tf

```

---

# Step 1 - Create Lab Folder

Go to repository:

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create:

```bash
mkdir -p terraform/lab-11-modules/modules/ec2

mkdir -p images/lab-11

touch labs/lab-11-terraform-modules.md
```

Enter:

```bash
cd terraform/lab-11-modules
```

---

# Step 2 - Create Root Files

Run:

```bash
touch main.tf

touch variables.tf

touch outputs.tf
```

Go into module:

```bash
cd modules/ec2
```

Create:

```bash
touch main.tf

touch variables.tf

touch outputs.tf
```

Return:

```bash
cd ../..
```

---

# Module Flow

```
Root Module

terraform apply


        |

        ▼


EC2 Module


        |

        ▼


AWS EC2 Instance
```

---

# Part 1 - Create EC2 Module

Now we create a reusable EC2 component.

---

# Step 3 - Module Variables

Open:

```
modules/ec2/variables.tf
```

Add:

```hcl
variable "ami_id" {

  description = "AMI ID"

  type = string

}


variable "instance_type" {

  description = "EC2 Instance Type"

  type = string

}


variable "name" {

  description = "Instance Name"

  type = string

}
```

---

# Step 4 - Module Resource

Open:

```
modules/ec2/main.tf
```

Add:

```hcl
resource "aws_instance" "this" {


  ami = var.ami_id


  instance_type = var.instance_type


  tags = {

    Name = var.name

  }


}
```

---

# Understanding Module Variables

The module does not know:

```
Which AMI

Which Instance Type

Which Name
```

The root module provides values.

---

# Step 5 - Module Output

Open:

```
modules/ec2/outputs.tf
```

Add:

```hcl
output "instance_id" {

  value = aws_instance.this.id

}


output "public_ip" {

  value = aws_instance.this.public_ip

}
```

---

# Module Created

Our reusable module:

```
modules/ec2

Input:

ami_id

instance_type

name


Output:

instance_id

public_ip
```

---

# Step 6 - Call Module From Root

Open:

```
main.tf
```

Add:

```hcl
module "web_server" {

  source = "./modules/ec2"


  ami_id = data.aws_ami.amazon_linux.id


  instance_type = "t2.micro"


  name = "module-test-server"

}
```

---

# What Happens?

Terraform reads:

```
Root main.tf
```

Then:

```
Go to modules/ec2
```

Then:

```
Create EC2
```

---

# Step 7 - Add AMI Data Source

Root:

Create:

```
data.tf
```

Add:

```hcl
data "aws_ami" "amazon_linux" {

  most_recent = true


  owners = [

    "amazon"

  ]


  filter {

    name = "name"

    values = [

      "al2023-ami-*-x86_64"

    ]

  }

}
```

---

# Step 8 - Initialize

Run:

```bash
terraform init
```

Terraform downloads:

```
Module

Provider
```

---

# Step 9 - Validate

Run:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```

---

# Step 10 - Plan

Run:

```bash
terraform plan
```

Expected:

```
module.web_server.aws_instance.this
```

---

# Summary

In Part 1 you learned:

✅ What modules are  
✅ Root module  
✅ Child module  
✅ Creating EC2 module  
✅ Passing variables  
✅ Returning outputs  
✅ Calling modules  

---

# End of Part 1

Next Part:

# Lab 11 Part 2 - Advanced Modules

You will learn:

- Module outputs
- Multiple module calls
- VPC module example
- Environment structure
- Production Terraform repository design
- Terraform Registry modules

---

# Part 2 - Advanced Terraform Modules

Now we will improve our module design.

A real company does not keep everything in:

```
main.tf
```

Instead they organize:

```
Modules

+

Environments

+

Reusable Infrastructure
```

---

# Enterprise Terraform Structure

A common production structure:

```
terraform-project

│
├── modules
│
│   ├── vpc
│   │
│   ├── ec2
│   │
│   ├── security-group
│   │
│   └── database
│
│
└── environments
    │
    ├── dev
    │
    ├── test
    │
    └── prod
```

---

# Why Separate Environments?

Development:

```
Small EC2

Cheap resources
```

Production:

```
Large EC2

High availability

More security
```

Same modules.

Different values.

---

# Module Reusability Example

One EC2 Module:

```
modules/ec2
```

Used by:

```
dev

test

prod
```

---

# Architecture

```
                 Root Module


                      |

        ----------------------------


        |                          |


     Dev Environment          Prod Environment


        |                          |


        ▼                          ▼


     EC2 Module                EC2 Module


        |                          |


        ▼                          ▼


     AWS EC2                  AWS EC2

```

---

# Step 11 - Create Environment Structure

Go to:

```
terraform/lab-11-modules
```

Create:

```bash
mkdir -p environments/dev

mkdir -p environments/prod
```

Structure becomes:

```
lab-11-modules

├── modules

│   └── ec2


└── environments

    ├── dev

    └── prod
```

---

# Step 12 - Create Dev Environment

Go:

```bash
cd environments/dev
```

Create:

```bash
touch main.tf

touch variables.tf

touch terraform.tfvars

touch outputs.tf
```

---

# Step 13 - Dev Variables

Open:

```
variables.tf
```

Add:

```hcl
variable "instance_type" {

  type = string

}


variable "environment" {

  type = string

}
```

---

# Step 14 - Dev Values

Open:

```
terraform.tfvars
```

Add:

```hcl
instance_type = "t2.micro"

environment = "dev"
```

---

# Step 15 - Call EC2 Module

Open:

```
main.tf
```

Add:

```hcl
module "dev_server" {

  source = "../../modules/ec2"


  ami_id = data.aws_ami.amazon_linux.id


  instance_type = var.instance_type


  name = "dev-web-server"

}
```

---

# Step 16 - Add Data Source

Create:

```
data.tf
```

Add:

```hcl
data "aws_ami" "amazon_linux" {


  most_recent = true


  owners = [

    "amazon"

  ]


  filter {

    name = "name"


    values = [

      "al2023-ami-*-x86_64"

    ]

  }

}
```

---

# Step 17 - Dev Output

Open:

```
outputs.tf
```

Add:

```hcl
output "server_id" {

  value = module.dev_server.instance_id

}


output "server_ip" {

  value = module.dev_server.public_ip

}
```

---

# Step 18 - Initialize Dev Environment

Run:

```bash
terraform init
```

---

# Step 19 - Validate

Run:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```

---

# Step 20 - Plan

Run:

```bash
terraform plan
```

Terraform shows:

```
module.dev_server.aws_instance.this
```

---

# Step 21 - Apply

Run:

```bash
terraform apply
```

Type:

```
yes
```

---

# What Happened?

Terraform:

1. Started from dev environment

2. Called EC2 module

3. Passed variables

4. Created EC2

5. Returned outputs

---

# Module Input and Output Flow

```
Environment


instance_type = t2.micro


        |


        ▼


EC2 Module


        |


        ▼


AWS EC2


        |


        ▼


Output


instance_id

public_ip

```

---

# Multiple Module Calls

You can call the same module many times.

Example:

```hcl
module "web" {

source = "../../modules/ec2"

name = "web-server"

}


module "app" {

source = "../../modules/ec2"

name = "app-server"

}
```

Result:

```
web-server

app-server
```

Both created from the same module.

---

# Terraform Registry Modules

Terraform also provides public modules.

Example:

Instead of manually creating VPC:

```
VPC

Subnets

Route Tables

Internet Gateway
```

You can use:

```
AWS VPC Module
```

from Terraform Registry.

---

# Production Module Rules

Good modules should:

✅ Accept variables

✅ Provide outputs

✅ Avoid hardcoded values

✅ Be reusable

✅ Have documentation

✅ Have version control

---

# Common Module Mistakes

## Mistake 1

Hardcoding values:

Wrong:

```hcl
instance_type = "t2.micro"
```

Better:

```hcl
variable "instance_type"
```

---

## Mistake 2

Too many resources in one module.

Bad:

```
VPC + EC2 + Database + Load Balancer
```

Better:

```
vpc module

ec2 module

database module
```

---

## Mistake 3

No outputs.

Modules should return useful information.

Example:

```
instance_id

private_ip

security_group_id
```

---

# Lab Verification Checklist

Verify:

✅ Module folder created  
✅ EC2 module works  
✅ Environment structure created  
✅ Variables passed correctly  
✅ Outputs returned correctly  
✅ Same module can be reused  

---

# Summary

You learned:

- Enterprise Terraform structure
- Environment separation
- Module reuse
- Module inputs
- Module outputs
- Production design patterns

---

# Git Save

Run:

```bash
git status

git add .

git commit -m "Complete Lab 11 Terraform Modules"

git push origin main
```

---

# Next Lab

# Lab 12 - Terraform AWS Networking Project

You will build:

```
VPC

Subnets

Internet Gateway

Route Tables

Security Groups

EC2 inside VPC
```

This will be your first complete AWS network deployment using Terraform.