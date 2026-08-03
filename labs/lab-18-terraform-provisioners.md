# Lab 18 - Terraform Provisioners

## Objective

In this lab, you will learn how to use **Terraform Provisioners** to execute commands during or after resource creation.

You will use:

- `local-exec`
- `remote-exec`
- `file`

These provisioners help automate tasks such as copying files, installing software, and running commands on EC2 instances.

> **Note:** Provisioners should be used only when necessary. Terraform recommends using cloud-init or User Data whenever possible.

---

## What You Will Build

- EC2 Instance
- SSH Connection
- Local Command Execution
- Remote Command Execution
- File Transfer
- Apache Installation

---

## What are Terraform Provisioners?

Provisioners allow Terraform to perform additional actions after a resource has been created or before it is destroyed.

Examples:

- Execute a local shell command
- Run commands on a remote EC2 instance
- Copy files to a remote server

---

## Types of Provisioners

| Provisioner | Purpose |
|-------------|---------|
| `local-exec` | Runs commands on your local machine |
| `remote-exec` | Runs commands on the remote EC2 instance |
| `file` | Copies files from your local machine to the EC2 instance |

---

## Architecture

```text
Local Machine
      │
      │
Terraform
      │
      ▼
EC2 Instance
      │
      ├── local-exec
      ├── file
      └── remote-exec
```

---

## Step 1 - Create the Lab File

```bash
touch labs/lab-18-terraform-provisioners.md
```

---

## Step 2 - Create the Project Directory

```bash
mkdir -p terraform/lab-18-provisioners

cd terraform/lab-18-provisioners
```

---

## Step 3 - Create Terraform Files

```bash
touch versions.tf
touch provider.tf
touch variables.tf
touch terraform.tfvars
touch main.tf
touch outputs.tf
```

---

## Step 4 - Create Supporting Files

```bash
touch install.sh

touch index.html
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
├── index.html
├── install.sh
├── main.tf
├── outputs.tf
├── provider.tf
├── terraform.tfvars
├── variables.tf
└── versions.tf
```

---

## What You Will Learn

- Terraform Provisioners
- local-exec
- remote-exec
- file Provisioner
- SSH Connection
- Remote Automation

---
# Step 1 - Create `versions.tf`

## Objective

Configure the Terraform version and AWS provider for this project.

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
- `required_providers` – Specifies the AWS provider required for the project.
- `source = "hashicorp/aws"` – Uses the official AWS provider.
- `version = "~> 6.0"` – Uses AWS Provider version **6.x**.

---

## Initialize the Project

Run:

```bash
terraform fmt
terraform init
terraform validate
```
---
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

      Environment = "Lab-18"

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
- `Project`, `Environment`, and `ManagedBy` help identify and organize AWS resources.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
-----------
# Step 3 - Create `variables.tf`

## Objective

Define the input variables used to create and connect to the EC2 instance.

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

variable "instance_type" {

  description = "EC2 Instance Type"

  type = string

}

variable "key_name" {

  description = "AWS EC2 Key Pair Name"

  type = string

}

variable "private_key_path" {

  description = "Path to the Private Key (.pem)"

  type = string

}
```

---

## Explanation

- `aws_region` – AWS Region where the EC2 instance will be created.
- `instance_type` – EC2 instance type.
- `key_name` – Existing AWS EC2 Key Pair name.
- `private_key_path` – Path to the private key (`.pem`) used by the `remote-exec` and `file` provisioners to connect to the EC2 instance over SSH.

Example:

```text
C:/Users/alfia/Downloads/terraform-key.pem
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
----------------
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

instance_type = "t2.micro"

key_name = "terraform-key"

private_key_path = "C:/Users/alfia/Downloads/terraform-key.pem"
```

> Replace the `private_key_path` with the location of your own `.pem` file.

---

## Explanation

- `aws_region` – AWS Region where the EC2 instance will be created.
- `instance_type` – EC2 instance type.
- `key_name` – Existing AWS EC2 Key Pair used to connect to the EC2 instance.
- `private_key_path` – Local path to the private key used by Terraform provisioners for SSH access.

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
---------------
# Step 5 - Create `install.sh`

## Objective

Create a shell script that installs and starts the Apache web server on the EC2 instance.

This script will be copied to the EC2 instance using the **file** provisioner and executed using the **remote-exec** provisioner.

---

## Open the File

```bash
nano install.sh
```

---

## Add the Following Code

```bash
#!/bin/bash

# Update installed packages
sudo dnf update -y

# Install Apache
sudo dnf install -y httpd

# Start Apache
sudo systemctl start httpd

# Enable Apache on boot
sudo systemctl enable httpd
```

---

## Explanation

### Shebang

```bash
#!/bin/bash
```

Runs the script using the Bash shell.

---

### Update Packages

```bash
sudo dnf update -y
```

Updates all installed packages.

---

### Install Apache

```bash
sudo dnf install -y httpd
```

Installs the Apache web server.

---

### Start Apache

```bash
sudo systemctl start httpd
```

Starts the Apache service immediately.

---

### Enable Apache

```bash
sudo systemctl enable httpd
```

Configures Apache to start automatically after every reboot.

---

## Make the Script Executable (Optional)

Run:

```bash
chmod +x install.sh
```

---

## Verify the File

Run:

```bash
cat install.sh
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```
--------------
# Step 6 - Create `index.html`

## Objective

Create a simple web page that will be copied to the EC2 instance using the **file** provisioner.

After Apache is installed, this page will be available from the EC2 instance.

---

## Open the File

```bash
nano index.html
```

---

## Add the Following Code

```html
<!DOCTYPE html>
<html>
<head>
    <title>Terraform Provisioners Lab</title>
