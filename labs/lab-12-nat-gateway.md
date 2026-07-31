Create
```bash
touch labs/lab-13-nat-gateway.md
```
Go inside:
```bash
cd terraform/lab-13-nat-gateway
```

### Step 2 - Create Files
```bash
touch versions.tf

touch provider.tf

touch variables.tf

touch terraform.tfvars

touch data.tf

touch vpc.tf

touch subnet.tf

touch igw.tf

touch nat.tf

touch route-table.tf

touch security-group.tf

touch ec2.tf

touch outputs.tf

touch README.md

touch user-data.sh
```
Architecture
```bash
                Internet

                    |

                    ▼

          Internet Gateway

                    |

        -----------------------

        |                     |

        ▼                     ▼

 Public Subnet         Private Subnet

        |                     |

        ▼                     ▼

 NAT Gateway          Private EC2

        |

        ▼

 Elastic IP
 ```

 Resources We'll Create
 ```bash
 ✓ VPC

✓ Public Subnet

✓ Private Subnet

✓ Internet Gateway

✓ Elastic IP

✓ NAT Gateway

✓ Public Route Table

✓ Private Route Table

✓ Public EC2

✓ Private EC2
```
### Step 1 - Create versions.tf
Create the file
```bash
nano versions.tf
```
```bash
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
### Explanation

- `terraform {}` – Defines Terraform project settings.
- `required_version` – Ensures Terraform version **1.5.0 or later** is used.
- `required_providers` – Specifies the providers required by the project.
- `source = "hashicorp/aws"` – Downloads the official AWS provider from HashiCorp.
- `version = "~> 6.0"` – Uses AWS Provider **6.x** and prevents automatic upgrades to **7.x**.
- 
After versions.tf
```bash
terraform fmt
terraform init
terraform validate
```
- `terraform fmt` – Formats Terraform files.
- `terraform init` – Initializes the project and downloads the AWS provider.
- `terraform validate` – Checks the configuration for syntax and validation errors.

## Step 2 – Create provider.tf
Open it:
```bash
nano provider.tf
```
```bash
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

### Explanation

- `provider "aws"` – Configures the AWS provider for this project.
- `region = var.aws_region` – Uses the AWS region stored in the `aws_region` variable.
- `default_tags` – Automatically adds the specified tags to supported AWS resources.
- `Project`, `Environment`, `ManagedBy` – Common tags used to organize, identify, and manage AWS resources.

```
After provider.tf
```bash
terraform fmt
terraform validate
```
## Step 3 – Create variables.tf
Open:
```bash
nano variables.tf
```
```bash
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

variable "private_subnet_cidr" {

  description = "Private Subnet CIDR"

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
### Explanation

- `variable` – Declares an input variable used in the Terraform configuration.
- `description` – Explains the purpose of the variable.
- `type = string` – Specifies that the variable accepts text values.
- Using variables makes the configuration reusable, flexible, and easier to maintain without modifying the main code.

```
After variables.tf
```bash
terraform fmt
terraform validate
```

## Step 4 – Create terraform.tfvars

This file contains the actual values for the variables
```bash
nano terraform.tfvars
```
```bash
aws_region = "ap-south-1"

vpc_cidr = "10.0.0.0/16"

public_subnet_cidr = "10.0.1.0/24"

private_subnet_cidr = "10.0.2.0/24"

availability_zone = "ap-south-1a"

instance_type = "t2.micro"

key_name = "terraform-key"
```
### Explanation

- `terraform.tfvars` stores the actual values for the variables declared in `variables.tf`.
- Terraform automatically loads this file during execution.
- Separating variable values from the main configuration makes the code reusable and easier to manage across different environments.
  
After terraform.tfvars
```bash
terraform fmt
terraform validate
```
## Step 5 – Create vpc.tf
What is a VPC?

A VPC (Virtual Private Cloud) is your own private network inside AWS.

Open the file:
```bash
nano vpc.tf
```
```bash
resource "aws_vpc" "main" {

  cidr_block = var.vpc_cidr

  enable_dns_support = true

  enable_dns_hostnames = true

  tags = {

    Name = "lab13-vpc"

  }

}
```
### Explanation

- `resource "aws_vpc" "main"` – Creates a new VPC.
- `cidr_block = var.vpc_cidr` – Defines the IP address range for the VPC.
- `enable_dns_support = true` – Enables DNS resolution within the VPC.
- `enable_dns_hostnames = true` – Assigns DNS hostnames to EC2 instances launched in the VPC.
- `tags` – Adds a name to help identify the VPC.

### What is DNS resolution?

DNS (Domain Name System) translates a hostname into an IP address.

For example:
```bash
amazon.com
      ↓
54.239.28.85
```
Instead of remembering IP addresses, applications use hostnames, and DNS finds the correct IP.

What does enable_dns_support = true do?

When this setting is enabled, resources inside your VPC can use AWS's built-in DNS server to resolve domain names.

For example, an EC2 instance can resolve:
```bash
google.com
amazonaws.com
s3.ap-south-1.amazonaws.com
```
into their corresponding IP addresses.

### Enable DNS Hostnames
```bash
enable_dns_hostnames = true
```

This allows EC2 instances to receive DNS names like:
```bash
ip-10-0-1-25.ap-south-1.compute.internal
```
Without this setting:
```bash
No hostname assigned
```
### Validate the Configuration 
After vpc.tf
```bash
terraform fmt
terraform validate
terraform plan
```
## Step 6 - Create `subnet.tf`

### What is a Subnet?

A **subnet** is a smaller network inside a VPC that helps organize and isolate AWS resources.

Create the file:

```bash
nano subnet.tf
```

Add the following configuration:

```hcl
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidr
  availability_zone       = var.availability_zone
  map_public_ip_on_launch = true

  tags = {
    Name = "lab13-public-subnet"
  }
}

resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidr
  availability_zone = var.availability_zone

  tags = {
    Name = "lab13-private-subnet"
  }
}
```

### Explanation

- `aws_subnet.public` – Creates a public subnet inside the VPC.
- `aws_subnet.private` – Creates a private subnet inside the VPC.
- `vpc_id` – Associates the subnet with the VPC.
- `cidr_block` – Defines the IP address range for the subnet.
- `availability_zone` – Specifies the Availability Zone where the subnet is created.
- `map_public_ip_on_launch = true` – Automatically assigns a **public IP address** to every EC2 instance launched in the subnet, allowing        it to be accessed from the internet (if security rules allow).
- The private subnet does **not** use `map_public_ip_on_launch`, so EC2 instances receive **only a private IP address** and cannot be            accessed directly from the internet.

### Validate the Configuration

Run the following commands:

```bash
terraform fmt
terraform validate
terraform plan
```

- `terraform fmt` – Formats Terraform files.
- `terraform validate` – Checks the configuration for syntax and validation errors.
- `terraform plan` – Shows the changes Terraform will make before creating resources.

### Step 7 - Create `igw.tf`

### What is an Internet Gateway?

An **Internet Gateway (IGW)** allows resources in a **public subnet** to communicate with the internet.

Create the file:

```bash
nano igw.tf
```

Add the following configuration:

```hcl
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "lab13-igw"
  }
}
```

### Explanation

- `resource "aws_internet_gateway" "main"` – Creates an Internet Gateway.
- `vpc_id = aws_vpc.main.id` – Attaches the Internet Gateway to the VPC.
- `tags` – Adds a name to help identify the Internet Gateway in the AWS Console.

### Validate the Configuration

Run the following commands:

```bash
terraform fmt
terraform validate
terraform plan
```

- `terraform fmt` – Formats Terraform files.
- `terraform validate` – Checks the configuration for syntax and validation errors.
- `terraform plan` – Shows the changes Terraform will make before creating resources.
---

## Step 8 - Create `nat.tf`

### What is a NAT Gateway?

A **NAT Gateway** allows EC2 instances in a **private subnet** to access the internet without exposing them to incoming internet traffic.

Create the file:

```bash
nano nat.tf
```

Add the following configuration:

```hcl
resource "aws_eip" "nat" {
  domain = "vpc"

  tags = {
    Name = "lab13-eip"
  }
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public.id

  depends_on = [
    aws_internet_gateway.main
  ]

  tags = {
    Name = "lab13-nat"
  }
}
```

### Explanation

- `aws_eip.nat` – Creates an Elastic IP for the NAT Gateway.
- `allocation_id` – Attaches the Elastic IP to the NAT Gateway.
- `subnet_id` – Places the NAT Gateway in the **public subnet**.
- `depends_on` – Ensures the Internet Gateway is created before the NAT Gateway.
- `tags` – Adds a name to help identify the NAT Gateway.

### Validate the Configuration

Run the following commands:

```bash
terraform fmt
terraform validate
terraform plan
```

## Step 9 - Create `route-table.tf`

### What is a Route Table?

A **Route Table** controls how network traffic is routed within a VPC.

Create the file:

```bash
nano route-table.tf
```

Add the following configuration:

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

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }

  tags = {
    Name = "lab13-private-route-table"
  }
}

resource "aws_route_table_association" "private" {
  subnet_id      = aws_subnet.private.id
  route_table_id = aws_route_table.private.id
}
```

