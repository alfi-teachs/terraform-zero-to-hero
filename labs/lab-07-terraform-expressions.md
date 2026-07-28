# Lab 07 - Terraform Expressions and Functions

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 07 |
| Lab Name | Terraform Expressions and Functions |
| Difficulty | Intermediate |
| Duration | 1 Hour |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will understand:

- What Terraform Expressions are
- How Terraform makes decisions
- Conditional expressions
- For expressions
- Splat expressions
- Built-in functions
- Dynamic blocks
- Real production examples

---

# Real World Scenario

Imagine you manage AWS infrastructure for:

```
Development

Testing

Production
```

You need different settings:

Development:

```
2 servers
small instances
no backup
```

Production:

```
10 servers
large instances
backup enabled
```

Instead of creating separate Terraform files,

Terraform can make decisions automatically.

This is done using Expressions.

---

# What Are Terraform Expressions?

Expressions are instructions that tell Terraform how to calculate values.

Example:

```hcl
var.environment == "prod"
```

Terraform asks:

```
Is environment production?
```

The answer decides what Terraform does next.

---

# Expression Types

Terraform supports:

```
1. Literal expressions

2. Reference expressions

3. Operators

4. Conditional expressions

5. For expressions

6. Function expressions
```

---

# Folder Structure

Create:

```
terraform/
└── lab-07/
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
mkdir -p terraform/lab-07
mkdir -p images/lab-07

touch labs/lab-07-terraform-expressions.md
```

Enter folder:

```bash
cd terraform/lab-07
```

---

# Step 2 - Copy Previous Files

Copy:

```bash
cp ../lab-06/versions.tf .
cp ../lab-06/provider.tf .
cp ../lab-06/variables.tf .
cp ../lab-06/terraform.tfvars .
cp ../lab-06/locals.tf .
cp ../lab-06/data.tf .
cp ../lab-06/outputs.tf .
```

Create:

```bash
touch main.tf
touch README.md
```

---

# Part 1 - Conditional Expressions

A conditional expression works like:

```
condition ? true_value : false_value
```

Example:

```hcl
var.environment == "prod" ? "large" : "small"
```

Meaning:

If:

```
environment = prod
```

return:

```
large
```

Otherwise:

```
small
```

---

# Step 3 - Create Conditional Local

Open:

```
locals.tf
```

Add:

```hcl
locals {

  instance_size = var.environment == "prod" ? "t3.large" : "t2.micro"

}
```

---

# Step 4 - Create Output

Open:

```
outputs.tf
```

Add:

```hcl
output "instance_size" {

  description = "Instance Size Based On Environment"

  value = local.instance_size

}
```

---

# Step 5 - Test

Run:

```bash
terraform fmt
```

Then:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```

Apply:

```bash
terraform apply
```

Expected:

```
instance_size = t2.micro
```

because your environment is currently:

```
dev
```

---

# Step 6 - Change Environment

Open:

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

Output:

```
instance_size = t3.large
```

Terraform changed the value automatically.

---

# Behind The Scenes

```
Variable

   |

   ▼

Condition Check

   |

   ▼

Choose Value

   |

   ▼

Local

   |

   ▼

Resource
```

---

# Part 2 - Operators

Terraform supports operators.

---

# Comparison Operators

Equal:

```hcl
==
```

Not Equal:

```hcl
!=
```

Greater Than:

```hcl
>
```

Less Than:

```hcl
<
```

---

# Logical Operators

AND:

```hcl
&&
```

OR:

```hcl
||
```

NOT:

```hcl
!
```

---

# Example

```hcl
var.environment == "prod" && var.enable_monitoring
```

Meaning:

Both conditions must be true.

---

# Real Production Example

Enable monitoring only for production:

```hcl
monitoring_enabled =
var.environment == "prod" ? true : false
```

Development:

```
false
```

Production:

```
true
```

---

# Assignment

Create a Local:

```hcl
delete_protection =
var.environment == "prod" ? true : false
```

Create an Output.

Test:

```
dev
```

and:

```
prod
```

---

# Summary

In Part 1 you learned:

- Terraform Expressions
- Conditional expressions
- Operators
- Environment-based decisions

---

# End of Part 1

Next Part:

- For Expressions
- Transforming Lists
- Transforming Maps
- Splat Expressions
- Real AWS examples

---

# Part 2 - For Expressions and Splat Expressions

Congratulations!

You learned how Terraform can make decisions using conditions.

Now we will learn how Terraform can process multiple values.

In real projects, you rarely manage only one resource.

You manage:

- Multiple EC2 instances
- Multiple subnets
- Multiple security groups
- Multiple environments

Terraform needs a way to work with collections of data.

This is where:

- For Expressions
- Splat Expressions

are used.

---

# What is a For Expression?

A For Expression allows Terraform to loop through a collection and create a new value.

Think of it like a loop.

Example:

Input:

```
[
 "dev",
 "test",
 "prod"
]
```

Terraform processes each item:

```
dev

test

prod
```

and creates a new output.

---

# Basic Syntax

```hcl
[
for item in collection :
result
]
```

Meaning:

```
For every item

