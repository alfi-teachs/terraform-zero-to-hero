# Lab 13 - Terraform AWS Application Load Balancer (ALB)

## Step 1 - Create the Lab File

Create the lab file:

```bash
touch labs/lab-13-terraform-aws-alb.md
```

---

## Step 2 - Create the Project Directory

```bash
mkdir -p terraform/lab-13-alb
cd terraform/lab-13-alb
```

---

## Step 3 - Create Terraform Files

```bash
touch versions.tf
touch provider.tf
touch variables.tf
touch terraform.tfvars
touch vpc.tf
touch subnet.tf
touch igw.tf
touch route-table.tf
touch security-group.tf
touch data.tf
touch user-data.sh
touch ec2.tf
touch alb.tf
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
alb.tf
data.tf
ec2.tf
igw.tf
outputs.tf
provider.tf
route-table.tf
security-group.tf
subnet.tf
terraform.tfvars
user-data.sh
variables.tf
versions.tf
vpc.tf
```

---

## Step 5 - Lab Overview

In this lab, you will create:

- VPC
- Two Public Subnets
- Internet Gateway
- Route Table
- Security Groups
- Two EC2 Web Servers
- Application Load Balancer (ALB)
- Target Group
- Listener
- Outputs

---

## Next Step

Next, create **`versions.tf`** and configure the Terraform version and AWS provider.

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

- `terraform {}` – Defines Terraform project settings.
- `required_version` – Ensures Terraform version **1.5.0 or later** is used.
- `required_providers` – Specifies the providers required by the project.
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

Configure the AWS provider and define default tags that will be applied to supported AWS resources.

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

      Environment = "Lab-13"

      ManagedBy   = "Terraform"

    }

  }

}
```

---

## Explanation

- `provider "aws"` – Configures the AWS provider.
- `region = var.aws_region` – Uses the AWS region from the `aws_region` variable.
- `default_tags` – Automatically applies the specified tags to supported AWS resources.
- `Project`, `Environment`, `ManagedBy` – Common tags used to organize and identify AWS resources.

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

Define the input variables used throughout the Terraform configuration.

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

variable "public_subnet_1_cidr" {

  description = "Public Subnet 1 CIDR"

  type = string

}

variable "public_subnet_2_cidr" {

  description = "Public Subnet 2 CIDR"

  type = string

}

variable "availability_zone_1" {

  description = "Availability Zone 1"

  type = string

}

variable "availability_zone_2" {

  description = "Availability Zone 2"

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

- `variable` – Declares an input variable.
- `description` – Describes the purpose of the variable.
- `type = string` – Specifies that the variable accepts text values.
- Variables make the Terraform configuration reusable and easier to maintain.

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

vpc_cidr = "10.0.0.0/16"

public_subnet_1_cidr = "10.0.1.0/24"

public_subnet_2_cidr = "10.0.2.0/24"

availability_zone_1 = "ap-south-1a"

availability_zone_2 = "ap-south-1b"

instance_type = "t2.micro"

key_name = "terraform-key"
```

---

## Explanation

- `terraform.tfvars` stores the actual values for the variables.
- Terraform automatically loads this file during execution.
- Keeping values separate from the main configuration makes the code reusable and easier to manage.

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
# Step 5 - Create `vpc.tf`

## Objective

Create a Virtual Private Cloud (VPC) to host the Application Load Balancer and EC2 instances.

---

## What is a VPC?

A **VPC (Virtual Private Cloud)** is your own private network inside AWS where you can securely launch AWS resources.

---

## Open the File

```bash
nano vpc.tf
```

---

## Add the Following Code

```hcl
resource "aws_vpc" "main" {

  cidr_block = var.vpc_cidr

  enable_dns_support = true

  enable_dns_hostnames = true

  tags = {

    Name = "lab13-vpc"

  }

}
```

---

## Explanation

- `resource "aws_vpc" "main"` – Creates a new VPC.
- `cidr_block = var.vpc_cidr` – Defines the IP address range for the VPC.
- `enable_dns_support = true` – Enables DNS resolution within the VPC.
- `enable_dns_hostnames = true` – Assigns DNS hostnames to EC2 instances.
- `tags` – Adds a name to identify the VPC in the AWS Console.

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
# Step 6 - Create `subnet.tf`

## Objective

Create two public subnets in different Availability Zones for high availability.

---

## What is a Public Subnet?

A **public subnet** is a subnet that can communicate with the internet through an Internet Gateway. Resources launched here can receive public IP addresses.

---

## Open the File

```bash
nano subnet.tf
```

---

## Add the Following Code

```hcl
resource "aws_subnet" "public_1" {

  vpc_id = aws_vpc.main.id

  cidr_block = var.public_subnet_1_cidr

  availability_zone = var.availability_zone_1

  map_public_ip_on_launch = true

  tags = {

    Name = "lab13-public-subnet-1"

  }

}

resource "aws_subnet" "public_2" {

  vpc_id = aws_vpc.main.id

  cidr_block = var.public_subnet_2_cidr

  availability_zone = var.availability_zone_2

  map_public_ip_on_launch = true

  tags = {

    Name = "lab13-public-subnet-2"

  }

}
```

