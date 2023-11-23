# AWS

<details>
<summary> EXERCISE 1: Create IAM user.
</summary>
  <br>
  First of all, you need an IAM user with correct permissions to execute the tasks below.<br>
  <ul>
  <li> Create a new IAM user using "Jane" as a username and "devops" as the user-group</li>
  <li> Give the "devops" group all needed permissions to execute the tasks below - with login and CLI credentials</li>
</ul>
  Note: Do that using the AWS UI with Admin User
  
##### Solution
***AWS UI:***
- go to AWS/IAM Dashboard/User/Create User <br>
  also generate password (for AWS UI access) + download csv containing credentials
- go to AWS/IAM Dashboard/User/Jane/Create access key <br>
  also  generate Access Key ID and Access Key Secret (for console access) + download csv containing credentials)
- go to AWS/User Groups/Create Group/ + add Jane to user Group
- WHERE DID I ADD PERMISSION 'Administrator Access' (in group or user?)
<br>

***AWS CLI:*** 
- ToDo
</details>

<details>
<summary>  EXERCISE 2: Configure AWS CLI
</summary>
<br>
You want to use the AWS CLI for the following tasks. So, to be able to interact with the AWS account from the AWS Command Line tool you need to configure it correctly:<br>
<br>
- Set credentials for that user for AWS CLI<br>
- Configure correct region for your AWS CLI<br>

##### Solution
Install AWS Client:<br>
<code>brew install awscli</code><br>
<br>
Check for success:<br>
<code>awscli --version</code><br>
<br>
Use downloaded csv files (created in Exercise 1) containing:<br>
- credentials (password) of user jane<br>
- access key ID and access key secret of user jane<br>
<br>
<code>aws configure</code>
<br>
The console will ask for:
<code>
AWS Access Key ID [None]: see csv file<br>
AWS Secret Access Key [None]: see csv file<br>
Default region name [eu-west-3]:  eu-west-3<br>
Default output format [json]: json<br>
</code>
</details>

<details>
<summary>
EXERCISE 3: Create VPC
</summary>
<br>
You want to create the EC2 Instance in a dedicated VPC, instead of using the default one. So, using the AWS CLI, you:<br>
<br>
<ul>
 <li>create a new VPC with 1 subnet </il>
 <li>create a security group in the VPC that will allow you access on ssh port 22 and will allow browser access to your Node application </il>
</lu>

#### Solution
<br> 
- Create VPC: <br> 
<code> aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text </code> <br>
Output: vpc-0d903b38f28d30648 <br>
<br>
- Create Subnet in VPC: <br>
<code> aws ec2 create-subnet --vpc-id vpc-0d903b38f28d30648 --cidr-block 10.0.1.0/16 --query Subnet.SubnetId --output text </code>  <br>
Output: subnet-0006e16700ea86e00 <br> 
</details>
