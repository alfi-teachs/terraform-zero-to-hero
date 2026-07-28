# Lab 04 - Terraform Outputs

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 04 |
| Lab Name | Terraform Outputs |
| Difficulty | Beginner |
| Duration | 45 Minutes |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will learn:

- What Terraform Outputs are
- Why Outputs are used
- How to create Outputs
- Sensitive Outputs
- Output Formatting
- How Terraform stores Outputs
- Production Use Cases

---

# Real World Scenario

Imagine you create an EC2 instance.

Terraform creates:

- Instance ID
- Public IP
- Private IP
- Public DNS

How do you know these values?

Do you log into the AWS Console every time?

No.

Terraform displays them automatically using Outputs.

---

# What is an Output?

An Output tells Terraform:

> "After creating infrastructure, display this information."

Think of Outputs as the **final report** after Terraform finishes its work.

---

# Example

Without Outputs:

```
terraform apply

✔ Infrastructure Created

Now open AWS Console...

Find the EC2...

Copy the Public IP...
```

With Outputs:

```
terraform apply

✔ Infrastructure Created

Outputs

Public IP

18.205.xx.xx

Instance ID

i-0123456789abcdef0
```

Much faster and easier.

---

# Where Are Outputs Used?

Outputs are commonly used to display:

- EC2 Public IP
- EC2 Private IP
- Instance ID
- VPC ID
- Subnet IDs
- Security Group ID
- S3 Bucket Name
- CloudFront Domain Name
- Load Balancer DNS Name
- RDS Endpoint

---

# Behind the Scenes

When Terraform creates infrastructure, it also creates a file called:

```
terraform.tfstate
```

This file stores:

- Resources
- Resource IDs
- Metadata
- Outputs

The flow looks like this:

```
Terraform Apply
        │
        ▼
AWS Creates Resource
        │
        ▼
Terraform Updates State
        │
        ▼
terraform.tfstate
        │
        ▼
Outputs Become Available
```

This is why the `terraform output` command works—it reads values from the state file.

---

# Folder Structure

```
terraform/
└── lab-04/
    ├── versions.tf
    ├── provider.tf
    ├── variables.tf
    ├── terraform.tfvars
    ├── outputs.tf
    ├── main.tf
    └── README.md
```

---

# Step 1 - Create Lab 04

Open Git Bash.

Go to your project.

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create the folders.

```bash
mkdir -p terraform/lab-04
mkdir -p images/lab-04

touch labs/lab-04-terraform-outputs.md
```

Move into the new lab.

```bash
cd terraform/lab-04
```

---

# Step 2 - Copy Configuration from Lab 03

Copy the reusable configuration files.

```bash
cp ../lab-03/versions.tf .
cp ../lab-03/provider.tf .
cp ../lab-03/variables.tf .
cp ../lab-03/terraform.tfvars .
```

This keeps all labs consistent.

---

# Step 3 - Create Remaining Files

Run:

```bash
touch outputs.tf
touch main.tf
touch README.md
```

Verify:

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

# Step 4 - Understanding outputs.tf

The file:

```
outputs.tf
```

contains every value that Terraform should display after a successful deployment.

An Output has three parts:

- Name
- Value
- Description (optional)

Basic structure:

```hcl
output "example_name" {
  value = "Hello Terraform"
}
```

---

# Step 5 - Create Your First Output

Open:

```
outputs.tf
```

Paste:

```hcl
output "welcome_message" {

  description = "First Terraform Output"

  value = "Welcome to Terraform Zero to Hero!"

}
```

This output doesn't depend on AWS resources.

It simply demonstrates how Outputs work.

---

# Step 6 - Format the Code

Run:

```bash
terraform fmt
```

---

# Step 7 - Initialize Terraform

Run:

```bash
terraform init
```

If you already initialized this folder earlier, Terraform may report that everything is already configured.

---

# Step 8 - Validate

Run:

```bash
terraform validate
```

Expected Output

```
Success! The configuration is valid.
```

