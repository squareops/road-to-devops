# Hands-on example of creating CODE PIPELINE from scratch: 

# Contents
- [Hands-on example of creating CODE PIPELINE from scratch:](#hands-on-example-of-creating-code-pipeline-from-scratch)
- [Contents](#contents)
- [Overview](#overview)
  - [Step - 1: Setup Wordpress on EC2](#step---1-setup-wordpress-on-ec2)
  - [Step - 2: Upload Source code on Github](#step---2-upload-source-code-on-github)
  - [Step – 3: Create Code Pipeline](#step--3-create-code-pipeline)
    - [Create a Code Deploy service role](#create-a-code-deploy-service-role)
    - [Setup Environment Variables](#setup-environment-variables)
    - [Create Code Deploy Application](#create-code-deploy-application)
    - [Create Source stage](#create-source-stage)
      - [Connecting using Github Version 1](#connecting-using-github-version-1)
      - [Connecting using Github Version 2 ( secure and recommended way )](#connecting-using-github-version-2--secure-and-recommended-way-)
    - [Create Build Stage](#create-build-stage)
    - [Create Code Deploy stage](#create-code-deploy-stage)

# Overview

AWS Services used are as follows:

1. GitHub as Source

2. AWS CodeBuild for building the artifact using buildspec.yml file

3. S3 Bucket to store the artifact

4. AWS CodeDeploy for deploying the configuration present in the artifact from AWS CodeBuild using appspec.yml file

5. AWS CodePipeline

**note: Codepipeline comprises of three stages i.e source, build and deploy.**

If you're following the tutorial, Module wise so as of now your WordPress website must be up and running. If not, please setup the WordPress on EC2 and then proceed with further steps  

## Step - 1: Setup Wordpress on EC2

Using this blog, you can setup the [WordPress Website](https://github.com/squareops/road-to-devops/blob/develop/Level-2/M2-WebApp2TierHA/L01-WorpressHA.md)

## Step - 2: Upload Source code on Github 

- For this example we have taken source code from squareops repository (https://github.com/sq-ldc/wordpress-example)


                git clone https://github.com/sq-ldc/wordpress-example

## Step – 3: Create Code Pipeline 

Code pipeline consists of 3 main stages i.e 

**1. Source**   

**2. Build**

**3. Deploy**   

### Create a Code Deploy service role 

1. Sign in to the AWS Management Console and open the IAM console at https://console.aws.amazon.com/iam/.

2. In the navigation pane, choose Roles, and then choose Create role.

3. On the Create role page, choose AWS service, and from the Choose the service that will use this role list, choose CodeDeploy.From Select your use case, choose your use case:

- For EC2/On-Premises deployments, choose CodeDeploy.

- For Amazon ECS deployments, choose CodeDeploy - ECS.

- For AWS Lambda deployments, choose CodeDeploy for Lambda.

4. Choose Next: Permissions.

5. On the Attached permissions policy page, the permission policy is displayed. Choose Next: Tags.

6. On the Review page, in Role name, enter a name for the service role (for example, CodeDeployServiceRole), and then choose Create role.

7. You can also enter a description for this service role in Role description.

8. If you want this service role to have permission to access all currently supported endpoints, you are finished with this procedure.

9. To restrict this service role from access to some endpoints, in the list of roles, browse to and choose the role you created, and continue to the next step.

10. On the Trust relationships tab, choose Edit trust relationship.

You should see the following policy, which provides the service role permission to access all supported endpoints:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "codedeploy.amazonaws.com"
                ]
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

11. The code deploy service code created with name **code-deploy-role-for-ec2**
    
      ![](Images/p19.png)

### Setup Environment Variables

For storing environment variables, you can you these two AWS services:

1. Parameter Store 
2. Secrets Manager

In this blog we will use Parameter store to keep environment variables 

- Click on Create Parameter in Systems Manager 

  ![](Images/p25.png)

- Enter parameter name and value as follows 

  ![](Images/p26.png)

- Now respectively store all the environment variables 
  
  ![](Images/p27.png)

### Create Code Deploy Application 

We will start by creating the Code Deploy application, for this go to the CODE DEPLOY service and click on **Create Application**

  ![](Images/p17.png)

Enter the name of the application and choose the platform of deployment, then click on **create application**

  ![](Images/p18.png)

Next click on **create deployment group**, then enter the name of the deployment group as **road-to-devops-application** and choose service role which you created above 


  ![](Images/p20.png)

Choose deployment type as **In-Place** and then choose the name of the EC2 instance on which application is setup

  ![](Images/p21.png)

Rest keep the configurations as default 

  ![](Images/p22.png)

Lastly check if you need to add loadbalancer and then click on **create deployment group**

  ![](Images/p23.png)

Check that you have successfully created **Deployment Group** 

  ![](Images/p24.png)

Now firstly go to the CodePipeline section and create the CodePipeline 

  ![](Images/p1.png)

Just enter the name of the CodePipeline such as for this blog, we are creating "road-to-devops-pipeline"

  ![](Images/p2.png)

  ![](Images/p3.png)

### Create Source stage 

- The first stage includes the Source where we create connection from github/bitbucket to the code pipeline using login credentials or code star connection

  ![](Images/p4.png)

#### Connecting using Github Version 1 

  ![](Images/p5.png)

Click on connect to Github and authorize with the Githubrepo 

  ![](Images/p6.png)


#### Connecting using Github Version 2 ( secure and recommended way )

Choose GITHUB version 2 in the source provider 

  ![](Images/p7.png)

Click on connect to github and enter the connection name 

  ![](Images/p8.png)

Then click on **Connect to Github** and click on **Authorize AWS connector for Github**

  ![](Images/p9.png)

After this it will redirect to the github source connection page 

  ![](Images/p10.png)

Here enter the Repository name and branch name which needs to be deployed 

### Create Build Stage 

Start with Code build stage and choose AWS Codebuild from the drop down list

  ![](Images/p11.png)

Click on create the build project and it will redirect to create a build project where we have to provide the name of the project and then the source provider which can be from git to S3 bucket **(using default s3 bucket )**.

  ![](Images/p12.png)

Keep Image environment settings as follows 

  ![](Images/p13.png)

Use build spec file stored in code as the code build project basically looks for the buildspec file in the root directory of the source provider . If the buildspec file has some other name then we can provide the same name in the project build. Once the project is created then we can build the project . 

  ![](Images/p14.png)

Last step is to click on "Continue to CodePipeline" 

  ![](Images/p15.png)

  ![](Images/p16.png)

Now click on next and add code deploy stage 

Here is the buildpec.yml file configuration which is stored in the root directory of the github repository 
```
version: 0.2

run-as: root

env:
  parameter-store:
    DB_NAME: "/road_to_devops/wordpress/DB_NAME"
    DB_USER: "/road_to_devops/wordpress/DB_USER"
    DB_PASSWORD: "/road_to_devops/wordpress/DB_PASSWORD"
    DB_HOST: "/road_to_devops/wordpress/DB_HOST"

phases:
  install:
    commands:
      - apt update -y
  pre_build:
    commands:
      - sed -i 's/database_name_here/'$DB_NAME'/' wp-config.php
      - sed -i 's/username_here/'$DB_USER'/' wp-config.php
      - sed -i 's/password_here/'$DB_PASSWORD'/' wp-config.php
      - sed -i 's/localhost/'$DB_HOST'/' wp-config.php
      - cat wp-config.php
  build:
    commands:
      - grep -Fq "road_to_devops" wp-config.php
artifacts:
  files:
    - '**/*'
```

- The buildspec file basically contains the information about variables to be taken from and then pre-build, build and post build steps.

- The build artifact stored in s3 bucket

After the successful build is done we can move to the next step called CodeDeploy.Here we will first create the application .After the application is created we will now create the deployment group where we can provide the information like service role , type of deployment, environment (like instances which can be on-premise or from autoscaling group) and the load balancer.

### Create Code Deploy stage 

In the Code Pipeline, the last stage is the deployment stage where you need to choose the deployment application and deployment group name which you have created above

  ![](Images/p28.png)

Click on next and review the code pipeline configuration. Then click on **Create Pipeline**

  ![](Images/p29.png)

Make sure you have given required permissions to your code build role, here we have given these permissions:

- AmazonSSMFullAccess
  
  ![](Images/p31.png)


Attach S3 full access policy to your EC2 instance IAM ROLE 

  ![](Images/p30.png)

Install code deploy agent as follows on ec2 instance or adding as user data in launch template: 

```
#!/bin/bash
sudo apt-get update
sudo apt-get install -y ruby
wget https://aws-codedeploy-us-east-1.s3.amazonaws.com/releases/codedeploy-agent_1.0-1.1597_all.deb
mkdir codedeploy-agent_1.0-1.1597_ubuntu20
dpkg-deb -R codedeploy-agent_1.0-1.1597_all.deb codedeploy-agent_1.0-1.1597_ubuntu20
sed 's/2.0/2.7/' -i ./codedeploy-agent_1.0-1.1597_ubuntu20/DEBIAN/control
dpkg-deb -b codedeploy-agent_1.0-1.1597_ubuntu20
dpkg -i codedeploy-agent_1.0-1.1597_ubuntu20.deb
sudo systemctl start codedeploy-agent
sudo systemctl enable codedeploy-agent

Note:- The minimum supported version of the CodeDeploy agent is 1.5.0. We can check the available version of code deploy agent using command “aws s3 ls s3://aws-codedeploy-region-identifier/releases/ | grep '\.deb$"

```

For checking the service 

        sudo service codedeploy-agent status

Here is the appspec.yml file configuration 

```
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/road-to-devops
file_exists_behavior: OVERWRITE
    
hooks:
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
      
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
      
  ValidateService:
    - location: scripts/curl_check.sh
      timeout: 300
      runas: root
```
- The appspec file contains information about the commands to be run during different hooks like ApplicationStop, ApplicationStart, BeforeInstall.
- scripts folder is created in root of the github directory which consists of 3 files: start, stop and validate

Successfully executed the deployment phase from the build artifact stored in s3

  ![](Images/p32.png)

  ![](Images/p33.png)

  ![](Images/p34.png)

  ![](Images/p35.png)

You can see that codepipeline has been executed successfully 

  ![](Images/p36.png)

Details of Instance, in which the application was deployed

![](Images/p37.png)

After hitting the Public IPv4 of the instance, the wordpress install page appeared

![](Images/d11.png)

![](Images/d12.png)

Hence, your application is deployed successfully using CI/CD 