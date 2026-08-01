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
# Step 12 - Create `launch-template.tf`

## Objective

Create a Launch Template that defines the configuration for EC2 instances launched by the Auto Scaling Group.

Instead of defining the EC2 configuration inside the ASG, AWS uses a Launch Template as a reusable blueprint.

---

## What is a Launch Template?

A **Launch Template** is a blueprint for launching EC2 instances.

It contains:

- Amazon Machine Image (AMI)
- Instance Type
- Key Pair
- Security Group
- User Data
- Tags

The Auto Scaling Group uses this template whenever it launches a new EC2 instance.

---

## Open the File

```bash
nano launch-template.tf
```

---

## Add the Following Code

```hcl
resource "aws_launch_template" "web" {

  name_prefix = "lab14-launch-template-"

  image_id = data.aws_ami.amazon_linux.id

  instance_type = var.instance_type

  key_name = var.key_name

  vpc_security_group_ids = [

    aws_security_group.web.id

  ]

  user_data = base64encode(file("${path.module}/user-data.sh"))

  tag_specifications {

    resource_type = "instance"

    tags = {

      Name = "lab14-web-server"

    }

  }

}
```

---

## Explanation

### Launch Template

```hcl
resource "aws_launch_template" "web"
```

Creates a Launch Template.

Terraform reference:

```hcl
aws_launch_template.web.id
```

---

### AMI

```hcl
image_id = data.aws_ami.amazon_linux.id
```

Uses the latest Amazon Linux 2023 AMI.

---

### Instance Type

```hcl
instance_type = var.instance_type
```

Uses the EC2 instance type from `terraform.tfvars`.

Example:

```hcl
instance_type = "t2.micro"
```

---

### Key Pair

```hcl
key_name = var.key_name
```

Uses your existing AWS EC2 Key Pair for SSH access.

---

### Security Group

```hcl
vpc_security_group_ids = [

  aws_security_group.web.id

]
```

Attaches the Web Server Security Group to every EC2 instance launched by the Auto Scaling Group.

---

### User Data

```hcl
user_data = base64encode(file("${path.module}/user-data.sh"))
```

Reads the `user-data.sh` file, converts it to Base64, and runs it automatically when each EC2 instance starts.

> **Note:** The Launch Template requires the User Data to be Base64 encoded.

---

### Tags

```hcl
tag_specifications {

  resource_type = "instance"

  tags = {

    Name = "lab14-web-server"

  }

}
```

Adds the **Name** tag to every EC2 instance created by the Auto Scaling Group.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```
---
# Step 13 - Create `alb.tf`

## Objective

Create an Application Load Balancer (ALB) that distributes incoming HTTP traffic across the EC2 instances launched by the Auto Scaling Group.

---

## What is an Application Load Balancer?

An **Application Load Balancer (ALB)** receives requests from users and forwards them to healthy EC2 instances.

Benefits:

- High Availability
- Load Balancing
- Health Checks
- Fault Tolerance

---

## Network Diagram

```text
                Internet
                    |
                    |
       Application Load Balancer
                    |
              Target Group
                    |
        Auto Scaling Group (ASG)
```

---

## Open the File

```bash
nano alb.tf
```

---

## Step 13.1 - Create Target Group

Add the following configuration:

```hcl
resource "aws_lb_target_group" "web" {

  name = "lab14-target-group"

  port = 80

  protocol = "HTTP"

  vpc_id = aws_vpc.main.id

  health_check {

    path = "/"

    protocol = "HTTP"

    matcher = "200"

  }

  tags = {

    Name = "lab14-target-group"

  }

}
```

---

## Explanation

- `aws_lb_target_group.web` – Creates a Target Group.
- `port = 80` – Sends traffic to port 80 on the EC2 instances.
- `protocol = "HTTP"` – Uses the HTTP protocol.
- `health_check` – Checks whether EC2 instances are healthy before routing traffic.

---

## Step 13.2 - Create the Application Load Balancer

Add the following configuration below the Target Group:

```hcl
resource "aws_lb" "main" {

  name = "lab14-alb"

  internal = false

  load_balancer_type = "application"

  security_groups = [

    aws_security_group.alb.id

  ]

  subnets = [

    aws_subnet.public_1.id,

    aws_subnet.public_2.id

  ]

  tags = {

    Name = "lab14-alb"

  }

}
```

---

## Explanation

- `internal = false` – Creates an internet-facing ALB.
- `load_balancer_type = "application"` – Creates an Application Load Balancer.
- `security_groups` – Attaches the ALB Security Group.
- `subnets` – Places the ALB in two public subnets for high availability.

---

## Step 13.3 - Create Listener

Add the following configuration below:

```hcl
resource "aws_lb_listener" "http" {

  load_balancer_arn = aws_lb.main.arn

  port = 80

  protocol = "HTTP"

  default_action {

    type = "forward"

    target_group_arn = aws_lb_target_group.web.arn

  }

}
```

---

## Explanation

- `aws_lb_listener.http` – Creates an HTTP listener.
- `port = 80` – Listens for incoming HTTP requests.
- `default_action` – Forwards requests to the Target Group.

---

## Traffic Flow

```text
User

