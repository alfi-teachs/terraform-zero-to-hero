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


---

# Step 8 - Create the GitHub Repository

Open GitHub.

Click:

New Repository

Repository Name:

terraform-zero-to-hero

Description:

Hands-on Terraform Labs from Beginner to Advanced using AWS.

Repository Visibility:

Public

Check:

✔ Add a README file

✔ Add .gitignore

Choose:

Terraform

Click:

Create Repository

---

# Step 9 - Clone the Repository

Open Git Bash.

Go to your Desktop.

Example:

```bash
cd ~/OneDrive/Desktop/terraform
```

Clone your repository.

Replace YOUR_GITHUB_USERNAME with your GitHub username.

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/terraform-zero-to-hero.git
```

Example:

```bash
git clone https://github.com/alskill/terraform-zero-to-hero.git
```

Move into the repository.

```bash
cd terraform-zero-to-hero
```

Verify.

```bash
pwd
```

Example Output

```text
/c/Users/YourName/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

---

# Step 10 - Check Git Status

Run:

```bash
git status
```

Expected Output:

```text
On branch main

Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

This means your repository is ready.

---

# Step 11 - Create the Project Folder Structure

Run the following command.

```bash
mkdir labs terraform images docs
```

Verify.

```bash
ls
```

Expected Output

```text
README.md
docs
images
labs
terraform
```

---

# Step 12 - Create the Lab 01 Folder

Run:

```bash
mkdir terraform/lab-01
mkdir images/lab-01
```

Verify.

```bash
ls terraform
```

Expected Output

```text
lab-01
```

---

# Step 13 - Create the Lab 01 Markdown File

Run:

```bash
touch labs/lab-01-install-terraform.md
```

Verify.

```bash
ls labs
```

Expected Output

```text
lab-01-install-terraform.md
```

---

# Step 14 - Create Terraform Files

Move into Lab 01.

```bash
cd terraform/lab-01
```

Create the required files.

```bash
touch provider.tf
touch main.tf
touch variables.tf
touch outputs.tf
touch terraform.tfvars
touch README.md
```

Verify.

```bash
ls
```

Expected Output

```text
README.md
main.tf
outputs.tf
provider.tf
terraform.tfvars
variables.tf
```

Return to the project root.

```bash
cd ../..
```

---

# Step 15 - Create Documentation Files

Run:

```bash
touch docs/terraform-cheat-sheet.md
touch docs/commands.md
touch docs/interview-questions.md
touch docs/troubleshooting.md
```

Verify.

```bash
ls docs
```

Expected Output

```text
commands.md
interview-questions.md
terraform-cheat-sheet.md
troubleshooting.md
```

---

# Step 16 - Verify the Complete Project Structure

Run:

```bash
find .
```

Expected Output

```text
.
./README.md
./docs
./docs/commands.md
./docs/interview-questions.md
./docs/terraform-cheat-sheet.md
./docs/troubleshooting.md
./images
./images/lab-01
./labs
./labs/lab-01-install-terraform.md
./terraform
./terraform/lab-01
./terraform/lab-01/main.tf
./terraform/lab-01/provider.tf
./terraform/lab-01/variables.tf
./terraform/lab-01/outputs.tf
./terraform/lab-01/terraform.tfvars
./terraform/lab-01/README.md
```

---

# Understanding the Folder Structure

## README.md

The main page of your GitHub repository.

It explains:

- What the repository is about
- Learning roadmap
- Prerequisites
- Repository structure

---

## labs/

Contains all lab guides.

Example:

```text
labs/
│
├── lab-01-install-terraform.md
├── lab-02-providers.md
├── lab-03-variables.md
```

Each Markdown file explains how to complete the lab.

---

## terraform/

Contains the Terraform code.

Example:

```text
terraform/
│
├── lab-01/
│
├── lab-02/
│
└── lab-03/
```

Each folder contains:

```text
provider.tf
main.tf
variables.tf
outputs.tf
terraform.tfvars
README.md
```

---

## images/

Contains screenshots.

Example:

```text
images/
│
├── lab-01/
│   ├── terraform-version.png
│   ├── git-status.png
│   └── folder-structure.png
```

---

## docs/

Contains reusable documentation.

Example:

```text
terraform-cheat-sheet.md

commands.md

interview-questions.md

