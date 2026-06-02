# Infrastructure as Code (IAC) & Terraform Basics

## 1. Introduction to IAC
- **Definition**: IAC means writing **code** (instead of clicking manually in AWS/Azure GUI) to create and manage infrastructure like servers, networks, databases, etc.  
- **Idea**: Just like you use code to build an app, you use code to build infrastructure.  
- **Benefits**:  
  - Repeatable → Same setup every time.  
  - Automated → Saves manual effort.  
  - Version-controlled → Stored in Git, so changes are trackable.  
  - Scalable → Easy to deploy infra across multiple environments (Dev, Test, Prod).  

👉 Example: Instead of manually creating an EC2 in AWS Console, you write a Terraform file and just run `terraform apply`.

---

## 2. Why we need IAC (Difference between Shell Script, Ansible, and IAC tools like Terraform)

| **Aspect** | **Shell Script** | **Ansible** | **IAC Tool (Terraform)** |
|------------|------------------|-------------|--------------------------|
| **Purpose** | Automates tasks (e.g., install software, copy files). | Config management + automation. | Full infra provisioning (VMs, networks, DBs). |
| **State awareness** | No state awareness → runs blindly. | Limited state tracking. | Maintains **state file** → knows what exists and what needs change. |
| **Idempotency** | ❌ No → may create duplicates. | ✅ Yes → ensures final state. | ✅ Yes → ensures infrastructure matches code. |
| **Cloud support** | Not cloud-focused. | Supports cloud but mainly for config mgmt. | Designed for multi-cloud infra provisioning. |
| **Example** | Bash script: `apt-get install nginx` | Ansible Playbook to install Nginx | Terraform code to create EC2 + attach security group + install Nginx |

👉 In short:  
- **Shell Script** = manual automation.  
- **Ansible** = config management & software deployment.  
- **Terraform (IAC)** = provisioning complete infra in a controlled, declarative way.  

---


## 3. Terraform Language (Basic Syntax)
Terraform files are written in **HCL (HashiCorp Configuration Language)**.  

Example:
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "my_ec2" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```

- **Provider** → Defines which cloud/service you are using (AWS, Azure, GCP).  
- **Resource** → Defines what infra you want (EC2, VPC, S3, etc.).  
- **Arguments** → Settings inside resources (`ami`, `instance_type`).  

👉 It’s **declarative** → you say *what you want*, Terraform figures out *how to do it*.  

---

## 4. Enlist the Blocks used in Terraform Language

Terraform has multiple **blocks** (building units):  

1. **provider** → Defines the provider (AWS, Azure, etc.)  
   ```hcl
   provider "aws" {
     region = "us-east-1"
   }
   ```

2. **resource** → Defines infrastructure resources.  
   ```hcl
   resource "aws_instance" "example" {
     ami           = "ami-12345"
     instance_type = "t2.micro"
   }
   ```

3. **variable** → Input values (like parameters).  
   ```hcl
   variable "region" {
     default = "us-east-1"
   }
   ```

4. **output** → Shows values after deployment.  
   ```hcl
   output "instance_ip" {
     value = aws_instance.example.public_ip
   }
   ```

5. **module** → Group of Terraform files reused as a package.  
   ```hcl
   module "vpc" {
     source = "./modules/vpc"
   }
   ```

6. **locals** → Define local variables.  
   ```hcl
   locals {
     env = "dev"
   }
   ```

7. **data** → Fetch existing info (e.g., latest AMI).  
   ```hcl
   data "aws_ami" "latest" {
     most_recent = true
     owners      = ["amazon"]
   }
   ```

---
### Terraform Instalation
```sh
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```
### Terraform file that creates an EC2 instance on AWS
```hcl
# Define the AWS provider
provider "aws" {
  region = "us-east-1"  
}

# Create an EC2 instance
resource "aws_instance" "my_ec2" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"              

  tags = {
    Name = "MyFirstEC2"
  }
}

```
Terraform Script to Deploy Security Group with HEREDOC in UserData

This guide covers the deployment of a Security Group using Terraform and explains the HEREDOC concept in UserData along with the key blocks in the script.
# Create Ec2 with Security Gruup and Userdata
```hcl
provider "aws" {
    region = "ap-south-1"
}