</head>
<body>
    <h1>Terraform Provisioners</h1>
    <p>Apache installed successfully using Terraform Provisioners.</p>
</body>
</html>
```

---

## Explanation

### HTML Document

```html
<!DOCTYPE html>
```

Defines the document as an HTML5 web page.

---

### Title

```html
<title>Terraform Provisioners Lab</title>
```

Sets the page title shown in the browser tab.

---

### Heading

```html
<h1>Terraform Provisioners</h1>
```

Displays the main heading on the web page.

---

### Paragraph

```html
<p>Apache installed successfully using Terraform Provisioners.</p>
```

Displays a confirmation message after Apache is installed.

---

## Verify the File

Run:

```bash
cat index.html
```

---

## Validate the Configuration

Run:

```bash
terraform fmt
terraform validate
```

---
# Step 7 - Create `main.tf`

## Objective

Launch an EC2 instance and use Terraform provisioners to:

- Run a command on the local machine
- Copy files to the EC2 instance
- Run commands on the EC2 instance

---

## Open the File

```bash
nano main.tf
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

resource "aws_security_group" "web" {

  name = "lab18-web-sg"

  description = "Allow SSH and HTTP"

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

}

resource "aws_instance" "web" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = var.instance_type

  key_name = var.key_name

  vpc_security_group_ids = [

    aws_security_group.web.id

  ]

  tags = {

    Name = "lab18-web-server"

  }

  connection {

    type = "ssh"

    user = "ec2-user"

    private_key = file(var.private_key_path)

    host = self.public_ip

  }

  provisioner "local-exec" {

    command = "echo EC2 Created: ${self.public_ip}"

  }

  provisioner "file" {

    source = "install.sh"

    destination = "/home/ec2-user/install.sh"

  }

  provisioner "file" {

    source = "index.html"

    destination = "/home/ec2-user/index.html"

  }

  provisioner "remote-exec" {

    inline = [

      "chmod +x /home/ec2-user/install.sh",

      "/home/ec2-user/install.sh",

      "sudo cp /home/ec2-user/index.html /var/www/html/index.html"

    ]

  }

}
```

---

## Explanation

### local-exec

```hcl
provisioner "local-exec"
```

Runs a command on your local machine after the EC2 instance is created.

Example output:

```text
EC2 Created: 13.233.120.45
```

---

### Connection

```hcl
connection
```

Creates an SSH connection to the EC2 instance using:

- SSH
- EC2 user
- Private key
- Public IP

---

### File Provisioner

```hcl
provisioner "file"
```

Copies files from your local machine to the EC2 instance.

Files copied:

- `install.sh`
- `index.html`

---

### Remote-exec

```hcl
provisioner "remote-exec"
```

Runs commands on the EC2 instance.

Commands executed:

- Makes the script executable
- Installs Apache
- Copies the web page to Apache's document root

---

## Validate the Configuration

Run:

```bash
terraform fmt

terraform validate

terraform plan
```
---
# Step 8 - Create `outputs.tf`

## Objective

Display useful information after the EC2 instance is created.

Terraform outputs make it easy to retrieve important resource details without opening the AWS Console.

---

## Open the File

```bash
nano outputs.tf
```

---

## Add the Following Code

```hcl
output "instance_id" {

  description = "EC2 Instance ID"

  value = aws_instance.web.id

}

output "public_ip" {

  description = "EC2 Public IP"

  value = aws_instance.web.public_ip

}

output "public_dns" {

  description = "EC2 Public DNS"

  value = aws_instance.web.public_dns

}
```

---

## Explanation

### Instance ID

```hcl
output "instance_id"
```

Displays the EC2 Instance ID.

Example:

```text
i-0123456789abcdef0
```

---

### Public IP

```hcl
output "public_ip"
```

Displays the public IP address of the EC2 instance.

Example:

```text
13.233.120.45
```

You can use this IP to:

- Access the Apache web page
- Connect using SSH

---

### Public DNS

```hcl
output "public_dns"
```

Displays the public DNS name of the EC2 instance.

Example:

```text
ec2-13-233-120-45.ap-south-1.compute.amazonaws.com
```

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
instance_id = "i-0123456789abcdef0"

public_ip = "13.233.120.45"

public_dns = "ec2-13-233-120-45.ap-south-1.compute.amazonaws.com"
```

---

## Test the Web Server

Open your browser and visit:

```text
http://<public-ip>
```

Example:

```text
http://13.233.120.45
```

You should see:

```text
Terraform Provisioners

Apache installed successfully using Terraform Provisioners.
```

---

## Summary

You learned how to use:

- `local-exec` Provisioner
- `file` Provisioner
- `remote-exec` Provisioner
- SSH Connection Block
- Terraform Outputs

You also automated:

- Copying files to EC2
- Installing Apache
- Deploying a web page

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
index.html
install.sh
main.tf
outputs.tf
provider.tf
terraform.tfvars
variables.tf
versions.tf
```

---

## Step 5 - Check Git Status

Run:

```bash
git status
```

---

## Step 6 - Commit Changes

```bash
git add .

git commit -m "Complete Lab 18 Terraform Provisioners"
```

---

## Step 7 - Push to GitHub

```bash
git push origin main
```

---

# Lab Completed

You learned:

- `local-exec` Provisioner
- `remote-exec` Provisioner
- `file` Provisioner
- SSH Connection Block
- File Transfer
- Remote Command Execution
- Provisioner Best Practices

---

# Congratulations!

You have successfully completed **Lab 18 – Terraform Provisioners**.

You can now automate post-deployment tasks such as copying files, running remote commands, and configuring EC2 instances using Terraform provisioners.

--------------