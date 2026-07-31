Create
```bash
touch labs/lab-13-nat-gateway.md
```
Go inside:
```bash
cd terraform/lab-13-nat-gateway
```

### Step 2 - Create Files
```bash
touch versions.tf

touch provider.tf

touch variables.tf

touch terraform.tfvars

touch data.tf

touch vpc.tf

touch subnet.tf

touch igw.tf

touch nat.tf

touch route-table.tf

touch security-group.tf

touch ec2.tf

touch outputs.tf

touch README.md

touch user-data.sh
```
Architecture
```bash
                Internet

                    |

                    ▼

          Internet Gateway

                    |

        -----------------------

        |                     |

        ▼                     ▼

 Public Subnet         Private Subnet

        |                     |

        ▼                     ▼

 NAT Gateway          Private EC2

        |

        ▼

 Elastic IP
 ```

 Resources We'll Create
 ```bash
 ✓ VPC

✓ Public Subnet

✓ Private Subnet

✓ Internet Gateway

✓ Elastic IP

✓ NAT Gateway

✓ Public Route Table

✓ Private Route Table

✓ Public EC2

✓ Private EC2
```
### Step 1 - Create versions.tf
Create the file
```bash
nano versions.tf
```
```bash
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

# Lab Cleanup

## Step 1 - Destroy AWS Resources

Run:

```bash
terraform destroy
```

Type:

```
yes
```

Verify:

```
Destroy complete!
```

---

## Step 2 - Verify AWS Console

Confirm these resources have been deleted:

```
✓ EC2

✓ Security Groups

✓ NAT Gateway

✓ Elastic IP

✓ Route Tables

✓ Internet Gateway

✓ Subnets

✓ VPC
```

---

## Step 3 - Remove Terraform Local Files

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

You should only see:

```
README.md

versions.tf

provider.tf

variables.tf

terraform.tfvars

main.tf

outputs.tf

...
```

---

## Step 5 - Git Status

Run:

```bash
git status
```

---

## Step 6 - Commit

```bash
git add .

git commit -m "Complete Lab 13 NAT Gateway"
```

---

## Step 7 - Push

```bash
git push origin main
```

---

# Lab Completed

You learned:

✅ NAT Gateway

✅ Elastic IP

✅ Private EC2

✅ Public EC2

✅ Public Route Table

✅ Private Route Table

✅ Production Networking
