# Lab 16 - Terraform Modules

## Objective

In this lab, you will learn how to create and use **Terraform Modules** to make your infrastructure reusable, organized, and easier to maintain.

Instead of writing the same Terraform code multiple times, you'll create reusable modules and call them from the root module.

---

## What You Will Build

- VPC Module
- EC2 Module
- Security Group Module
- Root Module
- Module Variables
- Module Outputs

---

## What is a Terraform Module?

A **Terraform Module** is a collection of Terraform files that work together to create a specific piece of infrastructure.

Examples:

- VPC Module
- EC2 Module
- Security Group Module
- RDS Module

Instead of copying code, you can reuse a module multiple times.

---

## Why Use Modules?

Without modules:

- Duplicate code
- Difficult to maintain
- Hard to reuse

With modules:

- Reusable code
- Easier maintenance
- Better organization
- Production best practice

---

## Project Structure

```text
terraform-modules/

├── main.tf
├── provider.tf
├── variables.tf
├── terraform.tfvars
├── outputs.tf
├── versions.tf
│
└── modules/
    ├── vpc/
    ├── ec2/
    └── security-group/
```

---

## Architecture

```text
             Root Module

        /        |        \

       /         |         \

  VPC Module  SG Module  EC2 Module
```

---

## Step 1 - Create the Lab File

```bash
touch labs/lab-16-terraform-modules.md
```

---

## Step 2 - Create the Project Directory

```bash
mkdir -p terraform/lab-16-modules

cd terraform/lab-16-modules
```

---

## Step 3 - Create Root Module Files

```bash
touch versions.tf
touch provider.tf
touch variables.tf
touch terraform.tfvars
touch main.tf
touch outputs.tf
```

---

## Step 4 - Create the Modules Directory

```bash
mkdir modules
```

---

## Step 5 - Create Module Folders

```bash
mkdir modules/vpc

mkdir modules/security-group

mkdir modules/ec2
```

---

## Step 6 - Create Module Files

### VPC Module

```bash
touch modules/vpc/main.tf
touch modules/vpc/variables.tf
touch modules/vpc/outputs.tf
```

---

### Security Group Module

```bash
touch modules/security-group/main.tf
touch modules/security-group/variables.tf
touch modules/security-group/outputs.tf
```

---

### EC2 Module

```bash
touch modules/ec2/main.tf
touch modules/ec2/variables.tf
touch modules/ec2/outputs.tf
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
├── versions.tf
└── modules
    ├── ec2
    │   ├── main.tf
    │   ├── outputs.tf
    │   └── variables.tf
    ├── security-group
    │   ├── main.tf
    │   ├── outputs.tf
    │   └── variables.tf
    └── vpc
        ├── main.tf
        ├── outputs.tf
        └── variables.tf
```

---

## What You Will Learn

- Root Module
- Child Module
- Module Inputs
- Module Outputs
- Module Reusability
- Terraform Best Practices

---

# Step 1 - Create `versions.tf`

## Objective

Configure the Terraform version and AWS provider required for the root module.

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
- `required_providers` – Specifies the providers used by the project.
- `source = "hashicorp/aws"` – Uses the official AWS provider.
- `version = "~> 6.0"` – Uses AWS Provider **6.x** while preventing automatic upgrades to **7.x**.

---

## Initialize the Project

Run:

```bash
terraform fmt
terraform init
terraform validate
```
-------
# Step 2 - Create `provider.tf`

## Objective

Configure the AWS provider and apply default tags to all supported resources.

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

      Environment = "Lab-16"

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
- `Project`, `Environment`, and `ManagedBy` help identify and organize resources.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---
# Step 3 - Create `variables.tf`

## Objective

Define the input variables that will be passed from the root module to the child modules.

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

variable "vpc_cidr" {

  description = "VPC CIDR Block"

  type = string

}

variable "public_subnet_cidr" {

  description = "Public Subnet CIDR"

  type = string

}

