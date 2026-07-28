# Lab 03 - Terraform Variables

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 03 |
| Lab Name | Terraform Variables |
| Difficulty | Beginner |
| Duration | 45 Minutes |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will learn:

- What Variables are
- Why Variables are important
- Input Variables
- Default Values
- Variable Types
- terraform.tfvars
- Variable References
- Variable Best Practices

---

# What is a Variable?

A Variable is a placeholder for a value.

Instead of writing the same value many times,

we store it once.

Example:

Instead of:

```
Region = ap-south-1

Region = ap-south-1

Region = ap-south-1
```

We write:

```
aws_region
```

Now we change it only once.

---

# Real Life Example

Think of Variables like a contact saved in your phone.

Instead of remembering:

9876543210

you save:

```
Mom
```

Whenever you call,

you simply choose:

```
Mom
```

Terraform Variables work the same way.

---

# Why Do We Need Variables?

Imagine your company has:

Development

Testing

Production

Development uses:

```
t3.micro
```

Testing uses:

```
t3.small
```

Production uses:

```
t3.medium
```

Without Variables,

you would edit your Terraform code every time.

With Variables,

you only change:

```
terraform.tfvars
```

Everything else stays exactly the same.

---

# Variable Types

Terraform supports multiple data types.

The most common are:

String

```
"ap-south-1"
```

Number

```
10
```

Boolean

```
true
```

List

```
[
"subnet-1",
"subnet-2"
]
```

Map

```
{
Environment = "Dev"
Owner = "Cloud Team"
}
```

Object

A collection of related values.

We will learn Objects in later labs.

---

# Folder Structure

```
terraform/
└── lab-03/
    ├── versions.tf
    ├── provider.tf
    ├── variables.tf
    ├── terraform.tfvars
    ├── outputs.tf
    ├── main.tf
    └── README.md
```

---

# Step 1 - Create Lab 03

Open Git Bash.

Go to your project.

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create folders.

```bash
mkdir -p terraform/lab-03
mkdir -p images/lab-03

touch labs/lab-03-terraform-variables.md
```

Move into Lab 03.

```bash
cd terraform/lab-03
```

---

# Step 2 - Create Terraform Files

Run:

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

# Step 3 - Copy Files from Lab 02

Instead of rewriting the same configuration,

copy these files.

```bash
cp ../lab-02/provider.tf .
cp ../lab-02/versions.tf .
```

Now Lab 03 already knows how to connect to AWS.

---

# Step 4 - Open VS Code

```bash
code .
```

Open:

```
terraform/lab-03
```

---

# Step 5 - Create Your First Variable

Open:

```
variables.tf
```

Paste:

```hcl
variable "aws_region" {

  description = "AWS Region"

  type = string

  default = "ap-south-1"

}
```

This creates your first Terraform variable.

---

# Step 6 - Use terraform.tfvars

Open:

```
terraform.tfvars
```

Paste:

```hcl
aws_region = "ap-south-1"
```

Instead of hardcoding values,

Terraform reads them from this file.

---

# Step 7 - Update provider.tf

Replace the region with:

```hcl
provider "aws" {

  region = var.aws_region

}
```

Notice:

```
var.aws_region
```

This tells Terraform:

Read the value from the variable.

Don't hardcode it.

---

# Summary

You have now created your first Terraform variable.

In Part 2 we will learn:

- Multiple Variables
- Variable Types
- Descriptions
- Defaults
- Variable Validation
- How Terraform Reads Variables

---

# Part 2 - Working with Multiple Variables

In Part 1, you created your first variable:

```hcl
variable "aws_region" {
  description = "AWS Region"
  type        = string
  default     = "ap-south-1"
}
```

Now let's create multiple variables that we will reuse in future labs.

These variables will later be used when creating:

- EC2 Instances
- Security Groups
- VPCs
- S3 Buckets
- IAM Resources

Instead of creating new variables every lab, we'll build a reusable foundation.

---

# Step 8 - Update variables.tf

Replace the contents of `variables.tf` with:

```hcl
variable "aws_region" {
  description = "AWS Region where resources will be created"
  type        = string
  default     = "ap-south-1"
}

variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "dev"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "terraform-zero-to-hero"
}

variable "owner" {
  description = "Resource owner"
  type        = string
  default     = "your-name"
}

variable "instance_type" {
  description = "EC2 Instance Type"
  type        = string
  default     = "t2.micro"
}
```

---

# Understanding Each Variable

## aws_region

```hcl
default = "ap-south-1"
```

Terraform creates resources in:

**Asia Pacific (Mumbai)**

---

## environment

```hcl
default = "dev"
```

Possible values:

- dev
- test
- staging
- production

Most companies separate infrastructure using environment names.

---

## project_name

```hcl
default = "terraform-zero-to-hero"
```

We'll use this value when naming AWS resources.

Example:

```
terraform-zero-to-hero-vpc

terraform-zero-to-hero-ec2

terraform-zero-to-hero-s3
```

---

## owner

```hcl
default = "your-name"
```

This identifies who owns the infrastructure.

Example Tag:

```
Owner = "Alfia"
```

Replace `"your-name"` with your own name.

---

## instance_type

```hcl
default = "t2.micro"
```

We'll use this in Lab 11 when launching an EC2 instance.

---

# Step 9 - Update terraform.tfvars

Replace the contents with:

```hcl
aws_region   = "ap-south-1"
environment  = "dev"
project_name = "terraform-zero-to-hero"
owner         = "Alfia"
instance_type = "t2.micro"
```

Notice that the values here override the defaults in `variables.tf`.

---

# Why Use terraform.tfvars?

Imagine you have three environments.

Development

```hcl
environment = "dev"
instance_type = "t2.micro"
```

Testing

```hcl
environment = "test"
instance_type = "t3.small"
```

Production

```hcl
environment = "prod"
instance_type = "t3.medium"
```

The Terraform code doesn't change.

Only the values change.

This is one of Terraform's biggest strengths.

---

# Step 10 - Validate the Configuration

Run:

```bash
terraform fmt
```

Then:

```bash
terraform validate
```

Expected Output:

```text
Success! The configuration is valid.
```

---

# Step 11 - View Variable Values

Terraform doesn't automatically display variables.

We'll do that in the next lab using **Outputs**.

For now, just understand that Terraform has loaded:

- aws_region
- environment
- project_name
- owner
- instance_type

These values are now available to any Terraform resource.

---

# Real-World Example

Imagine your company deploys the same application in three regions.

Instead of writing three different Terraform projects, you can use variables.

Example:

Mumbai

```hcl
aws_region = "ap-south-1"
```

Singapore

```hcl
aws_region = "ap-southeast-1"
```

Frankfurt

```hcl
aws_region = "eu-central-1"
```

The infrastructure code stays the same.

Only the variable values change.

---

# Lab Checkpoint

You have now:

- Created multiple reusable variables.
- Learned the purpose of `terraform.tfvars`.
- Separated configuration from code.
- Prepared reusable values for future AWS labs.

---

# What's Next?

In Part 3, you'll learn:

- Variable types in detail
- Lists
- Maps
- Booleans
- Numbers
- Variable validation
- How Terraform processes different data types


---

# Part 3 - Terraform Variable Types

Congratulations!

You now know how to create simple variables.

In this section, you'll learn about the different data types that Terraform supports.

Understanding variable types is extremely important because almost every Terraform project uses them.

---

# Why Do Variable Types Matter?

Imagine you ask Terraform to create:

- 3 EC2 Instances
- 5 Subnets
- Enable Versioning
- Add Resource Tags

These values are not all the same.

Some are:

- Text
- Numbers
- True/False
- Lists
- Maps

Terraform needs to know what kind of value it is working with.

---

# Terraform Variable Types

The most commonly used types are:

| Type | Example |
|------|----------|
| string | "Mumbai" |
| number | 5 |
| bool | true |
| list | ["a","b","c"] |
| map | { Name = "EC2" } |

---

# String Variables

A string stores text.

Example:

```hcl
variable "instance_name" {
  type    = string
  default = "web-server"
}
```

Possible values:

```text
web-server

database

bastion-host

terraform-demo
```

Strings are used for:

- Names
- Regions
- AMI IDs
- Instance Types

---

# Number Variables

A number stores numeric values.

Example:

```hcl
variable "disk_size" {
  type    = number
  default = 20
}
```

Possible values:

```text
10

20

50

100
```

We'll use this later when creating EBS volumes.

---

# Boolean Variables

Boolean means:

True or False

Example:

```hcl
variable "enable_monitoring" {
  type    = bool
  default = true
}
```

Possible values:

```text
true

false
```

Examples:

Enable Encryption

Enable Monitoring

Enable Versioning

Enable Logging

---

# List Variables

A list stores multiple values.

Example:

```hcl
variable "availability_zones" {
  type = list(string)

  default = [
    "ap-south-1a",
    "ap-south-1b"
  ]
}
```

Instead of one value,

you now have multiple values.

Lists are commonly used for:

- Subnets
- Availability Zones
- Security Groups
- CIDR Blocks

---

# Map Variables

A map stores key-value pairs.

Example:

```hcl
variable "common_tags" {

  type = map(string)

  default = {

    Environment = "Dev"

    Project = "Terraform"

    Owner = "Alfia"

  }

}
```

Map values are perfect for AWS Tags.

Instead of writing tags repeatedly,

you define them once.

---

# Step 12 - Update variables.tf

Append the following variables below the existing ones:

```hcl
variable "instance_name" {
  description = "EC2 Instance Name"
  type        = string
  default     = "terraform-demo"
}

variable "disk_size" {
  description = "EBS Volume Size"
  type        = number
  default     = 20
}

variable "enable_monitoring" {
  description = "Enable EC2 Monitoring"
  type        = bool
  default     = true
}

variable "availability_zones" {
  description = "Availability Zones"

  type = list(string)

  default = [
    "ap-south-1a",
    "ap-south-1b"
  ]
}

variable "common_tags" {

  description = "Common AWS Tags"

  type = map(string)

  default = {

    Environment = "Dev"

    Project = "Terraform Zero To Hero"

    Owner = "Alfia"

  }

}
```

---

# Step 13 - Update terraform.tfvars

Append the following values:

```hcl
instance_name     = "terraform-demo"
disk_size         = 20
enable_monitoring = true

availability_zones = [
  "ap-south-1a",
  "ap-south-1b"
]

common_tags = {
  Environment = "Dev"
  Project     = "Terraform Zero To Hero"
  Owner       = "Alfia"
}
```

---

# Understanding Lists

Imagine AWS has three Availability Zones.

```
Mumbai

├── ap-south-1a

├── ap-south-1b

└── ap-south-1c
```

Instead of creating three variables,

Terraform stores them as one list.

---

# Understanding Maps

Think of a map like a dictionary.

```
Owner

↓

Alfia

Environment

↓

Dev

Project

↓

Terraform Zero To Hero
```

Every AWS resource can reuse these tags.

---

# How We'll Use These Variables Later

EC2

```hcl
instance_type = var.instance_type
```

Tags

```hcl
tags = var.common_tags
```

Availability Zones

```hcl
availability_zone = var.availability_zones[0]
```

Disk Size

```hcl
volume_size = var.disk_size
```

Monitoring

```hcl
monitoring = var.enable_monitoring
```

Notice how everything becomes reusable.

---

# Validate the Configuration

Run:

```bash
terraform fmt
```

Then:

```bash
terraform validate
```

Expected Output:

```text
Success! The configuration is valid.
```

---

# Summary

You have now learned:

- String Variables
- Number Variables
- Boolean Variables
- List Variables
- Map Variables

These are the five most common variable types you'll use in Terraform projects.

---

# End of Part 3

In Part 4, you'll learn:

- Variable Validation
- Input Variable Precedence
- Environment Variables
- Passing Variables from the Command Line
- Best Practices
- Interview Questions
- Assignment
- Git Commit

---

# Part 4 - Variable Validation, Variable Precedence and Best Practices

Congratulations!

You now understand the five most common Terraform variable types.

In this final part, you'll learn how to make variables safer, easier to manage, and more suitable for real-world projects.

---

# What is Variable Validation?

Variable validation allows you to restrict what values users can enter.

Think of it like a security guard.

Without validation:

```
instance_type = "banana"
```

Terraform accepts it until AWS rejects it later.

With validation,

Terraform immediately reports the error before attempting to create infrastructure.

---

# Why Validation Matters

Imagine your production environment only allows three EC2 instance types:

- t2.micro
- t3.micro
- t3.small

Instead of allowing any value, we validate the input.

---

# Step 14 - Add Validation

Open:

```
variables.tf
```

Locate the variable:

```hcl
variable "instance_type"
```

Replace it with:

```hcl
variable "instance_type" {

  description = "EC2 Instance Type"

  type = string

  default = "t2.micro"

  validation {

    condition = contains(
      ["t2.micro", "t3.micro", "t3.small"],
      var.instance_type
    )

    error_message = "Only t2.micro, t3.micro or t3.small are allowed."

  }

}
```

---

# How Validation Works

Terraform checks:

```
Is the value inside this list?
```

If yes:

```
Continue
```

If no:

```
Stop immediately
```

---

# Test Validation

Change:

```hcl
instance_type = "t2.micro"
```

to

```hcl
instance_type = "banana"
```

Run:

```bash
terraform validate
```

Expected Output:

```text
Error:

Invalid value for variable "instance_type"

Only t2.micro, t3.micro or t3.small are allowed.
```

Now change it back:

```hcl
instance_type = "t2.micro"
```

---

# Variable Precedence

Terraform can receive variables from multiple locations.

If the same variable exists in multiple places, Terraform follows a priority order.

Highest priority wins.

---

# Order of Precedence

```
1. Command Line (-var)

↓

2. *.auto.tfvars

↓

3. terraform.tfvars

↓

4. Environment Variables

↓

5. Default Value
```

---

# Example

Suppose:

variables.tf

```hcl
default = "ap-south-1"
```

terraform.tfvars

```hcl
aws_region = "us-east-1"
```

Command Line

```bash
terraform plan -var="aws_region=eu-west-1"
```

Terraform uses:

```
eu-west-1
```

because command-line values have the highest priority.

---

# Passing Variables from the Command Line

Example:

```bash
terraform plan -var="instance_type=t3.micro"
```

No files are modified.

Terraform uses the value only for that command.

---

# Using Environment Variables

Terraform also supports environment variables.

Linux/macOS:

```bash
export TF_VAR_environment=dev
```

Windows PowerShell:

```powershell
$env:TF_VAR_environment="dev"
```

Terraform automatically reads variables prefixed with:

```
TF_VAR_
```

---

# Best Practices

## Good Practice

```hcl
instance_type = var.instance_type
```

## Avoid

```hcl
instance_type = "t2.micro"
```

---

## Good Practice

```hcl
tags = var.common_tags
```

## Avoid

```hcl
tags = {
  Name = "Server"
  Owner = "John"
}
```

---

## Good Practice

Use descriptive variable names.

Example:

```hcl
aws_region
```

instead of:

```hcl
r
```

---

# Common Mistakes

## Wrong Variable Name

```hcl
var.region
```

When the variable is:

```hcl
aws_region
```

Terraform reports:

```
Reference to undeclared input variable
```

---

## Wrong Data Type

Example:

```hcl
disk_size = "twenty"
```

Expected:

```hcl
disk_size = 20
```

---

## Incorrect Boolean

Wrong:

```hcl
enable_monitoring = "true"
```

Correct:

```hcl
enable_monitoring = true
```

---

# Lab Verification Checklist

Verify that:

- ✅ String variables work.
- ✅ Number variables work.
- ✅ Boolean variables work.
- ✅ List variables work.
- ✅ Map variables work.
- ✅ Validation works.
- ✅ terraform fmt succeeds.
- ✅ terraform validate succeeds.

---

# Assignment

Complete the following tasks.

### Task 1

Change:

```
environment
```

to

```
test
```

Run:

```bash
terraform validate
```

---

### Task 2

Change:

```
disk_size
```

from:

```
20
```

to:

```
50
```

Run:

```bash
terraform validate
```

---

### Task 3

Add another Availability Zone.

Example:

```hcl
availability_zones = [
  "ap-south-1a",
  "ap-south-1b",
  "ap-south-1c"
]
```

Validate again.

---

### Task 4

Add another tag.

Example:

```hcl
CostCenter = "Engineering"
```

---

# Interview Questions

## Beginner

1. What is an input variable?
2. Why are variables useful?
3. What is the purpose of `terraform.tfvars`?
4. What is the difference between a default value and a value in `terraform.tfvars`?
5. Name five Terraform variable types.

---

## Intermediate

6. What is variable validation?
7. What happens if validation fails?
8. What is variable precedence?
9. How do you pass variables on the command line?
10. How do environment variables work in Terraform?

---

## Practical

11. Why should you avoid hardcoding values?
12. How can variables make Terraform reusable?
13. When would you use a list instead of a string?
14. When would you use a map?
15. What are common mistakes when defining variables?

---

# Summary

After completing this lab, you can:

- Create input variables.
- Use strings, numbers, booleans, lists, and maps.
- Store values in `terraform.tfvars`.
- Validate variable values.
- Understand variable precedence.
- Pass variables from the command line.
- Use environment variables.
- Apply Terraform variable best practices.

---

# Save Your Work

Run:

```bash
git status

git add .

git commit -m "Complete Lab 03 - Terraform Variables"

git push origin main
```

---

# Next Lab

**Lab 04 – Terraform Outputs**

In the next lab, you'll learn:

- What outputs are
- Why outputs are useful
- How to display resource information
- Output formatting
- Sensitive outputs
- Using outputs in real AWS deployments

Congratulations! 🎉

You have completed **Lab 03 – Terraform Variables**.