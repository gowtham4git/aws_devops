# Deploying an AWS VM with Terraform
This guide provides an in-depth walkthrough of deploying an AWS Virtual Machine (VM) using Terraform. The deployment includes setting up essential networking components such as a Virtual Private Cloud (VPC), Subnets, an Internet Gateway, Route Tables, and Security Groups. This configuration is ideal for those looking to deploy basic infrastructure for a web application or remote SSH access.

## Table of Contents
* Introduction
+ Prerequisites
- Terraform Configuration
* Provider Configuration
- Networking Components
+ VPC
* Internet Gateway
+ Route Table
- Subnet
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

  
### Terraform Configuration
  
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