inside collection

return result
```

---

# Example 1 - Convert List Values

Create a Local.

Open:

```
locals.tf
```

Add:

```hcl
environment_upper = [
  for env in [
    "dev",
    "test",
    "prod"
  ] :
  upper(env)
]
```

---

# Understanding This

Input:

```
dev

test

prod
```

Terraform applies:

```
upper()
```

Result:

```
DEV

TEST

PROD
```

---

# Add Output

Open:

```
outputs.tf
```

Add:

```hcl
output "environment_upper" {

  value = local.environment_upper

}
```

---

# Test

Run:

```bash
terraform fmt
```

Validate:

```bash
terraform validate
```

Apply:

```bash
terraform apply
```

Output:

```
environment_upper = [
 "DEV",
 "TEST",
 "PROD"
]
```

---

# Real AWS Example

Imagine you have:

```
subnet-a

subnet-b

subnet-c
```

You can transform them:

Input:

```
subnet names
```

Output:

```
subnet tags
```

---

# For Expression with Maps

Terraform can also loop through maps.

Example:

```hcl
locals {

  server_types = {

    web = "t2.micro"

    app = "t3.micro"

    database = "t3.large"

  }

}
```

---

# Convert Map to List

Add:

```hcl
locals {

  server_names = [
    for name, type in local.server_types :
    name
  ]

}
```

Result:

```
[
 "web",
 "app",
 "database"
]
```

---

# Add Output

```hcl
output "server_names" {

  value = local.server_names

}
```

---

# Apply

Run:

```bash
terraform apply
```

Output:

```
server_names = [
 "web",
 "app",
 "database"
]
```

---

# Filtering With For Expressions

You can also filter values.

Syntax:

```hcl
[
for item in collection :
item
if condition
]
```

---

# Example

Create:

```hcl
locals {

  production_only = [
    for env in [
      "dev",
      "test",
      "prod"
    ] :
    env
    if env == "prod"
  ]

}
```

Result:

```
[
 "prod"
]
```

---

# Add Output

```hcl
output "production_only" {

  value = local.production_only

}
```

---

# What Are Splat Expressions?

A Splat Expression extracts the same attribute from multiple resources.

Instead of writing:

```hcl
aws_instance.web[0].public_ip

aws_instance.web[1].public_ip

aws_instance.web[2].public_ip
```

You can write:

```hcl
aws_instance.web[*].public_ip
```

Terraform returns all values.

---

# Example

Imagine three EC2 instances:

```
EC2-1

Public IP

10.0.0.1


EC2-2

Public IP

10.0.0.2


EC2-3

Public IP

10.0.0.3
```

Splat:

```hcl
aws_instance.web[*].public_ip
```

Returns:

```
[
10.0.0.1,
10.0.0.2,
10.0.0.3
]
```

---

# Why Splat Expressions Matter

They are useful for:

- Getting multiple IP addresses
- Getting multiple IDs
- Getting multiple DNS names
- Working with count-created resources

---

# Production Example

Terraform creates:

```
5 EC2 instances
```

You need:

```
All private IP addresses
```

Instead of manually listing them:

```hcl
instance1.private_ip
instance2.private_ip
instance3.private_ip
```

Use:

```hcl
aws_instance.web[*].private_ip
```

---

# Behind The Scenes

For Expression:

```
Collection

    |

    ▼

Loop Through Items

    |

    ▼

Transform Values

    |

    ▼

Return New Collection
```

Splat Expression:

```
Multiple Resources

    |

    ▼

Extract Same Attribute

    |

    ▼

Return List
```

---

# Common Mistakes

## Mistake 1

Wrong:

```hcl
for env in env
```

Correct:

```hcl
for env in environments
```

---

## Mistake 2

Using Splat on a single resource.

Wrong:

```hcl
aws_instance.web[*].id
```

when only one instance exists.

---

## Mistake 3

Forgetting the colon:

Wrong:

```hcl
for item in list item
```

Correct:

```hcl
for item in list : item
```

---

# Lab Verification Checklist

Verify:

✅ For expressions work  
✅ Map iteration works  
✅ Filtering works  
✅ Splat syntax understood  
✅ Outputs display correctly  
✅ terraform validate succeeds  

---

# Summary

You learned:

- For Expressions
- Looping through lists
- Looping through maps
- Filtering collections
- Splat Expressions
- Extracting values from multiple resources

---

# End of Part 2

Next Part:

# Part 3 - Dynamic Blocks and Advanced Expressions

You will learn:

- Dynamic blocks
- Nested configuration generation
- Real Security Group examples
- Advanced Terraform patterns

---

# Part 3 - Dynamic Blocks and Advanced Expressions

Congratulations!

You now understand:

- Conditional expressions
- Operators
- For expressions
- Splat expressions

Now we will learn Dynamic Blocks.

---

# What Are Dynamic Blocks?

A Dynamic Block allows Terraform to create repeated nested blocks automatically.

Normally, you write:

```hcl
ingress {

  from_port = 80

  to_port = 80

}

