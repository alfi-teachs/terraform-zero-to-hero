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