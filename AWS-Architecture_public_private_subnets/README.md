# AWS - Architecture with public and private subnets

### Intro

In this guide I explain how create an infrastructure network on AWS cloud following design best practicies.

The goal is create a two tier netowrk infrastructure with a DMZ for public accessible application and protect sensible infrastructure zones with private subnets.

A common usage example could be a LAMP application where a Load Balancer that distrubue the traffic across the Web Servers and/or Application Server should be exposed over the internet, instead the Web Servers and the database servers should be protected on private subnets.

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

Following this schema, if you wuold like to create a LAMP application, a frontend like apache,nginx could be placed on private subnet behind the ELB that need to be placed in public subnets, instead an Application server (like Wordpress, Drupal, Magento,..) and a backend (like a MySQL database, RDS) could be placed on private subnets as well.

* The Public Subnet are internet facing and could be proteced via Security Group
* The Private Subnet are not reachable via internet in any way

Kindly note that each component of this infrastructure must be hosted on multiple availability zone in order to guarantee high availability.
This will impact the creation of AWS resources that we are going to made.
Below it is reported the infrastructure image details:

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/0.infra.with.az.PNG).

In the next section I will show you how create in details an AWS architecture with public/private subnets.

#### Architecture details provisioning - practices part

The prerequisite to achive the goal described below are:

1. You have an AWS account
2. You have root or privileged user to create AWS Resources

The first resource that I'm going to create is the VPC.

In this example I used a CIDR address block like 10.0.0.0/20 but you can use the best address that fit your needs.
It is important to use a proper CIDR block address especially if you wuold like to connect the AWS Cloud to your on-prem environment (or another Cloud or another VPC in AWS). This because to connect the VPC with others network the ips address must not be overlapped.
In case you noticed that your VPC address are in overlapp with your local network you must destroy and recreate all the architecture (VPC, EC2, DB,..); up to now there is no way to migrate resources to another VPC and/or change VPC CIDR block.

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

In step 4 AWS ask about tenancy.
The tenancy specify if how AWS resources are dedicated to us or shared across multiple clients.
This is a very cool option when high level of security must be guarantee, but it is a costly solution :).
In this guide I leave Default tenancy.
More details about tenancy on AWS documentation here (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html).

### Create the Internet Gateway

Now we are going to create the Internet Gateway and then we need to attach it to a VPC in order to create an internet access.

1. Click on Internet Gateway section and then click on "Create Internet Gateway".
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/4.Create_IGW.PNG)
2. Choose a name of your Internet Gateway (in my example is test-igw)
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/5.Create_IGW.PNG)
3. Click on "Create"
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/6.Create_IGW.PNG)
4. As you can see the Internet Gateway is not attached to any VPC.
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/7.Create_IGW.PNG)
To attach it on our VPC click on "Actions" -> "Attach to VPC"
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/8.Attch_IGW_to_VPC.PNG)
5. Select our VPC previously created and click on "Attach"
![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/9.Attch_IGW_to_VPC.PNG)

Now our VPC "test-vpc" has an Internet Gateway.

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/10.Attch_IGW_to_VPC.PNG)

## Create subnets

In this section we are going to create our subnets.

We will create the following subnets:

* public-ELB-1a/b: This are public subnets on which we could create our public facing Load Balancer. a/b menas that we are going to create two subnet in two availability zones (named a and b).
* public-NAT-1a/b: This are public subnets on which we will create the two AWS NAT Gateway. Basically the idea is to create two NAT Gateway in order to avoid single point of failure in case of down of one availability zone.
* private-Application-1a/b: This are private subnets on which we could create our EC2 that will host our Application (like Wordpress, Drupal, Liferay, Magento,..).
* private-DB-1a/b: This are private subnets on which we could create our database.

Once created the above subnets we need to create the proper Route Table in order let them public or private.

Now start to create the subnets.

Click on "Subnets" and "Create subnet"

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/11.Create_subnet.PNG)

Now choose a name (like public-ELB-1a), select the VPC previously created, select the preferred availability zone and choose a CIDR block for this subnet.

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/12.Create_subnet.PNG)

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/13.Create_subnet.PNG)

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/14.Create_subnet.PNG)

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/15.Create_subnet.PNG)

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/16.Create_subnet.PNG)

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/16.Create_subnet_2.PNG)

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/17.Create_subnet.PNG)

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/18.Create_subnet.PNG)

The final sitution should be the following:

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/19.Create_subnet_final_status.PNG)

