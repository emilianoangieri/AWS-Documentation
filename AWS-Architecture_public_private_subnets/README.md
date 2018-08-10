# AWS-Architecture_public_private_subnets

### Intro

In this guide I explain step by step how create an infrastructure on AWS Cloud following design best practicies.

The goal is create a two tier infrastructure with a DMZ for public accessible application and protect sensible infrastructure zones with private subnets.

A common usage example could be a LAMP application on which Web Server should be exposed over the internet, instead the database servers should be protected on private subnets.

More details are available on AWS Documentation here https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Scenarios.html.


### Architecture goal

The architecture goal is reported in the image below.
