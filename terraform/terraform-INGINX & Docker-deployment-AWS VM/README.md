# Terraform AWS Infrastructure Deployment with NGINX and Docker
This article explains each component in a Terraform configuration designed to deploy an EC2 instance on AWS. The instance is preconfigured to host an NGINX server and Docker. The deployment includes VPC setup, subnet configuration, Internet Gateway, route table associations, and security groups, providing a straightforward yet robust environment suitable for web hosting or other Docker-based applications.
## Table of Contents
* Overview
+ Prerequisites
- Terraform Configuration
* Provider Configuration
- Networking Components
* Security Group
* EC2 Instance with NGINX and Docker
+ Deployment instructions
- Conclusion

  
## Overview
This setup provisions a VPC, subnet, Internet Gateway, and an EC2 instance configured to run an NGINX server with Docker installed. It includes a security group allowing SSH, HTTP, and HTTPS traffic, suitable for web application hosting and remote management.

## Prerequisites
Before proceeding, ensure you have the following:

* Terraform installed on your local machine.
* An AWS account with appropriate permissions to create resources such as VPCs, EC2 instances, and Security Groups.
* An AWS key pair in the specified region.

  
## Terraform Configuration
  
We begin by specifying the AWS provider in our Terraform configuration. Terraform uses this to understand which cloud resources to provision.

 ```bash
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}
```
The ```required_providers``` block ensures that Terraform fetches the correct version of the AWS provider plugin, ensuring compatibility.

## Provider Configuration
Next, we define the AWS provider. This specifies the AWS region where our infrastructure will be deployed. In this example, we are using the ```us-east-2``` region.

```bash
provider "aws" {
  region = "us-east-2"
}
```
## Networking Components
1.  VPC
2.  Internet Gateway
3.  Route Table
4.  Subnet
### 1. Virtual Private Cloud (VPC):
A VPC is a logically isolated network within AWS. Here, we define a VPC with a CIDR block of ```10.0.0.0/16```, which gives us plenty of private IP addresses to assign within our network.

```bash
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```
### 2. Internet Gateway
To allow instances within our VPC to access the internet for software updates or serving web traffic, we need to attach an Internet Gateway.
```bash
resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id
}
```

The Internet Gateway allows outbound internet access and inbound access based on security group rules.

### 3. Route Table
Next, we configure a route table for the VPC. The route table defines how traffic flows within the network. Here, we specify that traffic destined for ```0.0.0.0/0``` (all traffic) should be routed through the Internet Gateway.
```bash
resource "aws_route_table" "rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }
}
```

### 4. Subnet
A subnet is a smaller range of IP addresses within the VPC. We define a public subnet in the ```us-east-2a``` availability zone with a CIDR block of ```10.0.1.0/24```. This block provides 256 addresses, suitable for a small deployment.

```bash
resource "aws_subnet" "main" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-2a"
  map_public_ip_on_launch = true
}
```


By setting ```map_public_ip_on_launch = true```, we ensure that EC2 instances launched in this subnet will automatically receive public IP addresses, making them accessible from the internet.

### 5. Route Table Association
We associate the route table with the subnet so that resources in the subnet use the specified routing rules.

```bash
resource "aws_route_table_association" "a" {
  subnet_id      = aws_subnet.main.id
  route_table_id = aws_route_table.rt.id
}
```
## Security Group
Security Groups control inbound and outbound traffic to your instances. Here, we define a security group that allows SSH (port 22) and HTTP (port 80) traffic from any IP address.
```bash
resource "aws_security_group" "allow_web_traffic" {
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

```


* Port 22: Used for SSH access, enabling remote management of the instance.
+ Port 80: Used for HTTP traffic, making the instance capable of serving web traffic.
+ Port 443: Allows secure HTTPS traffic, ensuring encrypted access to web resources.
+ Port 0: Means all ports and protocols, Enables all outbound traffic from the instance.


## EC2 Instance with NGINX and Docker
The EC2 instance resource launches an ```ami-0862be96e41dcbf74``` image in a ```t2.micro``` instance type. It uses a user-data script to install NGINX and Docker.
```bash
resource "aws_instance" "gk_instance" {
  ami           = "ami-0862be96e41dcbf74"
  instance_type = "t2.micro"
  subnet_id     = aws_subnet.main.id
  key_name      = "us_east_2"
  vpc_security_group_ids = [aws_security_group.allow_web_traffic.id]

  user_data = <<-EOF
              #!/bin/bash
              sudo apt update
              sudo apt install -y nginx awscli
              echo 'Hello World from Terraform on NGINX!' > /var/www/html/index.html
              systemctl start nginx
              systemctl enable nginx

              # Install Docker
              sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
              curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
              sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
              sudo apt update
              sudo apt install -y docker-ce
              sudo systemctl start docker
              sudo systemctl enable docker

              # Add the default user to the docker group
              sudo usermod -aG docker ubuntu
              EOF

  tags = {
    Name = "gk_TerraformExample"
  }
}

```
The user-data script performs the following tasks:

* Updates the System: Ensures the latest packages are available.
* Installs NGINX and AWS CLI: Starts NGINX and writes a custom welcome message to ```/var/www/html/index.html```.
* Installs Docker: Adds Docker’s official repository, installs Docker, and enables it at startup.
* Configures Docker Permissions: Adds the default user to the Docker group for managing Docker without ```sudo```.

## Deployment Instructions
To deploy this infrastructure, follow these steps:

**Initialize Terraform**: Run the ```terraform init``` command to initialize the working directory containing the Terraform configuration files.
```bash
terraform init
```

**Plan the Deployment**: Generate an execution plan using ```terraform plan```. This will show you what Terraform will create or modify.
```bash
terraform plan
```
**Apply the Changes**: Use the ```terraform apply``` command to deploy the resources.
```bash
terraform apply
```
**Verify the Deployment**: After Terraform completes the deployment, you can log into the AWS console to verify the created resources, such as the VPC, subnets, and security groups.

## Conclusion
This Terraform setup deploys a web server with Docker installed on AWS, using a fully configured VPC, subnet, security group, and route table. The setup is flexible and can be extended to add additional instances, databases, or load balancers for a more complex environment.