variable "availability_zone" {

  description = "Availability Zone"

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

- `aws_region` – AWS Region where the infrastructure will be created.
- `vpc_cidr` – CIDR block for the VPC.
- `public_subnet_cidr` – CIDR block for the public subnet.
- `availability_zone` – Availability Zone for the subnet.
- `instance_type` – EC2 instance type.
- `key_name` – Existing AWS EC2 Key Pair name.

These variables will be passed from the **root module** to the **VPC**, **Security Group**, and **EC2** modules.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---------------
# Step 4 - Create `terraform.tfvars`

## Objective

Provide values for the variables defined in `variables.tf`.

---

## Open the File

```bash
nano terraform.tfvars
```

---

## Add the Following Code

```hcl
aws_region = "ap-south-1"

vpc_cidr = "10.0.0.0/16"

public_subnet_cidr = "10.0.1.0/24"

availability_zone = "ap-south-1a"

instance_type = "t2.micro"

key_name = "terraform-key"
```

---

## Explanation

- `aws_region` – AWS Region where resources will be created.
- `vpc_cidr` – CIDR block for the VPC.
- `public_subnet_cidr` – CIDR block for the public subnet.
- `availability_zone` – Availability Zone for the public subnet.
- `instance_type` – EC2 instance type.
- `key_name` – Existing AWS EC2 Key Pair used to connect to the EC2 instance.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
----------------
# Step 5 - Create `modules/vpc/main.tf`

## Objective

Create a reusable VPC module that provisions a VPC and a public subnet.

The root module will call this module whenever a VPC is required.

---

## Open the File

```bash
nano modules/vpc/main.tf
```

---

## Add the Following Code

```hcl
resource "aws_vpc" "main" {

  cidr_block = var.vpc_cidr

  enable_dns_support = true

  enable_dns_hostnames = true

  tags = {

    Name = "lab16-vpc"

  }

}

resource "aws_subnet" "public" {

  vpc_id = aws_vpc.main.id

  cidr_block = var.public_subnet_cidr

  availability_zone = var.availability_zone

  map_public_ip_on_launch = true

  tags = {

    Name = "lab16-public-subnet"

  }

}
```

---

## Explanation

### VPC

```hcl
resource "aws_vpc" "main"
```

Creates a VPC using the CIDR block passed from the root module.

---

### Public Subnet

```hcl
resource "aws_subnet" "public"
```

Creates a public subnet inside the VPC.

---

### Public IP Assignment

```hcl
map_public_ip_on_launch = true
```

Automatically assigns a public IP address to EC2 instances launched in this subnet.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
--------------------
# Step 6 - Create `modules/vpc/variables.tf`

## Objective

Define the input variables required by the VPC module.

These variables receive values from the root module when the VPC module is called.

---

## Open the File

```bash
nano modules/vpc/variables.tf
```

---

## Add the Following Code

```hcl
variable "vpc_cidr" {

  description = "VPC CIDR Block"

  type = string

}

variable "public_subnet_cidr" {

  description = "Public Subnet CIDR"

  type = string

}

variable "availability_zone" {

  description = "Availability Zone"

  type = string

}
```

---

## Explanation

- `vpc_cidr` – CIDR block used to create the VPC.
- `public_subnet_cidr` – CIDR block for the public subnet.
- `availability_zone` – Availability Zone where the subnet will be created.

These values are passed from the **root module** when calling the VPC module.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---
# Step 7 - Create `modules/vpc/outputs.tf`

## Objective

Export values from the VPC module so they can be used by other modules, such as the Security Group and EC2 modules.

---

## Open the File

```bash
nano modules/vpc/outputs.tf
```

---

## Add the Following Code

```hcl
output "vpc_id" {

  description = "VPC ID"

  value = aws_vpc.main.id

}

output "public_subnet_id" {

  description = "Public Subnet ID"

  value = aws_subnet.public.id

}
```

---

## Explanation

### VPC ID

```hcl
output "vpc_id"
```

Exports the ID of the VPC created by this module.

Example:

```text
vpc-0123456789abcdef0
```

The Security Group module will use this VPC ID to create security groups inside the VPC.

---

### Public Subnet ID

```hcl
output "public_subnet_id"
```

Exports the ID of the public subnet.

Example:

```text
subnet-0abc123456789def0
```

The EC2 module will use this subnet ID to launch EC2 instances.

---

## Module Output Flow

```text
Root Module

      │

      ▼

 VPC Module

      │

      ├── vpc_id

      └── public_subnet_id

      │

      ▼

Security Group Module

EC2 Module
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
----------------
# Step 8 - Create `modules/security-group/main.tf`

## Objective

Create a reusable Security Group module that allows SSH access to an EC2 instance.

The root module will use this Security Group when launching the EC2 instance.

---

## Open the File

```bash
nano modules/security-group/main.tf
```

---

## Add the Following Code

```hcl
resource "aws_security_group" "main" {

  name = "lab16-security-group"

  description = "Security Group for EC2"

  vpc_id = var.vpc_id

  ingress {

    description = "SSH"

    from_port = 22

    to_port = 22

    protocol = "tcp"

    cidr_blocks = ["0.0.0.0/0"]

  }

  ingress {

    description = "HTTP"

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

    Name = "lab16-security-group"

  }

}
```

---

## Explanation

### Security Group

```hcl
resource "aws_security_group" "main"
```

Creates a Security Group inside the VPC.

---

### VPC

```hcl
vpc_id = var.vpc_id
```

Creates the Security Group in the VPC received from the VPC module.

---

### SSH Rule

```hcl
from_port = 22
```

Allows SSH access on port **22**.

---

### HTTP Rule

```hcl
from_port = 80
```

Allows HTTP traffic on port **80** for the web server.

---

### Outbound Rule

```hcl
protocol = "-1"
```

Allows all outbound traffic.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---------------
# Step 9 - Create `modules/security-group/variables.tf`

## Objective

Define the input variables required by the Security Group module.

These variables receive values from the root module.

---

## Open the File

```bash
nano modules/security-group/variables.tf
```

---

## Add the Following Code

```hcl
variable "vpc_id" {

  description = "VPC ID"

  type = string

}
```

---

## Explanation

### VPC ID

```hcl
variable "vpc_id"
```

Receives the VPC ID from the root module.

The Security Group uses this value to create the security group inside the correct VPC.

Example:

```text
vpc-0123456789abcdef0
```

---

## Module Input Flow

```text
Root Module

      │

      ▼

VPC Module

      │

      ▼

vpc_id

      │

      ▼

Security Group Module
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
-----------
# Step 10 - Create `modules/security-group/outputs.tf`

## Objective

Export the Security Group ID so it can be used by other modules, such as the EC2 module.

---

## Open the File

```bash
nano modules/security-group/outputs.tf
```

---

## Add the Following Code

```hcl
output "security_group_id" {

  description = "Security Group ID"

  value = aws_security_group.main.id

}
```

---

## Explanation

### Security Group ID

```hcl
output "security_group_id"
```

Exports the ID of the Security Group created by this module.

Example:

```text
sg-0123456789abcdef0
```

The EC2 module will use this Security Group ID when launching the EC2 instance.

---

## Module Output Flow

```text
Root Module

      │

      ▼

Security Group Module

      │

      ▼

security_group_id

      │

      ▼

EC2 Module
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---------------------
# Step 11 - Create `modules/ec2/main.tf`

## Objective

Create a reusable EC2 module that launches an EC2 instance using the VPC and Security Group created by the other modules.

---

## Open the File

```bash
nano modules/ec2/main.tf
```

---

## Add the Following Code

```hcl
resource "aws_instance" "main" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = var.instance_type

  subnet_id = var.subnet_id

  vpc_security_group_ids = [

    var.security_group_id

  ]

  key_name = var.key_name

  tags = {

    Name = "lab16-ec2"

  }

}