---

# Step 9 - Apply the Configuration

Run:

```bash
terraform apply
```

Type:

```
yes
```

Because this lab contains no AWS resources yet, Terraform only evaluates the output.

Expected Output:

```
Outputs:

welcome_message = "Welcome to Terraform Zero to Hero!"
```

Congratulations!

You have created your first Terraform Output.

---

# Step 10 - View Outputs

Run:

```bash
terraform output
```

Expected Output

```
welcome_message = "Welcome to Terraform Zero to Hero!"
```

You can also retrieve a single output:

```bash
terraform output welcome_message
```

Expected Output

```
Welcome to Terraform Zero to Hero!
```

---

# End of Part 1

In Part 2, you'll learn:

- Multiple Outputs
- Output formatting
- Sensitive Outputs
- JSON Outputs
- Real AWS Outputs
- Production best practices

---

# Part 2 - Multiple Outputs and Output Types

Congratulations!

You have created your first Terraform Output.

Now let's explore how Outputs are used in real Terraform projects.

In production, a project rarely has just one Output.

A typical deployment may display:

- EC2 Instance ID
- Public IP
- Private IP
- Security Group ID
- VPC ID
- Subnet IDs
- Load Balancer DNS
- RDS Endpoint

Terraform can display all of these automatically.

---

# Step 11 - Create Multiple Outputs

Open:

```
outputs.tf
```

Replace the contents with:

```hcl
output "welcome_message" {

  description = "Welcome Message"

  value = "Welcome to Terraform Zero to Hero"

}

output "project_name" {

  description = "Project Name"

  value = var.project_name

}

output "environment" {

  description = "Deployment Environment"

  value = var.environment

}

output "aws_region" {

  description = "AWS Region"

  value = var.aws_region

}
```

Notice something interesting.

These Outputs are displaying variables.

We are not creating AWS resources yet.

Terraform is simply displaying values stored in variables.

---

# Step 12 - Format the Configuration

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

# Step 14 - Apply the Configuration

Run:

```bash
terraform apply
```

Type:

```
yes
```

Expected Output

```text
Outputs:

aws_region = "ap-south-1"

environment = "dev"

project_name = "terraform-zero-to-hero"

welcome_message = "Welcome to Terraform Zero to Hero"
```

Terraform now displays four Outputs.

---

# Step 15 - Display a Single Output

Instead of displaying everything,

you can display only one Output.

Example:

```bash
terraform output project_name
```

Expected Output

```text
terraform-zero-to-hero
```

Another example:

```bash
terraform output environment
```

Expected Output

```text
dev
```

---

# Step 16 - Display Outputs in JSON

Run:

```bash
terraform output -json
```

Expected Output

```json
{
  "aws_region": {
    "value": "ap-south-1"
  },
  "environment": {
    "value": "dev"
  },
  "project_name": {
    "value": "terraform-zero-to-hero"
  }
}
```

JSON output is useful when another tool or script needs to read Terraform Outputs.

---

# How Companies Use JSON Outputs

Imagine a deployment pipeline.

```
Terraform

↓

Creates EC2

↓

Returns Public IP

↓

Jenkins Reads JSON

↓

Runs Ansible

↓

Configures EC2
```

Automation tools often consume Terraform Outputs in JSON format.

---

# Step 17 - Output Formatting

Outputs can contain:

String

```hcl
output "message" {

  value = "Hello"

}
```

Number

```hcl
output "disk_size" {

  value = var.disk_size

}
```

Boolean

```hcl
output "monitoring_enabled" {

  value = var.enable_monitoring

}
```

List

```hcl
output "availability_zones" {

  value = var.availability_zones

}
```

Map

```hcl
output "tags" {

  value = var.common_tags

}
```

Terraform supports all major data types.

---

# Step 18 - Display List Output

Append this Output.

```hcl
output "availability_zones" {

  description = "Available Zones"

  value = var.availability_zones

}
```

Run:

```bash
terraform apply
```

