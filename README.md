# Skill-Test-3_Reattempt-EC2-Nginx-Deployment-with-Terraform
Assignment | Skill Test 3_Reattempt

AWS console Snapshot
<img width="1878" height="856" alt="AWS console snapshot" src="https://github.com/user-attachments/assets/e31f0fb5-fbc7-4325-8bd5-1d8e8b6efbba" />
AWS Configure snpahot
<img width="1483" height="652" alt="AWS configure Snapshot" src="https://github.com/user-attachments/assets/eb72373c-09e3-475e-97d9-efb4a10d0dec" />

## Ec2 Instance Runing snapshot

<img width="1366" height="768" alt="EC-2 Instance Snaphot" src="https://github.com/user-attachments/assets/9535ded7-2b44-4a32-b6b8-84318abb1e62" />
##cEc2 
## Ec2 Instance stop snapshot
<img width="1366" height="768" alt="EC-2 Instance Snaphot-2" src="https://github.com/user-attachments/assets/1a955d62-efed-4476-b17f-c115974684d8" />

## Terraform.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.65.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

## Variables.tf
variable "aws_region" {
  description = "AWS region where resources will be provisioned"
  default     = "ap-south-1"
}

variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  default     = "ami-01a00762f46d584a1"
}

variable "instance_type" {
  description = "Instance type for the EC2 instance"
  default     = "t2.micro"
}

## Outputs.tf
output "ec2_public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.jenkinsmaster.public_ip
}

## Ec2.tf 
provider "tls" {}
resource "tls_private_key" "generated" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "deployer" {
  key_name   = "capstone-user-key"
  public_key = tls_private_key.generated.public_key_openssh
}

resource "local_file" "private_key" {
  content         = tls_private_key.generated.private_key_pem
  filename        = "${path.module}/capstone-user-key.pem"
  file_permission = "0400"
}

resource "aws_default_vpc" "default" {

}

resource "aws_security_group" "allow_user_to_connect" {
  name        = "allow TLS"
  description = "Allow user to connect"
  vpc_id      = aws_default_vpc.default.id
  ingress {
    description = "port 22 allow"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

    ingress {
    description = "port 80 allow"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = " allow all outgoing traffic "
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
cidr_blocks = ["0.0.0.0/0"]
  }




  tags = {
    Name = "mysecurity"
  }
}

resource "aws_instance" "jenkinsmaster" {
  ami             = var.ami_id
  instance_type   = var.instance_type
  key_name        = aws_key_pair.deployer.key_name
  security_groups = [aws_security_group.allow_user_to_connect.name]
   user_data = <<-EOF
              #!/bin/bash
              apt update -y
              apt install nginx -y

              cat <<HTML > /var/www/html/index.html
              <!DOCTYPE html>
              <html>
              <head>
                  <title>Terraform Nginx Server</title>
              </head>
              <body>
                  <h1>Welcome to the Terraform-managed Nginx Server on Ubuntu</h1>
              </body>
              </html>
              HTML

              systemctl enable nginx
              systemctl restart nginx
              EOF

  tags = {
    Name = "terraform-nginx-server"
  }
}


## Terraform init








