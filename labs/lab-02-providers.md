# Lab 02 – Configure AWS Provider

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 02 |
| Lab Name | Configure AWS Provider |
| Difficulty | Beginner |
| Duration | 30–45 Minutes |
| Cloud | AWS |
| Services | IAM, AWS CLI, Terraform |

---

# Learning Objectives

By the end of this lab you will be able to:

✅ Explain what a Terraform Provider is

✅ Configure AWS CLI

✅ Authenticate Terraform with AWS

✅ Understand provider plugins

✅ Configure the AWS Provider

✅ Download the provider plugin

✅ Verify Terraform can communicate with AWS

---

# Real World Scenario

Imagine you join a company as a DevOps Engineer.

Your manager says:

> "Create an EC2 instance using Terraform."

Can Terraform directly create an EC2?

No.

Terraform first needs to know **where** it should create resources.

Should it create resources in:

- AWS?
- Azure?
- Google Cloud?
- VMware?

That's why Terraform uses **Providers**.

Providers act like translators between Terraform and cloud platforms.

---

# What is a Provider?

A Provider is a plugin that allows Terraform to communicate with another platform.

Think of Terraform as a universal remote control.

The Provider tells Terraform how to speak the language of AWS.

```
Terraform
     │
     │
AWS Provider Plugin
     │
     │
AWS APIs
     │
     ▼
AWS Cloud
```

Without a Provider,

Terraform cannot create anything.

---

# What is the AWS Provider?

The AWS Provider is maintained by HashiCorp.

It understands every AWS service including:

- EC2
- VPC
- IAM
- S3
- Route53
- CloudFront
- RDS
- Lambda
- EKS

Whenever you write Terraform code,

Terraform sends requests through the AWS Provider to AWS.

---

# Lab Architecture

```
                    +--------------------+
                    | Terraform          |
                    +---------+----------+
                              |
                              |
                       AWS Provider
                              |
                              |
                    AWS CLI Credentials
                              |
                              |
                    AWS Access Key
                    AWS Secret Key
                              |
                              ▼
                     AWS Cloud APIs
```

---

# Prerequisites

You must have:

- AWS Account
- Terraform Installed
- Git Installed
- VS Code
- AWS CLI Installed
- Internet Connection

---

# Step 1 - Verify Terraform

Open Git Bash.

Run:

```bash
terraform version
```

Expected Output

```
Terraform v1.x.x
```

---

# Step 2 - Verify AWS CLI

Run

```bash
aws --version
```

Expected Output

```
aws-cli/2.x.x
```

If AWS CLI is not installed,

Download it from:

https://aws.amazon.com/cli/

Install it.

Verify again.

---

# Step 3 - Verify AWS Login

Run

```bash
aws sts get-caller-identity
```

Expected Output

```json
{
  "Account":"123456789012",
  "Arn":"arn:aws:iam::123456789012:user/terraform-admin",
  "UserId":"AIDAxxxxxxxx"
}
```

If this command works,

Terraform will also be able to authenticate.

If it fails,

Configure AWS CLI.

---

# Step 4 - Configure AWS CLI

Run

```bash
aws configure
```

Example

```
AWS Access Key ID:
AKIAxxxxxxxx

AWS Secret Access Key:
********************************

Default region:
ap-south-1

Default output:
json
```

Verify again.

```bash
aws sts get-caller-identity
```

If successful,

move to the next step.

---

# Step 5 - Create Lab Folder

Navigate to the project.

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create the folders.

```bash
mkdir -p terraform/lab-02
mkdir -p images/lab-02
touch labs/lab-02-terraform-provider.md
```

Move into Lab 02.

```bash
cd terraform/lab-02
```

---

# Step 6 - Create Terraform Files

Run

```bash
touch versions.tf
touch provider.tf
touch variables.tf
touch terraform.tfvars
touch outputs.tf
touch main.tf
touch README.md
```

Verify.

```bash
ls
```

Expected Output

```
README.md
main.tf
outputs.tf
provider.tf
terraform.tfvars
variables.tf
versions.tf
```

---

# Step 7 - Understanding the Files

versions.tf

Stores:

- Terraform Version
- Provider Version

provider.tf

Stores:

AWS Authentication

