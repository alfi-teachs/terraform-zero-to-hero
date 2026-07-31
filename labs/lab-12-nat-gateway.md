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
## Step 10 - Create `data.tf`

### What is a Data Source?

A **Data Source** reads existing information from AWS instead of creating new resources.

Create the file:

```bash
nano data.tf
```

Add the following configuration:

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}
```

### Explanation

- `data "aws_ami" "amazon_linux"` – Finds an existing Amazon Linux AMI.
- `most_recent = true` – Selects the latest matching AMI.
- `owners = ["amazon"]` – Searches only AWS-published AMIs.
- `filter` – Finds Amazon Linux 2023 x86_64 AMIs that match the specified name pattern.
- `data.aws_ami.amazon_linux.id` – Returns the latest AMI ID, which can be used when creating an EC2 instance.

### Validate the Configuration

Run the following commands:

```bash
terraform fmt
terraform validate
terraform plan
```

## Step 11 - Create `security-group.tf`

### What is a Security Group?

A **Security Group** is a virtual firewall that controls inbound and outbound traffic for AWS resources.

Create the file:

```bash
nano security-group.tf
```

Add the following configuration:

```hcl
resource "aws_security_group" "bastion" {
  name        = "lab13-bastion-sg"
  description = "Security Group for Bastion Host"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "SSH from Anywhere"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "lab13-bastion-sg"
  }
}

resource "aws_security_group" "private" {
  name        = "lab13-private-sg"
  description = "Private EC2 Security Group"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "SSH from Bastion Host"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"

    security_groups = [
      aws_security_group.bastion.id
    ]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "lab13-private-sg"
  }
}
```

### Explanation

- `aws_security_group.bastion` – Creates a Security Group for the Bastion Host.
- `ingress` – Allows SSH (port 22) from anywhere (`0.0.0.0/0`) for this lab.
- `egress` – Allows all outbound traffic.
- `aws_security_group.private` – Creates a Security Group for the Private EC2 instance.
- `security_groups = [aws_security_group.bastion.id]` – Allows SSH only from the Bastion Host Security Group.

> **Note:** In production, replace `0.0.0.0/0` with your public IP address to improve security.

### Validate the Configuration

Run the following commands:

```bash
terraform fmt
terraform validate
terraform plan
```
## Step 12 - Create `user-data.sh`

### What is User Data?

**User Data** is a shell script that runs automatically the first time an EC2 instance starts. It is commonly used to install software and configure the instance.

Create the file:

```bash
nano user-data.sh
```

Add the following script:

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

### Explanation

- `#!/bin/bash` – Runs the script using the Bash shell.
- `dnf update -y` – Updates installed packages.
- `dnf install -y httpd` – Installs the Apache web server.
- `systemctl start httpd` – Starts the Apache service.
- `systemctl enable httpd` – Starts Apache automatically after reboot.
- `cat <<EOF ... EOF` – Creates the default `index.html` web page.

### (Optional) Make the Script Executable

```bash
chmod +x user-data.sh
```

### Validate the Configuration

Run the following commands:

```bash
terraform fmt
terraform validate
terraform plan
```
## Step 13 - Create `ec2.tf`

### What is an EC2 Instance?

An **EC2 instance** is a virtual server in AWS used to run applications.

Create the file:

```bash
nano ec2.tf
```

Add the following configuration:

```hcl
resource "aws_instance" "bastion" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.bastion.id]
  key_name               = var.key_name
  user_data              = file("${path.module}/user-data.sh")

  tags = {
    Name = "lab13-bastion-host"
  }
}

resource "aws_instance" "private" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.private.id
  vpc_security_group_ids = [aws_security_group.private.id]
  key_name               = var.key_name

  tags = {
    Name = "lab13-private-server"
  }
}
```

### Explanation