The next step is create the two Routing Table that will made the subnet public or private.

Before create the route table for private subnet we need to create the two NAT gateway.

### Create NAT Gateway

In order to create the NAT Gateway first we need the subnets IDs on which every single NAT will be created.
In our case the subnets IDs are subnet-04a2dbc4ba131a63f about a NAT in availability zone 1a and subnet-0604698c1c6279798 about NAT in availability zone 1b.

To check the subnet ID click on "Subnets" and filter by NAT keyword.

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/31.NAT.PNG)

Now click on "NAT Gateway" and click on "Create NAT Gateway".

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/30.NAT.PNG)

Now choose the the subnet ID (in my example subnet-04a2dbc4ba131a63f) click on "Create New EIP" and click on "Create NAT Gateway".
The NAT use the Elastic IP in order to keep the same static public ip avoiding any issue in case AWS need to change the underliyng resuorces.

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/32.NAT.PNG)

Wait until the NAT is available.

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/34.NAT.PNG)

Repeat this step also for the second subnet ID.

Now we have the two NAT gateway that we need to explicit in the Route Table rules about private subnets.

## Routing table to allow internet access

Basically AWS doesn't allow anything by default but you must specify a routing rule allow e.g. the internet access to your default resource.

In this section I'm going to create three Route Table:

* public-rtb: the route table that will make a subnet public, routing the traffic from 0.0.0.0/0 (anyway) to the Internet Gateway
* private-rtb-1a: the route table that will make a subnet private. This will route the traffic from 0.0.0.0/0 (anyway) to the internet via the NAT in the availability zone a.
* private-rtb-1b: the route table that will make a subnet private. This will route the traffic from 0.0.0.0/0 (anyway) to the internet via the NAT in the availability zone b.

Getting started creating the route table.

Click on "Route Tables" and "Crete Route Table".

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/20.Create_route_table.PNG)

Choose the name like "public-rtb" and select the proper VPC. Then click on "Create".

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/21.Create_route_table.PNG)

Now we need to configure the proper route for this public Route Table.

Click on "Routes" and "Edit".

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/22.Create_route_table.PNG)

Click on "Add another rule".

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/23.Create_route_table.PNG)

Now insert the route from anyway to the Internet Gateway previusly created:

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/24.Create_route_table.PNG)

We successfully create a public Route Table.

To let a subnet public we need to assign this route table to the subnets (we will do it in the next section).

Now we need to create the remaining private Route Table.

Click again on "Create Route Table" and now create the private-rtb-1a (do not forget to select the proper VPC).

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/25.Create_route_table.PNG)

Now assign the route from anyway to the NAT that we created previuosly in the availability zone 1a.

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/33.NAT.PNG)

Repeat this step also for the second Route Table private-rtb-1b assigning in this case the route from anyway to the NAT that we previuosly created in the availability zone 1b.

Now we need only to assign the proper Route Table to the proper subnets.

## Let subnet public or private

Now for all the public subnets assign the Route Table previously created and named "public-rtb".

First retrieve the Route Table ID of public-rtb Route Table.
Click on "Route Tables" and filter by "public".
Now keep track of Route Table ID.

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/28.Assign_route_table.PNG)

Click on 
Click on "Subnets" and select a public subnet (in this example public-ELB-1a) that we would like to make public.
Then click on "Route Table".


![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/26.Assign_route_table.PNG)

Now click on "Edit Route Table Association" and select the Route Table ID previuosly found (..ff7d).

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/27.Assign_route_table.PNG)

Click on "Save".

![alt text](https://github.com/emilianoangieri/AWS-Documentation/blob/master/AWS-Architecture_public_private_subnets/img/29.Assign_route_table.PNG)

Now the subnet "public-ELB-1a" is public accessible (kindly note that you need to explicit allow any inbound traffic via Security Group).

Repeat the step above for all public subnet.

About private subnet the procedure is the same.
The only one difference is select the Routing Table ID of private-rtb-1a and private-rtb-1b depending on where is located the subnet that you are updating (e.g. change the subnet private-Application-1b with the Route Table ID rtb-0d80b28289940df1b).

## Conclusion

We successfully created the network part of our infrastructure with a DMZ.

Let's start with the creation of the ELB and your preferred Application and database!




#### About author

My Name is Emiliano Angieri and I'm a Cloud engineer expert with passion for all cloud applications and best practicies.
Follow me on linkedin https://www.linkedin.com/in/emiliano-angieri-49908478/ or ask me a question to e.angieri@libero.it.
