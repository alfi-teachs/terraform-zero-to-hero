# Lab 05 - Terraform Locals

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 05 |
| Lab Name | Terraform Locals |
| Difficulty | Beginner |
| Duration | 45 Minutes |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will be able to:

- Explain what Terraform Locals are
- Differentiate Variables and Locals
- Create Local Values
- Use Locals in Terraform code
- Standardize resource naming
- Standardize AWS Tags
- Understand production naming conventions

---

# Real World Scenario

Imagine your company has hundreds of AWS resources.

Without standards, resources may look like:

```
EC2

Server1

MyServer

Linux01

ProductionServer

Web01
```

Everyone uses different names.

Now imagine a standard.

```
project-environment-resource

terraform-zero-to-hero-dev-ec2

terraform-zero-to-hero-dev-vpc

terraform-zero-to-hero-dev-s3
```

Everything becomes organized.

Terraform Locals help us achieve this.

---

# What Are Terraform Locals?

A Local is a value calculated once and reused many times.

Think of it as a shortcut.

Instead of repeating the same expression throughout your code, you define it once in a `locals` block and reference it wherever needed.

---

# Variables vs Locals

| Variables | Locals |
|-----------|--------|
| User provides the value | Terraform calculates the value |
| Can change between environments | Usually stays consistent within a project |
| Stored in `terraform.tfvars` | Stored in `locals.tf` or `main.tf` |
| Referenced with `var.name` | Referenced with `local.name` |

---

# Architecture

```
terraform.tfvars

↓

Variables

↓

Locals

↓

AWS Resources

↓

Outputs
```

Variables provide input.

Locals transform or combine that input.

Resources use the local values.

---

# Folder Structure

```
terraform/
└── lab-05/
    ├── versions.tf
    ├── provider.tf
    ├── variables.tf
    ├── terraform.tfvars
    ├── locals.tf
    ├── outputs.tf
    ├── main.tf
    └── README.md
```

---

# Step 1 - Create Lab Folder

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero

mkdir -p terraform/lab-05
mkdir -p images/lab-05

touch labs/lab-05-terraform-locals.md

cd terraform/lab-05
```

---

# Step 2 - Copy Files from Lab 04

```bash
cp ../lab-04/versions.tf .
cp ../lab-04/provider.tf .
cp ../lab-04/variables.tf .
cp ../lab-04/terraform.tfvars .
cp ../lab-04/outputs.tf .
```

Create the remaining files.

```bash
touch locals.tf
touch main.tf
touch README.md
```

---

# Step 3 - Create Your First Local

Open:

```
locals.tf
```

Paste:

```hcl
locals {

  project_prefix = "${var.project_name}-${var.environment}"

}
```

---

# Understanding This Local

Suppose:

```
project_name = terraform-zero-to-hero

environment = dev
```

Terraform combines them into:

```
terraform-zero-to-hero-dev
```

Instead of writing:

```
${var.project_name}-${var.environment}
```

over and over,

you can simply write:

```hcl
local.project_prefix
```

---

# Step 4 - Add Another Local

Append:

```hcl
locals {

  project_prefix = "${var.project_name}-${var.environment}"

  common_name = "${local.project_prefix}-resources"

}
```

Now Terraform creates:

```
terraform-zero-to-hero-dev-resources
```

Notice that one Local can reference another Local.

---

# Step 5 - Create Outputs

Open:

```
outputs.tf
```

Append:

```hcl
output "project_prefix" {

  description = "Standard Project Prefix"

  value = local.project_prefix

}

output "common_name" {

  description = "Common Resource Name"

  value = local.common_name

}
```

---

# Step 6 - Format

```bash
terraform fmt
```

---

# Step 7 - Initialize

```bash
terraform init
```

---

# Step 8 - Validate

```bash
terraform validate
```

Expected Output

```
Success! The configuration is valid.
```

---

# Step 9 - Apply

```bash
terraform apply
```

Type:

```
yes
```

Expected Output

```
Outputs:

project_prefix = "terraform-zero-to-hero-dev"