resource "aws_instance" "b69" {
    ami = "ami-07a00cf47dbbc844c"
    instance_type = "t3.micro"
    vpc_security_group_ids = [aws_security_group.my-sg.id]
    key_name = "sahil2.0"
    user_data = base64encode(<<-EOF
        #!/bin/bash
        apt update -y
        apt install nginx -y
        systemctl start nginx
        systemctl enable nginx
    EOF
    )

    tags = {
        Name = "MyFirstEc2"
    }
}

resource "aws_security_group" "my-sg" {
    name = "my-sg"
    description = "this sg is created through terraform"
    vpc_id = "vpc-0fd488cbdaf75f70f"

    ingress {
        from_port = 0
        to_port = 0
        protocol = 22
        cidr_blocks = ["0.0.0.0/0"]
    }

    ingress {
        from_port = 0
        to_port = 0
        protocol = 80
        cidr_blocks = ["0.0.0.0/0"]
    }
    ingress {
        from_port = 0
        to_port = 0
        protocol = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }
    egress {
        from_port        = 0
        to_port          = 0
        protocol         = "-1"
        cidr_blocks      = ["0.0.0.0/0"]
    }

    tags = {
        Name = "my-sg"
    }
}

```
---
## Autoscaling and loadbalancer through terraform
```hcl

# launch template 2
# asg - 2
# asg policy 2
# cloudwatch alarm 2
# target group - 2
# asg attachments 2
# alb
# listener
# listener rule


provider "aws" {
    region = "ap-south-1"
}

## launch_templates
resource "aws_launch_template" "home-temp" {
    name = "home-temp"
    image_id = "ami-07a00cf47dbbc844c"
    instance_type = "t3.micro"
    key_name = "sahil2.0"
    user_data = base64encode(<<-EOF
        #!/bin/bash
        sudo apt update -y
        sudo apt install nginx -y
        echo "welcome to homepage" > /var/www/html/index.html
        systemctl start nginx
        systemctl enable nginx
    EOF
    )
    tags = {
        Name = "home-temp"
    }
}  

resource "aws_launch_template" "cloth-temp" {
    name = "cloth-temp"
    image_id = "ami-07a00cf47dbbc844c"
    instance_type = "t3.micro"
    key_name = "sahil2.0"
    user_data = base64encode(<<-EOF
        #!/bin/bash
        sudo apt update -y
        sudo apt install nginx -y
        mkdir -p /var/www/html/cloth
        echo "SALE!!! SALE!!! SALE!!!" > /var/www/html/cloth/index.html
        systemctl start nginx
        systemctl enable nginx
    EOF
    )
    tags = {
        Name = "cloth-temp"
    }
}

##  Auto_scaling_group

resource "aws_autoscaling_group" "home-asg" {
    name = "home-asg"
    availability_zones = ["ap-south-1a", "ap-south-1b", "ap-south-1c"]
    desired_capacity = 1
    max_size = 1
    min_size = 1
    health_check_type = "EC2"

    launch_template {
        id = aws_launch_template.home-temp.id
        version = "$Latest"
    }

}

resource "aws_autoscaling_group" "cloth-asg" {
    name = "cloth-asg"
    availability_zones = ["ap-south-1a", "ap-south-1b", "ap-south-1c"]
    desired_capacity = 1
    max_size = 1
    min_size = 1
    health_check_type = "EC2"

    launch_template {
        id = aws_launch_template.cloth-temp.id
        version = "$Latest"
    }

}

## ASG_policy

resource "aws_autoscaling_policy" "home-policy" {
    name = "home-policy"
    autoscaling_group_name = aws_autoscaling_group.home-asg.name
    adjustment_type = "ChangeInCapacity"
    scaling_adjustment = -1
    cooldown = 120
}
resource "aws_autoscaling_policy" "cloth-policy" {
    name = "cloth-policy"
    autoscaling_group_name = aws_autoscaling_group.cloth-asg.name
    adjustment_type = "ChangeInCapacity"
    scaling_adjustment = -1
    cooldown = 120
}

## Cloudwatch_Alarms