Expected Output

```text
availability_zones = [
  "ap-south-1a",
  "ap-south-1b"
]
```

---

# Step 19 - Display Map Output

Append:

```hcl
output "common_tags" {

  description = "Common Tags"

  value = var.common_tags

}
```

Run:

```bash
terraform apply
```

Expected Output

```text
common_tags = {
  "Environment" = "Dev"
  "Owner" = "Alfia"
  "Project" = "Terraform Zero To Hero"
}
```

---

# Behind the Scenes

When you run:

```bash
terraform output
```

Terraform does NOT contact AWS.

Instead, it reads values from:

```
terraform.tfstate
```

The process looks like this:

```
terraform output

       │

       ▼

terraform.tfstate

       │

Reads Outputs

       │

Displays Values
```

This makes the command extremely fast.

---

# Summary

In this part you learned:

- Multiple Outputs
- Output data types
- JSON Outputs
- Displaying a single Output
- Displaying all Outputs
- How Terraform retrieves Outputs

---

# End of Part 2

In Part 3 you'll learn:

- Sensitive Outputs
- Real AWS Outputs
- Output Dependencies
- Production Use Cases
- Best Practices

---

# Part 3 - Sensitive Outputs and Production Use Cases

Congratulations!

You now know how to create Outputs.

But here's an important question.

Should Terraform display everything?

Imagine Terraform prints:

- Database Password
- AWS Secret Access Key
- API Token
- SSH Private Key

That would be a huge security risk.

Terraform solves this problem using **Sensitive Outputs**.

---

# What is a Sensitive Output?

A Sensitive Output tells Terraform:

> "This value exists, but don't display it on the screen."

Instead of showing the value,

Terraform hides it.

---

# Real World Example

Imagine your application uses:

```
Database Password

SuperSecretPassword123
```

Should everyone running Terraform see it?

No.

Instead Terraform displays:

```
db_password = (sensitive value)
```

The password is stored in the Terraform state,

but it is hidden from normal console output.

---

# Step 20 - Create a Sensitive Variable

Open:

```
variables.tf
```

Add:

```hcl
variable "database_password" {

  description = "Database Password"

  type = string

  sensitive = true

  default = "MySecurePassword123"

}
```

---

# Step 21 - Add a Sensitive Output

Open:

```
outputs.tf
```

Append:

```hcl
output "database_password" {

  description = "Database Password"

  value = var.database_password

  sensitive = true

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

Type:

```
yes
```

Expected Output

```
Outputs:

database_password = (sensitive value)

project_name = "terraform-zero-to-hero"

environment = "dev"
```

Notice something.

Terraform knows the value.

But it refuses to display it.

---

# Can Terraform Still Access It?

Yes.

Resources can still use:

```hcl
var.database_password
```

Only the screen output is hidden.

---

# Behind the Scenes

Terraform stores sensitive values inside the state file.

```
terraform apply

      │

      ▼

Creates Output

      │

      ▼

terraform.tfstate

      │

      ▼

Marks Output as Sensitive

      │

      ▼