common_name = "terraform-zero-to-hero-dev-resources"
```

---

# Summary

You learned:

- What Locals are
- Why Locals are useful
- Variables vs Locals
- Referencing Local values
- Creating reusable naming standards

---

# End of Part 1

In Part 2 you'll learn:

- Multiple Locals
- Local Maps
- Standardized AWS Tags
- String Functions
- Production naming conventions

---

# Part 2 - Production Naming Standards and Local Maps

Congratulations!

You have created your first Terraform Local.

Now let's learn how companies use Locals to create standardized resource names and reusable tags.

---

# Why Do Companies Use Naming Standards?

Imagine an AWS account with 500 resources.

Without naming standards:

```
EC2

Server1

Linux

TestVM

Production

WebServer01

DemoMachine
```

Can you immediately identify:

- Which project?
- Which environment?
- Who owns it?

No.

Now look at this:

```
terraform-zero-to-hero-dev-ec2

terraform-zero-to-hero-dev-vpc

terraform-zero-to-hero-dev-s3
```

Immediately you know:

- Project
- Environment
- Resource Type

This is why naming standards matter.

---

# Step 10 - Update locals.tf

Replace the contents with:

```hcl
locals {

  project_prefix = "${var.project_name}-${var.environment}"

  ec2_name = "${local.project_prefix}-ec2"

  vpc_name = "${local.project_prefix}-vpc"

  subnet_name = "${local.project_prefix}-subnet"

  security_group_name = "${local.project_prefix}-sg"

  s3_bucket_name = "${local.project_prefix}-bucket"

}
```

---

# Understanding These Locals

If:

```text
project_name = terraform-zero-to-hero

environment = dev
```

Terraform creates:

```
terraform-zero-to-hero-dev-ec2

terraform-zero-to-hero-dev-vpc

terraform-zero-to-hero-dev-subnet

terraform-zero-to-hero-dev-sg

terraform-zero-to-hero-dev-bucket
```

Notice that changing only:

```
environment = prod
```

automatically changes every resource name.

---

# Local Maps

Locals can also store maps.

This is one of the most common production practices.

Append this to the same locals block:

```hcl
common_tags = {

  Project = var.project_name

  Environment = var.environment

  Owner = var.owner

  ManagedBy = "Terraform"

}
```

Your complete locals block now contains both strings and a map.

---

# Why Use Local Tags?

Imagine creating:

- EC2
- VPC
- S3
- Security Group
- Internet Gateway

Without Locals:

You would repeat:

```hcl
tags = {

  Project = "terraform-zero-to-hero"

  Environment = "dev"

  Owner = "Alfia"

}
```

for every resource.

Instead:

```hcl
tags = local.common_tags
```

One definition.

Unlimited reuse.

---

# Step 11 - Update outputs.tf

Append:

```hcl
output "ec2_name" {

  description = "EC2 Name"

  value = local.ec2_name

}

output "vpc_name" {

  description = "VPC Name"

  value = local.vpc_name

}

output "common_tags" {

  description = "Standard Tags"

  value = local.common_tags

}
```

---

# Step 12 - Format

Run:

```bash
terraform fmt
```

---

# Step 13 - Validate

Run:

```bash
terraform validate
```

Expected Output

```text
Success! The configuration is valid.
```

---

# Step 14 - Apply

Run:

```bash
terraform apply
```

Expected Output

```text
Outputs:

ec2_name = "terraform-zero-to-hero-dev-ec2"

vpc_name = "terraform-zero-to-hero-dev-vpc"

common_tags = {

  "Environment" = "dev"

  "ManagedBy" = "Terraform"

  "Owner" = "Alfia"

  "Project" = "terraform-zero-to-hero"

}
```

---

# Behind the Scenes

Terraform evaluates Locals only once.

```
terraform apply

        │

        ▼

Read Variables

        │

        ▼

Calculate Locals

        │

        ▼

Create Resources

        │

        ▼

Generate Outputs
```

This improves readability because complex expressions are written once and reused.

---

# Real Production Example

A company deploys:

```
Development

Testing