resource "aws_cloudwatch_metric_alarm" "home-alarm" {
    alarm_description = "this is alarm for home-asg"
    alarm_actions = [aws_autoscaling_policy.home-policy.arn]
    alarm_name = "home-alarm"
    comparison_operator = "LessThanOrEqualToThreshold"
    namespace = "AWS/EC2"
    metric_name = "CPUUtilization"
    threshold = "20"
    evaluation_periods = "5"
    period = "30"
    statistic = "Average"

    dimensions = {
        AutoScalingGroupName = aws_autoscaling_group.home-asg.name
    }
}

resource "aws_cloudwatch_metric_alarm" "cloth-alarm" {
    alarm_description = "this is alarm for cloth-asg"
    alarm_actions = [aws_autoscaling_policy.cloth-policy.arn]
    alarm_name = "cloth-alarm"
    comparison_operator = "LessThanOrEqualToThreshold"
    namespace = "AWS/EC2"
    metric_name = "CPUUtilization"
    threshold = "20"
    evaluation_periods = "5"
    period = "30"
    statistic = "Average"

    dimensions = {
        AutoScalingGroupName = aws_autoscaling_group.cloth-asg.name
    }
}

## Target_group
 resource "aws_lb_target_group" "home-tg" {
    name = "home-tg"
    port = 80
    protocol  = "HTTP"
    vpc_id = "vpc-0fd488cbdaf75f70f"
 }

  resource "aws_lb_target_group" "cloth-tg" {
    name = "cloth-tg"
    port = 80
    protocol  = "HTTP"
    vpc_id = "vpc-0fd488cbdaf75f70f"
 }

 ## ASG_group_attachments

resource "aws_autoscaling_attachment" "home-attach" {
  autoscaling_group_name = aws_autoscaling_group.home-asg.id
  lb_target_group_arn    = aws_lb_target_group.home-tg.arn
}

resource "aws_autoscaling_attachment" "cloth-attach" {
  autoscaling_group_name = aws_autoscaling_group.cloth-asg.id
  lb_target_group_arn    = aws_lb_target_group.cloth-tg.arn
}

 ## ALB

 resource "aws_lb" "alb" {
    name = "alb"
    internal = false
    load_balancer_type = "application"
    subnets = ["subnet-0975c397075943a30", "subnet-012df937aaba75ee1"]
    security_groups = ["sg-0df0dec13f6d35176"]

 }

 ## Listner and Rules

 resource "aws_lb_listener" "alb-listener" {
    load_balancer_arn = aws_lb.alb.arn
    port = "80"
    protocol = "HTTP"

    default_action {
        type = "forward"
        target_group_arn = aws_lb_target_group.home-tg.arn
    }
 }

 resource "aws_lb_listener_rule" "cloth-rule" {
    listener_arn = aws_lb_listener.alb-listener.arn
    priority = 1

    action {
        type = "forward"
        target_group_arn = aws_lb_target_group.cloth-tg.arn
    }

    condition {
        path_pattern {
            values = ["/cloth/*"]
        }
    }
 }
```
---
## 📌 1. What is a Terraform Module?

- A **module** in Terraform is a **container for multiple resources** that are used together.  
- Think of a module as a **function in programming**:
  - You **define it once**,
  - **Pass input variables**,
  - **Get outputs**,
  - **Reuse it anywhere**.

By default, your Terraform project is the **root module**.  
You can create **child modules** inside a `modules/` folder and call them from your root module.

---


---

## 📌 3. Creating the Folder Structure on Server

- Step 1: Create project folder
```bash
mkdir terraform-project
cd terraform-project
```
- Step 2: Create module folders
```sh
mkdir -p modules/vpc modules/subnet modules/ec2
```
- Step 3: Create root files
```sh
touch main.tf variables.tf outputs.tf provider.tf
```
- Step 4: Create module files
```sh
touch modules/vpc/{main.tf,variables.tf,outputs.tf} \
      modules/subnet/{main.tf,variables.tf,outputs.tf} \
      modules/ec2/{main.tf,variables.tf,outputs.tf}
```

### Root/variable.tf
```hcl
# AWS Region
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

# VPC variables
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

# Subnet variables
variable "subnet_cidr" {
  description = "CIDR block for Subnet"
  type        = string
  default     = "10.0.1.0/24"
}

# EC2 variables
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```
### root/provider.tf
```hcl
provider "aws" {
  region = var.aws_region
}
```
### root/main.tf
```hcl
# VPC Module
module "vpc" {
  source   = "./modules/vpc"
  vpc_cidr = var.vpc_cidr
}