data "aws_ami" "amazon_linux" {

  most_recent = true

  owners = ["amazon"]

  filter {

    name = "name"

    values = ["al2023-ami-*-x86_64"]

  }

}
```

---

## Explanation

### EC2 Instance

```hcl
resource "aws_instance" "main"
```

Creates an EC2 instance.

---

### AMI

```hcl
ami = data.aws_ami.amazon_linux.id
```

Automatically selects the latest Amazon Linux 2023 AMI.

---

### Instance Type

```hcl
instance_type = var.instance_type
```

Uses the EC2 instance type passed from the root module.

---

### Subnet

```hcl
subnet_id = var.subnet_id
```

Launches the EC2 instance in the subnet received from the VPC module.

---

### Security Group

```hcl
vpc_security_group_ids = [

  var.security_group_id

]
```

Attaches the Security Group received from the Security Group module.

---

### Key Pair

```hcl
key_name = var.key_name
```

Uses the existing AWS EC2 Key Pair.

---

### Data Source

```hcl
data "aws_ami" "amazon_linux"
```

Finds the latest Amazon Linux 2023 AMI automatically.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
-------------
# Step 12 - Create `modules/ec2/variables.tf`

## Objective

Define the input variables required by the EC2 module.

These variables receive values from the root module.

---

## Open the File

```bash
nano modules/ec2/variables.tf
```

---

## Add the Following Code

```hcl
variable "instance_type" {

  description = "EC2 Instance Type"

  type = string

}

