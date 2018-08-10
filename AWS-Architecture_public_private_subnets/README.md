# AWS - Architecture with public and private subnets

### Intro

The aim of this guide is not replace the AWS documentation but is show step-by-step of create and combine AWS resources in cloud console in order to create an Architecture with public and private subnets.

In this guide I explain how create an infrastructure on AWS Cloud following design best practicies.

The goal is create a two tier infrastructure with a DMZ for public accessible application and protect sensible infrastructure zones with private subnets.

A common usage example could be a LAMP application on which Web Server should be exposed over the internet, instead the database servers should be protected on private subnets.

More details are available on AWS Documentation here https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Scenarios.html.


### Architecture design

The architecture goal is reported in the image below.

The first image represent an high level diagram of the AWS infrastructure that we are going to create.

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/high_level_diagram.PNG)

The main AWS resources that will be created are:

1. **Virtual Private Cloud** (better known as VPC).
It is a virtual network on which all the AWS resources (like ELB, EC2, DB,..) are created (more details available on AWS documentation here https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html).
2. **Security Group**.
The security group acts as a virtual firewall that allow the control inbound and outbound traffic (more details available on AWS documentation here https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_SecurityGroups.html).
2. **Public and Private subnets**.
A subnet is a range of IP addresses into a VPC (more details available on AWS documentation here https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html#what-is-vpc-subnet).
3. **Internet Gateway**.
An internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between instances into a VPC and the internet (more details available on AWS documentation here https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html).
4. **NAT**.
A network address translation (NAT) instance in a public subnet into a VPC enable instances in the private subnet to initiate outbound traffic to the Internet or other AWS services. On AWS The NAT could be configured with an EC2 instance (in this case is your responsibilities guarantee high availability and scalability) or via AWS NAT Services.
In this guide I will show you how create a secure and scalable infrastructure vi AWS NAT Services (that is always my best suggestion :)).
More details available on AWS documentation here https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_NAT_Instance.html.

As shown above all the internet traffic will reach our servers created on Public Subnets, hosted on our VPC passing thorugh a firewall named Security Group.
The communication between Public and Private subnet is allowed via another firewall (Security Group) placed above the Private Subnet.

Following this schema, if you wuold like to create a LAMP application, a frontend like apache,nginx could be placed on Public Subnets, instead an Application server (like Wordpress, Drupal, Magento,..) and a backend (like a MySQL database, RDS) could be placed on Private Subnets.

* The Public Subnet are internet facing and could be proteced via Security Group
* The Private Subnet are not reachable via internet in any way

In the next section I will show you how create in details an AWS architecture with public/private subnets.

#### Architecture details provisioning - practices part

The prerequisite to achive the goal described below are:

1. You have an AWS account
2. You have root or privileged user to create AWS Resources

The first item that I'm going to create is the VPC.

In this example I used a CIDR address block like 10.0.0.0/20 but you can use the best address that fit your needs.
It is important to use a proper CIDR block address especially if you wuold like to connect the AWS Cloud to your on-prem environment (or another Cloud or another VPC in AWS). This because to connect the VPC with others network the ips address must not be overlapped.
In case you noticed that your VPC address are in overlapp with your local network you must destroy and recreate all the architecture (VPC, EC2, DB,..); up to now there is no way to migrate resources to another VPN and/or change VPC CIDR block.

In this guide I will not connect my VPC to other network (is not the purpose of this document).

I used Ireland region.

### Create a VPC
1. Login on your AWS Account.
2. Move to VPC section
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/1.Choose_VPC.PNG)
3. Create new VPC
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/2.Create_VPC.PNG)
4. Choose a name (in my example is "test-vpc") and a CIDR block (in my example is 10.0.0.0/20). Then click on "Yes, Create".
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/3.Create_VPC.PNG)

### Create the Internet Gateway

Now we are going to create the Internet Gateway and then we need to attach it to a VPC in order to create an internet access.

1. Click on Internet Gateway section and then click on "Create Internet Gateway".
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/4.Create_IGW.PNG)