# Subnet Module
module "subnet" {
  source      = "./modules/subnet"
  vpc_id      = module.vpc.vpc_id
  subnet_cidr = var.subnet_cidr
}

# EC2 Module
module "ec2" {
  source        = "./modules/ec2"
  subnet_id     = module.subnet.subnet_id
  instance_type = var.instance_type
}
```
### root/outputs.tf
```hcl
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "subnet_id" {
  value = module.subnet.subnet_id
}

output "instance_id" {
  value = module.ec2.instance_id
}
```
## Modules
## modules/vpc
### variables.tf
```hcl
variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
}
```
### main.tf
```hcl
resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = "MyVPC"
  }
}
```
### outputs.tf
```hcl
output "vpc_id" {
  value = aws_vpc.this.id
}
```
## modules/subnet
### variabls.tf
```hcl
variable "vpc_id" {
  description = "VPC ID from VPC module"
  type        = string
}

variable "subnet_cidr" {
  description = "Subnet CIDR block"
  type        = string
}
```
### main.tf
```hcl
resource "aws_subnet" "this" {
  vpc_id     = var.vpc_id
  cidr_block = var.subnet_cidr

  tags = {
    Name = "MySubnet"
  }
}
```
### outputs.tf
```hcl
output "subnet_id" {
  value = aws_subnet.this.id
}
```
## modules/ec2
### variables.tf
```hcl
variable "subnet_id" {
  description = "Subnet ID from subnet module"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
}
```
### main.tf
```hcl
# Fetch latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "this" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id

  tags = {
    Name = "MyEC2"
  }
}
```
### outputs.tf
```hcl
output "instance_id" {
  value = aws_instance.this.id
}
```
# Multi-Environment Terraform (.tfvars)
You keep one set of Terraform code (resources, modules, variables) and switch values (CIDRs, sizes, tags, etc.) via per-environment .tfvars files. That way you don’t duplicate code—only the inputs change.
```bash
ec2-multi-env/
├─ main.tf
├─ variables.tf
├─ env/
│  ├─ dev.tfvars
│  ├─ stage.tfvars
│  └─ prod.tfvars
```

## create project root
```sh
mkdir ec2-multi-env
cd ec2-multi-env
```
## create terraform files
```sh
touch main.tf variables.tf
```
## Create Environment Directory
```sh
mkdir env
```
## Create Environment .tfvars Files
```sh
touch env/dev.tfvars env/stage.tfvars env/prod.tfvars
```
---
### main.tf
```hcl
provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "my_ec2" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = "ec2-${var.environment}"
    Env  = var.environment
  }
}
```
### variables.tf
```
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
}

variable "environment" {
  description = "Environment name (dev, stage, prod)"
  type        = string
}

variable "aws_region" {
  description = "AWS region"
  type        = string
}

variable "ami_id" {
  description = "AMI ID"
  type        = string
}
```
## Environment-specific .tfvars
### env/dev.tfvars
```hcl
environment   = "dev"
aws_region    = "us-east-1"
ami_id        = "ami-08c40ec9ead489470"
instance_type = "t2.micro"
```
### env/stage.tfvars
```hcl
environment   = "stage"
aws_region    = "us-east-1"
ami_id        = "ami-08c40ec9ead489470"
instance_type = "t3.small"
```
### env/prod.tfvars
```hcl
environment   = "prod"
aws_region    = "us-east-1"
ami_id        = "ami-08c40ec9ead489470"
instance_type = "t3.medium"
```
## Terraform Commands
```sh
terraform init
```
### deploy DEV
```sh
terraform apply -var-file="env/dev.tfvars"
```
### deploy stage
```sh
terraform apply -var-file="env/stage.tfvars"
```
### deploy prod
```sh
terraform apply -var-file="env/prod.tfvars"
```
---
---
# Terraform Workspace 

##  Objective
To understand how **Terraform Workspaces** work by creating the **same resource** in **different workspaces**, where only the **state file and resource name change**.

---

##  Concept Recap 

- Terraform workspaces allow you to use **one Terraform configuration**
- Each workspace has its **own state file**
- Resources are **separate**, even though code is the same
- `terraform.workspace` gives the **current workspace name**

---

## Step 1: Create Project Directory

```bash
mkdir terraform-workspace-demo
cd terraform-workspace-demo
```
## Step 2: Create Terraform File
```sh
touch main.tf
```
## Step 3: Add Terraform Configuration (main.tf)
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "example" {
  bucket = "example-bucket-${terraform.workspace}"
  acl    = "private"

  tags = {
    Name = "workspace-demo"
  }
}
```
Explanation - 
- ${terraform.workspace} automatically picks the active workspace name
- Each workspace creates a different S3 bucket
- Same code → different bucket names

