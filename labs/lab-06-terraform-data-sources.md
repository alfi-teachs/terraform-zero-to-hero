# Lab 06 - Terraform Data Sources

---

# Lab Information

| Item | Details |
|------|---------|
| Lab Number | 06 |
| Lab Name | Terraform Data Sources |
| Difficulty | Beginner |
| Duration | 1 Hour |
| Cloud | AWS |

---

# Learning Objectives

By the end of this lab you will understand:

- What Terraform Data Sources are
- Difference between Resources and Data Sources
- How Terraform reads existing AWS information
- How to fetch latest Amazon Linux AMI
- How to fetch Availability Zones
- How to use Data Sources with Resources
- Production use cases

---

# Real World Scenario

Imagine you are creating an EC2 instance.

Normally you might write:

```hcl
resource "aws_instance" "web" {

  ami = "ami-0123456789"

}
```

Problem:

AMI IDs are different in every AWS Region.

Example:

Mumbai:

```
ami-0abcdef123
```

Virginia:

```
ami-0987654321
```

Tomorrow AWS releases a new Amazon Linux version.

Your Terraform code becomes outdated.

---

# Solution

Terraform can ask AWS:

"Give me the latest Amazon Linux AMI."

Terraform receives the answer automatically.

This is called a:

# Data Source

---

# Resource vs Data Source

## Resource

Creates something.

Example:

```hcl
resource "aws_instance" "web" {

}
```

Terraform action:

```
Create EC2
```

---

## Data Source

Reads existing information.

Example:

```hcl
data "aws_ami" "amazon_linux" {

}
```

Terraform action:

```
Search AWS
Return AMI Information
```

---

# Simple Comparison

| Resource | Data Source |
|-|-|
| Creates infrastructure | Reads information |
| Managed by Terraform | Already exists |
| Uses resource block | Uses data block |
| Creates EC2 | Finds AMI |

---

# Architecture

```
Terraform

     |

     ▼

Data Source

     |

     ▼

AWS API

     |

     ▼

Returns Information

     |

     ▼

Terraform Uses It
```

---

# Folder Structure

Create:

```
terraform/
└── lab-06/
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

Go to your Terraform project:

```bash
cd ~/OneDrive/Desktop/terraform/terraform-zero-to-hero
```

Create folders:

```bash
mkdir -p terraform/lab-06
mkdir -p images/lab-06

touch labs/lab-06-terraform-data-sources.md
```

Move inside:

```bash
cd terraform/lab-06
```

---

# Step 2 - Copy Previous Lab Files

Copy common files:

```bash
cp ../lab-05/versions.tf .
cp ../lab-05/provider.tf .
cp ../lab-05/variables.tf .
cp ../lab-05/terraform.tfvars .
cp ../lab-05/locals.tf .
cp ../lab-05/outputs.tf .
```

Create new files:

```bash
touch data.tf
touch main.tf
touch README.md
```

Check:

```bash
ls
```

Expected:

```
README.md
data.tf
locals.tf
main.tf
outputs.tf
provider.tf
terraform.tfvars
variables.tf
versions.tf
```

---

# Step 3 - Create Your First Data Source

Open:

```
data.tf
```

Add:

```hcl
data "aws_availability_zones" "available" {

  state = "available"

}
```

---

# Understanding This

Terraform asks AWS:

"Show me all available Availability Zones."

AWS returns:

Example:

```
ap-south-1a

ap-south-1b

ap-south-1c
```

Terraform does not create them.

It only reads them.

---

# Step 4 - Create Output

Open:

```
outputs.tf
```

Append:

```hcl
output "availability_zones" {

  description = "Available AWS Availability Zones"

  value = data.aws_availability_zones.available.names

}
```

---

# Step 5 - Format

Run:

```bash
terraform fmt
```

---

# Step 6 - Initialize

Run:

```bash
terraform init
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

# Step 8 - Apply

Run:

```bash
terraform apply
```

Type:

```
yes
```

Expected:

```
Outputs:

availability_zones = [
 "ap-south-1a",
 "ap-south-1b",
 "ap-south-1c"
]
```

---

# Behind the Scenes

Terraform process:

```
terraform apply

        |

        ▼

Read data.tf

        |

        ▼

Ask AWS API

        |

        ▼

Receive Availability Zones

        |

        ▼

Store Information

        |

        ▼

Display Output
```

No AWS resource was created.

---

# Step 9 - Verify State

Run:

```bash
terraform show
```

You will see:

```
data.aws_availability_zones.available
```

Notice:

```
data.
```

This means Terraform is reading information.

---

# Real Production Usage

Companies use Data Sources for:

## Latest AMI

```
Find latest Amazon Linux
```

## Existing VPC

```
Use company VPC
```

## Existing Subnet

```
Deploy application
```

## Existing Security Groups

```
Attach to EC2
```

## AWS Account Information

```
Get Account ID
```

---

# Important Rule

Remember:

Resources:

```
Terraform creates
```

Data Sources:

```
Terraform searches
```

---

# Summary

You learned:

- What Data Sources are
- Resource vs Data Source
- Creating your first Data Source
- Reading Availability Zones
- Using Data Source outputs
- How Terraform communicates with AWS

---

# End of Part 1

In Part 2 you will learn:

- AWS AMI Data Source
- Finding latest Amazon Linux AMI
- Filtering AWS resources
- Using Data Sources with EC2
- Real production examples

---

# Part 2 - AWS AMI Data Source

Now we will use a real AWS example.

We will ask AWS:

"Give me the latest Amazon Linux AMI available in this region."

Terraform will find it automatically.

---

# Why Do We Need AMI Data Sources?

An AMI (Amazon Machine Image) is the operating system template used to launch EC2.

Examples:

```
Amazon Linux

Ubuntu

Windows Server

Red Hat Enterprise Linux
```

Every AMI has an ID.

Example:

```
ami-0abcdef123456789
```

The problem:

AMI IDs are different for:

- Different Regions
- Different Operating Systems
- Different Versions

---

# Without Data Source

Hardcoded:

```hcl
resource "aws_instance" "web" {

  ami = "ami-0123456789"

}
```

Problem:

Tomorrow AWS releases a new AMI.

Your code still uses the old one.

---

# With Data Source

Terraform:

```
Ask AWS

↓

Find Latest Amazon Linux

↓

Return AMI ID

↓

Use It
```

---

# Step 10 - Create AMI Data Source

Open:

```
data.tf
```

Add below the Availability Zone Data Source:

```hcl
data "aws_ami" "amazon_linux" {

  most_recent = true

  owners = [
    "amazon"
  ]

  filter {

    name = "name"

    values = [
      "al2023-ami-*-x86_64"
    ]

  }

  filter {

    name = "root-device-type"

    values = [
      "ebs"
    ]

  }

  filter {

    name = "virtualization-type"

    values = [
      "hvm"
    ]

  }

}
```

---

# Understanding Each Line

## most_recent

```hcl
most_recent = true
```

Means:

```
Give me the latest AMI
```

---

## owners

```hcl
owners = [
 "amazon"
]
```

Means:

Only use official Amazon images.

---

## Name Filter

```hcl
name = "name"
```

Search AMI names.

Example:

```
al2023-ami-2026xxxx
```

---

## Root Device Type

```hcl
root-device-type = ebs
```

Means:

Use EBS-backed storage.

---

## Virtualization Type

```hcl
virtualization-type = hvm
```

Modern EC2 instances use HVM virtualization.

---

# Step 11 - Create AMI Output

Open:

```
outputs.tf
```

Add:

```hcl
output "amazon_linux_ami_id" {

  description = "Latest Amazon Linux AMI ID"

  value = data.aws_ami.amazon_linux.id

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

Expected:

```
Success! The configuration is valid.
```

---

# Step 14 - Apply

Run:

```bash
terraform apply
```

Type:

```
yes
```

Expected Output:

```
amazon_linux_ami_id = ami-xxxxxxxxxxxx
```

Your AMI ID will be different depending on:

- AWS Region
- Current Amazon Linux release

---

# Behind the Scenes

The process:

```
terraform apply

        |

        ▼

Read data.aws_ami

        |

        ▼

AWS API Request

        |

        ▼

