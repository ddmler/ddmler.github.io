---
layout: post
title:  "AWS VPC creating a public and private subnet"
date:   2018-05-18 18:00:00 +0100
categories: aws
description: Creating a new VPC in AWS with a public and private subnet, a NAT gateway, routing and security group rules.
---

Part 1: AWS VPC creating a public and private subnet (you are here)

Part 2: [AWS EC2 & RDS install Laravel][part2]

This post is part 1 of a 2 part series. In this part we will create a virtual private network with a public and a private subnet with AWS VPC. In the next part we will launch a database into the private network and a webserver into the public network and install a Laravel application.

The first step is to create a new VPC. For this we use the IPv4 CIDR block: `10.0.0.0/16` and some name tag.

Next we create 1 public and 2 private subnets. We will need 2 private subnets to cover 2 availability zones which is a prerequisite to using AWS RDS in the second part of this series. If you don't want to use RDS you could just create 1 private subnet for now. For the public subnet use the following IPv4 CIDR block: `10.0.0.0/24` and for the two private ones use: `10.0.1.0/24 and 10.0.2.0/24`. Make sure that your two private subnets are in different availability zones.

Now create an internet gateway. For this only a name is needed. After it is created attach it to our VPC we created in the first step. Since the Instances in the private subnet are not connected to the internet, we will need to create a NAT gateway to let them access the internet for updates (but not letting the internet access them directly). So create a new NAT gateway, choose the public subnet and add an elastic IP to it.

The next step is to create 2 route tables. With these we want to configure that the instances in the public subnet can just access the internet via the internet gateway and the instances in the private subnet should go through the NAT gateway sitting in our public subnet. So create 2 route tables named public and private. After creating them configure their subnet associations to their respecting subnets. So the public route table should be associated to the public subnet and the private route table to both private subnets. Under routes add the following routes to the route tables:

Public route table:

| Destination | Target |
| --- | --- |
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | id of your internet gateway |
{: .blue-grey .darken-4}

Private route table:

| Destination | Target |
| --- | --- |
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | id of your nat gateway |
{: .blue-grey .darken-4}

The last step is to create 2 Security groups, which basically are simplified firewall rules, named: WebServer and DBServer. Again choose our VPC from step 1. After both are created set the Inbound and Outbound rules to:

DBServer Inbound Rules:

| Type | Source |
| --- | --- |
| ALL Traffic | id of your dbserver security group |
| MySQL/Aurora (3306) | id of you webserver security group |
{: .blue-grey .darken-4}

Protocol and port range will be set automatically when choosing the type correctly. You can of course choose another database like PostgreSQL here.

DBServer Outbound Rules:

| Type | Destination |
| --- | --- |
| ALL Traffic | id of your dbserver security group |
| HTTP (80) | 0.0.0.0/0 |
| HTTPS (443) | 0.0.0.0/0 |
{: .blue-grey .darken-4}

These rules define that all instances in this security group can communicate with each other however they want, instances from the webserver group can only access via mysql and the instances in the dbserver group can access the internet for updates via the NAT gateway.

WebServer Inbound Rules:

| Type | Source |
| --- | --- |
| ALL Traffic | id of your webserver security group |
| HTTP (80) | 0.0.0.0/0 |
| HTTPS (443) | 0.0.0.0/0 |
| SSH (22) | your home networks public ip address (or range) |
{: .blue-grey .darken-4}

The HTTP and HTTPS rules allow incoming traffic from the internet and the SSH rule allows you to access your servers via SSH.

WebServer Outbound Rules:

| Type | Destination |
| --- | --- |
| ALL Traffic | id of your webserver security group |
| HTTP (80) | 0.0.0.0/0 |
| HTTPS (443) | 0.0.0.0/0 |
| MySQL/Aurora (3306) | id of your dbserver security group |
{: .blue-grey .darken-4}

These rules allow outgoing web traffic to the whole internet or to our private network via mysql. The all traffic rule allows for communication between all webservers themselves.

And with that the VPC is done. When launching instances you will have to select the correct subnet and security group to use this VPC. In Part 2 we will launch a database and a webserver and get a Laravel application running on them.

[part2]: https://ddmler.github.io/aws/2018/05/25/aws-ec2-rds-install-laravel.html