↓

Application Load Balancer

↓

Target Group

↓

Auto Scaling Group

↓

Healthy EC2 Instance
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
# Step 14 - Create `auto-scaling-group.tf`

## Objective

Create an Auto Scaling Group (ASG) that automatically launches, replaces, and manages EC2 instances using the Launch Template.

The ASG will register its instances with the Application Load Balancer Target Group.

---

## What is an Auto Scaling Group?

An **Auto Scaling Group (ASG)** automatically manages a group of EC2 instances.

It can:

- Launch EC2 instances
- Replace unhealthy instances
- Maintain the desired number of instances
- Scale in or out based on demand

---

## Network Diagram

```text
                Internet
                    |
                    |
      Application Load Balancer
                    |
              Target Group
                    |
          Auto Scaling Group
              /          \
             /            \
      EC2 Instance    EC2 Instance
```

---

## Open the File

```bash
nano auto-scaling-group.tf
```

---

## Add the Following Code

```hcl
resource "aws_autoscaling_group" "web" {

  name = "lab14-asg"

  desired_capacity = var.desired_capacity

  min_size = var.min_size

  max_size = var.max_size

  vpc_zone_identifier = [

    aws_subnet.public_1.id,

    aws_subnet.public_2.id

  ]

  target_group_arns = [

    aws_lb_target_group.web.arn

  ]

  health_check_type = "ELB"

  health_check_grace_period = 300

  launch_template {

    id = aws_launch_template.web.id

    version = "$Latest"

  }

  tag {

    key = "Name"

    value = "lab14-web-server"

    propagate_at_launch = true

  }

}
```

---

## Explanation

### Auto Scaling Group

```hcl
resource "aws_autoscaling_group" "web"
```

Creates an Auto Scaling Group.

Terraform reference:

```hcl
aws_autoscaling_group.web.id
```

---

### Desired Capacity

```hcl
desired_capacity = var.desired_capacity
```

Starts the number of EC2 instances defined in `terraform.tfvars`.

Example:

```text
desired_capacity = 2
```

---

### Minimum Size

```hcl
min_size = var.min_size
```

Ensures at least the minimum number of EC2 instances are always running.

---

### Maximum Size

```hcl
max_size = var.max_size
```

Limits the maximum number of EC2 instances the ASG can launch.

---

### Public Subnets

```hcl
vpc_zone_identifier = [

  aws_subnet.public_1.id,

  aws_subnet.public_2.id

]
```

Launches EC2 instances across both public subnets for high availability.

---

### Target Group

```hcl
target_group_arns = [

  aws_lb_target_group.web.arn

]
```

Automatically registers every EC2 instance with the ALB Target Group.

---

### Health Check

```hcl
health_check_type = "ELB"
```

Uses the Application Load Balancer health checks.

If an EC2 instance becomes unhealthy, the ASG automatically terminates it and launches a replacement.

---

### Grace Period

```hcl
health_check_grace_period = 300
```

Waits **300 seconds (5 minutes)** before checking the health of a newly launched EC2 instance.

This gives the instance enough time to boot and run the User Data script.

---

### Launch Template

```hcl
launch_template {

  id = aws_launch_template.web.id

  version = "$Latest"

}
```