### Explanation

- `aws_route_table.public` – Creates a route table for the public subnet.
- `gateway_id = aws_internet_gateway.main.id` – Routes internet traffic through the Internet Gateway.
- `aws_route_table_association.public` – Associates the public subnet with the public route table.
- `aws_route_table.private` – Creates a route table for the private subnet.
- `nat_gateway_id = aws_nat_gateway.main.id` – Routes internet traffic through the NAT Gateway.
- `aws_route_table_association.private` – Associates the private subnet with the private route table.

### Validate the Configuration

Run the following commands:

```bash
terraform fmt
terraform validate
terraform plan
```
# Step 10 - Create data.tf

## Objective

In this step you will learn how to use a **Terraform Data Source**.

Instead of manually copying an AMI ID from the AWS Console, Terraform will automatically find the latest Amazon Linux 2023 AMI.

This makes your Terraform code portable across AWS regions and easier to maintain.

---

# What is a Data Source?

Terraform has two main concepts:

## Resources

Resources create infrastructure.

Examples:

- VPC
- EC2
- Subnet
- NAT Gateway

Example:

```hcl
resource "aws_vpc" "main" {
```

---

## Data Sources

Data Sources **read existing information** from AWS.

Examples:

- Latest Amazon Linux AMI
- Availability Zones
- Existing VPC
- Existing Security Group

Example:

```hcl
data "aws_ami" "amazon_linux" {
```

Data Sources do **not** create resources.

They only fetch information.

---

# Why Not Hardcode an AMI?

Bad example:

```hcl
ami = "ami-0123456789abcdef0"
```

Problems:

- Different in every AWS Region
- Changes over time
- May eventually be removed

Better approach:

Terraform searches AWS and automatically finds the latest AMI.

---

Open:

```bash
nano data.tf
```
---

# Paste the Following Code

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

# Explanation

## Data Block

```hcl
data "aws_ami" "amazon_linux"
```

Terraform searches AWS for an AMI.

Reference name:

```hcl
data.aws_ami.amazon_linux.id
```

---

## most_recent

```hcl
most_recent = true
```

If AWS has multiple matching AMIs:

```
Old

↓

New

↓

Newest
```

Terraform automatically selects the newest one.

---

## owners

```hcl
owners = ["amazon"]
```

Only search for AMIs published by AWS.

This prevents accidentally using an image published by another AWS account.

---

## Filter

```hcl
filter {

  name = "name"

  values = ["al2023-ami-*-x86_64"]

}
```

Terraform searches for images whose names match:

```
al2023-ami-*
```

Examples:

```
al2023-ami-2023.7.20250721-x86_64

al2023-ami-2023.6.20250618-x86_64
```

Terraform picks the latest matching image.

---

# How We Will Use It

Later, inside `ec2.tf`, we will write:

```hcl
ami = data.aws_ami.amazon_linux.id
```

Terraform will replace it with the latest AMI ID automatically.

Example:

```
ami-0abc123456789def0
```

You never need to update your Terraform code when AWS releases a new Amazon Linux 2023 AMI.

---

# Verify

Run:

```bash
cat data.tf
```

Expected:

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

# Current Project Structure

