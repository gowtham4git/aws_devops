# Deploying an AWS VM with Terraform
This guide provides an in-depth walkthrough of deploying an AWS Virtual Machine (VM) using Terraform. The deployment includes setting up essential networking components such as a Virtual Private Cloud (VPC), Subnets, an Internet Gateway, Route Tables, and Security Groups. This configuration is ideal for those looking to deploy basic infrastructure for a web application or remote SSH access.

## Table of Contents
* Introduction
+ Prerequisites
- Terraform Configuration
* Provider Configuration
- Networking Components
* Security Group
+ Deploying the Infrastructure
- Conclusion

  
## Introduction
In this guide, you’ll learn how to set up the core infrastructure required to launch an AWS EC2 instance using Terraform. This setup includes configuring a VPC, subnet, routing, and a security group to control access. Terraform will act as the Infrastructure-as-Code (IaC) tool that simplifies the process of creating and managing AWS resources in a repeatable manner.

## Prerequisites
Before proceeding, ensure you have the following:

* Terraform installed on your local machine.
* An AWS account with appropriate permissions to create resources such as VPCs, EC2 instances, and Security Groups.
* AWS CLI configured with access keys to interact with AWS services.

  
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
}
```


* Port 22: Used for SSH access, enabling remote management of the instance.
+ Port 80: Used for HTTP traffic, making the instance capable of serving web traffic.

## Deploying the Infrastructure
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
This Terraform configuration provides the basic networking setup needed to launch an EC2 instance with public internet access. It includes a VPC, subnet, Internet Gateway, routing, and security group configurations. You can extend this setup by adding EC2 instances, load balancers, or even an RDS database, depending on your needs.
