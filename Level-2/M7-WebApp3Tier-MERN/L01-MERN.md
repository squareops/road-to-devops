### Deploy a MERN Stack Application on AWS EC2

In this post we are going to setup a production ready web server from scratch on AWS EC2 instances and deploy MERN Stack application

### Contents:
- [What is MERN Stack Application ?](#what-is-mern-stack-application-)
- [Scope of this tutorial](#scope-of-this-tutorial)
- [GOALS](#goals)
- [KEY POINTS TO FOLLOW](#key-points-to-follow)
- [VPC ARCHITECTURE](#vpc-architecture)
  - [SUBNETS](#subnets)
  - [IGW](#igw)
  - [ELASTIC IP](#elastic-ip)
  - [NAT GATEWAY](#nat-gateway)
  - [ROUTE TABLES](#route-tables)
  - [Public Route Table:](#public-route-table)
  - [Private Route Table:](#private-route-table)
  - [Subnet Association:](#subnet-association)
- [APPLICATION ARCHITECTURE](#application-architecture)
  - [STEP 1: Create jump server](#step-1-create-jump-server)
  - [STEP 2: Create mongodb instances](#step-2-create-mongodb-instances)
  - [STEP 3: Create AMI and setup EC2 instance for Backend Node JS application](#step-3-create-ami-and-setup-ec2-instance-for-backend-node-js-application)
      - [Install CodeDeploy agent and CloudWatch agent](#install-codedeploy-agent-and-cloudwatch-agent)
        - [CODE DEPLOY AGENT:](#code-deploy-agent)
        - [CLOUD WATCH AGENT](#cloud-watch-agent)
  - [STEP 4: Create Instance in Private Subnet for nodejs application with the created template.](#step-4-create-instance-in-private-subnet-for-nodejs-application-with-the-created-template)
  - [STEP 5: Codebuild configuration](#step-5-codebuild-configuration)
  - [STEP 6: Create a domain in Route 53](#step-6-create-a-domain-in-route-53)
  - [STEP 7: Attach domain with the load balancer.](#step-7-attach-domain-with-the-load-balancer)
  - [STEP 8: Create a launch template from the AMI and use that in autoscaling group.](#step-8-create-a-launch-template-from-the-ami-and-use-that-in-autoscaling-group)
  - [STEP 9: Frontend Amplify setup](#step-9-frontend-amplify-setup)
  - [STEP 10: Testing](#step-10-testing)
  - [STEP 11: Create PIPELINE with the configuration as follows](#step-11-create-pipeline-with-the-configuration-as-follows)
    - [SLACK INTEGRATION IN PIPELINE](#slack-integration-in-pipeline)
    - [MONITORING](#monitoring)

## What is MERN Stack Application ?

A MERN Stack application is made up of a front app built with React and React frontend is connected to backend API built with Node.js + ExpressJS + MongoDB, hence the name MERN (MongoDB , Express JS , React, Nodejs). Other variations of stack include the MEAN Stack that has uses Angular as frontend and MERN Stack that use Vue.js as frontend.

## Scope of this tutorial
This tutorial is focused on setting a Cloud server on AWS EC2 then deployment and configuration of frontend and backend part of the MERN App to work. 

## GOALS
- Understanding Mongodb deployment with replication.
- Deployment of Nodejs application for backend.
- Deployment of Reactjs application with AWS amplify.
- Achieving HA and CI/CD with CodeBuild, CodeDeploy and CodePipeline.

## KEY POINTS TO FOLLOW
- Create VPC full architecture.
- Database must be in a private subnet.
- Create nodejs application deployment in a private subnet.
- Special care for security groups and ports.

## VPC ARCHITECTURE
For the complete VPC architecture we will first create a VPC and then we will create subnets (Public and Private) and associate them with Route Tables . We will create an Internet Gateway and a NAT Gateway to route traffic for subnets along with that will create a EIP and map it with NAT Gateway. 

- VPC Name: Squareops_VPC
- CIDR_Block: 192.168.0.0/16

 ![](Images/a1.png)

### SUBNETS
As per the architectural requirements, We will create two public and two private subnet so that we can have robust architecture.

- Public Subnets:
Names: Squareops_Public_subnet_1 && Squareops_Public_subnet_2
CIDR_Blocks: 192.168.1.0/24 && 192.168.2.0/24

 ![](Images/a2.png)

- Private Subnets:
Names: Squareops_Private_subnet_1 && Squareops_Private_subnet_2
CIDR_Blocks: 192.168.3.0/24 && 192.168.4.0/24

 ![](Images/a3.png)

### IGW
Enables resources (like EC2 instances) in your public subnets to connect to the internet if the resource has a public IPv4 address or an IPv6 address and associate it with VPC.

Name: Squareops_IGW

 ![](Images/a4.png)

### ELASTIC IP
Name: Squareops_EIP

 ![](Images/a5.png)

### NAT GATEWAY
Name: Squareops_NAT_Gateway

 ![](Images/a6.png)

### ROUTE TABLES
To route the traffic inside VPC we need to have a route table, so we can set rules to route traffic. As per the need, we will create one Private and one Public route table and associate subnets and gateways as required.

 ![](Images/a7.png)


### Public Route Table: 
In the Public route table we will associate Public Subnets and associate the route with the Internet Gateway.

- Name: Squareops_Public_Route_Table
- Associated with IGW: Squareops_IGW
- Associated Subnets Name: Squareops_Public_subnet_1 && Squareops_Public_subnet_2

 ![](Images/a8.png)

- Subnet Associations:

 ![](Images/a9.png)

### Private Route Table:
In the Public route table we will associate Private Subnets and associate the route with NAT Gateway so that the instances in the private subnet can go to the internet.

- Name: Squareops_Private_Route_Table
- Associated with NAT Gateway: Squareops_NAT_Gateway
- Associated Subnets Name: Squareops_Private_subnet_1 && Squareops_Private_subnet_2

 ![](Images/a10.png)


### Subnet Association:

 ![](Images/a11.png)

- NOTE: VPC complete architecture completed with all details and screenshots.

## APPLICATION ARCHITECTURE

As we have completed the complete architecture of VPC, We will now move to create resources 
and infrastructure for our application.
For that we will follow these steps:
1. Create a jump server in the public subnet so that we can connect to the instances Inside private subnet.
2. Create Mongo db instances in a private subnet and have 2 replicas for failover.
3. We will create an AMI for the Nodejs application installation setup so that we can have a template for the instance creation.
4. We will have our nodejs application in a private subnet and associate it with load balancer and for load management we will attach it with the auto scaling group.
5. After project setup on instances we will save the code and move it to GITHUB.
6. Build and Deploy the application via AWS CodeBuild (store artifacts in Bucket , CodeDeploy.
7. Create a domain in Route53 and attach it with the load balancer.
8. Create a certificate and attach it with the domain and make required changes in Load Balancer to make the website secure.
9. Create Autoscaling Group
10. Deploying React application to aws amplify
11. Implementing an env variable in a project.
12. Implementing CodePipeline to build and deploy applications.
13. Slack Integration with pipeline.
14. Monitoring

### STEP 1: Create jump server 
Create a jump server in the public subnet so that we can connect to the instancesInside private subnet. We will have a security group with SSH permission.
 - Name: Squareops_Jump_Server
 - Type: t3a.small
 - Key: Squareops_Jump_Server_Key
 - Subnet: Squareops_Public_Subnet_1
 - SG: Squareops_Jump_Server_SG[allow 22 port in inbound rules ]
 - Storage: gp2 , 8GB

 ![](Images/a12.png)

Security Group:

 ![](Images/a13.png)


### STEP 2: Create mongodb instances
Create Mongo db instances in a private subnet and have 2 replicas for failover.
- Primary DB Name: db-master
- Replication DB Names: db-node-1 && db-node-2
- Type: t3a.small
- Key: Squareops_DB_Server_Key 
- Availability Zone:
     - For Primary DB: ap-south-1a
     - For Replication DB: ap-south-1b
- Subnet: 
     - For Primary DB: Squareops_Private_Subnet_1
     - For Replication DB: Squareops_Private_Subnet_2
- SG: Squareops_DB_Server_SG[allow 27017 port in inbound rules ]
- Storage: gp2 , 8GB

 ![](Images/a14.png)

Security Group:

 ![](Images/a15.png)

Installation Steps:
```
apt update
sudo apt install dirmngr gnupg apt-transport-https ca-certificates software-properties-common
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
sudo add-apt-repository 'deb [arch=amd64] https://repo.mongodb.org/apt/ubuntu       focal/mongodb-org/4.4 multiverse'
sudo apt install mongodb-org -y
sudo systemctl start mongod
sudo systemctl enable mongod
mongo --eval 'db.runCommand({ connectionStatus: 1 })'
Open sudo vi /etc/mongod.conf
      Modify bindIp to 0.0.0.0 && add Replication set as
     replication:
          replSetName: rs0
sudo systemctl restart mongod
```
- Note: To make one instance Primary, login to that instance and initiate it and add the other two instances as replicas.

On the primary instance, create a user for authentication.

```
rs.initiate();
rs.add(‘worker1PrivateIP:27017’);
rs.add(‘worker2PrivateIP:27017’);
 rs.status();
use admin;
db.createUser(
{	user: "UserName",
	pwd: "Password",

	roles:[{role: "userAdminAnyDatabase" , db:"admin"}]})
```
Here is the mongodb connection string:
```
mongodb://squareopsuser:sqops321password@10.0.3.60:27017,10.0.4.237:27017,10.0.4.130:27017/conduit?authSource=admin&replicaSet=rs0
```

### STEP 3: Create AMI and setup EC2 instance for Backend Node JS application
Create an instance in a public subnet and do the required package installation like Nodejs,PM2, CodeDeploy Agent, CloudWatch Agent and make a template from that.After creating a template we can terminate the instance or can stop it, if we have further use.
- Name: Squareops_Node_Backend_Pub
- Type: t3a.small
- Key: Squareops_Web_Server_Key
- Subnet: Squareops_Public_Subnet_1
- SG: Squareops_Web_Server_SG[allow port 22 and 3000 in inbound rules]
- Storage: gp2 , 8GB

 ![](Images/a16.png)

Package Installation:

```
apt update
apt -get install net-tools -y
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
apt-get install gcc make -y
apt-get install nodejs -y 
npm  install pm2@latest -g
```
##### Install CodeDeploy agent and CloudWatch agent

###### CODE DEPLOY AGENT:
```
sudo apt update -y
sudo apt install ruby-full -y
wget https://aws-codedeploy-region.s3.region.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo service codedeploy-agent status
```
###### CLOUD WATCH AGENT
```
   Configure aws:
         curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install

wget https://s3.amazonaws.com/amazoncloudwatch-agent/debian/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
sudo apt-get update && sudo apt-get install collectd
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```
- Do manual setup
```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```

- To Do manual change in configuration file

          vi  /opt/aws/amazon-cloudwatch-agent/bin/config.json

- Check Status of Agent

          /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status

- To Start

          /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a start


- Create a template.
Name: SquareopsNodeJsAMI

 ![](Images/a17.png)

### STEP 4: Create Instance in Private Subnet for nodejs application with the created template.
- Name: Squareops_Node_Backend
- Type: t3a.small
- AMI: SquareopsNodeJsAMI
- Key: Squareops_Web_Server_Key
- Subnet: Squareops_Private_Subnet_1
- SG: Squareops_Web_Server_SG

 ![](Images/a18.png)

- Load Balancer Configuration

We will first create a target group and attach it with the  load balancer.

- Target Group Name: Squareops-nodejs-CICD-TG

 ![](Images/a19.png)

- Load Balancer Name: Squareops-Nodejs-Balancer

 ![](Images/a20.png)

- Security Group: Squareops_NodeJS_Balancer_SG

 ![](Images/a21.png)

- Note: Website result with Load Balancer 

 ![](Images/a22.png)


### STEP 5: Codebuild configuration 
Code Saved to GITHUB and here is the Link: https://github.com/RohitSquareops/BackEndNodeJS

-  Create a CodeBuild project and keep configurations as follows
 
Project Name: Squareops_NodeJS_Backend

 ![](Images/a23.png)

Configuration:

 ![](Images/a24.png)

Source 

 ![](Images/a25.png)

Environment

 ![](Images/a26.png)


Artifact

 ![](Images/a27.png)

 ![](Images/a28.png)


S3 BUCKET
Name: squareops-nodejs-bucket

 ![](Images/a29.png)

CODE DEPLOY

Application Name: BackEndNodeJS

 ![](Images/a30.png)


Deployment Group Name: BackEndNodeJS_DG

 ![](Images/a31.png)


### STEP 6: Create a domain in Route 53

- Domain Name: rtd.squareops.co.in
- Sub-Domain Name: namesquareops.rtd.squareops.co.in

 ![](Images/a32.png)

- Note: Website Result with Subdomain name:

 ![](Images/a33.png)


### STEP 7: Attach domain with the load balancer.

We will make a request for a certificate with our fully qualified domain name and create a canonical name in route 53 along with making required changes in load balancer for port forwarding with ACM certificate attachment.

Certificate Status:

 ![](Images/a34.png)


Create Record in route53

 ![](Images/a35.png)

Additional Details

 ![](Images/a36.png)

Making port forwarding in Load Balancer

 ![](Images/a37.png)

 ![](Images/a38.png)

 ![](Images/a39.png)

 ![](Images/a40.png)

- Note: Website Result with Subdomain and ACM certificate

 ![](Images/a41.png)

### STEP 8: Create a launch template from the AMI and use that in autoscaling group.

- Launch template Name: Squareops_Backend_ASG_Template

 ![](Images/a42.png)

- Autoscaling Group Name: Squareops_BAckend_ASG

 ![](Images/a43.png)

 ![](Images/a44.png)


- SPOT INSTANCE IN AUTO SCALING STRATEGY

Launch Template

 ![](Images/a45.png)

Modify Auto Scaling with Spot instance Template

 ![](Images/a46.png)

Instance Creating with SPOT type.

 ![](Images/a47.png)


### STEP 9: Frontend Amplify setup
We will first put our reactjs application code to github, later on we'll go to AWS and host a website under AWS AMPLIFY.

- Link to GITHUB Application: https://github.com/RohitSquareops/FrontEndReactJS

- We will give our backend domain name to the application for connectivity.

 ![](Images/a48.png)

On AWS AMPLIFY , We will first host a web application and choose our project location.

CodeRepo 

 ![](Images/a49.png)

Repo connectivity 

 ![](Images/a50.png)

Hosting Details

 ![](Images/a51.png)

Deployment 

 ![](Images/a52.png)

 ![](Images/a53.png)

### STEP 10: Testing

After completing the full application architecture, We’ll now make some changes in both Backend app as well as the frontend app.
In Backend Application, We will get the db connection string as a .env variable and use that in the application.As part of that will install “npm install dotenv --save” and some minor changes in app.js file so that it can read from .env.

 ![](Images/a54.png)

In the Frontend application, we’ll create an env variable in AMPLIFY [need clarification and session]

 ![](Images/a55.png)


In amplify.yml file we will add the env variable in .env file and later use in agent.js
Amplify.yml

 ![](Images/a56.png)


Help Resource: https://docs.aws.amazon.com/amplify/latest/userguide/environment-variables.html

 ![](Images/a57.png)


### STEP 11: Create PIPELINE with the configuration as follows

 ![](Images/a58.png)

 ![](Images/a59.png)

 ![](Images/a60.png)

 ![](Images/a61.png)

 ![](Images/a62.png)

#### SLACK INTEGRATION IN PIPELINE

![](Images/a63.png)

 ![](Images/a64.png)

#### MONITORING
Setup monitoring of Network, Servers, ASGs , ALB and Database. Configure alerts to slack

DASHBOARD Name: Squareops-Dashboard

 ![](Images/a65.png)

Alarms

 ![](Images/a66.png)

Autoscale setting for Scale on CPU and Memory Basic

 ![](Images/a67.png)

Setup for Instance memory utilization

 ![](Images/a68.png)

LOG GROUP: We have two log groups now, One for CodeBuild and another for Node instance system log.

 ![](Images/a69.png)

Retention Policy for log will be 3 days.

 ![](Images/a70.png)

Metrics Filter on Syslog

 ![](Images/a71.png)

 ![](Images/a72.png)

Alarm on Filter

Name: Syslog_Warning_Alarm

 ![](Images/a73.png)

 ![](Images/a74.png)








 







