# Lab 12 - Terraform AWS Networking Project

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 12 |
| Lab Name | AWS Networking with Terraform |
| Difficulty | Intermediate |
| Duration | 4-5 Hours |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will be able to:

- Create a custom VPC
- Create public and private subnets
- Create an Internet Gateway
- Create Route Tables
- Associate Route Tables with Subnets
- Launch an EC2 instance inside your VPC
- Understand AWS networking architecture

---

# Real World Scenario

A company wants a secure AWS network.

Requirements:

- One custom VPC
- One public subnet
- One private subnet
- Internet access only for the public subnet
- One web server
- One database subnet

---

# Final Architecture

```
                    Internet
                        |
                        |
                 Internet Gateway
                        |
                        |
               -------------------
               |                 |
             Route Table
               |
               |
        -----------------
        |               |
 Public Subnet    Private Subnet
        |               |
        |               |
     EC2 Web       Database
```

---

# Folder Structure

```
terraform/
└── lab-12-networking/

    ├── versions.tf
    ├── provider.tf
    ├── variables.tf
    ├── terraform.tfvars
    ├── main.tf
    ├── vpc.tf
    ├── subnet.tf
    ├── igw.tf
    ├── route-table.tf
    ├── security-group.tf
    ├── ec2.tf
    ├── outputs.tf
    └── README.md
```

---

# Step 1 - Create Project Folder

Go to your Terraform repository:

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create the project folders:

```bash
mkdir -p terraform/lab-12-networking

mkdir -p images/lab-12

touch labs/lab-12-terraform-aws-networking.md
```

Go inside:

```bash
cd terraform/lab-12-networking
```

---

# Step 2 - Create Terraform Files

Run:

```bash
touch versions.tf

touch provider.tf

touch variables.tf

touch terraform.tfvars

touch main.tf

touch vpc.tf

touch subnet.tf

touch igw.tf

touch route-table.tf

touch security-group.tf

touch ec2.tf

touch outputs.tf

touch README.md
```

---

# AWS Architecture We Will Build

```
VPC
 │
 ├── Public Subnet
 │      │
 │      └── EC2 Web Server
 │
 ├── Private Subnet
 │      │
 │      └── Future Database
 │
 ├── Internet Gateway
 │
 └── Route Table
```

---

# Step 3 - Terraform Version

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

      source  = "hashicorp/aws"

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

  type = string

}

variable "vpc_cidr" {

  type = string

}

variable "public_subnet_cidr" {

  type = string

}

variable "private_subnet_cidr" {

  type = string

}

variable "availability_zone" {

  type = string

}

variable "instance_type" {

  type = string

}
```

---

# Step 6 - Variable Values

Open:

```
terraform.tfvars
```

Add:

```hcl
aws_region = "ap-south-1"

vpc_cidr = "10.0.0.0/16"

public_subnet_cidr = "10.0.1.0/24"

private_subnet_cidr = "10.0.2.0/24"

availability_zone = "ap-south-1a"

instance_type = "t2.micro"
```

---

# AWS Resources We'll Build

| Order | Resource |
|-------|----------|
| 1 | VPC |
| 2 | Public Subnet |
| 3 | Private Subnet |
| 4 | Internet Gateway |
| 5 | Route Table |
| 6 | Route Table Association |
| 7 | Security Group |
| 8 | EC2 Instance |

---

# Understanding CIDR Blocks

```
VPC

10.0.0.0/16

│

├── Public

10.0.1.0/24

│

└── Private

10.0.2.0/24
```

A `/16` network is larger and contains many smaller `/24` subnet ranges.

---

# End of Part 1

Next Part:

## Lab 12 Part 2

We will create:

- Custom VPC
- Public Subnet
- Private Subnet
- Internet Gateway
- Route Table
- Route Table Association

This builds the core AWS network before launching any EC2 instances.

---

# Part 2 - Create VPC, Subnets, Internet Gateway and Route Table

Now we will build the AWS network.

Follow this order:

```
VPC

↓

Subnets

↓

Internet Gateway

↓

Route Table

↓

Route

↓

Subnet Association
```

---

# Step 7 - Create VPC

Open:

```
vpc.tf
```

Add:

```hcl
resource "aws_vpc" "main" {

  cidr_block = var.vpc_cidr

  enable_dns_support = true

  enable_dns_hostnames = true

  tags = {

    Name = "terraform-vpc"

    Environment = "dev"

  }

}
```

---

# What is Happening?

Terraform creates:

```
Custom VPC

CIDR:

10.0.0.0/16
```

DNS Support:

```
Enabled
```

DNS Hostnames:

```
Enabled
```

These settings allow EC2 instances to receive DNS hostnames.

---

# Step 8 - Create Public Subnet

Open:

```
subnet.tf
```

Add:

```hcl
resource "aws_subnet" "public" {

  vpc_id = aws_vpc.main.id

  cidr_block = var.public_subnet_cidr

  availability_zone = var.availability_zone

  map_public_ip_on_launch = true

  tags = {

    Name = "public-subnet"

  }

}
```

---

# Why map_public_ip_on_launch?

```
true
```

means:

Every EC2 launched inside this subnet automatically receives a Public IP.

Without it:

```
No Public IP

No Internet Access
```

---

# Step 9 - Create Private Subnet

Continue inside:

```
subnet.tf
```

Add:

```hcl
resource "aws_subnet" "private" {

  vpc_id = aws_vpc.main.id

  cidr_block = var.private_subnet_cidr

  availability_zone = var.availability_zone

  tags = {

    Name = "private-subnet"

  }

}
```

---

# Public vs Private

Public Subnet:

```
Public IP

Internet Access

Web Servers
```

Private Subnet:

```
No Public IP

No Direct Internet

Database

Application Servers
```

---

# Step 10 - Create Internet Gateway

Open:

```
igw.tf
```

Add:

```hcl
resource "aws_internet_gateway" "main" {

  vpc_id = aws_vpc.main.id

  tags = {

    Name = "terraform-igw"

  }

}
```

---

# Internet Gateway Purpose

Without Internet Gateway:

```
Internet

❌

VPC
```

With Internet Gateway:

```
Internet

↓

Internet Gateway

↓

VPC
```

---

# Step 11 - Create Route Table

Open:

```
route-table.tf
```

Add:

```hcl
resource "aws_route_table" "public" {

  vpc_id = aws_vpc.main.id

  route {

    cidr_block = "0.0.0.0/0"

    gateway_id = aws_internet_gateway.main.id

  }

  tags = {

    Name = "public-route-table"

  }

}
```

---

# Understanding the Route

```
0.0.0.0/0
```

means:

```
Any Destination

↓

Internet Gateway
```

So any internet traffic is sent through the Internet Gateway.

---

# Step 12 - Associate Route Table

Continue inside:

```
route-table.tf
```

Add:

```hcl
resource "aws_route_table_association" "public" {

  subnet_id = aws_subnet.public.id

  route_table_id = aws_route_table.public.id

}
```

---

# Why Association?

AWS does not automatically know which subnet should use which route table.

This resource connects:

```
Public Subnet

↓

Public Route Table
```

---

# Network Flow

Your network now looks like:

```
Internet

      |

      ▼

Internet Gateway

      |

      ▼

Public Route Table

      |

      ▼

Public Subnet

      |

      ▼

EC2 (Next Part)
```

---

# Step 13 - Format Files

Run:

```bash
terraform fmt
```
---

# Step 14 - Validate

Run:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```
---

# Step 15 - Plan

Run:

```bash
terraform plan
```

Terraform should show resources similar to:

```
+ aws_vpc.main

+ aws_subnet.public

+ aws_subnet.private

+ aws_internet_gateway.main

+ aws_route_table.public

+ aws_route_table_association.public
```
---

# Step 16 - Apply

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
✓ VPC

✓ Public Subnet

✓ Private Subnet

✓ Internet Gateway

✓ Route Table

✓ Route Table Association
```

---

# Step 17 - Verify in AWS Console

Go to:

```
AWS Console

↓

VPC Dashboard
```

Verify:

### VPC

```
terraform-vpc
```

### Subnets

```
public-subnet

private-subnet
```

### Internet Gateway

```
terraform-igw
```

### Route Table

```
public-route-table
```

---

# Architecture After Part 2

```
                 Internet
                     |
                     |
             Internet Gateway
                     |
                     |
          Public Route Table
                     |
          -------------------
          |                 |
          |                 |
   Public Subnet      Private Subnet
          |
          |
   (EC2 comes next)
```

---

# Lab Verification Checklist

Verify:

✅ Custom VPC created

✅ Public subnet created

✅ Private subnet created

✅ Internet Gateway attached

✅ Route table created

✅ Public subnet associated with route table

✅ Validation successful

---

# End of Part 2

Next Part:

# Part 3 - Launch EC2 into the Public Subnet

You will learn:

- Security Groups
- EC2 inside your custom VPC
- SSH access
- Apache Web Server
- Testing from your browser
- Complete AWS networking project
---

# Part 3 - Launch EC2 in the Custom VPC

Current architecture:

```

Internet

│

▼

Internet Gateway

│

▼

Route Table

│

▼

Public Subnet

│

▼

EC2 Web Server

### Step 18 - Find Latest Amazon Linux AMI

create 

```bash
data.tf
```
```bash
data "aws_ami" "amazon_linux" {

  most_recent = true

  owners = ["amazon"]

  filter {

    name = "name"

    values = ["al2023-ami-*-x86_64"]

  }

}
```
### Step 19 - Create Security Group

