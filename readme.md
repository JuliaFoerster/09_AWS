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
  
### Solution:
***AWS UI:***
- go to AWS/IAM Dashboard/User/Create User <br>
  also generate password (for AWS UI access) + download csv containing credentials
- go to AWS/IAM Dashboard/User/Jane/Create access key <br>
  generate Access Key ID and Access Key Secret (for console access) + download csv containing credentials)
- go to AWS/User Groups/Create Group/ + add Jane to user Group
- add permissions 'EC2FullAccess' to group devops. 
<br>

***AWS CLI:*** 
##### 1. Install AWS Client:<br>
<code>brew install awscli</code><br>
##### 2. Check for success:<br>
<code>awscli --version</code><br>
##### 3. Check if admin user has credentials  on my local machibe
<code> cat ~/.aws/config</code> <br>
if not: <code>aws configure</code>
<br>
##### 4. Create user 
<code>aws iam create-user --username jane</code>
##### 5. Create group 
<code>aws iam create-group --group-name devops2</code>
##### 6. Add use to group 
<code>aws iam add-user-to-group  --user-name jane --group-name devops2</code>
##### 7. Check if user is in group devops2
<code>aws iam get-group --group-name devops2</code>
##### 8. Give permission (policy) to create EC2 instance to users in group 
###### 8.1 Find policy identifier (for EC2 and VPC and all components under that service)
<code>aws iam list-policies --query 'Policies [?PolicyName==`AmazonEC2FullAccess`].Arn'</code> <br>
<code>aws iam list-policies --query 'Policies [?PolicyName==`AmazonVPCFullAccess`].Arn'</code>
###### 8.2 Attach policies (found above) to group
<code>aws iam attach-group-policy --group-name devops2 --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess</code> <br>
<code>aws iam attach-group-policy --group-name devops2 --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess</code>
###### 8.3 Validate
<code>aws iam list-attached-group-policies --group-name devops2</code>
</details>


<details>
<summary>  EXERCISE 2: Configure AWS CLI
</summary>
<br>
You want to use the AWS CLI for the following tasks. So, to be able to interact with the AWS account from the AWS Command Line tool you need to configure it correctly:<br>
<br>
- Set credentials for that user for AWS CLI<br>
- Configure correct region for your AWS CLI

### Solution: AWS UI Access (password)
##### 1. Configure password reset after first login
<code>aws iam create-login-profile --user jane --password <PASSWORD> --password-reset-required</code>
##### 2. Jane can't reset passwords -> Create permission for Jane
find policy ARN : <br>
<code>aws iam list-policies | grep Password</code><br>
"Arn": "arn:aws:iam::aws:policy/IAMUserChangePassword" <br>
<br>
<code>aws iam attach-user-policy --user-name jane --policy-arn arn:aws:iam::aws:policy/IAMUserChangePassword</code>
##### 3. Login UI + reset password
Login to UI requires: username, password and user id <br>
Find user ARN: <br>
<code>aws iam get-user --user-name jane</code><br>
"Arn": "arn:aws:iam::197796734648:user/jane"<br>
<br>
user id = 197796734648
<br>
### Solution: AWS CLI Access (Access Key ID, Access Key Secret)
##### 1. Save config file (keys) ~/.aws/credentials of admin user somewhere safe.
<code> mv ~/.aws/credentials ~/.aws/credentials_admin </code>
##### 2. Create config file for user jane
<code> aws iam create-access-key --user-name jane > key.txt</code><br>
OR via UI <br>
IAM/User/Jane/Create Access Key/Download csv file <br>
<code>aws configure</code>
##### 3. Validate credentials:
<code> cat ~/.aws/credentials</code>
<br>
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

