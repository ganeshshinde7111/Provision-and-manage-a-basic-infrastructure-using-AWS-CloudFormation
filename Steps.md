## Step 1: Connect to the AWS Cloud9 IDE
In this step, we connect to the AWS Cloud9 integrated development environment (IDE).

**1.1: Explore the Cloud9 Interface.**
- **Editor:** The top pane to the right is the editor. The editor is where we can do things such as write code, run terminal sessions, and change IDE settings. It is currently displaying the AWS Cloud9 Welcome Screen.
- **File Tree:** The left pane is the file tree. It shows a list of all files that are currently available and open in the editor.
  
![image](https://github.com/user-attachments/assets/cfc8a564-970a-4a5f-97c7-6512341980ad)


**1.2: Verify that the AWS CLI is installed in the Cloud9 Environment.**
- To verify the AWS CLI is installed and to display its version, run the following command in a terminal window:

        aws --version

- After that we get below Output,

![image](https://github.com/user-attachments/assets/bf697d05-db68-4c7f-b94e-8a777d82a7e5)


**1.3: Explore the basics of a CloudFormation Template**
- For basics of CloudFormation Template, we refer to the AWS CloudFormation documentation template-anatomy. https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy


## Step 2: Create an AWS CloudFormation template.
In this step, we use AWS Cloud9 to create an AWS CloudFormation template, and use that template to create a CloudFormation stack.

AWS CloudFormation provides a common language to model and provision resources in cloud environment. We code infrastructure with the CloudFormation template language, in either YAML or JSON format. Both YAML and JSON are data serialization languages but with different features. AWS CloudFormation only processes JSON; templates formatted as YAML are converted to JSON when the stack create is initiated.

For this step, we use YAML to code the CloudFormation templates.

**2.0: Create the AWS CloudFormation template**
- Create file name lab1.yaml under templates folder using AWS Cloud9.
- Copy below code and paste in lab1.yaml file.
  
```
AWSTemplateFormatVersion: 2010-09-09
Description: >-
  AWS CloudFormation Simple Infrastructure Template
  VPC_Single_Instance_In_Subnet: This template will show how to create a VPC and
  add an EC2 instance with an Elastic IP address and a security group.
Parameters:
  VPCCIDR:
    Description: CIDR Block for VPC
    Type: String
    Default: 10.199.0.0/16
    AllowedValues:
      - 10.199.0.0/16
  PUBSUBNET1:
    Description: Public Subnet 1
    Type: String
    Default: 10.199.10.0/24
    AllowedValues:
      - 10.199.10.0/24
  ## STEP 2.1 - BEGIN: Add the parameter definition for InstanceType
  InstanceType:
    Description:
    Type: 
    Default: 
    AllowedValues:
    ConstraintDescription:
  ## STEP 2.1 - END
  LatestAmiId:
    Description: Find the current AMI ID using System Manager Parameter Store
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

Resources:
  VPC:
    Type: 'AWS::EC2::VPC'
    Properties:
      CidrBlock: !Ref VPCCIDR
      EnableDnsSupport: 'true'
      EnableDnsHostnames: 'true'
      Tags:
        - Key: Application
          Value: !Ref 'AWS::StackId'
        - Key: Name
          Value: CF lab environment
  Subnet:
    Type: 'AWS::EC2::Subnet'
    DependsOn: VPC
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Ref PUBSUBNET1
      MapPublicIpOnLaunch: 'true'
      AvailabilityZone: !Select 
        - '0'
        - !GetAZs ''
      Tags:
        - Key: Application
          Value: !Ref 'AWS::StackId'
        - Key: Name
          Value: Public Subnet
  InternetGateway:
    Type: 'AWS::EC2::InternetGateway'
    DependsOn: VPC
    Properties:
      Tags:
        - Key: Application
          Value: !Ref 'AWS::StackId'
  AttachGateway:
    Type: 'AWS::EC2::VPCGatewayAttachment'
    DependsOn: VPC
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  RouteTable:
    Type: 'AWS::EC2::RouteTable'
    DependsOn: VPC
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Application
          Value: !Ref 'AWS::StackId'

  ## Route resource
  # STEP 2.2 - BEGIN: Add the resource definition for ROUTE
  Route:
    Type: 
    DependsOn:
    Properties:
  # STEP 2.2 - END
  SubnetRouteTableAssociation: 
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    DependsOn:
      - VPC
      - InternetGateway
    Properties:
      SubnetId: !Ref Subnet
      RouteTableId: !Ref RouteTable
  NetworkAcl:
    Type: 'AWS::EC2::NetworkAcl'
    DependsOn:
      - VPC
      - InternetGateway
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Application
          Value: !Ref 'AWS::StackId'
  InboundHTTPNetworkAclEntry:
    Type: 'AWS::EC2::NetworkAclEntry'
    DependsOn:
      - VPC
      - InternetGateway
    Properties:
      NetworkAclId: !Ref NetworkAcl
      RuleNumber: '100'
      Protocol: '6'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 0.0.0.0/0
      PortRange:
        From: '80'
        To: '80'
  InboundNetworkAclEntry:
    Type: 'AWS::EC2::NetworkAclEntry'
    DependsOn:
      - VPC
      - InternetGateway
    Properties:
      NetworkAclId: !Ref NetworkAcl
      RuleNumber: '101'
      Protocol: '6'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 0.0.0.0/0
      PortRange:
        From: '22'
        To: '22'
  InboundResponsePortsNetworkAclEntry:
    Type: 'AWS::EC2::NetworkAclEntry'
    DependsOn:
      - VPC
      - InternetGateway
    Properties:
      NetworkAclId: !Ref NetworkAcl
      RuleNumber: '102'
      Protocol: '6'
      RuleAction: allow
      Egress: 'false'
      CidrBlock: 0.0.0.0/0
      PortRange:
        From: '1024'
        To: '65535'
  OutBoundHTTPNetworkAclEntry:
    Type: 'AWS::EC2::NetworkAclEntry'
    DependsOn:
      - VPC
      - InternetGateway
    Properties:
      NetworkAclId: !Ref NetworkAcl
      RuleNumber: '100'
      Protocol: '6'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
      PortRange:
        From: '80'
        To: '80'
  OutBoundHTTPSNetworkAclEntry:
    Type: 'AWS::EC2::NetworkAclEntry'
    DependsOn:
      - VPC
      - InternetGateway
    Properties:
      NetworkAclId: !Ref NetworkAcl
      RuleNumber: '101'
      Protocol: '6'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
      PortRange:
        From: '443'
        To: '443'
  OutBoundResponsePortsNetworkAclEntry:
    Type: 'AWS::EC2::NetworkAclEntry'
    DependsOn:
      - VPC
      - InternetGateway
    Properties:
      NetworkAclId: !Ref NetworkAcl
      RuleNumber: '102'
      Protocol: '6'
      RuleAction: allow
      Egress: 'true'
      CidrBlock: 0.0.0.0/0
      PortRange:
        From: '1024'
        To: '65535'
  SubnetNetworkAclAssociation:
    Type: 'AWS::EC2::SubnetNetworkAclAssociation'
    Properties:
      SubnetId: !Ref Subnet
      NetworkAclId: !Ref NetworkAcl
  IPAddress:
    Type: 'AWS::EC2::EIP'
    DependsOn: AttachGateway
    Properties:
      Domain: vpc
      InstanceId: !Ref WebServerInstance
  InstanceSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      VpcId: !Ref VPC
      GroupDescription: Enable HTTP via port 80
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
    # STEP 5.1 - BEGIN: Change IP Range to 0.0.0.0/0
          CidrIp: 1.1.1.1/32
    # STEP 5.1 - END
  WebServerInstance:
    Type: 'AWS::EC2::Instance'
    DependsOn: AttachGateway
    Metadata:
      Comment: Install a simple application
      'AWS::CloudFormation::Init':
        config:
          packages:
            yum:
              httpd: []
          files:
            /var/www/html/index.html:
              content: !Join 
                - |+

                - - >-
                    <h1>Congratulations, you have successfully deployed a simple
                    infrastructure using AWS CloudFormation.</h1>
              mode: '000644'
              owner: root
              group: root
            /etc/cfn/cfn-hup.conf:
              content: !Join 
                - ''
                - - |
                    [main]
                  - stack=
                  - !Ref 'AWS::StackId'
                  - |+

                  - region=
                  - !Ref 'AWS::Region'
                  - |+

              mode: '000400'
              owner: root
              group: root
            /etc/cfn/hooks.d/cfn-auto-reloader.conf:
              content: !Join 
                - ''
                - - |
                    [cfn-auto-reloader-hook]
                  - |
                    triggers=post.update
                  - >
                    path=Resources.WebServerInstance.Metadata.AWS::CloudFormation::Init
                  - 'action=/opt/aws/bin/cfn-init -v '
                  - '         --stack '
                  - !Ref 'AWS::StackName'
                  - '         --resource WebServerInstance '
                  - '         --region '
                  - !Ref 'AWS::Region'
                  - |+

                  - |
                    runas=root
              mode: '000400'
              owner: root
              group: root
          services:
            sysvinit:
              httpd:
                enabled: 'true'
                ensureRunning: 'true'
              cfn-hup:
                enabled: 'true'
                ensureRunning: 'true'
                files:
                  - /etc/cfn/cfn-hup.conf
                  - /etc/cfn/hooks.d/cfn-auto-reloader.conf
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !Ref LatestAmiId
      Tags:
        - Key: Application
          Value: !Ref 'AWS::StackId'
        - Key: Name
          Value: Lab Host
      NetworkInterfaces:
        - GroupSet:
            - !Ref InstanceSecurityGroup
          AssociatePublicIpAddress: 'true'
          DeviceIndex: '0'
          DeleteOnTermination: 'true'
          SubnetId: !Ref Subnet
      UserData: !Base64 
        'Fn::Join':
          - ''
          - - |
              #!/bin/bash -xe
            - |
              yum update -y aws-cfn-bootstrap
            - '/opt/aws/bin/cfn-init -v '
            - '         --stack '
            - !Ref 'AWS::StackName'
            - '         --resource WebServerInstance '
            - '         --region '
            - !Ref 'AWS::Region'
            - |+

            - '/opt/aws/bin/cfn-signal -e $? '
            - '         --stack '
            - !Ref 'AWS::StackName'
            - '         --resource WebServerInstance '
            - '         --region '
            - !Ref 'AWS::Region'
            - |+

    CreationPolicy:
      ResourceSignal:
        Timeout: PT15M
##Outputs
Outputs:
  MySecurityGroup:
    Description: Application instance's security group name
    Value: 
        !GetAtt 
          - InstanceSecurityGroup
          - GroupId
# STEP 2.3 - BEGIN: Add the output definition for URL
  AppURL:
    Description: 
    Value: 
# STEP 2.3 - END
```

**2.1: Add a parameter to the template**
- In this step, we update a Parameter in the lab1.yaml template in the Cloud9 environment.
- Check 2.1 step in lab1.yaml and replace with below parameter,
```
  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.nano
    AllowedValues:
      - t2.nano
      - t2.micro
      - t2.small
    ConstraintDescription: must be a valid EC2 instance type.
```
![image](https://github.com/user-attachments/assets/a65ba04e-a303-4164-b4ea-badb2a7340d9)


**2.2: Add a resource to the template**
- In this step, we add a Resource to the lab1.yaml template in the Cloud9 environment.
- Check 2.2 step in lab1.yaml and replace with below resource,
```
  Route:
    Type: 'AWS::EC2::Route'
    DependsOn:
      - VPC
      - AttachGateway
    Properties:
      RouteTableId: !Ref RouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
```
![image](https://github.com/user-attachments/assets/a28ee6f1-53c6-4bc9-ad9e-54a4d1775b91)


**2.3: Add an output to the template**
- In this step, we add an Output to the lab1.yaml template in the Cloud9 environment.
- Check 2.3 step in lab1.yaml and replace with below output,
```
  AppURL:
    Description: Newly created application URL
    Value: !Sub 'http://${WebServerInstance.PublicIp}'
```
![image](https://github.com/user-attachments/assets/62fd4857-196f-47b2-b760-a50a8d3a4dd7)


**NOTE: The intrinsic !Sub function is used in the AppURL output definition above to return a usable URL when stack creation is completed.**

- On the File menu, select **Save** to save the template.


## Step 3: Provision an infrastructure using the stack’s CloudFormation template.
In this step, we launch the stack creation process from the AWS CLI and review all of the resources created on the CloudFormation console.

**3.1: Run CreateStack using the lab1.yaml template.**
- In this step, we run the create-stack command to initiate a stack creation from the CLI.
- In the AWS Cloud9 terminal, ensure you are in the templates folder by running the following command:
```
cd ~/environment/templates
```
![image](https://github.com/user-attachments/assets/f01a4572-f283-466f-8e85-d7067f5632c1)


- From the terminal, run the following command to launch the stack creation process using inline parameters to match the template definition.
```
aws cloudformation create-stack --stack-name Lab1 --parameters ParameterKey=InstanceType,ParameterValue=t2.micro --template-body file://lab1.yaml
```
![image](https://github.com/user-attachments/assets/31ea2ca0-f0a1-4d40-8f64-dec7f19511d1)

- The create-stack command calls the specified AWS CloudFormation template and initiates a stack creation. The parameter InstanceType tells CloudFormation what size Amazon EC2 instance to deploy.
- Successfully running create-stack returns a StackId at the command line.


**If your stack creation fails, refer to the troubleshooting steps:**

If your stack creation fails, the AWS CloudFormation console provides detailed information that we can use to identify the cause of the stack creation failure. Use that information to correct the template. When we have corrected the issue use the following command to delete the failed stack.

- Return to the AWS Cloud9 terminal and run the following command:
```
aws cloudformation delete-stack --stack-name Lab1
```
- Return to the AWS CloudFormation dashboard to observe your stack deletion. When the delete-stack has completed return to step 3.1 and re-run the create-stack command.


**3.2: Verify the status of the stack.**

In this step, we query the status of the stack creation process by running the describe-stack command in the CLI and viewing the process in the console.

- From the terminal, run the following command:
```
aws cloudformation describe-stacks --stack-name Lab1
```
![image](https://github.com/user-attachments/assets/55ebf489-36d7-4596-a70b-1074b95f3e47)

**NOTE: The describe-stacks command returns a large amount of information to the terminal. It presents information on every resource defined by our template, the current status in the build process, and specific attributes of the resource that are available at the time that we run describe-stacks. The AWS CloudFormation dashboard presents the same information, but in a friendlier format.**


- It takes CloudFormation a few minutes to complete the create-stack process.
- To query the status of the stack creation process, from the terminal, run the following command:
```
aws cloudformation describe-stacks --stack-name Lab1 --query "Stacks[0].StackStatus"
```
![image](https://github.com/user-attachments/assets/081d05da-3cc1-4893-ae2a-8aca164649d7)

- Return to the Environments browser tab to view the status in the console.
- At the top of the AWS Management Console, in the search bar, search for and choose CloudFormation.
- In the list of stacks, locate Lab1. Note the status of your Create Stack job.


**3.3: Review the stack resources created**

In this step, we review the Lab1 stack and exploring the Stack Info, Events, Resources and Outputs generated from the stack creation in the console.
![image](https://github.com/user-attachments/assets/1454654a-910d-4b10-8de2-2d2c76580895)
- In the CloudFormation console explore the stack events.
- Select the stack and explore the information available on each tab: Stack Info, Events, Resources and Outputs.
- On the Resources tab, a list of resources defined in the template are created. Identify some key resources we have created. Each resource has a unique Logical ID, Physical ID, Type, and Status.
- The following resources should be created:
  
  * AttachGateway
  * IPAddress
  * InboundHTTPNetworkAclEntry
  * InboundNetworkAclEntry
  * InboundResponsePortsNetworkAclEntry
  * InstanceSecurityGroup
  * InternetGateway
  * NetworkAcl
  * OutBoundHTTPNetworkAclEntry
  * OutBoundHTTPSNetworkAclEntry
  * OutBoundResponsePortsNetworkAclEntry
  * Route
  * RouteTable
  * Subnet
  * SubnetNetworkAclAssociation
  * SubnetRouteTableAssociation
  * VPC
  * WebServerInstance

- When the status of the Create Stack job shows CREATE_COMPLETE, open the Outputs tab.
- From the AppURL row, copy and paste the URL shown in the Value column in a new browser tab.
- A webpage displaying a time-out error is expected to appear with the following message: **This site can’t be reached.**
![image](https://github.com/user-attachments/assets/6ee9940b-5381-41a9-a7f5-47397c191a11)

**NOTE: The Public IP cannot be accessed at this time due to a Security Group restriction defined in the template. In the next step, this is fixed by modifying the Security Group resource.**

You have successfully provisioned a simple infrastructure using a CloudFormation template and identified a limitation in the Security Group resource.


## Step 4: Detect drift in a CloudFormation stack
In this task, we use AWS CloudFormation to detect changes that CloudFormation didn’t make. Make a change to your environment, instruct CloudFormation to detect any drift and then view the results.

Users can change resources outside of AWS CloudFormation. Drift detection can be used to identify stack resources that have been modified outside of AWS CloudFormation management.

**4.1: Modify environment to detect drift**

In this step, we modify the security group rules in the EC2 console.

- Open EC2 console.
- On the EC2 console, in the navigation pane, choose Security Groups and select the security group for your simple infrastructure.
- On the Inbound rules tab choose Edit inbound rules .
- On the Edit inbound rules page, on the listed Security group rule ID modify the Source type parameter by selecting Anywhere-IPv4 from the drop-down menu.
- Choose Save rules .


**4.2: Verify Security Group settings have changed**

In this step, we verify the changes made to the security group have allowed we to access the previously unreachable webpage.

- Return to the AWS Management Console, use the AWS search bar to search for CloudFormation and then choose the service from the list of results.
- Locate the Lab1 stack in the console view.
- On the Outputs tab, launch the URL shown in a new browser tab.
- When loaded, a webpage is displayed with the following message: Congratulations, you have successfully deployed a simple infrastructure using AWS CloudFormation.
![image](https://github.com/user-attachments/assets/f54e4869-5685-41f5-9a34-2fcefe07d90b)

**NOTE: Notice that changing the source IP address on the Security Group allowed traffic into the webpage URL. Modifying template resources via the console is a quick-fix but not a best practice.**


**4.3: Generate a drift report**

In this step, we identify and detect the drift in the template via the AWS Management console.

- Return to the CloudFormation > Stacks > Lab 1 browser tab.
- In the stack details pane, choose the Stack actions drop-down menu and then choose **Detect drift** .
![image](https://github.com/user-attachments/assets/1067f783-3c99-42dd-98a5-59828ee8a6f4)


- With stack selected, from the Stack actions drop-down menu, select **View drift results**.
![image](https://github.com/user-attachments/assets/d8e77eac-dc10-4cf8-96c7-e948a1428fe5)


- In the Resource drift status section, select the InstanceSecurityGroup resource that has the status of **MODIFIED**.
![image](https://github.com/user-attachments/assets/5a4b6abb-a781-461e-b175-e44a0941925c)


- Select **View drift details** to check more about what resources that changed.
![image](https://github.com/user-attachments/assets/92a54c61-a5b5-4246-b0b9-973dcfd6980a)

Drift detection allows us to detect whether a stack’s actual configuration differs from its expected configuration. A resource is considered to have drifted if any of its actual property values differ from the expected property values. This includes if the property or resource has been deleted. In this lab we manually changed a resource value. The most direct manner to address this drift is to manually modify the resource back to the expected value. If the change is due to the deployment of another stack, rollback the changes of the other stack. A last resort and the most destructive is to delete stack and redeploy it.


**4.4: View drift detection results via the CLI**

In this step, we identify and detect the drift in the template via the Cloud9 CLI.

- Return to the AWS Cloud9 terminal to review the details of the stack drift detection operation using AWS CLI.
- In the AWS Cloud9 terminal, enter the AWS CLI CloudFormation describe-stack-resource-drifts command with the following parameters:
  –stack-name [Paste the stack name here]
  –stack-resource-drift-status-filters MODIFIED DELETED

```
aws cloudformation describe-stack-resource-drifts --stack-name Lab1 --stack-resource-drift-status-filters MODIFIED DELETED
```
![image](https://github.com/user-attachments/assets/e65ef56e-ed0d-4804-9567-01b296acef6f)

- In the results, look for the Resource Type SecurityGroup and then look for the PropertyDifferences. The results should mirror the drift information that was displayed on the AWS CloudFormation dashboard.
- We have successfully detected a drift in the CloudFormation template.
![image](https://github.com/user-attachments/assets/a65b5597-ffa0-437c-be6d-8b3343946947)


## Step 5: Update the stack using a change set
In the previous step, we manually changed a resource value. The best practice to change the environment is through the template updates. Template updates can be easily done using a changeset.

In this step, we modify the Security Group resource to the expected value in the template. we edit the lab1 template modifying the SecurityGroupRules, review those changes as part of the Change Set, and then implement the changes to the environment.

**5.1: Modify the Security Group Rules**

Earlier in step 3, the InstanceSecurityGroup was modified outside of the CloudFormation template. In this step, a change is made in the template for the stack to implement the modification when creating.

- In the AWS Cloud9 Environment pane, open the context menu for lab1.yaml file you edited earlier and choose Duplicate.
- Rename the duplicate file lab1-CS.yaml, and open the file in the AWS Cloud9 editor.
<img align="left" width="100" height="100" src="![image](https://github.com/user-attachments/assets/56d1f645-04e6-4a71-a5f1-3ff121e9328c)">


  Modify the InstanceSecurityGroup resource to allow access to the AppURL:
- Under the SecurityGroupIngress property value, note the CidrIP listed as 1.1.1.1/32
- In order to make the webpage accessible, change the CidrIP value to 0.0.0.0/0
![image](https://github.com/user-attachments/assets/d8617233-f189-4d48-9bc8-aad14e05107c)


- Save the file.


**5.2: Create the change set**

In this step, we run the create-change-set command to create a change set for the lab1.yaml template.

- In the AWS Cloud9 terminal, ensure you are in the templates folder by running the following command:
```
cd ~/environment/templates
```
![image](https://github.com/user-attachments/assets/d8443516-4187-4d3c-9d86-203d7c220c7d)

- From the terminal, run the following command to launch the stack change set process:
```
aws cloudformation create-change-set --stack-name Lab1 --change-set-name Lab1ChangeSet --parameters ParameterKey=InstanceType,ParameterValue=t2.micro --template-body file://lab1-CS.yaml
```
![image](https://github.com/user-attachments/assets/616dfa72-229c-4b96-bf44-257a99aaa233)

- After the change set is processed, AWS CloudFormation returns a StackId and Id. To see the changes that you have staged, return to the CloudFormation browser tab.


**If your change set creation fails, refer to the troubleshooting steps:**

If change set creation fails, the AWS CloudFormation console provides detailed information that we can use to identify the cause of the failure. Use that information to correct template and deploy the change set again.

When we have corrected the issue use the following command to delete your change set:
```
aws cloudformation delete-change-set --change-set-name Lab1ChangeSet --stack-name Lab1
```
Return to the AWS CloudFormation dashboard to observe change set deletion. When the delete-change-set has completed re-run the execute-change-set command.

- To create the change set, return back to Step 5.2


**5.3: Review the changes**

In this step, we review the details of the change set in the console.

- Choose Lab1 from the top of the drift results window to return to stack.
- On the Change sets tab, select Lab1ChangeSet. Review the details of the change-set in the Changes tab.
![image](https://github.com/user-attachments/assets/d4af1514-364c-441d-917c-1c07c76f68a0)


**5.4: Run the change set**

In this step, we run the change set created using the console.

- In the Lab1ChangeSet window, verify the changes are as expected for the InstanceSecurityGroup resource.
- At the top-right of the page, choose **Execute change set**.
![image](https://github.com/user-attachments/assets/dc8f2942-c59b-4883-8601-d3359ae9bb17)

- A pop-up window prompts you to choose how to handle resources in the event of a stack failure.
- With the default Roll back all stack resources chosen, select **Execute change set**.
![image](https://github.com/user-attachments/assets/c4984d40-6011-41c9-8b13-c0dbc288550e)

- Wait for the stack update to complete and the Status to change to **UPDATE_COMPLETE**.
  Lab1ChangeSet is no longer available, and there is a new entry under Last executed change set.
- Select the Change set id and review the information.
  * Status as **CREATE_COMPLETE**
  * Execution Status as **EXECUTE_COMPLETE**


**5.5: Verify the AppURL is working**

In this step, we access the AppURL link to ensure the security group change made is successfully implemented.

- Return to the Environments browser tab, return to the Lab1 stack console view.
- On the Outputs tab, launch the URL shown in a new browser tab.
- When loaded, a webpage is displayed with the following message: **“Congratulations, you have successfully deployed a simple infrastructure using AWS CloudFormation”.**
![image](https://github.com/user-attachments/assets/be3a9a88-c9f4-47ba-8626-25a55708d687)