```
---

# Summary

In this step you learned:

- What a Data Source is
- Difference between Resources and Data Sources
- How to automatically find the latest Amazon Linux AMI
- Why using a Data Source is better than hardcoding an AMI ID

---

# Next Step

Next we will create:

## security-group.tf

You will learn:

- Security Groups
- Inbound Rules
- Outbound Rules
- SSH (Port 22)
- HTTP (Port 80)
- ICMP (Ping)
- Security Group best practices

# Step 11 - Create `security-group.tf`

## Objective

Create Security Groups for:

- Bastion Host (Public EC2)
- Private EC2

A Security Group acts as a virtual firewall that controls inbound and outbound traffic.

---

## What We Will Create

| Security Group | Purpose |
|---------------|---------|
| `bastion-sg` | Allow SSH from your computer |
| `private-ec2-sg` | Allow SSH only from the Bastion Host |

---

Open the file:

```bash
nano security-group.tf
```

---

## Bastion Host Security Group

Paste the following:

```hcl
resource "aws_security_group" "bastion" {

  name        = "lab13-bastion-sg"

  description = "Security Group for Bastion Host"

  vpc_id      = aws_vpc.main.id

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

    Name = "lab13-bastion-sg"

  }

}
```

---
After security-group.tf
```bash
terraform fmt
terraform validate
terraform plan
```
## Explanation

### Name

```hcl
name = "lab13-bastion-sg"
```

Security Group name shown in AWS.

---

### VPC

```hcl
vpc_id = aws_vpc.main.id
```

Creates the Security Group inside our VPC.

---

### Ingress Rule

```hcl
from_port = 22

to_port = 22

protocol = "tcp"
```

Allows SSH connections.

---

### CIDR Block

```hcl
cidr_blocks = ["0.0.0.0/0"]
```

Allows SSH from anywhere.

> **Note:** This is acceptable for a learning lab. In production, restrict this to your public IP address (for example, `203.0.113.10/32`) instead of allowing the whole internet.

---

### Egress

```hcl
protocol = "-1"
```

Allows all outbound traffic.

---

## Private EC2 Security Group

Paste below the previous resource.

```hcl
resource "aws_security_group" "private" {

  name        = "lab13-private-sg"

  description = "Private EC2 Security Group"

  vpc_id      = aws_vpc.main.id

  ingress {

    description = "SSH from Bastion Host"

    from_port = 22

    to_port = 22

    protocol = "tcp"

    security_groups = [

      aws_security_group.bastion.id

    ]

  }

  egress {

    from_port = 0

    to_port = 0

    protocol = "-1"

    cidr_blocks = ["0.0.0.0/0"]

  }

  tags = {

    Name = "lab13-private-sg"

  }

}
```

---

## Explanation

### Why Use `security_groups`?

Instead of allowing SSH from an IP address, we allow SSH from another Security Group.

```hcl
security_groups = [

  aws_security_group.bastion.id

]
```

This means:

- The Bastion Host can SSH to the Private EC2.
- Your laptop cannot SSH directly to the Private EC2.
- The Private EC2 has no direct SSH exposure to the internet.

This is a common production pattern.

---

## Verify

Run:

```bash
cat security-group.tf
```

---

## Summary

You created two Security Groups:

- **Bastion Security Group**
  - Allows SSH from the internet (for this lab).
  - Allows all outbound traffic.

- **Private EC2 Security Group**
  - Allows SSH only from the Bastion Host.
  - Allows all outbound traffic.

These Security Groups prepare the network for launching the Bastion Host and the Private EC2 instance.

---

## Next Step

Next we will create **`user-data.sh`**, which automatically installs software when the EC2 instances start.

# Step 12 - Create `user-data.sh`

## Objective

Create a User Data script that automatically updates the EC2 instance and installs the Apache web server when it launches.

---

## What is User Data?

User Data is a shell script that EC2 runs automatically during the first boot.

Common use cases:

- Install packages
- Update the operating system
- Install web servers
- Configure applications
- Start services

---

Open the file:

```bash
nano user-data.sh
```

---

## Add the Following Code

```bash
#!/bin/bash

# Update all installed packages
dnf update -y

# Install Apache Web Server
dnf install -y httpd

# Start Apache
systemctl start httpd

# Enable Apache at boot
systemctl enable httpd

# Create a simple web page
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Terraform Lab 13</title>
</head>
<body>
    <h1>Terraform NAT Gateway Lab</h1>
    <p>Apache installed successfully using User Data.</p>
