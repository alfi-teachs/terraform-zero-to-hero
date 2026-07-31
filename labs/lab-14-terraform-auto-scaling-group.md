# Lab 14 - Terraform Auto Scaling Group (ASG)

## Objective

In this lab, you will create an **Auto Scaling Group (ASG)** that automatically launches and replaces EC2 instances.

The Auto Scaling Group will use the **Application Load Balancer (ALB)** created in the previous lab to distribute traffic across healthy instances.

---

## What You Will Build

- Launch Template
- Auto Scaling Group (ASG)
- Desired Capacity
- Minimum Capacity
- Maximum Capacity
- Automatic EC2 Replacement
- Integration with an Application Load Balancer
- Health Checks

---

## Architecture

```text
                    Internet
                        |
                        |
          Application Load Balancer
                        |
                  Target Group
                        |
              Auto Scaling Group
                /             \
               /               \
      EC2 Instance        EC2 Instance
        (Auto)              (Auto)
               \               /
                \             /
              Launch Template
```

---

## Step 1 - Create the Lab File

Create the lab file:

```bash
touch labs/lab-14-terraform-auto-scaling-group.md
```

---

## Step 2 - Create the Project Directory

```bash
mkdir -p terraform/lab-14-auto-scaling-group
cd terraform/lab-14-auto-scaling-group
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
touch launch-template.tf
touch auto-scaling-group.tf
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
auto-scaling-group.tf
data.tf
igw.tf
launch-template.tf
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

## What You Will Learn

- Launch Templates
- Auto Scaling Groups
- Desired Capacity
- Minimum Capacity
- Maximum Capacity
- Automatic EC2 Replacement
- Health Checks
- Application Load Balancer Integration
- High Availability
- Fault Tolerance

---

## Lab Outcome

At the end of this lab, you will have:

- A VPC with two public subnets
- An Internet Gateway
- A Route Table
- An Application Load Balancer
- A Launch Template
- An Auto Scaling Group
- Two EC2 instances managed automatically by the ASG
- Automatic replacement of unhealthy EC2 instances
- An ALB distributing traffic across healthy instances

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

      Environment = "Lab-14"

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
- `Project`, `Environment`, and `ManagedBy` help organize and identify AWS resources.

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

Define the input variables used throughout the Auto Scaling Group project.

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

variable "desired_capacity" {

  description = "Desired number of EC2 instances"

  type = number

}

variable "min_size" {

  description = "Minimum number of EC2 instances"

  type = number

}

variable "max_size" {

  description = "Maximum number of EC2 instances"

  type = number

}
```

---

## Explanation

- `aws_region` – AWS Region where resources will be created.
- `vpc_cidr` – CIDR block for the VPC.
- `public_subnet_1_cidr` – CIDR block for the first public subnet.
- `public_subnet_2_cidr` – CIDR block for the second public subnet.
- `availability_zone_1` – Availability Zone for the first subnet.
- `availability_zone_2` – Availability Zone for the second subnet.
- `instance_type` – EC2 instance type for the Launch Template.
- `key_name` – Existing AWS EC2 Key Pair used for SSH.
- `desired_capacity` – Number of EC2 instances the ASG tries to maintain.
- `min_size` – Minimum number of EC2 instances the ASG keeps running.
- `max_size` – Maximum number of EC2 instances the ASG can launch.

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

desired_capacity = 2

min_size = 2

max_size = 4
```

---

## Explanation

- `terraform.tfvars` stores the actual values for the Terraform variables.
- Terraform automatically loads this file during execution.
- `desired_capacity = 2` launches two EC2 instances.
- `min_size = 2` ensures at least two EC2 instances are always running.
- `max_size = 4` allows the Auto Scaling Group to scale up to four EC2 instances when needed.

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

Create a Virtual Private Cloud (VPC) for the Auto Scaling Group infrastructure.

The VPC provides an isolated network where the Application Load Balancer and EC2 instances will run.

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

    Name = "lab14-vpc"

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
---
# Step 6 - Create `subnet.tf`

## Objective

Create two public subnets in different Availability Zones.

The Application Load Balancer and Auto Scaling Group will use these subnets to provide high availability.

---

## What is a Public Subnet?

A **public subnet** is a subnet that has a route to an Internet Gateway. EC2 instances launched in the subnet can receive public IP addresses.

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

    Name = "lab14-public-subnet-1"

  }

}

resource "aws_subnet" "public_2" {

  vpc_id = aws_vpc.main.id

  cidr_block = var.public_subnet_2_cidr

  availability_zone = var.availability_zone_2

  map_public_ip_on_launch = true

  tags = {

    Name = "lab14-public-subnet-2"

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
- `tags` – Adds names to identify the subnets in the AWS Console.

---

## Why Two Public Subnets?

Using two public subnets in different Availability Zones provides:

- High Availability
- Fault Tolerance
- Better application uptime

The Application Load Balancer distributes traffic between instances running in both subnets.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```