Search Amazon AMIs

        |

        ▼

Return Latest AMI ID

        |

        ▼

Store Result
```

Terraform did not create an AMI.

It only searched AWS.

---

# Step 15 - Test Different Regions

Open:

```
terraform.tfvars
```

Change:

```hcl
aws_region = "us-east-1"
```

Run:

```bash
terraform apply
```

You will receive a different AMI ID.

Why?

Because every AWS Region has different AMI IDs.

---

# Step 16 - View Data Source Information

Run:

```bash
terraform show
```

Look for:

```
data.aws_ami.amazon_linux
```

You will see:

```
id

name

owner_id

architecture

creation_date
```

Terraform collected this information from AWS.

---

# Data Source vs Resource Example

## Data Source

```hcl
data "aws_ami" "amazon_linux" {

}
```

Meaning:

```
Find existing AMI
```

---

## Resource

```hcl
resource "aws_instance" "web" {

}
```

Meaning:

```
Create EC2 instance
```

---

# Production Architecture

In real companies:

```
Terraform

        |

        ▼

Data Source

        |

        ▼

Find Latest AMI

        |

        ▼

Create EC2

        |

        ▼

Output Instance Details
```

This prevents outdated infrastructure.

---

# Common Mistakes

## Mistake 1

Wrong owner:

```hcl
owners = ["ubuntu"]
```

when searching Amazon Linux.

Result:

```
No AMI found
```

---

## Mistake 2

Wrong AMI name filter.

Example:

```hcl
values = [
 "wrong-image-*"
]
```

Result:

```
Your query returned no results
```

---

## Mistake 3

Wrong region.

Example:

```
AMI exists in Mumbai

but you search in Virginia
```

Result:

```
No matching AMI
```

---

# Lab Verification Checklist

Verify:

✅ Availability Zones Data Source works  
✅ AMI Data Source works  
✅ Latest AMI is retrieved  
✅ Output displays AMI ID  
✅ Region change returns a different AMI  
✅ terraform validate succeeds  

---

# Summary

You learned:

- AWS AMI Data Source
- AMI filtering
- Finding latest images
- Region dependency
- Using Data Sources in production

---

# End of Part 2

Next:

# Part 3 - Using Data Sources with EC2

You will learn:

- Create your first EC2 resource using a Data Source
- Combine Variables + Locals + Data Sources
- Add Tags
- Add Outputs
- Understand the complete Terraform flow

This will become the foundation for your Lab 10 EC2 Mini Project.

---

# Part 3 - Using Data Sources with EC2

Congratulations!

You can now:

- Read Availability Zones from AWS
- Find the latest Amazon Linux AMI
- Create reusable variables
- Create naming standards using Locals

Now we will combine these concepts.

We will create an EC2 instance using the AMI returned by the Data Source.

---

# Architecture

The Terraform flow will look like this:

```
terraform.tfvars

        |

        ▼

Variables

        |

        ▼

Locals

        |

        ▼

Data Source

        |

        ▼

Find Latest AMI

        |

        ▼

EC2 Resource

        |

        ▼

Outputs
```

---

# Step 17 - Create EC2 Resource

Open:

```
main.tf
```

Add:

```hcl
resource "aws_instance" "web" {

  ami = data.aws_ami.amazon_linux.id

  instance_type = var.instance_type


  tags = merge(

    local.common_tags,

    {

      Name = local.ec2_name

    }

  )

}
```

---

# Understanding the Code

## AMI

```hcl
ami = data.aws_ami.amazon_linux.id
```

Terraform gets the AMI from:

```
AWS Data Source
```

Not from a hardcoded value.

---

## Instance Type

```hcl
instance_type = var.instance_type
```

This comes from:

```
terraform.tfvars
```

Example:

```
t2.micro
```

---

## Tags

```hcl
tags = merge()
```

We combine:

Common Tags:

```
Project
Environment
Owner
ManagedBy
```

with:

```
Name
```

Final result:

```
Name = terraform-zero-to-hero-dev-ec2

Project = Terraform