- `aws_instance.bastion` – Creates a Bastion Host in the public subnet.
- `aws_instance.private` – Creates a Private EC2 instance in the private subnet.
- `ami = data.aws_ami.amazon_linux.id` – Uses the latest Amazon Linux 2023 AMI.
- `instance_type` – Uses the EC2 instance type from `terraform.tfvars`.
- `subnet_id` – Launches the instance in the specified subnet.
- `vpc_security_group_ids` – Attaches the appropriate Security Group.
- `key_name` – Uses an existing AWS Key Pair for SSH access.
- `user_data = file("${path.module}/user-data.sh")` – Runs the User Data script when the Bastion Host starts.
- `tags` – Adds a name to identify the EC2 instance in the AWS Console.

### Validate the Configuration

Run the following commands:

```bash
terraform fmt
terraform validate
terraform plan
```
## Step 14 - Create `outputs.tf`

### What are Terraform Outputs?

**Outputs** display useful information about the infrastructure after `terraform apply` completes.

Create the file:

```bash
nano outputs.tf
```

Add the following configuration:

```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_id" {
  description = "Public Subnet ID"
  value       = aws_subnet.public.id
}

output "private_subnet_id" {
  description = "Private Subnet ID"
  value       = aws_subnet.private.id
}

output "internet_gateway_id" {
  description = "Internet Gateway ID"
  value       = aws_internet_gateway.main.id
}

output "nat_gateway_id" {
  description = "NAT Gateway ID"
  value       = aws_nat_gateway.main.id
}

output "bastion_public_ip" {
  description = "Public IP of Bastion Host"
  value       = aws_instance.bastion.public_ip
}

output "bastion_public_dns" {
  description = "Public DNS of Bastion Host"
  value       = aws_instance.bastion.public_dns
}

output "private_instance_private_ip" {
  description = "Private IP of Private EC2"
  value       = aws_instance.private.private_ip
}
```

### Explanation

- `output` – Displays resource information after `terraform apply`.
- `value` – Specifies the resource attribute to display.
- `vpc_id` – Displays the VPC ID.
- `public_subnet_id` – Displays the Public Subnet ID.
- `private_subnet_id` – Displays the Private Subnet ID.
- `internet_gateway_id` – Displays the Internet Gateway ID.
- `nat_gateway_id` – Displays the NAT Gateway ID.
- `bastion_public_ip` – Displays the Bastion Host's public IP address.
- `bastion_public_dns` – Displays the Bastion Host's public DNS name.
- `private_instance_private_ip` – Displays the Private EC2 instance's private IP address.

### Apply the Configuration

Run the following commands:

```bash
terraform fmt
terraform validate
terraform plan
terraform apply
```

- `terraform fmt` – Formats Terraform files.
- `terraform validate` – Checks the configuration for syntax and validation errors.
- `terraform plan` – Shows the changes Terraform will make before creating resources.
- `terraform apply` – Creates the infrastructure and displays the output values.

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

Confirm the following resources have been deleted:

- EC2 Instances
- Security Groups
- NAT Gateway
- Elastic IP
- Route Tables
- Internet Gateway
- Subnets
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

> **Note:** These commands remove local Terraform state and cache files only. Your `.tf` configuration files remain unchanged.

---

## Step 4 - Verify Cleanup

Run:

```bash
ls -la
```

Verify that your Terraform configuration files (such as `versions.tf`, `provider.tf`, `variables.tf`, `terraform.tfvars`, `outputs.tf`, etc.) are still present.

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
git commit -m "Complete Lab 13 NAT Gateway"
```

---

## Step 7 - Push to GitHub

```bash
git push origin main
```

### Lab Summary

In this lab, you learned how to:

- Create a VPC and Subnets
- Configure an Internet Gateway and NAT Gateway
- Create Public and Private Route Tables
- Launch Public and Private EC2 Instances
- Configure Security Groups
- Use User Data to automate EC2 setup
- Use Terraform Outputs to display resource information
- Clean up AWS resources using `terraform destroy`