</body>
</html>
EOF
```

---

## Explanation

### Shebang

```bash
#!/bin/bash
```

Tells Linux to execute this file using the Bash shell.

---
After user-data.sh
```bash
terraform fmt
terraform validate
terraform plan
```
### Update Packages

```bash
dnf update -y
```

Updates all installed packages to the latest available versions.

The `-y` option automatically answers "yes" to prompts.

---

### Install Apache

```bash
dnf install -y httpd
```

Installs the Apache HTTP Server.

---

### Start Apache

```bash
systemctl start httpd
```

Starts the Apache service immediately.

---

### Enable Apache

```bash
systemctl enable httpd
```

Ensures Apache starts automatically after every reboot.

---

### Create Web Page

```bash
cat <<EOF > /var/www/html/index.html
...
EOF
```

Creates a simple HTML page that will be served by Apache.

---

## Make the Script Executable (Optional)

Run:

```bash
chmod +x user-data.sh
```

This isn't required by Terraform when using the `file()` function, but it's a good habit if you also want to run the script manually.

---

## Verify

Display the file:

```bash
cat user-data.sh
```

---

## Summary

In this step you created a User Data script that:

- Updates the operating system
- Installs Apache
- Starts the Apache service
- Enables Apache on boot
- Creates a default web page

In the next step, we'll attach this script to the EC2 instance so it runs automatically during instance creation.

# Step 13 - Create `ec2.tf`

## Objective

Create two EC2 instances:

- Bastion Host (Public Subnet)
- Private EC2 (Private Subnet)

The Bastion Host will be used to SSH into the Private EC2.

---

Open:

```bash
nano ec2.tf
```

# Bastion Host

Paste the following code.

```hcl
resource "aws_instance" "bastion" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = var.instance_type

  subnet_id = aws_subnet.public.id

  vpc_security_group_ids = [

    aws_security_group.bastion.id

  ]

  key_name = var.key_name

  user_data = file("${path.module}/user-data.sh")

  tags = {

    Name = "lab13-bastion-host"

  }

}
```
After ec2.tf
```bash
terraform fmt
terraform validate
terraform plan
```


---

## Explanation

### AMI

```hcl
ami = data.aws_ami.amazon_linux.id
```

Uses the latest Amazon Linux 2023 AMI from `data.tf`.

---

### Instance Type

```hcl
instance_type = var.instance_type
```

Reads the value from:

```hcl
terraform.tfvars
```

Example:

```hcl
instance_type = "t2.micro"
```

---

### Subnet

```hcl
subnet_id = aws_subnet.public.id
```

Launches the Bastion Host inside the Public Subnet.

---

### Security Group

```hcl
vpc_security_group_ids = [

  aws_security_group.bastion.id

]
```

Attaches the Bastion Security Group.

---

### Key Pair

```hcl
key_name = var.key_name
```

Uses the AWS Key Pair that already exists in your account.

Example:

```hcl
key_name = "terraform-key"
```

---

### User Data

```hcl
user_data = file("${path.module}/user-data.sh")
```

Reads the contents of `user-data.sh` and runs it during the first boot of the EC2 instance.

---

### Tags

```hcl
tags = {

  Name = "lab13-bastion-host"

}
```

This name appears in the AWS Console.

---

# Private EC2

Paste the following below the Bastion Host resource.

```hcl
resource "aws_instance" "private" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = var.instance_type

  subnet_id = aws_subnet.private.id

  vpc_security_group_ids = [

    aws_security_group.private.id

  ]

  key_name = var.key_name

  tags = {

    Name = "lab13-private-server"

  }

}
```

---

## Explanation

This EC2 instance is similar to the Bastion Host, but with two important differences.

### Private Subnet

```hcl
subnet_id = aws_subnet.private.id
```

This launches the instance in the Private Subnet.

It will not receive a public IP address because:

- The subnet does not assign public IPs.
- It has no direct route to the Internet Gateway.

---

### Security Group

```hcl
aws_security_group.private.id
```

Only the Bastion Host is allowed to SSH into this instance.

Your laptop cannot connect directly.

---

## Verify

Run:

```bash
cat ec2.tf
```

---

## Summary

This file creates:

- 1 Bastion Host in the Public Subnet
- 1 Private EC2 in the Private Subnet
- Uses the latest Amazon Linux 2023 AMI
- Uses the existing AWS Key Pair
- Attaches the correct Security Groups
- Runs the User Data script on the Bastion Host

# Step 14 - Create `outputs.tf`

## Objective

Terraform Outputs display useful information after infrastructure is created.

Instead of searching the AWS Console, Terraform prints important values directly in the terminal.

---
Open the file:

```bash
nano outputs.tf
```

---

# Add the Following Code

```hcl
output "vpc_id" {

  description = "VPC ID"

  value = aws_vpc.main.id

}