## Step 4: Initialize Terraform
```sh
terraform init
```
## Step 5: Check Existing Workspaces
```sh
terraform workspace list
```
## Step 6: Create New Workspaces
```sh
terraform workspace new dev
terraform workspace new stage
terraform workspace new prod
```
## Step 7: Switch Between Workspaces
```sh
terraform workspace select <workspace_name>
```
## terraform workspace select <workspace_name>
```sh
terraform workspace select dev
terraform apply
```
---
# Difference Between Terraform Modules, .tfvars, and Workspaces

| Aspect | Modules | .tfvars | Workspaces |
|------|--------|---------|-----------|
| What it is | A way to organize and reuse Terraform code | A file used to provide variable values | A mechanism to maintain separate state files |
| Primary purpose | Avoid code duplication | Change configuration values without changing code | Isolate Terraform state |
| Affects Terraform code | Yes | No | No |
| Affects variable values | No | Yes | No |
| Affects state file | No | No | Yes |
| Reusability | High – same module can be used multiple times | Not reusable code, only values | Not reusable, only state separation |
| Typical use case | Large or repeated infrastructure components | Different configurations for dev, stage, prod | Logical separation of infrastructure states |
| Common examples | VPC module, EC2 module, RDS module | instance_type, region, CIDR blocks | default, dev, test |
| Recommended for production | Yes | Yes | Limited (use carefully) |
| Learning curve | Medium | Easy | Easy to Medium |
| Mental model | Code structure | Configuration values | State management |

## One-Line Summary

- Modules define how infrastructure is built.
- .tfvars define what values are used.
- Workspaces define where Terraform stores its state.
---
## Terraform Loops
Terraform provides powerful constructs for iterating over collections like `list` and `map`. The primary looping mechanisms are `count`, `for_each`, and `for`.

### 1. `count`
- **Definition**: The `count` parameter allows you to specify how many instances of a resource to create.
- **Usage**: Works well for creating identical resources.

#### Example:
```hcl
resource "aws_instance" "example" {
  count         = 3
  ami           = "ami-12345678"
  instance_type = "t2.micro"
}
```
In this example, three EC2 instances are created.

#### Accessing Instances:
```hcl
aws_instance.example[0]  # First instance
aws_instance.example[1]  # Second instance
aws_instance.example[2]  # Third instance
```

### 2. `for_each`
- **Definition**: The `for_each` meta-argument allows iterating over `map` or `set` types to create resources with distinct properties.
- **Usage**: Useful when resource properties vary.

#### Example:
```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_s3_bucket" "example" {
  for_each = {
    dev  = "dev-bucket-unique-1"
    prod = "prod-bucket-unique-2"
  }

  bucket = each.value
}



```
This creates two S3 buckets: `dev-bucket` and `prod-bucket`.

#### Accessing Instances:
```hcl
aws_s3_bucket.example["dev"]  # Dev bucket
aws_s3_bucket.example["prod"] # Prod bucket
```

### 3. `for`
- **Definition**: The `for` expression is used to transform or filter collections.
- **Usage**: Commonly used in variables and outputs.

#### Example:
```hcl
variable "names" {
  default = ["Alice", "Bob", "Charlie"]
}

output "uppercase_names" {
  value = [for name in var.names : upper(name)]
}
```
This outputs the names in uppercase: `["ALICE", "BOB", "CHARLIE"]`.

#### Filtering with `for`:
```hcl
output "filtered_names" {
  value = [for name in var.names : name if length(name) > 3]
}
```
This filters names longer than three characters.

---

## Comparison Table
| Feature      | `count`                  | `for_each`                  | `for`                    |
|--------------|--------------------------|-----------------------------|--------------------------|
| Input Type   | Number                   | Map or Set                  | List, Map, or Set        |
| Use Case     | Create identical items   | Create unique items         | Transform or filter data |
| Example      | EC2 instances            | S3 buckets with unique IDs  | Modify list of names     |