variable "subnet_id" {

  description = "Subnet ID"

  type = string

}

variable "security_group_id" {

  description = "Security Group ID"

  type = string

}

variable "key_name" {

  description = "AWS EC2 Key Pair Name"

  type = string

}
```

---

## Explanation

### Instance Type

```hcl
variable "instance_type"
```

Receives the EC2 instance type from the root module.

Example:

```text
t2.micro
```

---

### Subnet ID

```hcl
variable "subnet_id"
```

Receives the subnet ID from the VPC module.

The EC2 instance is launched in this subnet.

---

### Security Group ID

```hcl
variable "security_group_id"
```

Receives the Security Group ID from the Security Group module.

The EC2 instance uses this Security Group to control network access.

---

### Key Pair

```hcl
variable "key_name"
```

Receives the existing AWS EC2 Key Pair name from the root module.

This key pair is used to connect to the EC2 instance using SSH.

---

## Module Input Flow

```text
Root Module
     │
     ├── instance_type
     ├── key_name
     │
     ▼
VPC Module ───────────────► subnet_id
     │
     ▼
Security Group Module ────► security_group_id
     │
     ▼
EC2 Module
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---------------
# Step 13 - Create `modules/ec2/outputs.tf`

## Objective

Export the EC2 instance details so they can be accessed from the root module.

---

## Open the File

```bash
nano modules/ec2/outputs.tf
```

---

## Add the Following Code

```hcl
output "instance_id" {

  description = "EC2 Instance ID"

  value = aws_instance.main.id

}

output "public_ip" {

  description = "EC2 Public IP"

  value = aws_instance.main.public_ip

}

output "public_dns" {

  description = "EC2 Public DNS"

  value = aws_instance.main.public_dns

}
```

---

## Explanation

### Instance ID

```hcl
output "instance_id"
```

Exports the EC2 Instance ID.

Example:

```text
i-0123456789abcdef0
```

---

### Public IP

```hcl
output "public_ip"
```

Exports the public IP address assigned to the EC2 instance.

Example:

```text
13.233.120.45
```

You can use this IP to connect to the EC2 instance using SSH.

---

### Public DNS

```hcl
output "public_dns"
```

Exports the public DNS name of the EC2 instance.

Example:

```text
ec2-13-233-120-45.ap-south-1.compute.amazonaws.com
```

You can use either the public IP or the public DNS name to connect to the EC2 instance.

---

## Module Output Flow

```text
EC2 Module

     │

     ├── instance_id

     ├── public_ip

     └── public_dns

     │

     ▼

Root Module
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
-----------------------
# Step 14 - Create `main.tf`

## Objective

Use the root module to call the **VPC**, **Security Group**, and **EC2** modules.

The root module connects all child modules to build the complete infrastructure.

---

## Open the File

```bash
nano main.tf
```

---

## Step 14.1 - Call the VPC Module

Add the following code:

```hcl
module "vpc" {

  source = "./modules/vpc"

  vpc_cidr = var.vpc_cidr

  public_subnet_cidr = var.public_subnet_cidr

  availability_zone = var.availability_zone

}
```

---

## Explanation

- `module "vpc"` – Calls the VPC module.
- `source` – Specifies the location of the module.
- Passes the VPC variables from the root module to the VPC module.

---

## Step 14.2 - Call the Security Group Module

Add the following code below the VPC module:

```hcl
module "security_group" {

  source = "./modules/security-group"