Production
```

Only this changes:

```hcl
environment = "dev"
```

or

```hcl
environment = "test"
```

or

```hcl
environment = "prod"
```

Terraform automatically generates consistent names for every environment.

---

# Summary

In this part you learned:

- Standard resource naming
- Local maps
- Standard AWS tags
- Reusing Local values
- Production naming conventions

---

# End of Part 2

In Part 3 you'll learn:

- String functions
- Local expressions
- Conditional Locals
- Dynamic naming
- Production examples

---

# Part 3 - Terraform Functions with Locals

Congratulations!

You now know how to create Local values.

In this section you'll learn how Terraform functions make your code smarter.

Functions allow Terraform to:

- Change text
- Join values
- Split values
- Replace text
- Calculate lengths
- Format names
- Build reusable naming standards

Functions are used in almost every production Terraform project.

---

# What is a Function?

Think of a function as a machine.

You give it an input.

It returns an output.

Example

Input

```
terraform
```

Function

```
upper()
```

Output

```
TERRAFORM
```

---

# Common Terraform Functions

| Function | Purpose |
|----------|---------|
| upper() | Convert text to uppercase |
| lower() | Convert text to lowercase |
| title() | Capitalize words |
| replace() | Replace text |
| join() | Join multiple values |
| split() | Split text |
| format() | Format strings |
| length() | Count items |

---

# Step 15 - Update locals.tf

Append these Local values inside the existing `locals {}` block.

```hcl
project_upper = upper(var.project_name)

project_lower = lower(var.project_name)

project_title = title(var.project_name)
```

---

# Understanding upper()

Input

```
terraform-zero-to-hero
```

Output

```
TERRAFORM-ZERO-TO-HERO
```

---

# Understanding lower()

Input

```
Terraform-Zero-To-Hero
```

Output

```
terraform-zero-to-hero
```

---

# Understanding title()

Input

```
terraform zero to hero
```

Output

```
Terraform Zero To Hero
```

---

# Step 16 - Add Outputs

Open:

```
outputs.tf
```

Append:

```hcl
output "project_upper" {

  value = local.project_upper

}

output "project_lower" {

  value = local.project_lower

}

output "project_title" {

  value = local.project_title

}
```

---

# Step 17 - Using replace()

Append this Local.

```hcl
project_without_dash = replace(
  var.project_name,
  "-",
  "_"
)
```

Input

```
terraform-zero-to-hero
```

Output

```
terraform_zero_to_hero
```

Add Output.

```hcl
output "project_without_dash" {

  value = local.project_without_dash

}
```

---

# Step 18 - Using join()

Append:

```hcl
environment_list = join(
  "-",
  [
    var.project_name,
    var.environment,
    "aws"
  ]
)
```

Output

```
terraform-zero-to-hero-dev-aws
```

Output Block

```hcl
output "environment_list" {

  value = local.environment_list

}
```

---

# Step 19 - Using split()

Append:

```hcl
project_parts = split(
  "-",
  var.project_name
)
```

Terraform creates:

```
terraform

zero

to

hero
```

Output

```hcl
output "project_parts" {

  value = local.project_parts

}
```

---

# Step 20 - Using length()

Append:

```hcl
project_length = length(var.project_name)
```

Output

```hcl
output "project_length" {

  value = local.project_length

}
```

Example

```
24
```

(The exact number depends on the text.)

---

# Step 21 - Using format()

Append:

```hcl
ec2_display_name = format(
  "%s-%s-%s",
  var.project_name,
  var.environment,
  "ec2"
)
```

Output

```
terraform-zero-to-hero-dev-ec2
```

Output Block

```hcl
output "ec2_display_name" {

  value = local.ec2_display_name

}
```

---

# Step 22 - Format the Configuration

Run:

```bash
terraform fmt
```

---

# Step 23 - Validate

Run:

```bash
terraform validate
```

Expected Output

```
Success! The configuration is valid.
```

---

# Step 24 - Apply

Run:

```bash
terraform apply
```

Expected Output

```
Outputs

project_upper

TERRAFORM-ZERO-TO-HERO

project_lower

terraform-zero-to-hero

project_title

Terraform Zero To Hero

project_without_dash

