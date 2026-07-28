# Lab 08 - Terraform Meta Arguments

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 08 |
| Lab Name | Terraform Meta Arguments |
| Difficulty | Intermediate |
| Duration | 2 Hours |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will understand:

- What Terraform Meta Arguments are
- Difference between count and for_each
- Creating multiple resources
- Managing dependencies
- Controlling resource lifecycle
- Production usage patterns

---

# What Are Meta Arguments?

Meta Arguments are special Terraform arguments that control how resources behave.

They are not AWS-specific.

Terraform provides them.

Examples:

```hcl
resource "aws_instance" "web" {

  count = 3

}
```

Terraform understands:

```
Create 3 instances
```

---

# Meta Arguments We Learn

## 1. count

Creates multiple identical resources.

Example:

```
3 EC2 instances
```

---

## 2. for_each

Creates resources from a collection.

Example:

```
web server

database server

application server
```

---

## 3. depends_on

Creates explicit dependencies.

Example:

```
Create VPC first

Then create EC2
```

---

## 4. lifecycle

Controls resource changes.

Example:

```
Prevent accidental deletion
```

---

# Architecture

```
Terraform Configuration

        |

        ▼

Meta Arguments

        |

        ▼

Resource Creation Logic

        |

        ▼

AWS Infrastructure
```

---

# Folder Structure

Create:

```
terraform/
└── lab-08/
    ├── versions.tf
    ├── provider.tf
    ├── variables.tf
    ├── terraform.tfvars
    ├── locals.tf
    ├── data.tf
    ├── main.tf
    ├── outputs.tf
    └── README.md
```

---

# Step 1 - Create Lab Folder

Go to your project:

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create:

```bash
mkdir -p terraform/lab-08

mkdir -p images/lab-08

touch labs/lab-08-terraform-meta-arguments.md
```

Enter:

```bash
cd terraform/lab-08
```

---

# Step 2 - Copy Previous Files

Run:

```bash
cp ../lab-07/versions.tf .
cp ../lab-07/provider.tf .
cp ../lab-07/variables.tf .
cp ../lab-07/terraform.tfvars .
cp ../lab-07/locals.tf .
cp ../lab-07/data.tf .
cp ../lab-07/outputs.tf .
```

Create:

```bash
touch main.tf

touch README.md
```

---

# Part 1 - Count Meta Argument

## What is count?

`count` creates multiple copies of the same resource.

Example:

```hcl
count = 3
```

Terraform creates:

```
resource[0]

resource[1]

resource[2]
```

---

# Real World Example

Company needs:

```
3 Web Servers
```

Instead of writing:

```
EC2-1

EC2-2

EC2-3
```

Use:

```hcl
count = 3
```

---

# Step 3 - Create Variable

Open:

```
variables.tf
```

Add:

```hcl
variable "instance_count" {

  description = "Number of EC2 instances"

  type = number

  default = 3

}
```

---

# Step 4 - Create Multiple EC2 Instances

Open:

```
main.tf
```

Add:

```hcl
resource "aws_instance" "web" {

  count = var.instance_count


  ami = data.aws_ami.amazon_linux.id


  instance_type = var.instance_type


  tags = {

    Name = "${local.ec2_name}-${count.index}"

    Environment = var.environment

  }

}
```

---

# Understanding count.index

Terraform automatically creates indexes.

Example:

```
count = 3
```

Creates:

```
count.index = 0

count.index = 1

count.index = 2
```

---

# Resource Names

Terraform creates:

```
terraform-zero-to-hero-dev-ec2-0

terraform-zero-to-hero-dev-ec2-1

terraform-zero-to-hero-dev-ec2-2
```

---

# Step 5 - Add Outputs

Open:

```
outputs.tf
```

Add:

```hcl
output "instance_ids" {

  description = "All EC2 Instance IDs"

  value = aws_instance.web[*].id

}


output "public_ips" {

  description = "All Public IPs"

  value = aws_instance.web[*].public_ip

}
```

---

# Step 6 - Format

Run:

```bash
terraform fmt
```

---

# Step 7 - Validate

Run:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```

---

# Step 8 - Plan

Run:

```bash
terraform plan
```

Terraform shows:

```
aws_instance.web[0]

aws_instance.web[1]