ingress {

  from_port = 443

  to_port = 443

}
```

This becomes difficult when you have many rules.

Dynamic blocks solve this problem.

---

# Real World Example

A company needs a Security Group:

Allow:

```
HTTP 80

HTTPS 443

SSH 22
```

Without Dynamic Blocks:

You manually create:

```
3 ingress blocks
```

With Dynamic Blocks:

You provide a list:

```
80

443

22
```

Terraform creates the blocks automatically.

---

# Dynamic Block Syntax

Basic structure:

```hcl
dynamic "block_name" {

  for_each = collection

  content {

    attribute = block_name.value

  }

}
```

---

# Architecture

```
Variable

    |

    ▼

List of Values

    |

    ▼

Dynamic Block

    |

    ▼

Multiple Resource Blocks

```

---

# Step 29 - Create Security Group Example

Open:

```
variables.tf
```

Add:

```hcl
variable "allowed_ports" {

  description = "Ports allowed in security group"

  type = list(number)

  default = [
    22,
    80,
    443
  ]

}
```

---

# Step 30 - Create Security Group Resource

Open:

```
main.tf
```

Add:

```hcl
resource "aws_security_group" "web" {

  name = local.security_group_name


  dynamic "ingress" {

    for_each = var.allowed_ports


    content {

      from_port = ingress.value

      to_port = ingress.value

      protocol = "tcp"

      cidr_blocks = [
        "0.0.0.0/0"
      ]

    }

  }


  egress {

    from_port = 0

    to_port = 0

    protocol = "-1"

    cidr_blocks = [
      "0.0.0.0/0"
    ]

  }


  tags = local.common_tags

}
```

---

# Understanding Dynamic Block

This part:

```hcl
for_each = var.allowed_ports
```

loops through:

```
[
22,
80,
443
]
```

Terraform creates:

First:

```
ingress 22
```

Second:

```
ingress 80
```

Third:

```
ingress 443
```

---

# Generated Terraform Result

Terraform internally creates:

```hcl
ingress {

from_port = 22

to_port = 22

}


ingress {

from_port = 80

to_port = 80

}


ingress {

from_port = 443

to_port = 443

}
```

You only wrote one block.

---

# Step 31 - Add Output

Open:

```
outputs.tf
```

Add:

```hcl
output "security_group_id" {

  description = "Security Group ID"

  value = aws_security_group.web.id

}
```

---

# Step 32 - Format

Run:

```bash
terraform fmt
```

---

# Step 33 - Validate

Run:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```

---

# Step 34 - Plan

Run:

```bash
terraform plan
```

You should see:

```
aws_security_group.web
```

being created.

---

# Step 35 - Apply

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
Security Group

Inbound Rules:

22

80

443
```

---

# Verify in AWS Console

Go to:

```
AWS Console

↓

EC2

↓

Security Groups
```

You should see:

```
terraform-zero-to-hero-dev-sg
```

Rules:

```
SSH 22

HTTP 80

HTTPS 443
```

---

# Real Production Usage

Dynamic blocks are commonly used for:

## Security Groups

Example:

```
Multiple ingress rules
```

---

## IAM Policies

Example:

```
Multiple permissions
```

---

## Load Balancers

Example:

```
Multiple listener rules
```

---

## Route Tables

Example:

```
Multiple routes
```

---

# Dynamic Block vs For Expression

They are different.

## For Expression

Creates values.

Example:

```
List

Map

String
```

Example:

```hcl
[
for item in list :
item
]
```

---

## Dynamic Block

Creates Terraform configuration blocks.

Example:

```hcl
dynamic "ingress"
```

Creates:

```
multiple ingress blocks
```

---

# Common Mistakes

## Mistake 1

Using dynamic block for simple values.

Wrong:

```hcl
dynamic "name"
```

Dynamic blocks are for nested blocks.

---

## Mistake 2

Wrong iterator reference.

Incorrect:

```hcl
value
```

Correct:

```hcl
ingress.value
```

---

## Mistake 3

Forgetting collection type.

Dynamic blocks need:

```
list

set

map
```

---

# Lab Verification Checklist

Verify:

✅ Dynamic block created  
✅ Security Group created  
✅ Multiple rules generated  
✅ Variables used correctly  
✅ Output displays ID  
✅ Terraform validation passes  

---

# Summary

You learned:

- Dynamic Blocks
- Generating repeated configuration
- Security Group example
- Dynamic block vs For expression
- Production use cases

---

# End of Lab 07

You have completed:

✅ Conditional Expressions  
✅ Operators  
✅ For Expressions  
✅ Splat Expressions  
✅ Dynamic Blocks  

---

# Git Save

Run:

```bash
git status

git add .

git commit -m "Complete Lab 07 - Terraform Expressions"

git push origin main
```

---

# Next Lab

## Lab 08 - Terraform Meta Arguments

You will learn:

- count
- for_each
- depends_on
- lifecycle
- Creating multiple resources
- Resource dependencies
- Production patterns

After Lab 08, you will be able to create multiple EC2 instances and infrastructure components automatically.