terraform_zero_to_hero

environment_list

terraform-zero-to-hero-dev-aws

project_parts

[
  terraform,
  zero,
  to,
  hero
]

project_length

24

ec2_display_name

terraform-zero-to-hero-dev-ec2
```

---

# Behind the Scenes

Terraform evaluates functions before creating resources.

```
Variables

        │

        ▼

Functions

        │

        ▼

Locals

        │

        ▼

Resources

        │

        ▼

Outputs
```

Functions do not create infrastructure.

They simply calculate values.

---

# Real Production Example

Suppose your company requires every EC2 instance name to follow this format:

```
company-environment-application-resource
```

Instead of typing names manually:

```
acme-dev-web-ec2

acme-test-web-ec2

acme-prod-web-ec2
```

Terraform can generate them automatically with:

```hcl
format(
  "%s-%s-%s-%s",
  "acme",
  var.environment,
  "web",
  "ec2"
)
```

This reduces mistakes and keeps naming consistent across environments.

---

# Summary

You have learned how to use:

- upper()
- lower()
- title()
- replace()
- join()
- split()
- format()
- length()

These functions are used regularly in production Terraform projects.

---

# End of Part 3

In Part 4 you'll learn:

- Conditional Locals
- Complex Expressions
- Best Practices
- Interview Questions
- Assignment
- Git Commands
- Cleanup
- Complete Lab 05

---

# Part 4 - Conditional Locals, Best Practices and Lab Completion

Congratulations!

You now know:

- Variables
- Outputs
- Locals
- Terraform Functions

Now let's learn one final feature of Locals before moving on to Data Sources.

---

# What are Conditional Expressions?

Sometimes Terraform needs to make decisions.

For example:

Development

```
t2.micro
```

Production

```
t3.large
```

Instead of manually changing the instance type every time,

Terraform can decide automatically.

---

# Terraform Conditional Syntax

Terraform uses:

```hcl
condition ? true_value : false_value
```

Think of it like asking a question.

```
Is environment production?

Yes

↓

Use t3.large

No

↓

Use t2.micro
```

---

# Step 25 - Create Conditional Local

Open:

```
locals.tf
```

Append:

```hcl
instance_type = var.environment == "prod" ? "t3.large" : "t2.micro"
```

Explanation

If:

```
environment = "prod"
```

Terraform returns:

```
t3.large
```

Otherwise:

```
t2.micro
```

---

# Step 26 - Another Conditional Example

Append:

```hcl
backup_enabled = var.environment == "prod" ? true : false
```

Development

```
false
```

Production

```
true
```

Many companies enable backups only for production.

---

# Step 27 - Dynamic Resource Name

Append:

```hcl
resource_name = format(
  "%s-%s-%s",
  var.project_name,
  var.environment,
  "web"
)
```

Terraform generates:

```
terraform-zero-to-hero-dev-web
```

Change the environment:

```
prod
```

Terraform automatically generates:

```
terraform-zero-to-hero-prod-web
```

---

# Step 28 - Output the Values

Open:

```
outputs.tf
```

Append:

```hcl
output "calculated_instance_type" {

  description = "Instance Type Based on Environment"

  value = local.instance_type

}

output "backup_enabled" {

  description = "Backup Enabled"

  value = local.backup_enabled

}

output "resource_name" {

  description = "Generated Resource Name"

  value = local.resource_name

}
```

---

# Step 29 - Format

Run:

```bash
terraform fmt
```

---

# Step 30 - Validate

Run:

```bash
terraform validate
```

Expected Output

```text
Success! The configuration is valid.
```

---

# Step 31 - Apply

Run:

```bash
terraform apply
```

Expected Output

```
Outputs

calculated_instance_type

t2.micro

backup_enabled

false

resource_name

terraform-zero-to-hero-dev-web
```

Now edit:

```
terraform.tfvars
```

Change:

```hcl
environment = "prod"
```

Run:

```bash
terraform apply
```

Terraform now displays:

```
Outputs

calculated_instance_type

t3.large

backup_enabled

true

resource_name