---
## Example
```hcl
# Define the AWS provider
provider "aws" {
  region = "ap-south-1"   
}

# Create an EC2 instance
resource "aws_instance" "my_ec2" {
  for_each = toset(var.ami_ids)
  ami = each.value
  instance_type = "t3.micro"

#  count = 3
  tags = {
    Name = "MyFirstEC2"
  }
}

variable "ami_ids" {
    default = ["ami-07a00cf47dbbc844c", "ami-0685bcc683dadb6b9"]
    type = list(string)
}

output "public_ip" {
    value = { for instance in aws_instance.my_ec2: instance.id => instance.arn }
}

variable "names" {
  default = ["Alice", "Bob", "Charlie"]
}

output "uppercase_names" {
  value = [for name in var.names : upper(name)]
}
```
---
# Terraform Commands and Provisioners

## Terraform Commands

### 1. **Taint Command**
The `taint` command marks a resource for recreation during the next `terraform apply`. This is useful when a specific resource needs to be replaced without altering the rest of the infrastructure.

#### **Syntax:**
```bash
terraform taint <resource_name>
```

#### **Example:**
```bash
terraform taint aws_instance.my_instance
```
This marks the `aws_instance.my_instance` resource for recreation.

---

### 2. **Import Command**
The `import` command allows importing existing infrastructure resources into Terraform state. This is helpful when managing resources created outside of Terraform.

#### **Syntax:**
```bash
terraform import <resource_type>.<resource_name> <resource_id>
```

#### **Example:**
```bash
terraform import aws_instance.my_instance i-0abcd1234efgh5678
```
This imports the AWS EC2 instance with ID `i-0abcd1234efgh5678` into Terraform as `aws_instance.my_instance`.

---

### 3. **Destroy Command**
The `destroy` command removes all resources defined in the configuration.

#### **Targeted Destroy (-t)**
You can destroy specific resources using the `-target` flag.

#### **Syntax:**
```bash
terraform destroy -target=<resource_type>.<resource_name>
```

#### **Example:**
```bash
terraform destroy -target=aws_instance.my_instance
```
This removes only the `aws_instance.my_instance` resource.

---
## terraform provision blocks
```hcl
# Define the AWS provider
provider "aws" {
  region = "us-east-1"   
}

# Create an EC2 instance
resource "aws_instance" "my_ec2" {
  ami           = "ami-0ecb62995f68bb549" 
  instance_type = "t3.micro"  
  key_name = "nv" 


  provisioner "local-exec" {
    command = "touch abc.txt"            
  }
  provisioner "file" {
  source      = "apache2.sh"
  destination = "/home/ubuntu/apache2.sh"
  }
  connection {
    type     = "ssh"
    user     = "ubuntu"
    private_key = file("nv")
    host     = self.public_ip
  }
    provisioner "remote-exec" {
    inline = [
        "bash /home/ubuntu/apache2.sh",
        "touch remote.txt"
    ]
  }
  tags = {
    Name = "MyFirstEC2"
  }
}
```
## EKS Cluster Through Terraform

#role
#VPC
#Subnet
```hcl
provider "aws" {
    region = "ap-south-1"
}

resource "aws_iam_role" "cluster" {
  name = "eks-cluster-example"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "sts:AssumeRole",
          "sts:TagSession"
        ]
        Effect = "Allow"
        Principal = {
          Service = "eks.amazonaws.com"
        }
      },
    ]
  })
}

resource "aws_iam_role_policy_attachment" "cluster_AmazonEKSClusterPolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.cluster.name
}


data "aws_vpc" "default" {
    default = true
}
data "aws_subnets" "default"{
    filter {
        name = "vpc-id"
        values = [data.aws_vpc.default.id]
    }
}

resource "aws_eks_cluster" "cluster" {
    name = "cluster"
    role_arn = aws_iam_role.cluster.arn
    access_config {
        authentication_mode = "API"
    }
    version  = "1.34"

    vpc_config {
        subnet_ids = data.aws_subnets.default.ids
    }
    depends_on = [
    aws_iam_role_policy_attachment.cluster_AmazonEKSClusterPolicy
  ]
}
```