  vpc_id = module.vpc.vpc_id

}
```

---

## Explanation

- `module "security_group"` – Calls the Security Group module.
- `module.vpc.vpc_id` – Uses the VPC ID exported by the VPC module.

---

## Step 14.3 - Call the EC2 Module

Add the following code below the Security Group module:

```hcl
module "ec2" {

  source = "./modules/ec2"

  instance_type = var.instance_type

  subnet_id = module.vpc.public_subnet_id

  security_group_id = module.security_group.security_group_id

  key_name = var.key_name

}
```

---

## Explanation

- `module "ec2"` – Calls the EC2 module.
- `instance_type` – Passes the EC2 instance type.
- `subnet_id` – Uses the Public Subnet ID exported by the VPC module.
- `security_group_id` – Uses the Security Group ID exported by the Security Group module.
- `key_name` – Passes the existing AWS EC2 Key Pair.

---

## Module Dependency Flow

```text
Root Module
     │
     ▼
 VPC Module
     │
     ├── vpc_id
     ├── public_subnet_id
     ▼
Security Group Module
     │
     ├── security_group_id
     ▼
 EC2 Module
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```
---------------------
# Step 15 - Create `outputs.tf`

## Objective

Display the values exported by the child modules after the infrastructure is created.

Outputs allow you to view important resource information without opening the AWS Console.

---

## Open the File

```bash
nano outputs.tf
```

---

## Add the Following Code

```hcl
output "vpc_id" {

  description = "VPC ID"

  value = module.vpc.vpc_id

}

output "public_subnet_id" {

  description = "Public Subnet ID"

  value = module.vpc.public_subnet_id

}

output "security_group_id" {

  description = "Security Group ID"

  value = module.security_group.security_group_id

}

output "instance_id" {

  description = "EC2 Instance ID"

  value = module.ec2.instance_id

}

output "public_ip" {

  description = "EC2 Public IP"

  value = module.ec2.public_ip

}

output "public_dns" {

  description = "EC2 Public DNS"

  value = module.ec2.public_dns

}
```

---

## Explanation

### VPC ID

```hcl
module.vpc.vpc_id
```

Displays the VPC ID exported by the VPC module.

---

### Public Subnet ID

```hcl
module.vpc.public_subnet_id
```

Displays the Public Subnet ID created by the VPC module.

---

### Security Group ID

```hcl
module.security_group.security_group_id
```

Displays the Security Group ID created by the Security Group module.

---

### EC2 Instance ID

```hcl
module.ec2.instance_id
```

Displays the EC2 Instance ID created by the EC2 module.

---

### Public IP

```hcl
module.ec2.public_ip
```

Displays the EC2 Public IP address.

---

### Public DNS

```hcl
module.ec2.public_dns
```

Displays the EC2 Public DNS name.

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
vpc_id = "vpc-0123456789abcdef0"

public_subnet_id = "subnet-0123456789abcdef0"

security_group_id = "sg-0123456789abcdef0"

instance_id = "i-0123456789abcdef0"

public_ip = "13.233.120.45"

public_dns = "ec2-13-233-120-45.ap-south-1.compute.amazonaws.com"
```

---

## Verify in AWS Console

Confirm the following resources were created:

- VPC
- Public Subnet
- Security Group
- EC2 Instance

---

## Summary

You created:

- VPC Module
- Security Group Module
- EC2 Module
- Root Module
- Module Inputs
- Module Outputs

You also learned:

- Root Modules
- Child Modules
- Module Variables
- Module Outputs
- Reusable Infrastructure
- Module Communication

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

---

## Step 2 - Verify AWS Console

Confirm these resources have been deleted:

- EC2 Instance
- Security Group
- Public Subnet
- VPC

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
main.tf
outputs.tf
provider.tf
terraform.tfvars
variables.tf
versions.tf
modules/
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

git commit -m "Complete Lab 16 Terraform Modules"
```

---

## Step 7 - Push to GitHub

```bash
git push origin main
```

---

# Lab Completed

You learned:

- Terraform Modules
- Root Module
- Child Modules
- Module Inputs
- Module Outputs
- Module Reusability
- Module Communication
- Infrastructure Cleanup

---

# Congratulations!

You have successfully completed **Lab 16 – Terraform Modules**.

You can now build reusable and production-ready Terraform configurations using modules instead of duplicating code.