Open:
```bash
security-group.tf
```
```bash
resource "aws_security_group" "web" {

  name = "terraform-web-sg"

  description = "Allow SSH and HTTP"

  vpc_id = aws_vpc.main.id

  ingress {

    from_port = 22

    to_port = 22

    protocol = "tcp"

    cidr_blocks = ["0.0.0.0/0"]

  }

  ingress {

    from_port = 80

    to_port = 80

    protocol = "tcp"

    cidr_blocks = ["0.0.0.0/0"]

  }

  egress {

    from_port = 0

    to_port = 0

    protocol = "-1"

    cidr_blocks = ["0.0.0.0/0"]

  }

  tags = {

    Name = "terraform-web-sg"

  }

}
```
Security Group Explanation

This allows:
```bash
SSH

Port 22

↓

Remote Login


HTTP

Port 80

↓

Website Access
```
Everything else remains blocked unless another rule allows it.

### Step 20 - Create User Data Script

Create:
```bash 
user-data.sh
```
```bash
#!/bin/bash

dnf update -y

dnf install -y httpd

systemctl enable httpd

systemctl start httpd

cat <<EOF > /var/www/html/index.html
<html>
<head>
<title>Terraform Lab</title>
</head>
<body>

<h1>Congratulations!</h1>

<h2>Your Terraform Networking Lab Works!</h2>

<p>Custom VPC</p>

<p>Public Subnet</p>

<p>Internet Gateway</p>

<p>Route Table</p>

<p>EC2 Instance</p>

<p>Apache Installed Automatically</p>

</body>
</html>
EOF
```
### Step 21 - Create EC2

Open:

```bash
ec2.tf
```
```bash
resource "aws_instance" "web" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = var.instance_type

  subnet_id = aws_subnet.public.id

  vpc_security_group_ids = [

    aws_security_group.web.id

  ]

  associate_public_ip_address = true

  user_data = file("user-data.sh")

  tags = {

    Name = "terraform-web-server"

  }

}
```
Why associate_public_ip_address?

Without:
```bash
Public IP

❌

No browser access
```
With:
```bash
Public IP

↓

Website works
```
### Step 22 - Create Outputs

Open:
```bash
outputs.tf
```
```bash
output "vpc_id" {

  value = aws_vpc.main.id

}

output "public_subnet_id" {

  value = aws_subnet.public.id

}

output "private_subnet_id" {

  value = aws_subnet.private.id

}

output "instance_id" {

  value = aws_instance.web.id

}

output "public_ip" {

  value = aws_instance.web.public_ip

}

output "website" {

  value = "http://${aws_instance.web.public_ip}"

}

```
### Step 23 - Format

Run:
```bash
terraform fmt
```
### Step 24 - Validate

Run:
```bash
terraform validate
```
Expected:
Success! The configuration is valid.
### Step 25 - Plan

Run:
```bash
terraform plan
```
Terraform should display resources similar to:
```bash
aws_vpc.main

aws_subnet.public

aws_subnet.private

aws_internet_gateway.main

aws_route_table.public

aws_security_group.web

aws_instance.web
```
### Step 26 - Apply

Run:
```bash
terraform apply
```
Terraform creates everything in dependency order.
### Step 27 - Verify in AWS

Open:
```bash
AWS Console

↓

VPC
```
Check:
```bash
VPC

Public Subnet

Private Subnet

Internet Gateway

Route Table
```
Verify:
```bash
terraform-web-server

Running
```
### Step 28 - Test Website

Terraform outputs:
```bash
website =
http://xx.xx.xx.xx
```
Open the URL in your browser.

Expected page:
```bash

Congratulations!

Your Terraform Networking Lab Works!
```

Cleanup

When you're finished:
```bash
terraform destroy
```
```bash
ls -la
```
Remove the .terraform directory:
```bash
rm -rf .terraform
```
Remove the lock file:
```bash
rm -f .terraform.lock.hcl
```
Verify:
```bash
ls -la
```
This removes all resources and helps avoid AWS charges.
Git Save
```bash
git status

git add .

git commit -m "Complete Lab 12 AWS Networking Project"

git push origin main
```
Interview Questions

Beginner
```bash
What is a VPC?
Difference between a public and private subnet?
Why do we need an Internet Gateway?
What is a Route Table?
What is a Security Group?
```
Intermediate
```bash
Why is map_public_ip_on_launch important?
Why associate a Route Table with a subnet?
What happens if an Internet Gateway is missing?
Difference between Security Groups and NACLs?
Why use User Data?
```
Advanced
```bash
How would you make the private subnet access the internet?
Where would you place an RDS database?
Why should application servers be in private subnets?
What AWS resources are required for a highly available VPC?
How would you improve this architecture for production?
```