aws_instance.web[2]
```

---

# Step 9 - Apply

Run:

```bash
terraform apply
```

Type:

```
yes
```

Terraform creates:

```
3 EC2 instances
```

---

# Verify

AWS Console:

```
EC2

↓

Instances
```

You should see:

```
terraform-zero-to-hero-dev-ec2-0

terraform-zero-to-hero-dev-ec2-1

terraform-zero-to-hero-dev-ec2-2
```

---

# Behind The Scenes

Terraform process:

```
Read count value

        |

        ▼

Create Indexes

        |

        ▼

Create Multiple Resources

        |

        ▼

Store Each Resource In State
```

---

# count Advantages

Good for:

```
Same resource

Same configuration

Different index
```

Examples:

- Multiple identical servers
- Multiple workers
- Development environments

---

# count Limitations

Problem:

Changing the middle item can recreate resources.

Example:

Before:

```
server[0]

server[1]

server[2]
```

Remove:

```
server[1]
```

Terraform may shift indexes.

This is why we use:

```
for_each
```

for complex resources.

---

# Summary

You learned:

- What Meta Arguments are
- count argument
- count.index
- Creating multiple EC2 instances
- Splat outputs

---

# End of Part 1

Next Part:

# Part 2 - for_each Meta Argument

You will learn:

- count vs for_each
- Creating named resources
- Using maps
- Production recommended patterns
- Multiple EC2 roles example

---

# Part 2 - for_each Meta Argument

Now we will learn `for_each`.

---

# What is for_each?

`for_each` creates multiple resources using a:

- Map
- Set

Instead of creating:

```
server-0

server-1

server-2
```

It creates:

```
web

application

database
```

Each resource gets its own name.

---

# count vs for_each

| count | for_each |
|------|----------|
| Uses numbers | Uses names |
| Uses index | Uses keys |
| Good for identical resources | Good for different resources |
| server[0] | server["web"] |
| Easier but less flexible | More production friendly |

---

# Example Scenario

A company needs three servers:

```
web-server

app-server

database-server
```

Each server has a different instance type.

Example:

```
web-server

t2.micro


app-server

t3.micro


database-server

t3.large
```

`count` cannot handle this easily.

`for_each` is designed for this.

---

# Step 10 - Create Server Map Variable

Open:

```
variables.tf
```

Add:

```hcl
variable "servers" {

  description = "Server configuration"

  type = map(string)

  default = {

    web = "t2.micro"

    app = "t3.micro"

    database = "t3.large"

  }

}
```

---

# Understanding The Map

Terraform sees:

```
key              value


web              t2.micro


app              t3.micro


database         t3.large
```

The key becomes the resource name.

---

# Step 11 - Create EC2 with for_each

Open:

```
main.tf
```

Add:

```hcl
resource "aws_instance" "servers" {

  for_each = var.servers


  ami = data.aws_ami.amazon_linux.id


  instance_type = each.value


  tags = {

    Name = "${local.project_prefix}-${each.key}"

    Role = each.key

    Environment = var.environment

  }

}
```

---

# Understanding each.key

Example:

```
web
```

becomes:

```
terraform-zero-to-hero-dev-web
```

---

# Understanding each.value

Example:

```
t2.micro
```

becomes:

```
instance_type = t2.micro
```

---

# Terraform Creates

```
aws_instance.servers["web"]

aws_instance.servers["app"]

aws_instance.servers["database"]
```

---

# Step 12 - Add Outputs

Open:

```
outputs.tf
```

Add:

```hcl
output "server_ids" {

  description = "EC2 IDs"

  value = {
    for name, instance in aws_instance.servers :
    name => instance.id
  }

}


output "server_public_ips" {

  description = "Public IP Addresses"

  value = {
    for name, instance in aws_instance.servers :
    name => instance.public_ip
  }

}
```

---

# Understanding Output For Expression

Terraform receives:

```
web  -> instance id

app  -> instance id

database -> instance id
```

Output becomes:

```text
{
 web = i-xxxx

 app = i-yyyy

 database = i-zzzz
}
```

---

# Step 13 - Format

Run:

```bash
terraform fmt
```

---

# Step 14 - Validate

Run:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```

---

# Step 15 - Plan

Run:

```bash
terraform plan
```

Terraform shows:

```
aws_instance.servers["web"]

aws_instance.servers["app"]

aws_instance.servers["database"]
```

---

# Step 16 - Apply

Run:

```bash
terraform apply
```

Type:

```
yes
```

Terraform creates:

```
3 EC2 Instances
```

---

# Verify in AWS

Go to:

```
AWS Console

↓

EC2

↓

Instances
```

You should see:

```
terraform-zero-to-hero-dev-web

terraform-zero-to-hero-dev-app

terraform-zero-to-hero-dev-database
```

---

# Why for_each is Better

Imagine you remove:

```
app-server
```

With count:

```
server[1] removed

server[2] may become server[1]
```

Terraform may recreate resources.

With for_each:

Before:

```
web

app

database
```

Remove:

```
app
```

Terraform only removes:

```
app
```

The others remain unchanged.

---

# Real Production Usage

for_each is commonly used for:

## Multiple EC2 Roles

```
web

app

database
```

---

## Multiple Buckets

```
logs

backup

application
```

---

## Multiple IAM Users

```
developer

admin

auditor
```

---

## Multiple Security Groups

```
web

database

internal
```

---

# Common Mistakes

## Mistake 1

Using list instead of map.

Wrong:

```hcl
[
"web",
"app"
]
```

Better:

```hcl
{
web = "t2.micro"

app = "t3.micro"
}
```

---

## Mistake 2

Using count.index with for_each.

Wrong:

```hcl
count.index
```

Correct:

```hcl
each.key
```

or:

```hcl
each.value
```

---

# Lab Verification Checklist

Verify:

✅ for_each creates multiple resources  
✅ each.key works  
✅ each.value works  
✅ Outputs display maps  
✅ Resources have stable names  
✅ Terraform state contains named resources  

---

# Summary

You learned:

- for_each Meta Argument
- Maps with Terraform
- each.key
- each.value
- count vs for_each
- Production resource patterns

---

# End of Part 2

Next Part:

# Part 3 - depends_on and lifecycle

You will learn:

- Resource dependencies
- Explicit dependencies
- Preventing accidental deletion
- Ignoring changes
- Production safety controls

---

# Part 3 - depends_on and lifecycle Meta Arguments

Congratulations!

You learned:

- count
- for_each

Now we learn two more powerful Meta Arguments:

```
depends_on

lifecycle
```

---

# 1. depends_on Meta Argument

## What is depends_on?

Terraform automatically understands many dependencies.

Example:

```
VPC

↓

Subnet

↓

EC2
```

Terraform knows:

```
Create VPC first

Then Subnet

Then EC2
```

But sometimes Terraform cannot understand a dependency.

In that case we use:

```hcl
depends_on
```

---

# Real World Example

Imagine:

```
IAM Role

↓

EC2 Instance
```

Terraform may not know that EC2 requires the IAM role.

We manually tell Terraform:

```
Create IAM Role first

Then create EC2
```

---

# depends_on Syntax

Example:

```hcl
resource "aws_instance" "web" {

  depends_on = [
    aws_iam_role.example
  ]

}
```

Meaning:

```
Wait until IAM Role exists
```

---

# Example Architecture

Without depends_on:

```
Terraform

   |

   ▼

Create Resources
(random order)
```

With depends_on:

```
IAM Role

   |

   ▼

EC2 Instance
```

---

# Step 17 - Create Dependency Example

Open:

```
main.tf
```

Add:

```hcl
resource "aws_s3_bucket" "example" {

  bucket = "${local.project_prefix}-bucket"

}


resource "aws_instance" "dependent_server" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = var.instance_type


  depends_on = [

    aws_s3_bucket.example

  ]


  tags = {

    Name = "dependent-server"

  }

}
```

---

# Understanding This

Terraform will do:

Step 1:

```
Create S3 Bucket
```

Step 2:

```
Create EC2 Instance
```

because EC2 depends on S3.

---

# Important Note

Use depends_on only when Terraform cannot automatically understand the relationship.

Do not add it everywhere.

Terraform already handles most dependencies automatically.

---

# 2. lifecycle Meta Argument

Now we learn lifecycle.

Lifecycle controls what Terraform does during:

- Creation
- Updates
- Replacement
- Deletion

