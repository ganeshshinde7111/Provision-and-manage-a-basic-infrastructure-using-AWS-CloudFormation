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
- check 2.1 step in lab1.yaml and replace with below parameter,
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

**2.2: Add a resource to the template**
- In this step, we add a Resource to the lab1.yaml template in the Cloud9 environment.
- check 2.2 step in lab1.yaml and replace with below resource,
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

**2.3: Add an output to the template**

- In this step, we add an Output to the lab1.yaml template in the Cloud9 environment.

- Check 2.3 step in lab1.yaml and replace with below output,
```
  AppURL:
    Description: Newly created application URL
    Value: !Sub 'http://${WebServerInstance.PublicIp}'
```
**NOTE:** The intrinsic !Sub function is used in the AppURL output definition above to return a usable URL when stack creation is completed.

- On the File menu, select **Save** to save the template.

## Step 3: Provision an infrastructure using the stack’s CloudFormation template.
In this step, we launch the stack creation process from the AWS CLI and review all of the resources created on the CloudFormation console.

**3.1: Run CreateStack using the lab1.yaml template.**

- In this step, we run the create-stack command to initiate a stack creation from the CLI.

- In the AWS Cloud9 terminal, ensure you are in the templates folder by running the following command:
```
cd ~/environment/templates
```

- From the terminal, run the following command to launch the stack creation process using inline parameters to match the template definition.
```
aws cloudformation create-stack --stack-name Lab1 --parameters ParameterKey=InstanceType,ParameterValue=t2.micro --template-body file://lab1.yaml
```

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
**NOTE:** The describe-stacks command returns a large amount of information to the terminal. It presents information on every resource defined by our template, the current status in the build process, and specific attributes of the resource that are available at the time that we run describe-stacks. The AWS CloudFormation dashboard presents the same information, but in a friendlier format.

- It takes CloudFormation a few minutes to complete the create-stack process.

- To query the status of the stack creation process, from the terminal, run the following command:
```
aws cloudformation describe-stacks --stack-name Lab1 --query "Stacks[0].StackStatus"
```

- Return to the Environments browser tab to view the status in the console.

- At the top of the AWS Management Console, in the search bar, search for and choose CloudFormation.

- In the list of stacks, locate Lab1. Note the status of your Create Stack job.

**3.3: Review the stack resources created**
In this step, we review the Lab1 stack and exploring the Stack Info, Events, Resources and Outputs generated from the stack creation in the console.

- In the CloudFormation console explore the stack events.

- Select the stack and explore the information available on each tab: Stack Info, Events, Resources and Outputs.

- On the Resources tab, a list of resources defined in the template are created. Identify some key resources we have created. Each resource has a unique Logical ID, Physical ID, Type, and Status.

- The following resources should be created:
*AttachGateway*
*IPAddress*
*InboundHTTPNetworkAclEntry*
*InboundNetworkAclEntry*
*InboundResponsePortsNetworkAclEntry*
*InstanceSecurityGroup*
*InternetGateway*
*NetworkAcl*
*OutBoundHTTPNetworkAclEntry*
*OutBoundHTTPSNetworkAclEntry*
*OutBoundResponsePortsNetworkAclEntry*
*Route*
*RouteTable*
*Subnet*
*SubnetNetworkAclAssociation*
*SubnetRouteTableAssociation*
*VPC*
*WebServerInstance*

- When the status of the Create Stack job shows CREATE_COMPLETE, open the Outputs tab.

- From the AppURL row, copy and paste the URL shown in the Value column in a new browser tab.

- A webpage displaying a time-out error is expected to appear with the following message: **This site can’t be reached.**

**NOTE:** The Public IP cannot be accessed at this time due to a Security Group restriction defined in the template. In the next step, this is fixed by modifying the Security Group resource.

You have successfully provisioned a simple infrastructure using a CloudFormation template and identified a limitation in the Security Group resource.