variables.tf

Stores:

Input Variables

terraform.tfvars

Stores:

Variable Values

main.tf

Stores:

AWS Resources

outputs.tf

Displays Resource Information

README.md

Explains the Lab

---

# End of Part 1

---

# Part 2 - Writing Your First Terraform Configuration

Congratulations!

In Part 1, you learned:

- What a Provider is
- Why Providers are required
- How Terraform communicates with AWS
- How to configure AWS CLI
- How to create the Lab 02 folder structure

Now it's time to write your first real Terraform code.

---

# What is a Terraform Block?

The `terraform` block tells Terraform:

- Which Terraform version is required
- Which provider is required
- Which version of the provider should be used

This ensures everyone working on the project uses the same versions.

---

# Step 8 - Open the Project in VS Code

Go to your project directory.

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Open VS Code.

```bash
code .
```

Open:

```
terraform/
└── lab-02/
```

---

# Step 9 - Configure versions.tf

Open:

```
terraform/lab-02/versions.tf
```

Replace the contents with:

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

### Explanation

`required_version`

Specifies the minimum Terraform version.

```
required_version = ">= 1.5.0"
```

This means:

Use Terraform version **1.5.0 or higher**.

---

`required_providers`

Tells Terraform which provider plugin is needed.

```
aws
```

is simply the provider name.

---

`source`

```
hashicorp/aws
```

Means:

Download the AWS Provider maintained by HashiCorp.

---

`version`

```
~> 6.0
```

Means:

Use version 6.x.

Terraform will automatically download the latest compatible version.

---

# Step 10 - Configure variables.tf

Open:

```
terraform/lab-02/variables.tf
```

Paste:

```hcl
variable "aws_region" {

  description = "AWS Region"

  type = string

  default = "ap-south-1"

}
```

### Explanation

This creates a variable named:

```
aws_region
```

Instead of hardcoding:

```
ap-south-1
```

everywhere,

we use:

```
var.aws_region
```

This makes the code reusable.

---

# Step 11 - Configure terraform.tfvars

Open:

```
terraform/lab-02/terraform.tfvars
```

Paste:

```hcl
aws_region = "ap-south-1"
```

If you want another region later,

you only change this file.

---

# Step 12 - Configure provider.tf

Open:

```
terraform/lab-02/provider.tf
```

Paste:

```hcl
provider "aws" {

  region = var.aws_region

}
```

### Explanation

The provider block tells Terraform:

Connect to AWS

Use the region stored in:

```
var.aws_region
```

Terraform will automatically read the AWS credentials that you configured using:

```bash
aws configure
```

---

# Step 13 - Leave main.tf Empty

Open:

```
main.tf
```

Leave it empty.

Why?

Because this lab focuses only on configuring the provider.

We will create our first AWS resource in **Lab 06**.

---

# Step 14 - Leave outputs.tf Empty

Open:

```
outputs.tf
```

Leave it empty.

We don't have any resources yet.

---

# Step 15 - Verify Folder Structure

Your folder should now look like this:

```
terraform/
└── lab-02/
    ├── versions.tf
    ├── provider.tf
    ├── variables.tf
    ├── terraform.tfvars
    ├── outputs.tf
    ├── main.tf
    └── README.md
```

---

# Step 16 - Format the Code

Run:

```bash
terraform fmt
```

Expected Output

```
provider.tf
variables.tf
versions.tf
```

Terraform automatically formats the files.

---

# Step 17 - Validate the Configuration

Run:

```bash
terraform validate
```

If you haven't initialized the provider yet, you may see an error similar to:

```
Missing required provider
```

This is expected.

In Part 3, you'll run:

```bash
terraform init
```

which downloads the AWS provider plugin.

---

# Summary

In this part you learned:

- The purpose of the `terraform` block
- How to specify Terraform versions
- How to specify AWS provider versions
- How to create input variables
- How to configure the AWS provider
- Why `main.tf` is still empty
- Why `terraform.tfvars` is useful

---

# End of Part 2

In Part 3, you'll:

- Run `terraform init`
- Download the AWS provider plugin
- Understand the `.terraform` directory
- Verify the provider installation
- Test communication with AWS
- Learn what happens behind the scenes during initialization

---