---

# lifecycle Options

The important ones are:

```
create_before_destroy

prevent_destroy

ignore_changes
```

---

# Option 1 - create_before_destroy

Normally:

Terraform:

```
Delete old resource

↓

Create new resource
```

Problem:

Your application may have downtime.

---

With:

```hcl
create_before_destroy = true
```

Terraform does:

```
Create new resource

↓

Move traffic

↓

Delete old resource
```

---

# Example

```hcl
lifecycle {

  create_before_destroy = true

}
```

---

# Real Production Usage

Used with:

- Load Balancers
- Servers
- Auto Scaling Groups

to reduce downtime.

---

# Option 2 - prevent_destroy

This protects important resources.

Example:

Production database.

You don't want someone running:

```bash
terraform destroy
```

and deleting it.

---

Example:

```hcl
lifecycle {

  prevent_destroy = true

}
```

---

# What Happens?

If someone tries:

```bash
terraform destroy
```

Terraform shows:

```
Error:

Resource cannot be destroyed
because prevent_destroy is enabled
```

---

# Resources Usually Protected

Companies protect:

- Production databases
- S3 buckets containing data
- Critical networking components

---

# Option 3 - ignore_changes

Sometimes AWS changes values automatically.

Example:

AWS automatically modifies:

```
Tags

Metadata

Settings
```

Terraform sees:

```
Difference detected
```

and tries to change it back.

---

Use:

```hcl
lifecycle {

  ignore_changes = [
    tags
  ]

}
```

Terraform ignores tag changes.

---

# Example Resource

```hcl
resource "aws_instance" "web" {


  ami = data.aws_ami.amazon_linux.id


  instance_type = var.instance_type


  lifecycle {

    ignore_changes = [

      tags

    ]

  }

}
```

---

# Production Example

A company has:

Terraform:

```
Environment = Production
```

Security team adds:

```
SecurityReviewed = Yes
```

AWS tags become:

```
Environment = Production

SecurityReviewed = Yes
```

Terraform normally wants to remove the extra tag.

With:

```hcl
ignore_changes
```

Terraform leaves it alone.

---

# Complete Meta Argument Flow

```
count

↓

Create Multiple Resources


for_each

↓

Create Named Resources


depends_on

↓

Control Creation Order


lifecycle

↓

Control Resource Changes
```

---

# Common Mistakes

## Mistake 1

Using prevent_destroy everywhere.

Problem:

You cannot clean up your environment.

Use only for critical resources.

---

## Mistake 2

Adding depends_on everywhere.

Problem:

Terraform becomes slower.

Only use when required.

---

## Mistake 3

Ignoring too many changes.

Problem:

Terraform loses control of resources.

---

# Lab Verification Checklist

Verify:

✅ depends_on understood  
✅ Resource ordering understood  
✅ create_before_destroy understood  
✅ prevent_destroy understood  
✅ ignore_changes understood  
✅ Production use cases understood  

---

# Interview Questions

## Beginner

1. What are Terraform Meta Arguments?
2. What does count do?
3. What does for_each do?
4. What is depends_on?
5. What is lifecycle?

---

## Intermediate

6. Difference between count and for_each?
7. When should you use depends_on?
8. What does prevent_destroy do?
9. What is ignore_changes?
10. Why use create_before_destroy?

---

## Advanced

11. Why is for_each preferred in production?
12. How does Terraform automatically detect dependencies?
13. When can lifecycle rules cause problems?
14. Why should prevent_destroy be used carefully?
15. How do Meta Arguments improve infrastructure management?

---

# Summary

You completed:

✅ count  
✅ for_each  
✅ depends_on  
✅ lifecycle  
✅ Production safety patterns  

---

# Git Save

Run:

```bash
git status

git add .

git commit -m "Complete Lab 08 - Terraform Meta Arguments"

git push origin main
```

---

# Next Lab

## Lab 09 - Terraform Lifecycle and State Management

You will learn:

- Terraform state file deeply
- Remote state
- State locking
- Import resources
- Terraform refresh
- State commands
- Production state management

After Lab 09, you will be ready for the **Lab 10 EC2 Mini Project**, where we combine everything:

Variables + Locals + Data Sources + Meta Arguments + State + AWS EC2.