output "public_subnet_id" {

  description = "Public Subnet ID"

  value = aws_subnet.public.id

}

output "private_subnet_id" {

  description = "Private Subnet ID"

  value = aws_subnet.private.id

}

output "internet_gateway_id" {

  description = "Internet Gateway ID"

  value = aws_internet_gateway.main.id

}

output "nat_gateway_id" {

  description = "NAT Gateway ID"

  value = aws_nat_gateway.main.id

}

output "bastion_public_ip" {

  description = "Public IP of Bastion Host"

  value = aws_instance.bastion.public_ip

}

output "bastion_public_dns" {

  description = "Public DNS of Bastion Host"

  value = aws_instance.bastion.public_dns

}

output "private_instance_private_ip" {

  description = "Private IP of Private EC2"

  value = aws_instance.private.private_ip

}
```
After outputs.tf
```bash
terraform fmt
terraform validate
terraform plan
terraform apply
```
---

# Explanation

## Output Block

```hcl
output "vpc_id" {

  value = aws_vpc.main.id

}
```

After `terraform apply`, Terraform prints the VPC ID.

Example:

```
vpc_id = "vpc-0123456789abcdef0"
```

---

## Public Subnet ID

```hcl
output "public_subnet_id"
```

Displays the ID of the Public Subnet.

Example:

```
subnet-0a12bc34de56fg789
```

---

## Private Subnet ID

```hcl
output "private_subnet_id"
```

Displays the ID of the Private Subnet.

---

## Internet Gateway ID

```hcl
output "internet_gateway_id"
```

Displays the Internet Gateway ID.

---

## NAT Gateway ID

```hcl
output "nat_gateway_id"
```

Displays the NAT Gateway ID.

---

## Bastion Public IP

```hcl
output "bastion_public_ip"
```

Example:

```
43.205.120.15
```

You'll use this IP to SSH into the Bastion Host.

---

## Bastion Public DNS

```hcl
output "bastion_public_dns"
```

Example:

```
ec2-43-205-120-15.ap-south-1.compute.amazonaws.com
```

You can SSH using either the IP address or the DNS name.

---

## Private EC2 Private IP

```hcl
output "private_instance_private_ip"
```

Example:

```
10.0.2.145
```

This IP is only reachable from within the VPC, such as from the Bastion Host.

---

# Verify

Run:

```bash
cat outputs.tf
```

---

# Summary

This file prints important information after deployment:

- VPC ID
- Public Subnet ID
- Private Subnet ID
- Internet Gateway ID
- NAT Gateway ID
- Bastion Public IP
- Bastion Public DNS
- Private EC2 Private IP

These outputs help you verify the infrastructure and connect to your instances without searching through the AWS Console.


# Lab Cleanup

## Step 1 - Destroy AWS Resourcesnano provider.tf

Run:

```bash
terraform destroy
```

Type:

```
yes
```

Verify:

```
Destroy complete!
```

---

## Step 2 - Verify AWS Console

Confirm these resources have been deleted:

```
✓ EC2

✓ Security Groups

✓ NAT Gateway

✓ Elastic IP

✓ Route Tables

✓ Internet Gateway

✓ Subnets

✓ VPC
```

---

## Step 3 - Remove Terraform Local Files

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

You should only see:

```
README.md

versions.tf

provider.tf

variables.tf

terraform.tfvars

main.tf

outputs.tf

...
```

---

## Step 5 - Git Status

Run:

```bash
git status
```

---

## Step 6 - Commit

```bash
git add .

git commit -m "Complete Lab 13 NAT Gateway"
```

---

## Step 7 - Push

```bash
git push origin main
```

---

# Lab Completed

You learned:

✅ NAT Gateway

✅ Elastic IP

✅ Private EC2

✅ Public EC2

✅ Public Route Table

✅ Private Route Table

✅ Production Networking