# Part 3 - Initialize Terraform and Download the AWS Provider

Congratulations!

You have written your first Terraform configuration.

Now it's time to initialize the project.

This is one of the most important steps in every Terraform project.

---

# What Does terraform init Do?

The `terraform init` command prepares your working directory.

Think of it like installing dependencies before running an application.

When you run:

```bash
terraform init
```

Terraform performs several tasks automatically:

- Reads all `.tf` files
- Checks the Terraform version
- Checks required providers
- Downloads provider plugins
- Creates the `.terraform` directory
- Creates the `.terraform.lock.hcl` file
- Prepares the working directory

Every new Terraform project begins with:

```bash
terraform init
```

---

# Step 18 - Move to the Lab Directory

Open Git Bash.

Navigate to the Lab 02 directory.

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero/terraform/lab-02
```

Verify your location.

```bash
pwd
```

Example Output

```text
/c/Users/YourName/OneDrive/Desktop/terraform/terraform-zero-to-hero/terraform/lab-02
```

---

# Step 19 - Check the Files Before Initialization

Run:

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
versions.tf
```

Notice that there is **no** `.terraform` directory yet.

Terraform has not downloaded anything.

---

# Step 20 - Initialize Terraform

Run:

```bash
terraform init
```

Terraform begins reading:

- versions.tf
- provider.tf

It notices:

```
Provider:
hashicorp/aws
```

Terraform connects to the Terraform Registry.

It downloads the AWS Provider.

---

# Expected Output

You should see output similar to:

```text
Initializing the backend...

Initializing provider plugins...

- Finding hashicorp/aws versions matching "~> 6.0"...

- Installing hashicorp/aws v6.x.x...

- Installed hashicorp/aws v6.x.x

Terraform has been successfully initialized!
```

The exact version number may differ.

That is normal.

---

# What Just Happened?

Terraform downloaded the AWS Provider plugin.

Think of the provider as a translator.

```
Terraform
      │
      ▼
AWS Provider Plugin
      │
      ▼
AWS APIs
      │
      ▼
AWS Cloud
```

Without this plugin,

Terraform cannot communicate with AWS.

---

# Step 21 - Check the Directory Again

Run:

```bash
ls -la
```

Expected Output

```text
.terraform
.terraform.lock.hcl
README.md
main.tf
outputs.tf
provider.tf
terraform.tfvars
variables.tf
versions.tf
```

Notice two new items.

---

# The .terraform Directory

Terraform created:

```
.terraform/
```

This directory contains:

- Provider plugins
- Metadata
- Internal Terraform files

Do not edit this folder manually.

---

# The .terraform.lock.hcl File

Terraform also created:

```
.terraform.lock.hcl
```

This file records the exact provider versions used.

Example:

```
hashicorp/aws

Version:

6.18.0
```

This ensures everyone on the team uses the same provider version.

Always commit this file to Git.

---

# Step 22 - View Installed Providers

Run:

```bash
terraform providers
```

Expected Output

```text
Providers required by configuration:

.

└── provider[registry.terraform.io/hashicorp/aws]
```

This confirms Terraform recognizes the AWS Provider.

---

# Step 23 - Validate the Configuration

Run:

```bash
terraform validate
```

Expected Output

```text
Success! The configuration is valid.
```

Terraform has checked:

- Syntax
- Provider configuration
- Variables

Everything is ready.

---

# Step 24 - Format the Configuration

Run:

```bash
terraform fmt
```

Terraform automatically formats your code.

Keeping consistent formatting is considered a best practice.

---

# Step 25 - View Provider Information

Run:

```bash
terraform version
```

Example Output

```text
Terraform v1.x.x

+ provider registry.terraform.io/hashicorp/aws v6.x.x
```

Notice that Terraform now displays the installed AWS Provider.

---

# Understanding the Initialization Process

The initialization process looks like this:

```
You run:

terraform init

        │

        ▼

Terraform reads versions.tf

        │

        ▼

Terraform finds:

hashicorp/aws

        │

        ▼

Downloads Provider

        │

        ▼

Creates:

.terraform/

        │

        ▼

Creates:

.terraform.lock.hcl

        │

        ▼

Initialization Complete
```

---

# Common Mistakes

