# Lab 01 - Install Terraform on Windows

## Lab Information

| Item | Details |
|------|---------|
| Lab Number | 01 |
| Lab Name | Install Terraform |
| Difficulty | Beginner |
| Duration | 15-20 Minutes |
| Prerequisites | Windows 10/11, Internet Connection |

---

# Objective

In this lab you will learn:

- What Terraform is
- Why Terraform is used
- What Infrastructure as Code (IaC) means
- How to install Terraform
- How to verify the installation
- Your first Terraform commands

By the end of this lab you will have Terraform installed and ready for all future labs.

---

# What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool developed by HashiCorp.

Instead of creating AWS resources manually using the AWS Console, Terraform allows you to define your infrastructure in code.

For example, instead of clicking multiple buttons to create:

- EC2 Instance
- VPC
- Security Group
- S3 Bucket

You simply write a few lines of Terraform code and Terraform creates everything automatically.

---

# What is Infrastructure as Code (IaC)?

Infrastructure as Code means managing infrastructure using code instead of manually clicking through a web console.

Traditional Method:

Administrator

↓

Login to AWS

↓

Click EC2

↓

Click Launch Instance

↓

Configure Instance

↓

Launch

Terraform Method:

Write Code

↓

terraform apply

↓

Infrastructure Created Automatically

Benefits:

- Faster deployments
- Repeatable infrastructure
- Version controlled
- Less human error
- Easy automation
- Easy collaboration

---

# Why Learn Terraform?

Terraform is one of the most popular DevOps tools.

Companies use Terraform to automate cloud infrastructure.

Examples include:

- AWS
- Microsoft Azure
- Google Cloud Platform (GCP)
- Oracle Cloud
- VMware
- Kubernetes

Terraform allows you to manage infrastructure consistently across multiple cloud providers.

---

# Prerequisites

Before starting this lab, ensure you have:

- Windows 10 or Windows 11
- Administrator access
- Internet connection
- Git Bash installed
- Visual Studio Code installed

---

# Lab Architecture

+---------------------------+
|      Windows Laptop       |
|---------------------------|
| VS Code                   |
| Git Bash                  |
| Terraform CLI             |
+------------+--------------+
             |
             |
      terraform version
             |
             v
     Terraform Installed

---

# Step 1 - Create the Project Folder

Open Git Bash.

Run the following commands:

```bash
mkdir -p ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
pwd
```

Expected Output:

```text
/c/Users/<username>/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

If your Desktop is not inside OneDrive, use:

```bash
mkdir -p ~/Desktop/terraform/terraform-zero-to-hero
cd ~/Desktop/terraform/terraform-zero-to-hero
```

---

# Step 2 - Check Whether Terraform is Already Installed

Run:

```bash
terraform version
```

Possible Output 1:

```text
Terraform v1.x.x
```

If you see this, Terraform is already installed.

You can skip to Step 6.

Possible Output 2:

```text
terraform: command not found
```

or

```text
'terraform' is not recognized...
```

If you see either of these messages, continue with the installation.

---

# Step 3 - Download Terraform

Open your browser.

Visit:

https://developer.hashicorp.com/terraform/downloads

Download the Windows AMD64 version.

After downloading you will have a ZIP file similar to:

terraform_1.xx.x_windows_amd64.zip

---

# Step 4 - Extract Terraform

Extract the ZIP file.

Inside it you will find:

terraform.exe

Create a folder:

C:\Terraform

Move terraform.exe into:

C:\Terraform

The final structure should be:

C:\Terraform
│
└── terraform.exe

---

# Step 5 - Add Terraform to Windows PATH

Press the Windows key.

Search for:

Environment Variables

Open:

Edit the system environment variables

Click:

Environment Variables

Under User Variables select:

Path

Click:

New

Add:

C:\Terraform

Click OK on every window.

Close Git Bash.

Open Git Bash again.

Terraform is now available from any folder.

---

# Step 6 - Verify Installation

Run:

```bash
terraform version
```

Expected Output:

```text
Terraform v1.x.x
on windows_amd64
```

Your version number may be different.

That is perfectly fine.

---

# Step 7 - View Terraform Help

Run:

```bash
terraform
```

Expected Output:

```text
Usage: terraform [global options] <subcommand>

Main commands:

init

plan

apply

destroy

validate

fmt

version
```

Congratulations!

Terraform is successfully installed.