### Solution:
##### ====== Create VPC and subnet  ========
##### 1. Create VPC:
<code> aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output text </code> <br>
Output: vpc-04411448155c5c404 <br>
<br>
##### 2. Create Subnet in VPC:
<code> aws ec2 create-subnet --vpc-id vpc-04411448155c5c404 --cidr-block 10.0.1.0/24 --query Subnet.SubnetId --output text </code>  <br>
Output: subnet-0dcd59104af3b4016 <br>
<br>
##### 3. Validate: 
<code>aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-04411448155c5c404"</code>
<br>
##### ====== Make  subnet public by attaching a internet gateway  ========
##### 1. Create Internet Gateway
<code>aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text</code>
igw-0943c735026803291 <br>

##### 2. Attach Internet Gateway to the VPC
<code> aws ec2 attach-internet-gateway --vpc-id vpc-04411448155c5c404 --internet-gateway-id igw-0943c735026803291 </code>

##### 3. Create Route Table (like a virtual router in our VPC)
<code>aws ec2 create-route-table --vpc-id vpc-04411448155c5c404 --query RouteTable.RouteTableId --output text</code>
rtb-01e4614195e247971 <br>

##### 4. Create Route rule for handling all traffic between internet & VPC
<code> aws ec2 create-route --route-table-id rtb-01e4614195e247971 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0943c735026803291 </code>

##### 5. Valide your custom route table has correct configuraton, 1 local and 1 interent gateway routes
<code>aws ec2 describe-route-tables --route-table-id rtb-01e4614195e247971</code>

    {
    "RouteTables": [
        {
            "Associations": [],
            "PropagatingVgws": [],
            "RouteTableId": "rtb-01e4614195e247971",
            "Routes": [
                {
                    "DestinationCidrBlock": "10.0.0.0/16",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                },
                {
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "GatewayId": "igw-0943c735026803291",
                    "Origin": "CreateRoute",
                    "State": "active"
                }
            ],
            "Tags": [],
            "VpcId": "vpc-04411448155c5c404",
            "OwnerId": "197796734648"
        }
    ]
    }
##### 6. Associate subnet with the route table to allow internet traffic in the subnet of our VPC
<code>aws ec2 associate-route-table  --subnet-id subnet-0dcd59104af3b4016 --route-table-id rtb-01e4614195e247971</code>
AssociationId": "rtbassoc-0c6d6c4b85d6b0f50"

##### ====== Create security group in the VPC to allow access on port 22 ======
##### 1. Find ID of the VPC created above:<br>
<code>aws ec2 describe-vpcs</code><br>
-> vpc-04411448155c5c404<br>
##### 2. Create Security Group for VPC
<code>aws ec2 create-security-group --group-name sgjane --description "My Sg" --vpc-id vpc-04411448155c5c404</code><br>
-> "GroupId": "sg-05b422fe026aaaa3e"<br>
##### 3. Create Rule to open port 22 for ssh
Allow access on port 22 from all sources<br>
<code>aws ec2 authorize-security-group-ingress --group-id sg-05b422fe026aaaa3e --protocol tcp --port 22 --cidr 0.0.0.0/0</code><br>
-> sgr-0d89f1f54c1cf0b67
##### 4. Validate ingress rule
<code>aws ec2  describe-security-groups --group-id sg-05b422fe026aaaa3e</code>
</details>


<details>
<summary>
EXERCISE 4: Create EC2 Instance
</summary>
<br>
Once the VPC is created, using the AWS CLI, you:<br>
Create an EC2 instance in that VPC with the security group you just created and ssh key file<br>
<br>
aws ec2 create-key-pair --key-name JanesKeyPair --query 'KeyMaterial' --output text>MyKeyPair.pem
<br>
aws ec2 run-instances --image-id ami-081dce9b7a9078540 --count 1 --instance-type t2.micro  --key-name MyKeyPair.pem --security-group-id sg-05b422fe026aaaa3e --subnet-id subnet-0dcd59104af3b4016
<br>

### Solution: 
</details>

<details>
<summary>EXERCISE 5: SSH into the server and install Docker on it
</summary>
<br>
Once the EC2 instance is created successfully, you want to prepare the server to run Docker containers. So you:
<br>
- ssh into the server and <br>
- install Docker on it to run the dockerized application later

### Solution: 
</details>

