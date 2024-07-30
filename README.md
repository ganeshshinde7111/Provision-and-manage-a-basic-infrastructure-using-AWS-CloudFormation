# Provision-and-manage-a-basic-infrastructure-using-AWS-CloudFormation
This repository contains an AWS CloudFormation template and associated resources for creating a simple development environment. The project focuses on modifying an AWS CloudFormation template to set up an Apache web server with HTTP access, hosted in a dedicated VPC with a single public subnet and a small Amazon EC2 instance. 
The project involves:
   1. Using AWS Cloud9 to modify an existing CloudFormation template.
   2. Deploying a stack using the AWS CLI and AWS Management Console.
   3. Detecting drift in the stack after manual modifications.
   4. Updating the stack using CloudFormation's change set functionality.

##OBJECTIVES
 - Build AWS CloudFormation templates in YAML to define infrastructure resources.
 - Create a CloudFormation stack to provision infrastructure and review the outputs.
 - Detect changes made to your resources and generate drift reports.
 - Update your infrastructure using CloudFormation change sets.

![image](https://github.com/user-attachments/assets/4f4d9714-b309-4d69-91f3-e4d3576aa900)
using AWS Cloud9, we modify an AWS CloudFormation template to build a simple Amazon Virtual Private Cloud (Amazon VPC) environment and deploy a web server with a simple web page. Second, verifying that the webpage renders properly, we manually modify an Amazon EC2 Security Group and use the Detect drift command to detect the change. Lastly, we create a change set to update the Security Group with the expected values.