terraform-zero-to-hero-prod-web
```

Notice that you never edited the Local.

You only changed the variable.

---

# Behind the Scenes

Terraform evaluates everything in this order:

```
Read Variables

        │

        ▼

Evaluate Functions

        │

        ▼

Evaluate Conditionals

        │

        ▼

Calculate Locals

        │

        ▼

Create Resources

        │

        ▼

Generate Outputs
```

Understanding this order makes debugging much easier.

---

# Real Production Example

Suppose your company has three environments.

Development

```
Instance Type

t2.micro
```

Testing

```
Instance Type

t3.micro
```

Production

```
Instance Type

t3.large
```

Instead of maintaining three different Terraform projects, you can use conditional logic to choose values based on the environment.

---

# Best Practices

Always:

- Keep Variables as user input.
- Use Locals for calculated values.
- Store common tags in a Local.
- Store naming standards in a Local.
- Use descriptive Local names.
- Keep calculations simple and readable.

Avoid:

- Hardcoding repeated values.
- Creating unnecessary Locals.
- Using unclear names such as `local.value1`.

---

# Common Mistakes

## Wrong Reference

Incorrect:

```hcl
var.project_prefix
```

Correct:

```hcl
local.project_prefix
```

---

## Circular Reference

Incorrect:

```hcl
locals {

  value1 = local.value2

  value2 = local.value1

}
```

Terraform cannot evaluate circular references.

---

## Using Locals for User Input

Don't do this:

```hcl
locals {

  aws_region = "ap-south-1"

}
```

Use a Variable instead.

---

# Lab Verification Checklist

Verify that:

- ✅ Local values display correctly.
- ✅ Local maps work.
- ✅ String functions work.
- ✅ Conditional expressions work.
- ✅ Generated names change when the environment changes.
- ✅ `terraform fmt` succeeds.
- ✅ `terraform validate` succeeds.

---

# Assignment

## Task 1

Change:

```hcl
environment = "test"
```

Run:

```bash
terraform apply
```

Observe the changes.

---

## Task 2

Create a Local named:

```hcl
s3_name
```

Expected format:

```
terraform-zero-to-hero-dev-s3
```

Display it using an Output.

---

## Task 3

Create a Local named:

```hcl
database_name
```

Expected format:

```
terraform-zero-to-hero-dev-db
```

Display it using an Output.

---

## Task 4

Add another tag:

```hcl
Department = "Cloud"
```

Verify it appears in the Output.

---

# Interview Questions

## Beginner

1. What is a Terraform Local?
2. How are Locals different from Variables?
3. How do you reference a Local?
4. Why are Locals useful?
5. Where are Locals usually defined?

---

## Intermediate

6. What is a conditional expression?
7. What Terraform functions have you used?
8. Why use Local maps?
9. How do Locals improve readability?
10. Can a Local reference another Local?

---

## Advanced

11. Can a Local reference a Variable?
12. Can a Variable reference a Local?
13. Why should naming conventions use Locals?
14. What is a circular reference?
15. How are Locals evaluated?

---

# Summary

You have completed:

- Local values
- Local maps
- Terraform functions
- Conditional expressions
- Dynamic naming
- Production naming standards

You are now writing Terraform much closer to the way experienced engineers do.

---

# Save Your Work

Run:

```bash
git status

git add .

git commit -m "Complete Lab 05 - Terraform Locals"

git push origin main
```

---

# Congratulations!

You have completed:

- ✅ Lab 01 – Install Terraform
- ✅ Lab 02 – Configure AWS Provider
- ✅ Lab 03 – Terraform Variables
- ✅ Lab 04 – Terraform Outputs
- ✅ Lab 05 – Terraform Locals

---

# Next Lab

## Lab 06 – Terraform Data Sources

You'll learn:

- What Data Sources are
- Why they are used
- Fetching existing AWS resources
- Looking up the latest Amazon Linux AMI
- Looking up Availability Zones
- Using Data Sources with EC2
- Data Sources vs Resources
- Production best practices

By the end of Lab 06, you'll stop hardcoding AWS IDs and start dynamically retrieving information from AWS, which is the standard approach in production Terraform projects.