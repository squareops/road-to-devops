### Deploy a MERN Stack Application on AWS EC2

In this post we are going to setup a production ready web server from scratch on AWS EC2 instances and deploy MERN Stack application

### Contents:
- [What is MERN Stack Application ?](#what-is-mern-stack-application-)
- [Scope of this tutorial](#scope-of-this-tutorial)
- [GOALS](#goals)
- [KEY POINTS TO FOLLOW](#key-points-to-follow)
- [VPC ARCHITECTURE](#vpc-architecture)
- [APPLICATION ARCHITECTURE](#application-architecture)
  - [STEP 1: Create jump server](#step-1-create-jump-server)
  - [STEP 2: Setup Mongodb instances](#step-2-setup-mongodb-instances)
  - [STEP 3: Create AMI and setup EC2 instance for Backend Node JS application](#step-3-create-ami-and-setup-ec2-instance-for-backend-node-js-application)
    - [Package Installation (NODEJS, PM2)](#package-installation-nodejs-pm2)
    - [Install CODE DEPLOY AGENT](#install-code-deploy-agent)
    - [Configure AWS-CLI](#configure-aws-cli)
    - [Install CLOUD WATCH AGENT](#install-cloud-watch-agent)
  - [STEP 4: Create Instance in Private Subnet for nodejs application with the created template.](#step-4-create-instance-in-private-subnet-for-nodejs-application-with-the-created-template)
  - [STEP 5: Create Load Balancer](#step-5-create-load-balancer)
  - [STEP 6: Create a launch template from the AMI and use that in autoscaling group.](#step-6-create-a-launch-template-from-the-ami-and-use-that-in-autoscaling-group)
  - [STEP 7: Setup code deploy](#step-7-setup-code-deploy)
  - [STEP 8: Store environment variables](#step-8-store-environment-variables)
  - [STEP 9: Codebuild Configurations](#step-9-codebuild-configurations)
  - [STEP 10: Codepipeline Configurations](#step-10-codepipeline-configurations)
  - [STEP 11: Create a domain in Route 53](#step-11-create-a-domain-in-route-53)
  - [STEP 12: Frontend Amplify setup](#step-12-frontend-amplify-setup)
  - [STEP 13: Observability (Monitoring \& Logging)](#step-13-observability-monitoring--logging)

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

**This network stack can be used in deploying any production environment**

We will deploy a secure network stack on cloudFormation which will create these AWS resources 
- VPC 
- Public Subnets 
- Private Subnets 
- Route Tables 
- IGW 
- NAT 
- NAT EIP

Create a yaml file with name "vpc_secure.yml" and store it on your local system. Then paste the following code in it. 

```
AWSTemplateFormatVersion: 2010-09-09
Description: This template will deploy a work station
Metadata: 
  AWS::CloudFormation::Interface: 
   ParameterGroups: 
      - Label: 
          default: "Configurations"
        Parameters: 
          - VpcCIDR
          - PublicSubnet1ID
          - PublicSubnet2ID
          - PrivateSubnet1ID
          - PrivateSubnet2ID
          - KeyPairName
          - AppSG
          - ALBSecurityGroup
          - amiId
          - InstanceVolumeSize
          - InstanceType
          - min
          - max
          - DesiredCapacity
          - CPUPolicyTargetValue
          - SSLCert
          - OnDemandBaseCapacity
          - OnDemandPercentageAboveBaseCapacity
          - SpotMaxPrice
          - EstimatedInstanceWarmup
          - HealthCheckPath

Parameters:
  VpcCIDR:
    Description: Enter the Ip range (CIDR notation) for this VPC
    Type: String
    Default: 10.0.0.0/16
  PublicSubnet1CIDR:
    Description: CIDR for public subnet 1.
    Type: String
    Default: 10.0.1.0/24
  PublicSubnet2CIDR:
    Description: CIDR for public subnet 2.
    Type: String
    Default: 10.0.2.0/24
  PrivateSubnet1CIDR:
    Description: CIDR for private subnet 1.
    Type: String
    Default: 10.0.3.0/24
  PrivateSubnet2CIDR:
    Description: CIDR for private subnet 2.
    Type: String
    Default: 10.0.4.0/24
  KeyPairName: 
    Description: Amazon EC2 Key Pair
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be a valid Key Pair
  AppSG:
    Description: Name of application security group. 
    Type: AWS::EC2::SecurityGroup::Id
  ALBSecurityGroup:
    Description: Name of LoadBalancer security group. 
    Type: AWS::EC2::SecurityGroup::Id
  amiId:
    Description: Ami id of the machine for private network
    Type: String 
    AllowedPattern: '[-a-zA-Z0-9]*'
    ConstraintDescription: must contain only alphanumeric characters.
  InstanceVolumeSize:
    Description: Application instance EBS volume size in GiBs.
    Type: Number
    Default: 30
  InstanceType:
    Description: WebServer EC2 instance type for Private network.Refer:- https://aws.amazon.com/ec2/instance-types/
    Type: String
    Default: t3a.medium
    AllowedValues:
    - t3a.medium
    ConstraintDescription: must be a valid EC2 instance type.
  min:
    Description: Minimum Ec2 instances for Auto Scaling group.
    Default: 2
    Type: Number
  max:
    Description: Maximum Ec2 instances for Auto Scaling group.
    Default: 6
    Type: Number
  DesiredCapacity:
    Description: The number to ec2 instances you want to launch.
    Type: Number
    Default: 2
  CPUPolicyTargetValue:
    Description: Target value for Target based scaling group.
    Default: 60
    Type: Number
  SSLCert:
    Description: SSL Certificate ARN
    Type: String
    ConstraintDescription: must be a valid Certificate ARN
  OnDemandBaseCapacity:
    Description: Minimum numer of OnDemand instances to launched for AutoScaling.
    Type: Number
    Default: 2
  OnDemandPercentageAboveBaseCapacity:
    Description: for example, 20 specifies 20% On-Demand Instances, 80% Spot Instances
    Type: Number
    Default: 50
  SpotMaxPrice:
    Description: Maximum Bid price for spot instance. It depends on instance size.
    Type: Number
    Default: 0.005
  EstimatedInstanceWarmup:
    Description: Instance Warmup time for ASG
    Type: Number
    Default: 300
  HealthCheckPath:
    Description: Instance Warmup time for ASG
    Type: String
    Default: /

#Resources to be created
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsHostnames: true
      EnableDnsSupport: true
      InstanceTenancy: "default"
      Tags:
      - Key: Name
        Value: !Sub vpc-${AWS::StackName}

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
      - Key: Name
        Value: !Sub IGW-${AWS::StackName}

  AttachIGW:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref InternetGateway
      VpcId: !Ref VPC

  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: !Ref PublicSubnet1ID
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ '0', !GetAZs '' ]
      MapPublicIpOnLaunch: true       #it will assign public ip on launch
      Tags:
      - Key: Name
        Value: !Sub Public-Subnet1-${AWS::StackName}

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: !Ref PublicSubnet2ID
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ '1', !GetAZs '' ]
      MapPublicIpOnLaunch: true       #it will assign public ip on launch
      Tags:
      - Key: Name
        Value: !Sub Public-Subnet2-${AWS::StackName}

  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: !Ref PrivateSubnet1ID
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ '0', !GetAZs '' ]
      Tags:
      - Key: Name
        Value: !Sub Private-Subnet1-${AWS::StackName}  

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: !Ref PrivateSubnet2ID
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ '1', !GetAZs '' ]
      Tags:
      - Key: Name
        Value: !Sub Private-Subnet2-${AWS::StackName} 

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
      - Key: Name
        Value: !Sub PublicRouteTable-${AWS::StackName}

  PublicRoute:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnet1RouteTableAssoication:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet1

  PublicSubnet2RouteTableAssoication:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet2

  PrivateRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
      - Key: Name
        Value: !Sub PrivateRouteTable-${AWS::StackName}

  PrivateRoute:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NAT

  PrivateSubnet1RouteTableAssoication:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PrivateRouteTable
      SubnetId: !Ref PrivateSubnet1

  PrivateSubnet2RouteTableAssoication:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PrivateRouteTable
      SubnetId: !Ref PrivateSubnet2
  
  DeployAppLaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateData:
        ImageId: !Ref amiId
        InstanceType: !Ref InstanceType
        KeyName: !Ref KeyPairName
        BlockDeviceMappings:
          - DeviceName: '/dev/sda1'
            Ebs: 
              DeleteOnTermination: true
              Encrypted: false
              VolumeSize: !Ref InstanceVolumeSize
              VolumeType: gp2
        IamInstanceProfile:
          Arn:
            Fn::GetAtt:
            - S3accessProfile
            - Arn
        SecurityGroupIds:
          - !Ref AppSG

  DeployAppASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      VPCZoneIdentifier: 
        - !Ref PrivateSubnet1ID
        - !Ref PrivateSubnet2ID
      DesiredCapacity: !Ref DesiredCapacity
      TargetGroupARNs:
        - !Ref WebAppTargetGroup
      MaxSize: !Ref max
      MinSize: !Ref min
      MixedInstancesPolicy:
        InstancesDistribution:
          OnDemandAllocationStrategy: prioritized
          OnDemandBaseCapacity: !Ref OnDemandBaseCapacity
          OnDemandPercentageAboveBaseCapacity: !Ref OnDemandPercentageAboveBaseCapacity
          SpotAllocationStrategy: lowest-price
          SpotInstancePools: 3
          SpotMaxPrice: !Ref SpotMaxPrice
        LaunchTemplate:
          LaunchTemplateSpecification:
            LaunchTemplateId: !Ref DeployAppLaunchTemplate
            Version: !GetAtt DeployAppLaunchTemplate.LatestVersionNumber
      Tags:
          - Key: Name
            Value: !Sub ASG-${AWS::StackName}
            PropagateAtLaunch: True
    UpdatePolicy:
      AutoScalingReplacingUpdate:
        WillReplace: True

  AppScalingPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref DeployAppASG
      PolicyType: TargetTrackingScaling 
      EstimatedInstanceWarmup: !Ref EstimatedInstanceWarmup #-> 30
      TargetTrackingConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
        TargetValue: !Ref CPUPolicyTargetValue


Outputs:
  VPCID:
    Description: VPC
    Value: !Ref VPC
  PublicSubnet1:
    Description: Public Subnet1
    Value: !Ref PublicSubnet1
  PublicSubnet2:
    Description: Public Subnet1
    Value: !Ref PublicSubnet2
  PrivateSubnet1:
    Description: Private Subnet1
    Value: !Ref PrivateSubnet1
    Export:
      Name: !Sub "${AWS::StackName}-PrivateSubnet1ID"
  PrivateSubnet2:
    Description: Private Subnet1
    Value: !Ref PrivateSubnet2
    Export:
      Name: !Sub "${AWS::StackName}-PrivateSubnet2ID"
  ASG:
    Description: Information about the Auto scaling group
    Value: !Ref DeployAppASG
  AppSecruityGroup:
    Description: Application security group
    Value: !Ref AppSG
  ALBSG:
    Description: application load balancer security group. 
    Value: !Ref ALBSecurityGroup

```
Next go the cloudFormation AWS service and upload the yml file as follows, then click on next 

![](Images/b1.png)

Enter the stack name and click on next 

![](Images/b2.png)

Review the stack and submit 

![](Images/b3.png)

Now check the AWS resources created after successful stack deployment 

![](Images/b4.png)

## APPLICATION ARCHITECTURE

As we have completed with the architecture of VPC, We will now move to create resources and infrastructure for our application.For that we will follow these steps:
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
Create a jump server in the public subnet so that we can connect to the instances Inside private subnet. We will have a security group with SSH permission.

Now go to EC2 dashboard and launch an EC2 instance with the following configurations 

1. Click on launch instance 

![](Images/b5.png)

2. Write the name and choose OS as ubuntu 
    
![](Images/b6.png)

3. Next choose instance type as **t3a.small** and select the key pair 
   
![](Images/b7.png)

4. Select the vpc which you have deployed earlier and choose the public subnet, then create a new security group
   
![](Images/b8.png)

5. Lastly choose the storage as 8gb and if you want to connect using ssm, you can attach role in advanced settings 
   
![](Images/b9.png)
  
### STEP 2: Setup Mongodb instances

Create Mongo db instances in a private subnet and have 2 replicas for failover.
- Primary DB Name: mongodb-master
- Replication DB Names: mongodb-node-1 && mongodb-node-2

1. Click on launch instance and use the following configurations to launch 3 instances 

Enter the name for master mongodb and later you can update the name for secondary mongodb nodes using EC2 console 
![](Images/b10.png)
![](Images/b11.png)
![](Images/b12.png)
![](Images/b13.png)
![](Images/b14.png)

2. Verify all the 3 instances are launched in private subnet and hence **NO PUBLIC IP** is attached to these instances 

![](Images/b17.png)

3. Allow port 27017 in mongodb security group 

![](Images/b18.png)

4. Now SSH into the each DB instance and run the following commands  
   
**Installation Steps:**
```
apt update
sudo apt install dirmngr gnupg apt-transport-https ca-certificates software-properties-common
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
sudo add-apt-repository 'deb [arch=amd64] https://repo.mongodb.org/apt/ubuntu       focal/mongodb-org/4.4 multiverse'
sudo apt install mongodb-org -y
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
mongo --eval 'db.runCommand({ connectionStatus: 1 })'
```
![](Images/b15.png)

- Open sudo vi /etc/mongod.conf
    - Modify bindIp to 0.0.0.0 
    - add Replication set as
     replication:
          replSetName: rs0

![](Images/b16.png)

- Now restart the mongodb service 

      sudo systemctl restart mongod

![](Images/b19.png)    

5. To make one instance Primary, login to that instance and initiate it and add the other two instances as replicas.

Run the following command to login into the mongo shell of primary/master node

    mongo 

![](Images/b20.png)

On the primary instance, add the worker nodes 

```
rs.initiate();
```
![](Images/b21.png)

```
rs.add(‘worker1PrivateIP:27017’);
rs.add(‘worker2PrivateIP:27017’);

```
![](Images/b22.png)

```
 rs.status();
```
![](Images/b23.png)

**create a user for authentication.**

```
use admin;

db.createUser(
{	user: "UserName",
	pwd: "Password",

	roles:[{role: "userAdminAnyDatabase" , db:"admin"}]})
```

![](Images/b24.png)

Now exit the mongo shell and connect with the mongodb string in the following format 
```
mongodb://roadtodevops:Roadtodevops123@10.0.4.163:27017,10.0.4.20:27017,10.0.4.38:27017/conduit?authSource=admin&replicaSet=rs0
```

![](Images/b25.png)

### STEP 3: Create AMI and setup EC2 instance for Backend Node JS application

Create an instance in a public subnet and do the required package installation like Nodejs,PM2, CodeDeploy Agent, CloudWatch Agent and make a template from that.After creating a template we can terminate the instance or can stop it, if we have further use.

- Name: road-to-devops-backend
- Type: t3a.small
- Subnet: Public_Subnet_1
- SG: road-to-devops-backend-sg [allow port 22 and 3000 in inbound rules]
- Storage: gp2 , 8GB

![](Images/b26.png)

![](Images/b27.png)

![](Images/b28.png)

Now SSH into the instance and run the following commands:

#### Package Installation (NODEJS, PM2)

```
apt update
apt install net-tools -y
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
apt-get install gcc make -y
apt-get install nodejs -y 
npm  install pm2@latest -g
```
Now verify the version installed 

 ![](Images/b29.png)

#### Install CODE DEPLOY AGENT
```
sudo apt update -y
sudo apt install ruby-full -y
wget https://aws-codedeploy-region.s3.region.amazonaws.com/latest/install (update the region)
chmod +x ./install
sudo ./install auto
sudo service codedeploy-agent status
```
 ![](Images/b30.png)

#### Configure AWS-CLI

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
 ![](Images/b31.png)

#### Install CLOUD WATCH AGENT

firstly add SSM FULL ACCESS policy to the IAM role attached to the instance

 ![](Images/b32.png)

Now run the following commands to install cloudwatch agent 

```
wget https://s3.amazonaws.com/amazoncloudwatch-agent/debian/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
sudo apt-get update && sudo apt-get install collectd
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```
Here it will ask few questions for monitoring metrics and logs, answer then accordingly 

```
root@ip-10-0-1-76:~# sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
================================================================
= Welcome to the Amazon CloudWatch Agent Configuration Manager =
=                                                              =
= CloudWatch Agent allows you to collect metrics and logs from =
= your host and send them to CloudWatch. Additional CloudWatch =
= charges may apply.                                           =
================================================================
On which OS are you planning to use the agent?
1. linux
2. windows
3. darwin
default choice: [1]:

Trying to fetch the default region based on ec2 metadata...
Are you using EC2 or On-Premises hosts?
1. EC2
2. On-Premises
default choice: [1]:

Which user are you planning to run the agent?
1. root
2. cwagent
3. others
default choice: [1]:

Do you want to turn on StatsD daemon?
1. yes
2. no
default choice: [1]:

Which port do you want StatsD daemon to listen to?
default choice: [8125]

What is the collect interval for StatsD daemon?
1. 10s
2. 30s
3. 60s
default choice: [1]:

What is the aggregation interval for metrics collected by StatsD daemon?
1. Do not aggregate
2. 10s
3. 30s
4. 60s
default choice: [4]:

Do you want to monitor metrics from CollectD? WARNING: CollectD must be installed or the Agent will fail to start
1. yes
2. no
default choice: [1]:

Do you want to monitor any host metrics? e.g. CPU, memory, etc.
1. yes
2. no
default choice: [1]:

Do you want to monitor cpu metrics per core?
1. yes
2. no
default choice: [1]:

Do you want to add ec2 dimensions (ImageId, InstanceId, InstanceType, AutoScalingGroupName) into all of your metrics if the info is available?
1. yes
2. no
default choice: [1]:

Do you want to aggregate ec2 dimensions (InstanceId)?
1. yes
2. no
default choice: [1]:

Would you like to collect your metrics at high resolution (sub-minute resolution)? This enables sub-minute resolution for all metrics, but you can customize for specific metrics in the output json file.
1. 1s
2. 10s
3. 30s
4. 60s
default choice: [4]:

Which default metrics config do you want?
1. Basic
2. Standard
3. Advanced
4. None
default choice: [1]:

Current config as follows:
{
        "agent": {
                "metrics_collection_interval": 60,
                "run_as_user": "root"
        },
        "metrics": {
                "aggregation_dimensions": [
                        [
                                "InstanceId"
                        ]
                ],
                "append_dimensions": {
                        "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
                        "ImageId": "${aws:ImageId}",
                        "InstanceId": "${aws:InstanceId}",
                        "InstanceType": "${aws:InstanceType}"
                },
                "metrics_collected": {
                        "collectd": {
                                "metrics_aggregation_interval": 60
                        },
                        "disk": {
                                "measurement": [
                                        "used_percent"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "mem": {
                                "measurement": [
                                        "mem_used_percent"
                                ],
                                "metrics_collection_interval": 60
                        },
                        "statsd": {
                                "metrics_aggregation_interval": 60,
                                "metrics_collection_interval": 10,
                                "service_address": ":8125"
                        }
                }
        }
}
Are you satisfied with the above config? Note: it can be manually customized after the wizard completes to add additional items.
1. yes
2. no
default choice: [1]:

Do you have any existing CloudWatch Log Agent (http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AgentReference.html) configuration file to import for migration?
1. yes
2. no
default choice: [2]:

Do you want to monitor any log files?
1. yes
2. no
default choice: [1]:
2
Saved config file to /opt/aws/amazon-cloudwatch-agent/bin/config.json successfully.
Current config as follows:
{
        "agent": {
                "metrics_collection_interval": 60,
                "run_as_user": "root"
        },
        "metrics": {
                "aggregation_dimensions": [
                        [
                                "InstanceId"
                        ]
                ],
                "append_dimensions": {
                        "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
                        "ImageId": "${aws:ImageId}",
                        "InstanceId": "${aws:InstanceId}",
                        "InstanceType": "${aws:InstanceType}"
                },
                "metrics_collected": {
                        "collectd": {
                                "metrics_aggregation_interval": 60
                        },
                        "disk": {
                                "measurement": [
                                        "used_percent"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "mem": {
                                "measurement": [
                                        "mem_used_percent"
                                ],
                                "metrics_collection_interval": 60
                        },
                        "statsd": {
                                "metrics_aggregation_interval": 60,
                                "metrics_collection_interval": 10,
                                "service_address": ":8125"
                        }
                }
        }
}
Please check the above content of the config.
The config file is also located at /opt/aws/amazon-cloudwatch-agent/bin/config.json.
Edit it manually if needed.
Do you want to store the config in the SSM parameter store?
1. yes
2. no
default choice: [1]:

What parameter store name do you want to use to store your config? (Use 'AmazonCloudWatch-' prefix if you use our managed AWS policy)
default choice: [AmazonCloudWatch-linux]

Trying to fetch the default region based on ec2 metadata...
Which region do you want to store the config in the parameter store?
default choice: [us-east-1]

Which AWS credential should be used to send json config to parameter store?
1. ASIAUP4W3X5URxxXUPQJ5T(From SDK)
2. Other
default choice: [1]:

Please make sure the creds you used have the right permissions configured for SSM access.
Which AWS credential should be used to send json config to parameter store?
1. ASIAUP4W3X5URxxXUPQJ5T(From SDK)
2. Other
default choice: [1]:

Successfully put config to parameter store AmazonCloudWatch-linux.
Program exits now.
root@ip-10-0-1-76:~#
```

- You can also Do manual setup:

```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```

- To Do manual change in configuration file

          vi  /opt/aws/amazon-cloudwatch-agent/bin/config.json

- To Start

          /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a start

- Check Status of Agent

          /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status

 ![](Images/b33.png)

Now create the AMI of this instance on which all the configurations have been installed 

- Go to the EC2 dashboard console and select **actions** from the drop down list, then click on **create image**
  
   ![](Images/b34.png)

- Write the name of the AMI and click on **Create Image**

 ![](Images/b35.png)

### STEP 4: Create Instance in Private Subnet for nodejs application with the created template.
- Name: road-to-devops-backend
- Type: t3a.small
- AMI: backend-ami
- Subnet: Private_Subnet

 ![](Images/b36.png)

 ![](Images/b37.png)

  ![](Images/b38.png)

### STEP 5: Create Load Balancer 

Click on **create loadbalancer**

 ![](Images/b39.png)

Now choose application loadbalancer and click on next

 ![](Images/b40.png)

Enter the name of loadbalancer and keep **internet facing**

 ![](Images/b41.png)

Choose public subnets 

 ![](Images/b42.png)

Create a new security group for alb

 ![](Images/b46.png)

Create target group with following configuration 

![](Images/b43.png)

![](Images/b44.png)

![](Images/b45.png)

Now check that target group has been attached to the loadbalancer 

![](Images/b47.png)

Lastly click on create loadbalancer and check that load balancer has been created successfully 

Now add listener as follows 

![](Images/b73.png)

**note: allow port 80 and 443 from everywhere in alb sg**

### STEP 6: Create a launch template from the AMI and use that in autoscaling group.

Create launch template with following configurations 

![](Images/b48.png)
![](Images/b49.png)
![](Images/b50.png)
![](Images/b51.png)

Launch AutoScaling group using the launch template created above 

![](Images/b52.png)
![](Images/b53.png)
![](Images/b54.png)
![](Images/b55.png)

now click on next and create AutoScaling group 

### STEP 7: Setup code deploy

click on **create application** in code deploy 

![](Images/b56.png)

now enter the application name and compute type, then click on **create application**

![](Images/b57.png)

create deployment group with following configurations 

![](Images/b58.png)
![](Images/b59.png)
![](Images/b60.png)

now click on **create deployment group**

**note: allow port 300 from alb-sg in backend-sg**

### STEP 8: Store environment variables 

We are using parameter store in systems manager to store DB_URL 

![](Images/b70.png)

### STEP 9: Codebuild Configurations

clone the code from github: https://github.com/sq-ldc/nodejs-example

Create the build project with the following configurations

![](Images/b61.png)

Either you can choose a public repo with the above github link or you can clone the code and setup a GITHUB REPOSITORY for your project, then use it under private repository 

![](Images/b62.png)

![](Images/b63.png)

![](Images/b64.png)

![](Images/b65.png)

click on **create build project**

Now also add **ssmfullaccess** policy in code build role

![](Images/b71.png)

### STEP 10: Codepipeline Configurations

Create Code pipeline with the following configurations 

![](Images/b66.png)

![](Images/b67.png)

![](Images/b68.png)

![](Images/b69.png)

Now click on next and review the pipeline, then click on **create pipeline**

### STEP 11: Create a domain in Route 53

- Domain Name: labs.squareops.in
- Sub-Domain Name: backend.labs.squareops.in

![](Images/b72.png)

now test the url

![](Images/b74.png)

### STEP 12: Frontend Amplify setup
- We will first put our reactjs application code to github

- Link to GITHUB Application: https://github.com/sq-ldc/reactjs-example

- On AWS AMPLIFY , We will first host a web application and choose our project location.

**CodeRepo**

 ![](Images/b75.png)

**Repo connectivity**

 ![](Images/b76.png)

 ![](Images/b77.png)

**Deployment**

 ![](Images/b78.png)

**Set environment variables**

![](Images/b79.png)

**Amplyfy.yml configuration**

```
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm install
    build:
      commands:
        - echo "BACKEND_URL=${DOMAIN_URL}" > .env
        - npm run build
  artifacts:
    # IMPORTANT - Please verify your build output directory
    baseDirectory: /
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```
Amplify pipeline has been deployed successfully 

![](Images/b80.png)

![](Images/b81.png)

### STEP 13: Observability (Monitoring & Logging)

Use can check Cloudwatch metrics for monitoring and log groups  
- ec2 nodes 
- application loadbalancer 
- endpoints monitoring

![](Images/d1.png)

**MERN STACK HAD BEEN DEPLOYED SUCCESSFULLY WITH THE FOLLOWING COMPONENTS:**

1. Three node mongodb setup is up and running in private subnets
2. backend deployed on EC2 instance with private
3. frontend deployed on Amplify 
4. DB connection string stored as ENVIRONMENT VARIABLE in AWS parameter store.
5. In the loadbalancer security group, just allow ports 80 and 443 from everywhere (0.0.0.0)
6. In the application security group, allow traffic from loadbalancer security group 
7. domain management is done using AWS ROUTE 53 
8. Network stack deployed can be used in production environments 
9. CI/CD setup for both backend and frontend 