## Running terraform init from the Wrong Directory

Incorrect:

```bash
cd terraform
terraform init
```

Correct:

```bash
cd terraform/lab-02
terraform init
```

Always run Terraform commands inside the folder containing your `.tf` files.

---

## No Internet Connection

Terraform downloads providers from the Terraform Registry.

Without internet access, initialization will fail.

---

## Unsupported Terraform Version

If Terraform reports that your version does not satisfy:

```
required_version
```

upgrade Terraform before continuing.

---

# Lab Verification Checklist

Verify the following:

- ✅ AWS CLI is configured
- ✅ versions.tf created
- ✅ provider.tf created
- ✅ variables.tf created
- ✅ terraform.tfvars created
- ✅ terraform init completed
- ✅ AWS Provider downloaded
- ✅ .terraform directory created
- ✅ .terraform.lock.hcl created
- ✅ terraform validate successful
- ✅ terraform fmt completed

---

# Summary

You now understand:

- Why `terraform init` is required
- What happens during initialization
- How Terraform downloads provider plugins
- Purpose of the `.terraform` directory
- Purpose of `.terraform.lock.hcl`
- How to validate and format Terraform configurations

---

# End of Part 3

In Part 4, you will:

- Understand AWS authentication in detail
- Learn how Terraform uses AWS CLI credentials
- Verify communication with AWS
- Learn best practices
- Troubleshoot common authentication errors
- Complete Lab 02 with interview questions, assignments, and Git commands

---

# Part 4 - AWS Authentication, Best Practices and Lab Completion

Congratulations!

You have successfully configured your first Terraform Provider.

Although Terraform has not yet created any AWS resources, your environment is now fully prepared.

From the next labs onward, Terraform will communicate directly with AWS.

---

# How Does Terraform Authenticate with AWS?

One of the most common interview questions is:

**How does Terraform authenticate with AWS?**

Terraform does not store your AWS username and password.

Instead, Terraform uses AWS credentials.

The most common authentication method is the AWS CLI.

The authentication flow looks like this:

```
                +------------------+
                | Terraform        |
                +--------+---------+
                         |
                         |
                  AWS Provider
                         |
                         |
                Reads Credentials
                         |
                         |
         ~/.aws/credentials
                         |
                         |
                    AWS APIs
                         |
                         |
                      AWS Cloud
```

Terraform automatically reads credentials from your AWS CLI configuration.

---

# Where Does AWS CLI Store Credentials?

On Windows, AWS CLI stores credentials in:

```
C:\Users\<YourUserName>\.aws\
```

Inside this folder you will find:

```
credentials

config
```

Example:

credentials

```
[default]

aws_access_key_id=AKIAXXXXXXXXXXXXX

aws_secret_access_key=XXXXXXXXXXXXXXXXXXXXXXXX
```

config

```
[default]

region=ap-south-1

output=json
```

Terraform reads these files automatically.

You do not need to write your AWS keys inside Terraform code.

---

# Never Do This

Avoid writing credentials directly in `provider.tf`:

```hcl
provider "aws" {

  access_key = "AKIAxxxxxxxx"

  secret_key = "xxxxxxxxxxxx"

  region = "ap-south-1"

}
```

Why?

Because:

- Credentials may be exposed.
- They could be committed to GitHub.
- Anyone with the repository could use your AWS account.

Always use the AWS CLI or IAM roles.

---

# Verify AWS Authentication

Run:

```bash
aws sts get-caller-identity
```

Example Output:

```json
{
  "UserId": "AIDAEXAMPLE123456",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/terraform-admin"
}
```

If this command succeeds, Terraform can authenticate with AWS.

---

# Verify Terraform Can Use the Provider

Run:

```bash
terraform providers
```

Expected Output:

```
Providers required by configuration:

.

└── provider[registry.terraform.io/hashicorp/aws]
```

This confirms that the AWS Provider is installed and recognized.

---

# Review of Important Terraform Files

## versions.tf

Purpose:

- Specifies the required Terraform version.
- Specifies required provider versions.

---

## provider.tf

Purpose:

- Configures the AWS Provider.
- Sets the AWS Region.
- Connects Terraform to AWS.

---

## variables.tf

Purpose:

- Defines input variables.