Console Hides Value
```

Important:

Sensitive outputs are **hidden in normal console output**, but the value still exists in the Terraform state file.

This is why protecting your Terraform state is so important.

---

# Production Examples

Sensitive Outputs include:

Database Password

```text
db_password
```

API Token

```text
api_token
```

SSH Private Key

```text
private_key
```

OAuth Secret

```text
oauth_secret
```

Access Token

```text
access_token
```

Never display these values in logs.

---

# Step 25 - Output a List

Append:

```hcl
output "availability_zones" {

  value = var.availability_zones

}
```

Apply again.

Expected Output

```text
availability_zones = [
  "ap-south-1a",
  "ap-south-1b"
]
```

---

# Step 26 - Output a Map

Append:

```hcl
output "common_tags" {

  value = var.common_tags

}
```

Expected Output

```text
common_tags = {
  "Environment" = "Dev"
  "Owner" = "Alfia"
  "Project" = "Terraform Zero To Hero"
}
```

---

# Production Use Cases

Outputs are frequently used by:

Terraform

↓

Creates Infrastructure

↓

Outputs Values

↓

Jenkins

↓

Ansible

↓

Application Deployment

Example:

Terraform creates an EC2 instance.

Terraform outputs:

```
54.201.xx.xx
```

Ansible reads:

```
54.201.xx.xx
```

and automatically configures the server.

No manual work is required.

---

# Best Practices

Always:

- Output useful information such as IDs, IPs, DNS names, and endpoints.
- Mark passwords, secrets, and tokens as `sensitive`.
- Use descriptive output names.
- Keep outputs focused on information other tools or users need.

Avoid:

- Displaying passwords or secrets in plain text.
- Creating outputs that are never used.
- Giving outputs vague names such as `output1`.

---

# Common Mistakes

## Forgetting Sensitive

Wrong:

```hcl
output "password" {

  value = var.database_password

}
```

Correct:

```hcl
output "password" {

  value = var.database_password

  sensitive = true

}
```

---

## Wrong Output Reference

Wrong:

```hcl
value = var.region
```

Correct:

```hcl
value = var.aws_region
```

---

# Lab Verification Checklist

Verify that:

- ✅ Outputs display correctly.
- ✅ Sensitive outputs are hidden.
- ✅ List outputs display correctly.
- ✅ Map outputs display correctly.
- ✅ `terraform output` works.
- ✅ `terraform output -json` works.
- ✅ `terraform fmt` succeeds.
- ✅ `terraform validate` succeeds.

---

# Summary

You have learned:

- Sensitive Outputs
- Output data types
- Production use cases
- Best practices
- Common mistakes

---

# End of Part 3

In Part 4, we'll complete Lab 04 with:

- Output Dependencies
- Interview Questions
- Assignments
- Git Commands
- AWS Examples
- Cleanup

---

# Part 4 - Output Dependencies, Cleanup and Interview Preparation

Congratulations!

You have learned how Terraform Outputs work.

Now let's understand what happens behind the scenes and how Outputs are used in real production environments.

---

# Output Dependencies

Terraform automatically understands dependencies.

For example:

```
EC2 Instance

↓

Public IP

↓

Output
```

Terraform knows that it cannot display the Public IP until the EC2 instance has been created.

You do not need to manually specify this dependency.

Example:

```hcl
output "public_ip" {

  value = aws_instance.web.public_ip

}
```

Terraform first creates:

```
aws_instance.web
```

Then retrieves:

```
public_ip
```

Finally displays it.

---

# Dependency Flow

```
terraform apply

        │

        ▼

Create EC2 Instance

        │

        ▼

Receive Public IP

        │

        ▼

Store in terraform.tfstate

        │

        ▼

Display Output
```

Terraform automatically determines the correct execution order.

---

# Outputs After Resource Creation

In upcoming labs, your Outputs will look like this.

Example:

```hcl
output "instance_id" {

  description = "EC2 Instance ID"

  value = aws_instance.web.id

}

output "public_ip" {

  description = "Public IP Address"

  value = aws_instance.web.public_ip

}

output "private_ip" {

  description = "Private IP Address"

  value = aws_instance.web.private_ip

}

output "availability_zone" {

  description = "Availability Zone"

  value = aws_instance.web.availability_zone

}
```

These Outputs will appear automatically after:

```bash
terraform apply
```

Example:

```
Outputs

instance_id = i-0123456789abcdef0

public_ip = 43.204.xx.xx

private_ip = 172.31.xx.xx

availability_zone = ap-south-1a
```

---

# Production Example

Imagine Terraform creates:

```
Application Load Balancer
```

Terraform outputs:

```
alb_dns_name
```

A deployment pipeline can use that Output to:

- Update Route 53
- Configure monitoring
- Notify developers
- Run automated tests

Outputs often become the bridge between Terraform and other automation tools.

---

# How Outputs Help Automation

```
Terraform

     │