---

## Explanation

- `aws_subnet.public_1` – Creates the first public subnet.
- `aws_subnet.public_2` – Creates the second public subnet.
- `vpc_id` – Associates the subnet with the VPC.
- `cidr_block` – Defines the subnet's IP address range.
- `availability_zone` – Places each subnet in a different Availability Zone.
- `map_public_ip_on_launch = true` – Automatically assigns a public IP to EC2 instances launched in the subnet.
- `tags` – Adds names to identify the subnets.

---

## Why Two Public Subnets?

Application Load Balancers require **at least two subnets in different Availability Zones** for high availability.

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
# Step 7 - Create `igw.tf`

## Objective

Create an Internet Gateway (IGW) and attach it to the VPC.

The Internet Gateway allows resources in the **public subnets** to communicate with the internet.

---

## Network Diagram

```text
Internet
    |
    |
Internet Gateway
    |
    |
   VPC
```

---

## Open the File

```bash
nano igw.tf
```

---

## Add the Following Code

```hcl
resource "aws_internet_gateway" "main" {

  vpc_id = aws_vpc.main.id

  tags = {

    Name = "lab13-igw"

  }

}
```

---

## Explanation

- `resource "aws_internet_gateway" "main"` – Creates an Internet Gateway.
- `vpc_id = aws_vpc.main.id` – Attaches the Internet Gateway to the VPC.
- `tags` – Adds a name to identify the Internet Gateway.

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
# Step 8 - Create `route-table.tf`

## Objective

Create a Public Route Table and associate it with both public subnets.

This allows EC2 instances and the Application Load Balancer to access the internet through the Internet Gateway.

---

## Network Diagram

```text
                Internet
                    |
            Internet Gateway
                    |
            Public Route Table
               /           \
              /             \
     Public Subnet 1   Public Subnet 2
```

---

## Open the File

```bash
nano route-table.tf
```

---

## Add the Following Code

```hcl
resource "aws_route_table" "public" {

  vpc_id = aws_vpc.main.id

  route {

    cidr_block = "0.0.0.0/0"

    gateway_id = aws_internet_gateway.main.id

  }

  tags = {

    Name = "lab13-public-route-table"

  }

}

resource "aws_route_table_association" "public_1" {

  subnet_id = aws_subnet.public_1.id

  route_table_id = aws_route_table.public.id

}

resource "aws_route_table_association" "public_2" {

  subnet_id = aws_subnet.public_2.id

  route_table_id = aws_route_table.public.id

}
```

---

## Explanation

- `aws_route_table.public` – Creates a public route table.
- `gateway_id = aws_internet_gateway.main.id` – Routes internet traffic through the Internet Gateway.
- `aws_route_table_association.public_1` – Associates Public Subnet 1 with the public route table.
- `aws_route_table_association.public_2` – Associates Public Subnet 2 with the public route table.

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
# Step 9 - Create `security-group.tf`

## Objective

Create Security Groups for:

- Application Load Balancer (ALB)
- EC2 Web Servers

The ALB will accept HTTP requests from the internet and forward them to the EC2 instances.

---

## What We Will Create

| Security Group | Purpose |
|---------------|---------|
| `alb-sg` | Allow HTTP traffic from the internet |
| `web-sg` | Allow HTTP traffic only from the ALB and SSH from your computer |

---

## Open the File

```bash
nano security-group.tf
```

---

## ALB Security Group

Add the following configuration:

```hcl
resource "aws_security_group" "alb" {

  name        = "lab13-alb-sg"

  description = "Security Group for Application Load Balancer"

  vpc_id = aws_vpc.main.id

  ingress {

    description = "HTTP from Internet"

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

    Name = "lab13-alb-sg"

  }

}
```

---

## EC2 Security Group

Add the following configuration below the previous resource:

```hcl
resource "aws_security_group" "web" {

  name        = "lab13-web-sg"

  description = "Security Group for Web Servers"

  vpc_id = aws_vpc.main.id

  ingress {

    description = "HTTP from ALB"

    from_port = 80

    to_port = 80

    protocol = "tcp"

    security_groups = [

      aws_security_group.alb.id

    ]

  }

  ingress {

    description = "SSH from Anywhere"

    from_port = 22

    to_port = 22

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

    Name = "lab13-web-sg"

  }

}
```

---

## Explanation

- `aws_security_group.alb` – Creates a Security Group for the Application Load Balancer.
- `HTTP (80)` – Allows users on the internet to access the ALB.
- `aws_security_group.web` – Creates a Security Group for the EC2 web servers.
- `security_groups = [aws_security_group.alb.id]` – Allows HTTP traffic only from the ALB.
- `SSH (22)` – Allows SSH access for administration (recommended to restrict this to your public IP in production).
- `egress` – Allows all outbound traffic.

> **Note:** In production, replace `0.0.0.0/0` for SSH with your public IP address.

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


