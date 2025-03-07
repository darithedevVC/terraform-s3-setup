# Terraform S3 Setup 🚀

As of 2022, the top three cloud providers Amazon (AWS), Microsoft (Azure), and Google (GCP) accounted for 66% of cloud services revenue worldwide, with AWS as number one on that list.[[1]](https://techcrunch.com/2023/02/06/even-as-cloud-infrastructure-market-growth-slows-microsoft-continues-to-gain-on-amazon/#:~:text=The%20Big%20Three%20cloud%20providers,SaaS%2C%20which%20is%20measured%20separately.) 
<br><br>
Cloud professionals are expected to be familiar with several frameworks and tools used daily in cloud development. I am dedicating time to learning about how to use cloud development in my projects, focusing on deploying, scaling, amd managing applications efficiently. For this project, I set up a cloud account on AWS and learned the basics of Terraform. I encourage you to follow along with what I learned. 

### What is Terraform?

Terraform is an infrastructre as code (IaC) tool developed by Hashicorp that allows you to manage cloud infrastructure with configuration files (Json or HCL). Terraform simplifies infrastructure management in any cloud environment and has the ability to create, change, and manage with version control across different environments and teams. 

# What I did

**Cloud Account Setup & Terraform Basics**
1. I created a new free tier [AWS account](https://aws.amazon.com).
1. I also created an [IAM Identity Center User](https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html) (formerly SSO) and assigned it permission
2. I downloaded and configured [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) using the GUI installer
3. I installed [Terraform CLI](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) using Homebrew 
4. I wrote my first [Terraform configuration](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-build) !!!! :D
5. but the configuration kept failing so I googled alot [[2]](https://developer.hashicorp.com/terraform/intro) [[3]](https://www.youtube.com/watch?v=-_XwzwMeqGY) [[4]](https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_iam-s3.html)[[5]](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/infrastructure-as-code) >:
6. Oops no S3 container ("bucket") but theres an EC2 instance running???
7. And then VICTORY! ```terraform apply``` [finally worked!](https://kodekloud.com/blog/how-to-create-aws-s3-bucket-using-terraform/) and I created my first S3 bucket :D
8. ```terraform destroy```
8. Documentation of my process and experiences
<br><br>**Bonus**: Learned some [Markdown basics](https://stackedit.io/app#) for the creation of this readme

# Documentation: The Process

### Getting Started

If you want to test out this project or follow along please follow the steps below: 
<br>***NOTE***: Before you can run this project make sure that you have an AWS account, IAM user with assigned privileges, and the latest version of AWS CLI and Terraform CLI installed.
<br>

1. Clone or fork this project
2. Open with preferred IDE or text editor

### Running this project

1. run the ```aws configure``` command

```
$ aws configure
AWS Access Key ID [None]: Access_Key_ID
AWS Secret Access Key [None]: Secret_key
Default region name [None]: Region
Default output format [None]: Output (json or hcl)
```
2. In the working directory create a directory for the configuration of the S3 bucket

```
mkdir test-terraform-aws-s3
```
```
cd test-terraform-aws-s3
```
3. open ```main.tf``` file in IDE or text editor and edit this configuration to your specifications:
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region  = "[add-your-region]"

  # Remove comment for profile name only if terraform apply fails due to invalid credentials
  #profile = "[profile-name]"
}

resource "aws_s3_bucket" "[name_bucket]" {
  # Make sure that the bucket name is unique 
  bucket  = "name_bucket"
  tags    = {
    Name          = "BucketTag"
	Environment    = "Production"
  }
}
```
4. run the ```terraform init``` command to initialize the directory
5. run the ```terraform validate``` command to validate the syntax of the configuration
6. run the ```terraform plan``` command to generate an execution plan
7. run the ```terraform apply``` command to create the S3 container on AWS
8. verify that the S3 bucket was created by going to the AWS Console S3 service page
9. run the ```terraform destroy``` command to destroy the S3 bucket

### What Worked

- The process of creating a new AWS account, creating a new IAM Identity Center user, and assiging it privilages was a straight forward process. I appreciated that the aws console had access to the tutorials which I could access at any time (or hide). 
- Running the ```terraform destroy``` command

### What Didn't Work

- Confusing IAM and IAM Identity Center and not knowing the difference. After assigning priviliges to the IAM Identity Center user I created, I was having trouble signing in the the aws console. I even opened a ticket with AWS support because I could not login to my new user account with ther correct user name and password. The issue was that I was trying to sign in to: 

```
https://[account_ID_or_alias].signin.aws.amazon.com/console
```
instead of:
```
https://[managing_instance].awsapps.com/start 
```
I ended up resolving the issue after reading [aws support documents](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_identity-management.html#intro-identity-users) and watching a youtube video explaining the difference between IAM and IAM Identity Center users. 
- error : **configuring Terraform AWS Provider: no valid credential sources for Terraform AWS Provider found.** when running ```terraform apply``` command

What fixed it was running the ```aws configure sso``` command
```
$ aws configure sso
SSO session name (Recommended): IAM_Identity_Center_UserName
SSO start URL [None]: https://[managing_instance].awsapps.com/start/#
SSO region [None]: [region]
SSO registration scopes [None]: sso:account:access
Attempting to automatically open the SSO authorization page in your default browser.
If the browser does not open or you wish to use a different device to authorize this request, open the following URL:


The only AWS account available to you is:
Using the account ID 
The only role available to you is: 
Using the role name ""
Default client Region []: 
CLI default output format (json if not specified) [None]: json
Profile name []: Profile Name
To use this profile, specify the profile name using --profile, as shown:

aws sts get-caller-identity --profile Profile Name
```
To verify that your profile was successfully created run the ```aws sts get-caller-identity --profile``` command and include your profile name after ```--profile```
<br><br>
To login as the same user use the ```aws sso login --profile``` command adding the profile name after ```--profile```
- When successfully running the ```terraform apply``` command, I could not find the S3 container because I used the wrong [terraform resource syntax](https://developer.hashicorp.com/terraform/language/resources/syntax). Instead of creating a S3 container, I created a EC2 server.

What fixed this issue was change the resource block from ["aws_instance_"](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)
```
# creates aws EC3 server
resource "aws_instance" "app_server" {
  ami           = "ami-0fc82f4dabc05670b"
  instance_type = "t2.micro"

  tags = {
    Name = "My_EC2_Server"
  }
}
```
to resource "aws_s3_bucket
```
# creates aws S3 container
resource "aws_s3_bucket" "[name-bucket]" {
  bucket  = "name-bucket"
  tags    = {
    Name          = "BucketTag"
	Environment    = "Production"
    }
}
```

# Supplemental Resources

I have attached some links for what I read that was interesting while I was searching for solutions for issues that came up during the configuration process. 

- I found an interesting reddit post that explains [when Terriform](https://www.reddit.com/r/Terraform/comments/17xcpvq/can_someone_help_me_explain_when_is_terraform/) is needed
- Terraform [marketshare](https://6sense.com/tech/configuration-management/terraform-market-share) in configuration management
- What AWS EC2 is and if it's a good option for [hosting a website](https://stackoverflow.com/questions/919224/explain-what-amazon-ec2-is-and-can-i-use-it-as-a-web-hosting-service)
- Using ```.gitignore``` [file](https://docs.github.com/en/get-started/git-basics/ignoring-files) to ignore files you dont want uploaded to GitHub 
- How to write a [good README file](https://www.freecodecamp.org/news/how-to-write-a-good-readme-file/)
- [Why correctly configuring your Virtual Private Cloud (VPC) is important](https://www.missioncloud.com/blog/is-your-aws-virtual-private-cloud-really-private)

# References

1. https://techcrunch.com/2023/02/06/even-as-cloud-infrastructure-market-growth-slows-microsoft-continues-to-gain-on-amazon/#:~:text=The%20Big%20Three%20cloud%20providers,SaaS%2C%20which%20is%20measured%20separately
2. https://developer.hashicorp.com/terraform/intro
3. https://www.youtube.com/watch?v=-_XwzwMeqGY
4. https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_iam-s3.html 
5. https://developer.hashicorp.com/terraform/tutorials/aws-get-started/infrastructure-as-code
