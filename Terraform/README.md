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