Environment = Dev
```

---

# What is merge()?

`merge()` combines multiple maps.

Example:

Map 1:

```hcl
{
 Environment = "dev"
}
```

Map 2:

```hcl
{
 Name = "web-server"
}
```

Result:

```hcl
{
 Environment = "dev"
 Name = "web-server"
}
```

---

# Step 18 - Add EC2 Outputs

Open:

```
outputs.tf
```

Append:

```hcl
output "instance_id" {

  description = "EC2 Instance ID"

  value = aws_instance.web.id

}


output "public_ip" {

  description = "EC2 Public IP"

  value = aws_instance.web.public_ip

}


output "private_ip" {

  description = "EC2 Private IP"

  value = aws_instance.web.private_ip

}


output "instance_ami" {

  description = "AMI Used"

  value = aws_instance.web.ami

}
```

---

# Step 19 - Format

Run:

```bash
terraform fmt
```

---

# Step 20 - Initialize

Run:

```bash
terraform init
```

---

# Step 21 - Validate

Run:

```bash
terraform validate
```

Expected:

```
Success! The configuration is valid.
```

---

# Step 22 - Plan

Before creating resources, always check the plan.

Run:

```bash
terraform plan
```

Terraform will show:

```
+ create aws_instance.web
```

Example:

```
Resource actions:

  + aws_instance.web

Plan: 1 to add, 0 to change, 0 to destroy
```

---

# Step 23 - Apply

Run:

```bash
terraform apply
```

Type:

```
yes
```

Terraform will:

1. Query AWS for AMI
2. Create EC2 instance
3. Apply Tags
4. Store state
5. Display Outputs

---

# Expected Output

Example:

```
Outputs:

instance_id =
"i-0123456789abcdef"

public_ip =
"13.xxx.xxx.xxx"

private_ip =
"172.31.xxx.xxx"

instance_ami =
"ami-xxxxxxxx"
```

Your values will be different.

---

# Verify in AWS Console

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
terraform-zero-to-hero-dev-ec2
```

Check:

- Instance State: Running
- Instance Type: t2.micro
- AMI: Amazon Linux
- Tags applied

---

# Behind the Scenes

Complete Terraform workflow:

```
terraform apply

        |

        ▼

Read Variables

        |

        ▼

Calculate Locals

        |

        ▼

Search AWS AMI

        |

        ▼

Create EC2 Instance

        |

        ▼

Apply Tags

        |

        ▼

Update State File

        |

        ▼

Display Outputs
```

This is the complete Terraform lifecycle.

---

# Why This Approach Is Better

Old method:

```hcl
ami = "ami-123456"
```

Problems:

- Hardcoded
- Region dependent
- Becomes outdated

---

Professional method:

```hcl
ami = data.aws_ami.amazon_linux.id
```

Benefits:

- Dynamic
- Always gets latest image
- Works across environments
- Easier maintenance

---

# Production Example

A company has:

```
Development

Testing

Production
```

All environments use:

```
Latest Amazon Linux
```

Terraform automatically finds the correct AMI in each AWS Region.

No engineer manually updates AMI IDs.

---

# Lab Verification Checklist

Verify:

✅ Data Source retrieves AMI  
✅ EC2 instance is created  
✅ EC2 uses dynamic AMI  
✅ Tags are applied  
✅ Outputs show EC2 information  
✅ Terraform state updated  
✅ AWS Console shows instance  

---

# Cleanup

After testing:

Destroy the instance.

Run:

```bash
terraform destroy
```

Type:

```
yes
```

Terraform removes:

```
EC2 Instance
```

---

# Summary

You learned:

- Data Sources with Resources
- Dynamic AMI selection
- Creating EC2 using Data Source
- Using merge() for tags
- Terraform workflow
- Complete AWS deployment flow

---

# End of Lab 06

You have completed:

✅ Availability Zone Data Source  
✅ AMI Data Source  
✅ EC2 with Data Source  
✅ Outputs  
✅ Dynamic Infrastructure

---

# Next Lab

## Lab 07 - Terraform Expressions and Functions

You will learn:

- Conditional expressions
- For expressions
- Splat expressions
- Dynamic blocks
- Advanced Terraform logic

These skills are required before learning:

- count
- for_each
- modules
- production Terraform patterns

