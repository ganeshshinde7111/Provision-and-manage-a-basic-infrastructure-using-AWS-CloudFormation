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

AWS CloudFormation provides a common language to model and provision resources in your cloud environment. You code your infrastructure with the CloudFormation template language, in either YAML or JSON format. Both YAML and JSON are data serialization languages but with different features. AWS CloudFormation only processes JSON; templates formatted as YAML are converted to JSON when the stack create is initiated.

For this step, we use YAML to code the CloudFormation templates.

**2.1: Add a parameter to the template**

