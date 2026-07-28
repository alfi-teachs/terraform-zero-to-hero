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

---

# Part 2 - Create EC2 Web Server

Now we will create:

```
EC2 Instance

+

Apache Web Server

+

Outputs
```

---

# Step 10 - Create User Data Script

Open:

```
user-data.sh
```

Add:

```bash
#!/bin/bash

yum update -y

yum install httpd -y

systemctl start httpd

systemctl enable httpd


echo "<html>

<h1>

Terraform AWS Web Server

</h1>

<h2>

Deployed Automatically Using Terraform

</h2>

</html>" > /var/www/html/index.html
```

---

# Understanding User Data

User Data runs automatically when EC2 starts.

Flow:

```
Terraform Creates EC2

        |

        ▼

AWS Starts Instance

        |

        ▼

User Data Executes

        |

        ▼

Install Apache

        |

        ▼

Start Website
```

---

# Step 11 - Create EC2 Resource

Open:

```
main.tf
```

Add:

```hcl
resource "aws_instance" "web" {


  ami = data.aws_ami.amazon_linux.id


  instance_type = var.instance_type


  vpc_security_group_ids = [

    aws_security_group.web.id

  ]


  user_data = file("user-data.sh")


  tags = merge(

    local.common_tags,

    {

      Name = local.server_name

    }

  )


}
```

---

# Understanding EC2 Configuration

## AMI

```hcl
ami = data.aws_ami.amazon_linux.id
```

Terraform automatically finds:

```
Latest Amazon Linux AMI
```

---

## Instance Type

```hcl
instance_type = var.instance_type
```

From:

```
terraform.tfvars
```

Example:

```
t2.micro
```

---

## Security Group

```hcl
vpc_security_group_ids
```

Attaches:

```
SSH Rule

HTTP Rule
```

---

## User Data

```hcl
user_data = file("user-data.sh")
```

Runs:

```
Apache Installation
```

---

# Step 12 - Create Outputs

Open:

```
outputs.tf
```

Add:

```hcl
output "instance_id" {

  description = "EC2 Instance ID"

  value = aws_instance.web.id

}


output "public_ip" {

  description = "Public IP Address"

  value = aws_instance.web.public_ip

}


output "website_url" {

  description = "Website URL"

  value = "http://${aws_instance.web.public_ip}"

}
```

---

# Step 13 - Initialize Terraform

Run:

```bash
terraform init
```

Expected:

```
Terraform has been successfully initialized
```

---

# Step 14 - Format Files

Run:

```bash
terraform fmt
```

---

# Step 15 - Validate Configuration

Run:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```

---

# Step 16 - Review Plan

Run:

```bash
terraform plan
```

Terraform should show:

```
aws_security_group.web

aws_instance.web
```

Expected:

```
Plan: 2 to add, 0 to change, 0 to destroy
```

---

# Step 17 - Deploy Infrastructure

Run:

```bash
terraform apply
```

Type:

```
yes
```

Terraform will:

```
1. Find latest AMI

2. Create Security Group

3. Create EC2 Instance

4. Attach Security Group

5. Run User Data

6. Install Apache

7. Display Outputs
```

---

# Expected Output

Example:

```
instance_id =
"i-0123456789"

public_ip =
"13.xxx.xxx.xxx"

website_url =
"http://13.xxx.xxx.xxx"
```

Your IP will be different.

---

# Step 18 - Test Website

Copy the output:

Example:

```
http://13.xxx.xxx.xxx
```

Open browser.

You should see:

```
Terraform AWS Web Server

Deployed Automatically Using Terraform
```

---

# Step 19 - Verify EC2

AWS Console:

```
EC2

↓

Instances
```

Check:

```
Instance State:

Running
```

Tags:

```
Project = terraform-web

Environment = dev

ManagedBy = Terraform
```

---

# Step 20 - Check Apache Manually

Connect using SSH:

```bash
ssh ec2-user@PUBLIC_IP
```

Check Apache:

```bash
systemctl status httpd
```

Expected:

```
active (running)
```

Check website:

```bash
curl localhost
```

Expected:

```html
Terraform AWS Web Server
```

---

# Complete Terraform Architecture

Your final deployment:

```
                 Terraform

                     |

        ----------------------------

        |                          |

 Data Source                  Variables

        |                          |

 Latest AMI                 Configuration

        |

        ▼

 Security Group

        |

        ▼

 EC2 Instance

        |

        ▼

 User Data

        |

        ▼

 Apache Website

        |

        ▼

 Browser Access
```

---

# Step 21 - Destroy Resources

After testing:

```bash
terraform destroy
```

Type:

```
yes
```

Terraform removes:

```
EC2 Instance

Security Group
```

---

# Production Improvements

In real companies we would add:

```
VPC

Private Subnets

Load Balancer

Auto Scaling Group

CloudWatch Monitoring

IAM Roles

HTTPS Certificate

Route 53 DNS
```

---

# Lab Verification Checklist

Verify:

✅ EC2 created using Terraform  
✅ Latest AMI selected dynamically  
✅ Security Group attached  
✅ Apache installed automatically  
✅ Website accessible  
✅ Outputs working  
✅ Destroy successful  

---

# Lab 10 Completed

You have built:

```
Complete AWS EC2 Deployment

Using Terraform
```

Skills used:

✅ Variables  
✅ Locals  
✅ Data Sources  
✅ Security Groups  
✅ EC2  
✅ User Data  
✅ Outputs  
✅ Terraform Workflow  

---

# Git Save

Run:

```bash
git status

git add .

git commit -m "Complete Lab 10 AWS EC2 Terraform Project"

git push origin main
```

---

# Next Lab

## Lab 11 - Terraform Modules

You will learn:

- Why modules are used
- Creating reusable infrastructure
- Module structure
- Calling modules
- Production Terraform repository design

This is the step where your Terraform project starts looking like an enterprise DevOps project.