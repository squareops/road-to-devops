# Install the unified CloudWatch agent on the AMI and use SSM Parameter store to configure unified CloudWatch agent

1. Installation of unified CloudWatch Agent on EC2 instance

```
#!/bin/bash
sudo mkdir /tmp/cwa
cd /tmp/cwa
sudo wget https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/AmazonCloudWatchAgent.zip -O AmazonCloudWatchAgent.zip
sudo apt-get install -y unzip
sudo unzip -o AmazonCloudWatchAgent.zip
sudo ./install.sh
sudo mkdir -p /usr/share/collectd/
sudo touch /usr/share/collectd/types.db 
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
{ 
"status": "running", 
"starttime": "2020-06-07T10:04:41+00:00", 
"version": "1.245315.0" 
}
systemctl status amazon-cloudwatch-agent.service
```

Run the CodeDeploy Agent wizard after this, and SSM Parameter Store will be configured there itself.

2. Code to use SSM Parameter Store to configure Unified CloudWatch Agent

```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c ssm:AmazonCloudWatch-AadeshUbuntu –s
```
3. Created a CloudWatch Dashboard, added EC2 and RDS Metrics

![](Images/a3.png)

4. Shipped logs from the instance:

a. Access log of Nginx Webserver (access.log)

![](Images/a4.png)

b. Error log of Nginx Web Server (error.log)

![](Images/a5.png)

c. syslog

![](Images/a6.png)

5. Created metric filter for 404 status code access.log

![](Images/a7.png)

a. Created a 404 Alarm based on this metric and an SNS Alert also executed

![](Images/a8.png)

6. Alarms

![](Images/a9.png)

7. Chaos Testing

a. EC2 SNS Trigger for CPUUtilization Metric

![](Images/a10.png)

Command used for stress testing:

            sudo apt install stress
            stress –c 4 –m 6 –d 4

b. RDS

i. SNS alert for DatabaseConnections Metric

![](Images/a11.png)

Script used to test DB Connections

![](Images/a12.png)


c. SNS alert for FreeStorageSpace Metric

a. ELB SNS Trigger for HealthyHostCount Metric

![](Images/a13.png)

b. Tested by manually terminating target instances.

![](Images/a14.png)