troubleshooting.md
```

These files are shared across all labs.

---

# Checkpoint

At this point you should have:

✅ Terraform Installed

✅ Git Repository

✅ Folder Structure

✅ Lab Folder

✅ Terraform Files

✅ Documentation Folder

You are now ready to start writing Terraform code in the next part.

---

# Part 3 - Your First Terraform Configuration

Congratulations!

You have successfully installed Terraform and created your project structure.

Now it's time to write your first Terraform configuration.

Don't worry if you've never written Terraform before.

We'll understand every single line.

---

# What is a Terraform Configuration?

A Terraform configuration is simply one or more files ending with:

.tf

Example:

main.tf

provider.tf

variables.tf

outputs.tf

Terraform reads all *.tf files in the current directory.

---

# Terraform File Purpose

Our Lab 01 folder contains:

terraform/
└── lab-01/
    ├── provider.tf
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    ├── terraform.tfvars
    └── README.md

Let's understand each file.

---

## provider.tf

This file tells Terraform which cloud provider to use.

Examples:

AWS

Azure

Google Cloud

VMware

Kubernetes

Without a provider, Terraform does not know where to create resources.

In later labs we will configure AWS here.

For now, leave it empty.

---

## main.tf

This is the most important file.

It contains the infrastructure you want Terraform to create.

Examples:

EC2

VPC

S3 Bucket

Security Group

IAM User

For Lab 01 we are only learning the workflow, so leave it empty.

---

## variables.tf

Stores input variables.

Example:

Instead of hardcoding:

Instance Type = t2.micro

you can create a variable:

instance_type

and change it whenever needed.

We will learn variables in Lab 03.

Leave this file empty.

---

## terraform.tfvars

Stores values for variables.

Example:

instance_type = "t2.micro"

Terraform automatically loads this file.

Leave it empty.

---

## outputs.tf

Displays information after Terraform creates resources.

Example:

Public IP

Instance ID

VPC ID

Subnet ID

Load Balancer DNS

Leave it empty.

---

## README.md

This README belongs only to Lab 01.

It explains:

Purpose

Files

Commands

Expected Output

Notes

---

# Step 17 - Open the Project in Visual Studio Code

Open Git Bash.

Navigate to your project.

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Open VS Code.

```bash
code .
```

If the command does not work:

Open VS Code

Click

File

Open Folder

Choose:

terraform-zero-to-hero

---

# Step 18 - Open Lab 01

In VS Code open:

terraform

↓

lab-01

You should see:

provider.tf

main.tf

variables.tf

outputs.tf

terraform.tfvars

README.md

---

# Step 19 - Save Empty Files

Even though they are empty,

press

CTRL + S

for each file.

This ensures VS Code recognizes them.

---

# Step 20 - Initialize Terraform

Open Terminal inside VS Code.

Or Git Bash.

Navigate to:

```bash
cd terraform/lab-01
```

Verify.

```bash
pwd
```

Expected Output

```text
.../terraform-zero-to-hero/terraform/lab-01
```

Now initialize Terraform.

```bash
terraform init
```

Expected Output

```text
Terraform initialized in an empty directory!

The directory has no Terraform configuration files.
You may begin working with Terraform immediately...
```

This is expected because we haven't written any resources yet.

The important thing is that Terraform runs successfully.

---

# What Does terraform init Do?

This is always the first command you run.

It prepares your working directory.

terraform init performs several tasks:

• Checks Terraform configuration

• Downloads required providers

• Creates Terraform working directory

• Creates lock file

Think of it like installing dependencies before running a project.

You only need to run it:

• First time

• After changing providers

• After deleting the .terraform folder

---

# Step 21 - Check New Files

Run:

```bash
ls -la
```

Expected Output

```text
.terraform/

.terraform.lock.hcl

main.tf

outputs.tf

provider.tf

terraform.tfvars

variables.tf

README.md
```

Notice two new items.

Terraform created them automatically.

---

# What is the .terraform Folder?

This folder stores:

Downloaded providers

Plugin binaries

Internal metadata

Never edit it manually.

---

# What is .terraform.lock.hcl?

This file locks provider versions.

Example:

AWS Provider 6.2.0

Every developer on the team uses the same provider version.

Always commit this file to Git.

---

# Step 22 - Check Terraform Version Again

Run:

```bash
terraform version
```

Example Output

```text
Terraform v1.x.x
on windows_amd64
```

---

# Step 23 - Validate Configuration

Run:

```bash
terraform validate
```

Expected Output

```text
Success! The configuration is valid.
```

Since there are no resources yet, Terraform simply confirms the directory is valid.

---

# Summary

Today you learned:

✔ What Terraform configuration is

✔ Purpose of each Terraform file

✔ How to open the project in VS Code

✔ How to initialize Terraform

✔ What the .terraform folder is

✔ What the lock file is

✔ Why terraform init is the first command

✔ How to validate a configuration

You have successfully completed the first hands-on interaction with Terraform.