Uses the latest version of the Launch Template whenever the ASG launches a new EC2 instance.

---

### Tag

```hcl
tag {

  key = "Name"

  value = "lab14-web-server"

  propagate_at_launch = true

}
```

Automatically adds the **Name** tag to every EC2 instance created by the ASG.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
terraform plan
```
------------
# Step 15 - Create `outputs.tf`

## Objective

Display useful information after the infrastructure is created.

Terraform Outputs allow you to quickly access important resource details without opening the AWS Console.

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

  value = aws_vpc.main.id

}

output "public_subnet_1_id" {

  description = "Public Subnet 1 ID"

  value = aws_subnet.public_1.id

}

output "public_subnet_2_id" {

  description = "Public Subnet 2 ID"

  value = aws_subnet.public_2.id

}

output "alb_dns_name" {

  description = "Application Load Balancer DNS Name"

  value = aws_lb.main.dns_name

}

output "target_group_arn" {

  description = "Target Group ARN"

  value = aws_lb_target_group.web.arn

}

output "launch_template_id" {

  description = "Launch Template ID"

  value = aws_launch_template.web.id

}

output "autoscaling_group_name" {

  description = "Auto Scaling Group Name"

  value = aws_autoscaling_group.web.name

}
```

---

## Explanation

- `vpc_id` – Displays the VPC ID.
- `public_subnet_1_id` – Displays the first public subnet ID.
- `public_subnet_2_id` – Displays the second public subnet ID.
- `alb_dns_name` – Displays the DNS name of the Application Load Balancer.
- `target_group_arn` – Displays the Target Group ARN.
- `launch_template_id` – Displays the Launch Template ID.
- `autoscaling_group_name` – Displays the Auto Scaling Group name.

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

## Verify the Deployment

Display the outputs:

```bash
terraform output
```

Example:

```text
alb_dns_name = "lab14-alb-123456789.ap-south-1.elb.amazonaws.com"

autoscaling_group_name = "lab14-asg"
```

---

## Test the Application

Open the ALB DNS name in your browser:

```text
http://<alb_dns_name>
```

Refresh the page several times.

You should see different **Instance IDs**, confirming that the Application Load Balancer is distributing traffic across the EC2 instances managed by the Auto Scaling Group.

---

## Test Auto Scaling Group Recovery

1. Open the **EC2 Console**.
2. Terminate one of the EC2 instances created by the Auto Scaling Group.
3. Wait a few minutes.

The Auto Scaling Group automatically launches a new EC2 instance to maintain the desired capacity.

Refresh the ALB page again.

You should see the new **Instance ID** after the replacement instance becomes healthy.

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
              /          \
             /            \
      EC2 Instance    EC2 Instance
          (Auto)          (Auto)
```

---

## Summary

You created:

- VPC
- Two Public Subnets
- Internet Gateway
- Route Table
- Security Groups
- Launch Template
- Auto Scaling Group
- Application Load Balancer
- Target Group
- Listener
- Outputs

You also learned:

- Launch Templates
- Auto Scaling Groups (ASG)
- Desired, Minimum, and Maximum Capacity
- Automatic EC2 Replacement
- High Availability
- Fault Tolerance
- Application Load Balancer Integration

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

Confirm the following resources have been deleted:

- Auto Scaling Group
- Launch Template
- Application Load Balancer
- Target Group
- Listener
- EC2 Instances
- Security Groups
- Route Table
- Internet Gateway
- Subnets
- VPC

> **Note:** Wait until the Auto Scaling Group finishes deleting before the Launch Template and Load Balancer are removed.

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

> **Note:** These commands remove only local Terraform state and cache files. Your `.tf` configuration files remain unchanged.

---

## Step 4 - Verify Cleanup

Run:

```bash
ls -la
```

Verify your project files are still present:

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

git commit -m "Complete Lab 14 Auto Scaling Group"
```

---

## Step 7 - Push to GitHub

```bash
git push origin main
```

---

# Lab Completed

You learned:

- Launch Templates
- Auto Scaling Groups (ASG)
- Desired, Minimum, and Maximum Capacity
- Automatic EC2 Replacement
- Application Load Balancer Integration
- Health Checks
- High Availability
- Fault Tolerance
- Infrastructure Cleanup