Creates Infrastructure

     │

Outputs

     │

Jenkins Pipeline

     │

Reads Output

     │

Runs Ansible

     │

Deploys Application

     │

Application Ready
```

Outputs eliminate manual copy-and-paste steps.

---

# Cleanup

This lab did not create any AWS resources.

However, after future labs, you should always clean up resources you no longer need.

To remove infrastructure:

```bash
terraform destroy
```

Terraform will display a plan showing what will be deleted.

Type:

```
yes
```

Terraform deletes the infrastructure and updates the state file.

---

# Common Output Commands

Initialize the project:

```bash
terraform init
```

Validate the configuration:

```bash
terraform validate
```

Format the configuration:

```bash
terraform fmt
```

Apply the configuration:

```bash
terraform apply
```

Display all Outputs:

```bash
terraform output
```

Display one Output:

```bash
terraform output project_name
```

Display Outputs as JSON:

```bash
terraform output -json
```

Destroy infrastructure:

```bash
terraform destroy
```

---

# Lab Verification Checklist

Verify that:

- ✅ Outputs display correctly.
- ✅ Sensitive Outputs are hidden.
- ✅ JSON Output works.
- ✅ Single Output works.
- ✅ Multiple Outputs work.
- ✅ Validation succeeds.
- ✅ Formatting succeeds.

---

# Assignment

## Task 1

Create an Output named:

```text
course_name
```

Value:

```text
Terraform Zero to Hero
```

---

## Task 2

Create an Output named:

```text
student_name
```

Use the value from a variable instead of hardcoding it.

---

## Task 3

Create an Output named:

```text
deployment_environment
```

Display:

```
dev
```

using the existing variable.

---

## Task 4

Run:

```bash
terraform output
```

Verify all Outputs.

---

## Task 5

Run:

```bash
terraform output -json
```

Observe the JSON structure.

---

# Interview Questions

## Beginner

1. What is a Terraform Output?
2. Why are Outputs useful?
3. Where are Outputs defined?
4. Which command displays Outputs?
5. Which command displays Outputs in JSON?

---

## Intermediate

6. Where does Terraform store Outputs?
7. What is a Sensitive Output?
8. Why should passwords be marked as sensitive?
9. Does `terraform output` contact AWS?
10. How do Outputs help automation?

---

## Advanced

11. How can Outputs be used between Terraform modules?
12. How are Outputs used in CI/CD pipelines?
13. Why is the Terraform state file important?
14. What happens if the state file is deleted?
15. Why should state files be stored securely?

---

# Best Practices

Always:

- Use meaningful Output names.
- Output values that are useful after deployment.
- Mark secrets as sensitive.
- Protect the Terraform state file.
- Keep Outputs simple and focused.

Avoid:

- Outputting passwords or secrets in plain text.
- Creating unnecessary Outputs.
- Using unclear names.

---

# Summary

After completing this lab, you can:

- Create Outputs.
- Display single and multiple Outputs.
- Display JSON Outputs.
- Create Sensitive Outputs.
- Understand how Outputs are stored.
- Explain how Outputs support automation.
- Follow Output best practices.

---

# Save Your Work

Run:

```bash
git status

git add .

git commit -m "Complete Lab 04 - Terraform Outputs"

git push origin main
```

---

# Congratulations!

You have successfully completed:

- ✅ Lab 01 – Install Terraform
- ✅ Lab 02 – Configure AWS Provider
- ✅ Lab 03 – Terraform Variables
- ✅ Lab 04 – Terraform Outputs

You now have a solid Terraform foundation.

---

# Next Lab

## Lab 05 – Terraform Locals

In the next lab, you will learn:

- What Locals are
- Locals vs Variables
- Naming conventions
- String interpolation
- Local expressions
- Tag standardization
- Production naming standards
- Best practices
- Real AWS examples

By the end of Lab 05, you'll be writing Terraform code that follows the same patterns used in professional DevOps projects.