Example:

```
aws_region
```

---

## terraform.tfvars

Purpose:

Stores actual values for variables.

Example:

```hcl
aws_region = "ap-south-1"
```

---

## main.tf

Purpose:

Contains AWS resources.

Examples:

- EC2
- VPC
- S3
- IAM

Currently empty.

We will begin creating AWS resources in later labs.

---

## outputs.tf

Purpose:

Displays information after deployment.

Examples:

- Instance ID
- Public IP
- VPC ID
- Bucket Name

Currently empty.

---

# Best Practices

Always:

- Use variables instead of hardcoded values.
- Run `terraform fmt` before committing code.
- Run `terraform validate` before `terraform plan`.
- Keep Terraform code in Git.
- Commit `.terraform.lock.hcl`.
- Use IAM users or IAM roles instead of the AWS root account.

Never:

- Commit AWS Access Keys.
- Commit Secret Access Keys.
- Store credentials in Terraform files.
- Modify the `.terraform` directory manually.

---

# Common Errors

## Error

```
No valid credential sources found
```

Solution:

Run:

```bash
aws configure
```

Verify:

```bash
aws sts get-caller-identity
```

---

## Error

```
InvalidClientTokenId
```

Cause:

The AWS Access Key or Secret Access Key is incorrect.

Solution:

Configure the AWS CLI again.

---

## Error

```
ExpiredToken
```

Cause:

Temporary credentials have expired.

Solution:

Refresh the credentials or sign in again.

---

## Error

```
Error acquiring the state lock
```

Cause:

Another Terraform process is using the state.

Solution:

Wait until the other operation completes or unlock the state if appropriate.

---

# Lab Verification Checklist

Verify that:

- ✅ AWS CLI is installed.
- ✅ AWS CLI is configured.
- ✅ `aws sts get-caller-identity` works.
- ✅ `terraform init` completed successfully.
- ✅ `terraform validate` succeeded.
- ✅ `terraform fmt` completed.
- ✅ `terraform providers` lists the AWS provider.
- ✅ `.terraform` directory exists.
- ✅ `.terraform.lock.hcl` exists.

---

# Assignment

Complete the following tasks:

1. Run:

```bash
terraform version
```

2. Run:

```bash
terraform providers
```

3. Run:

```bash
terraform fmt
```

4. Run:

```bash
terraform validate
```

5. Run:

```bash
aws sts get-caller-identity
```

6. Verify that the `.terraform` directory and `.terraform.lock.hcl` file exist.

---

# Interview Questions

## Beginner

1. What is a Terraform Provider?
2. Why does Terraform require a Provider?
3. What is the purpose of `provider.tf`?
4. What is the purpose of `versions.tf`?
5. What is the purpose of `terraform.tfvars`?

---

## Intermediate

6. What does `terraform init` do?
7. Why do we use variables?
8. Why should we avoid hardcoding AWS credentials?
9. What is the purpose of `.terraform.lock.hcl`?
10. What is stored inside the `.terraform` directory?

---

## Practical

11. Which command downloads provider plugins?
12. Which command verifies the configuration?
13. Which command formats Terraform code?
14. How does Terraform authenticate with AWS?
15. How can you verify your AWS CLI credentials?

---

# Key Takeaways

After completing this lab, you can:

- Explain what a Terraform Provider is.
- Configure the AWS Provider.
- Configure AWS CLI.
- Authenticate Terraform with AWS.
- Understand the purpose of `versions.tf`.
- Understand the purpose of `provider.tf`.
- Understand how Terraform downloads provider plugins.
- Use `terraform init`, `terraform fmt`, and `terraform validate`.

---

# Save Your Work

Run the following commands:

```bash
git status

git add .

git commit -m "Complete Lab 02 - Configure AWS Provider"

git push origin main
```

---

# What's Next?

**Lab 03 – Terraform Variables**

In the next lab, you will learn:

- What variables are
- Input variables
- Default values
- Variable types
- `terraform.tfvars`
- Variable validation
- Passing variables from the command line
- Environment variables
- Best practices for reusable Terraform code

Congratulations! 🎉

You have completed **Lab 02 – Configure AWS Provider** and your Terraform environment is now ready to build real AWS infrastructure.