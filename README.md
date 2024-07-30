# Provision-and-manage-a-basic-infrastructure-using-AWS-CloudFormation
This repository contains an AWS CloudFormation template and associated resources for creating a simple development environment. The project focuses on modifying an AWS CloudFormation template to set up an Apache web server with HTTP access, hosted in a dedicated VPC with a single public subnet and a small Amazon EC2 instance.
The project involves:
     1. Using AWS Cloud9 to modify an existing CloudFormation template.
     2. Deploying a stack using the AWS CLI and AWS Management Console.
     3. Detecting drift in the stack after manual modifications.
     4. Updating the stack using CloudFormation's change set functionality.

## Objectives
 - Build AWS CloudFormation templates in YAML to define infrastructure resources.
 - Create a CloudFormation stack to provision infrastructure and review the outputs.
 - Detect changes made to our resources and generate drift reports.
 - Update your infrastructure using CloudFormation change sets.

![image](https://github.com/user-attachments/assets/4f4d9714-b309-4d69-91f3-e4d3576aa900)

using AWS Cloud9, we modify an AWS CloudFormation template to build a simple Amazon Virtual Private Cloud (Amazon VPC) environment and deploy a web server with a simple web page. Second, verifying that the webpage renders properly, we manually modify an Amazon EC2 Security Group and use the Detect drift command to detect the change. Lastly, we create a change set to update the Security Group with the expected values.

## Step 1: Connect to the AWS Cloud9 IDE
        1.1: Explore the Cloud9 interface.
        1.2: Verify that the AWS CLI is installed in the Cloud9 environment.
        1.3: Explore the basics of a CloudFormation template.

## Step 2: Create an AWS CloudFormation template
        2.1: Add a parameter to the template.
        2.2: Add a resource to the template.
        2.3: Add an output to the template.

## Step 3: Provision an infrastructure using the stack’s CloudFormation template
        3.1: Run create-stack using the lab1.yaml template.
        3.2: Verify the status of the stack.
        3.3: Review the stack resources created.

## Step 4: Detect drift in a CloudFormation stack
        4.1: Modify our environment.
        4.2: Verify security group settings have changed.
        4.3: Generate a drift report.
        4.4: View drift detection results via the CLI.

## Step 5: Update the stack using a change set
        5.1: Modify the security group rules.
        5.2: Create the change set.
        5.3: Review the changes.
        5.4: Run the change set.
        5.5: Verify the app URL is working.


Feel free to reach out for any assistance or clarification. Happy deploying!
   
