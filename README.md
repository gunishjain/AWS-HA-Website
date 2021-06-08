# AWS High Availibility Website
This Project Demonstrates How to Setup a Wordpress Website which is Highly Available and Scalable using AWS.
Everything was done using different AWS Services 

## Services Used in the Project
- EC2 Instance
- AMI
- Application Load Balancer
- CloudFront
- RDS MySQL
- Security Groups
- Launch Configuration
- Auto Scaling  Group
- Route 53
- S3 Bucket
- IAM 

### Project Setup

#### Creating S3 Bucket: Creating S3 Bucket:

We will create two S3 bucket for this project. 
First one will store Media Files and the other will store all the code configuration from wordpress. This will act as a backup if something goes wrong with our website.

![Buckets](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/buckets.JPG)
------------


#### Creating CloudFront Distribution:
We will create a cloudfront distribution in which the origin path should be our media bucket since the media will be fetched through cloudfront.

![CLDFN](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/cloudfront_create.JPG)

Leave other Settings as deafult

![CLDFN](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/cloudfront.JPG)

------------
#### Creating Two Security Groups:
We will create Security Group for our EC2 instance and RDS Instance
EC2 allows ssh and public access while RDS can be accessed only by ec2 thats why The SG for ec2 is present in SG of RDS.

![SG](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/sg-ec2.JPG)
![SG](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/sg-rds.JPG)

------------

#### Creating RDS Instance:
> Since we are building the project in Free Tier Multi AZ feature is disabled but can be enabled( It will cost some charge!)

We choose MySQL database
Provide Login Credentials
Public Access is Disabled Since EC2 instance will access it.
Add our own Security Group that was created.
Other Setting are left Default.
Backups and snapshot is disabled.

![RDS](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/rds.JPG)
------------

#### Creating IAM Role
We are creating an IAM Role so that Our EC2 user could access the S3 Bucket to send data and fetch it.

The Policy is attached by searching S3FullAccess provide a suitable name and create it.

![IAMR](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/Iam_role.JPG)
![IAMS](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/role_summary.JPG)

------------

#### Launching EC2 Instance
We launch an ec2 instance using Linux AMI 2 as our base image.
In the configuration page attach our created role and add the script given [EC2-Script](https://github.com/gunishjain/AWS-HA-Website/blob/main/EC2_Script.txt "EC2-Script") in place of **User Data**
![EC2C](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/ec2-role.JPG)

![EC2C](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/ec2-launch.JPG)

Add the security group which we created in previous steps. And follow the steps to launch it.


------------


#### Wordpress Setup
Take the IPv4 Address and access the setup page in browser.

![WPS](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/wp-startup.JPG)

Enter the details for the database and provide the host address from the RDS endpoint.
Finish the setup and we have successfully launched our wordpress website hosted on ec2 instance.

![WPI](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/wp-isntall.JPG)
![WPE](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/wordpress-page-edit.JPG)

> **Note**: If there are any error while setting up the database for wordpress, we can fix this problem by SSH into our instance and editing wp-config-sample.php file and adding the db details using any linux file editor. 

> wp-config-sample is found in /var/www/html 


------------

#### Creating Backup in S3
Using SSH we can access our instance and copy all our files into s3 bucket  using the command:

`aws s3 cp --recursive /var/www/html/wp-content s3://yourbucketname`

![S3Copy](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/s3-copy.JPG)

![S3Back](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/bucket-backup.JPG)

------------
#### Creating Application Load Balancer

We create a Application Load Balancer, continue to setup according to your region. Provide the ec2 Security Group.

In Health Check section provide the path as awsproject.html since it was created using our script code.
This will perform health check by accessing it.
Now using DNS name of ALB we can easily access our wordpress website.
![ALBSum](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/ALB-summary.JPG)

![ALBS](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/ALB-success.JPG)

------------
#### Creating AMI from EC2 instance
AMI is created so that it could be used later in our Launch Configuration:

![AMIC](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/ami-creation.JPG)
![AMID](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/ami-done.JPG)


------------

#### Creating Launch Configuration
Create launch configuration by providing our AMI and setting up the instance according to our ec2 instance.

------------

#### Creating AutoScaling Group
Now we create ASG using our Launch Configuration
We can provide scaling policies and Minimum and Maximum instances to be launched.
In my case I added 2 minimum instances.

![ASGC](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/ASG.JPG)

![ASGE](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/asg-ec2.JPG)


You can see in the screenshot one more instance was automatically launched since 2 minmum instances need to be launched.

------------
#### Cloudfront Setup for Wordpress

We need to edit the .htaccess file and provide Cloudfront Origin Host address as shown in the screenshot below:

![HTA](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/htaccess.JPG)

This will allow proper functioning of Cloudfront.


------------

#### Route53 Configuration

You can use your custom Domain Name to access our Wordpress Website.
Create a hosted zone for the domain.
Create a record set for ALB as shown in screenshots:

![HZ](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/hosted-zone.JPG)
![RS](https://github.com/gunishjain/AWS-HA-Website/blob/main/Screenshots/record-set.JPG)


In Route Traffic To add your ALB DNS.

> **Note: Update the NameServers found in Your AWS hosted zone to your domain provider settings**



###### Website Setup is Done! 

------------

##### Resources Used:

[Freenom.com](https://www.freenom.com/en/index.html?lang=en "Freenom.com")  -- For Free Domain Name

[Wordpress.org](https://wordpress.org/ "Wordpress.org") -- Wordpress Setup