------------
# Step 7 - Create `igw.tf`

## Objective

Create an Internet Gateway (IGW) and attach it to the VPC.

The Internet Gateway allows the Application Load Balancer and EC2 instances in the public subnets to communicate with the internet.

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

    Name = "lab14-igw"

  }

}
```

---

## Explanation

- `resource "aws_internet_gateway" "main"` – Creates an Internet Gateway.
- `vpc_id = aws_vpc.main.id` – Attaches the Internet Gateway to the VPC.
- `tags` – Adds a name to identify the Internet Gateway in the AWS Console.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```
-------------------
# Step 8 - Create `route-table.tf`

## Objective

Create a Public Route Table and associate it with both public subnets.

This enables the Application Load Balancer and Auto Scaling Group instances to access the internet through the Internet Gateway.

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

    Name = "lab14-public-route-table"

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
--------------------
# Step 9 - Create `security-group.tf`

## Objective

Create Security Groups for:

- Application Load Balancer (ALB)
- Auto Scaling Group EC2 Instances

The ALB accepts HTTP requests from the internet and forwards them to the EC2 instances managed by the Auto Scaling Group.

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

  name        = "lab14-alb-sg"

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

    Name = "lab14-alb-sg"

  }

}
```

---

## EC2 Security Group

Add the following configuration below the previous resource:

```hcl
resource "aws_security_group" "web" {

  name        = "lab14-web-sg"

  description = "Security Group for Auto Scaling EC2 Instances"

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

    Name = "lab14-web-sg"

  }

}
```

---

## Explanation

- `aws_security_group.alb` – Creates a Security Group for the Application Load Balancer.
- `HTTP (80)` – Allows users on the internet to access the ALB.
- `aws_security_group.web` – Creates a Security Group for the Auto Scaling EC2 instances.
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
-------------------
# Step 10 - Create `data.tf`

## Objective

Use a Terraform Data Source to automatically retrieve the latest Amazon Linux 2023 AMI.

This ensures your Auto Scaling Group always launches EC2 instances with the latest Amazon Linux image.

---

## What is a Data Source?

A **Data Source** reads existing information from AWS without creating any resources.

Examples include:

- AMIs
- Availability Zones
- VPCs
- Security Groups

---

## Open the File

```bash
nano data.tf
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
```

---

## Explanation

- `data "aws_ami" "amazon_linux"` – Searches AWS for an Amazon Linux AMI.
- `most_recent = true` – Selects the latest available AMI.
- `owners = ["amazon"]` – Uses only official AWS-published AMIs.
- `filter` – Searches for Amazon Linux 2023 x86_64 images.

Terraform will later reference this AMI using:

```hcl
data.aws_ami.amazon_linux.id
```

---

## Why Use a Data Source?

Instead of hardcoding an AMI ID:

```hcl
ami = "ami-0123456789abcdef0"
```

Terraform automatically finds the latest Amazon Linux 2023 AMI for the selected AWS Region.

This makes the configuration easier to maintain and portable across regions.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```
-------------------
# Step 11 - Create `user-data.sh`

## Objective

Create a User Data script that automatically updates the EC2 instance, installs Apache, and creates a web page when the instance starts.

Every EC2 instance launched by the Auto Scaling Group will run this script automatically.

---

## What is User Data?

**User Data** is a shell script that runs automatically the first time an EC2 instance starts.

Common uses:

- Update packages
- Install software
- Configure applications
- Start services

---

## Open the File

```bash
nano user-data.sh
```

---

## Add the Following Code

```bash
#!/bin/bash

dnf update -y

dnf install -y httpd

systemctl enable httpd

systemctl start httpd

INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)

cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Terraform ASG Lab</title>
</head>
<body>
    <h1>Terraform Auto Scaling Group Lab</h1>
    <h2>Instance ID: $INSTANCE_ID</h2>
    <p>This EC2 instance was launched by the Auto Scaling Group.</p>
</body>
</html>
EOF
```

---

## Explanation

- `#!/bin/bash` – Runs the script using the Bash shell.
- `dnf update -y` – Updates installed packages.
- `dnf install -y httpd` – Installs the Apache web server.
- `systemctl enable httpd` – Starts Apache automatically after reboot.
- `systemctl start httpd` – Starts the Apache service.
- `INSTANCE_ID=$(...)` – Retrieves the EC2 instance ID from the Instance Metadata Service.
- `cat <<EOF ... EOF` – Creates a web page displaying the EC2 instance ID.

---

## Why Display the Instance ID?

The Auto Scaling Group may launch or replace EC2 instances automatically.

When you refresh the ALB DNS name in your browser, the displayed **Instance ID** lets you see which EC2 instance handled your request.

If an instance is replaced, you'll see a new Instance ID after the replacement is complete.

---

## (Optional) Make the Script Executable

```bash
chmod +x user-data.sh
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```
----------------
