# Lab 10 - Terraform AWS EC2 Production Project

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 10 |
| Lab Name | AWS EC2 Production Deployment |
| Difficulty | Intermediate |
| Duration | 3-4 Hours |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will build:

✅ AWS EC2 Instance

✅ Security Group

✅ Dynamic AMI Lookup

✅ Variables

✅ Locals

✅ Outputs

✅ User Data

✅ Remote State

✅ Terraform Best Practices

---

# Project Scenario

A company wants to deploy a simple web server.

Requirements:

- Use latest Amazon Linux AMI
- Create EC2 instance
- Install Apache automatically
- Allow HTTP traffic
- Allow SSH access
- Add proper tags
- Store Terraform state remotely

---

# Final Folder Structure

Create:

```
terraform/
└── lab-10-aws-ec2-project/

    ├── versions.tf

    ├── provider.tf

    ├── backend.tf

    ├── variables.tf

    ├── terraform.tfvars

    ├── locals.tf

    ├── data.tf

    ├── security-group.tf

    ├── main.tf

    ├── user-data.sh

    ├── outputs.tf

    └── README.md
```

---

# Step 1 - Create Project Folder

Go to your Terraform repository:

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create:

```bash
mkdir -p terraform/lab-10-aws-ec2-project

mkdir -p images/lab-10

touch labs/lab-10-terraform-aws-ec2-project.md
```

Enter:

```bash
cd terraform/lab-10-aws-ec2-project
```

---

# Step 2 - Create Terraform Files

Run:

```bash
touch versions.tf

touch provider.tf

touch backend.tf

touch variables.tf

touch terraform.tfvars

touch locals.tf

touch data.tf

touch security-group.tf

touch main.tf

touch user-data.sh

touch outputs.tf

touch README.md
```

---

# Step 3 - Terraform Version Configuration

Open:

```
versions.tf
```

Add:

```hcl
terraform {

  required_version = ">= 1.5.0"


  required_providers {

    aws = {

      source = "hashicorp/aws"

      version = "~> 6.0"

    }

  }

}
```

---

# Step 4 - AWS Provider

Open:

```
provider.tf
```

Add:

```hcl
provider "aws" {

  region = var.aws_region

}
```

---

# Step 5 - Variables

Open:

```
variables.tf
```

Add:

```hcl
variable "aws_region" {

  description = "AWS Region"

  type = string

}


variable "environment" {

  description = "Environment Name"

  type = string

}


variable "instance_type" {

  description = "EC2 Instance Type"

  type = string

}


variable "project_name" {

  description = "Project Name"

  type = string

}
```

---

# Step 6 - Terraform Variables Values

Open:

```
terraform.tfvars
```

Add:

```hcl
aws_region = "ap-south-1"

environment = "dev"

instance_type = "t2.micro"

project_name = "terraform-web"
```

---

# Step 7 - Create Locals

Open:

```
locals.tf
```

Add:

```hcl
locals {

  common_tags = {

    Project = var.project_name

    Environment = var.environment

    ManagedBy = "Terraform"

  }


  server_name = "${var.project_name}-${var.environment}-server"

}
```

---

# Step 8 - Find Latest Amazon Linux AMI

Open:

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


  filter {

    name = "root-device-type"

    values = [

      "ebs"

    ]

  }


}
```

---

# Step 9 - Create Security Group

Open:

```
security-group.tf
```

Add:

```hcl
resource "aws_security_group" "web" {


  name = "${var.project_name}-sg"


  description = "Allow Web Traffic"


  ingress {


    from_port = 22

    to_port = 22

    protocol = "tcp"

    cidr_blocks = [

      "0.0.0.0/0"

    ]

  }


  ingress {


    from_port = 80

    to_port = 80

    protocol = "tcp"

    cidr_blocks = [

      "0.0.0.0/0"

    ]

  }


  egress {


    from_port = 0

    to_port = 0

    protocol = "-1"

    cidr_blocks = [

      "0.0.0.0/0"

    ]

  }


  tags = local.common_tags

}
```

---

# What We Created

Security Group allows:

```
SSH

Port 22


HTTP

Port 80
```

---

# End of Part 1

Next Part:

## Lab 10 Part 2

We will create:

- EC2 Instance
- User Data Apache Installation
- Outputs
- Apply Infrastructure
- Test Website
- Destroy Cleanup

This will complete your first production-style Terraform AWS deployment.

