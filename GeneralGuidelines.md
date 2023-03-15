# General Guidelines : 

1. This road map uses AWS ( Amazon Web Services ) to build primary concepts of cloud. For this you need to signup on [AWS account](https://aws.amazon.com/console/)

2. All work suggested being done on "**Ubuntu 20.04**" AMI. 

    - Recommended AWS Region - N.Virginia us-east-1

    - Recommended EC2 instance types - t3a.nano, t3a.micro and t3a.small, t4g.nano, t4g.micro, and t4g.small

    - Use AWS systems manager to SSH in maximum possible cases, instead of native SSH client

    - Never use the root user to execute commands, instead use sudo with your user. ( do not use sudo -i or sudo -su )

    - Use GP3 EBS Volume with max size up to 15 GB Max

    - Recommended RDS instance types - db.t3.micro and db.t3.small .Use General Purpose Storage type and not provisioned IOPS for RDS

    - Launch on-demand instances. Create a cron runs every night at 9 PM which stops the running ec2 instances and RDS Databases. You can start the servers next day when you start working. Here are scripts 

        - [stop EC2 instances](https://github.com/sq-ia/aws-stop-start-services/blob/develop/ec2-instances-stop-start/stop-EC2.md) 

        - [stop RDS](https://github.com/sq-ia/aws-stop-start-services/blob/develop/rds-stop-start/stop-RDS.md
        - )

    - ##### Do not push AWS account Access keys / Secret keys to any git repository.

3. All tutorials and video links will be followed by quick hands-on with the concepts mentioned in tutorial

4. Tagging instructions for resources in AWS account ( TO BE FOLLOWED STRICTLY ). Aws-tagging-best-practices: https://www.cloudforecast.io/blog/aws-tagging-best-practices/ 

    - Do not use names in the resource name. Instead add a tag with the Owner Key (Owner:<your-name>).

    - Always check for the cost of resources you are launching and take only small resources for this RoadMap

    - Check CloudWatch, data transfer, alarm, and Route 53 health check costs


### Note: Kindly follow these instructions STRICTLY while going through all levels of the Roadmap 
