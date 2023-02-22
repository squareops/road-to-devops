##### Below is the Boto script to stop all RDS

```
import boto3
from datetime import datetime

# Get current time in format yyyy-mm-dd
# define boto3 the connection
current_date = datetime.today().strftime('%Y-%m-%d')
# get all regions
available_regions = boto3.Session().get_available_regions('rds')
  
def lambda_handler(event, context):

    for region in available_regions:
        rds = boto3.client('rds', region_name=region)
        

      # Define instances 
        instances = rds.describe_db_instances()

        stopInstances = []   
    

      # Define and locate tags.
        for instance in instances["DBInstances"]:
    
            tags = rds.list_tags_for_resource(ResourceName=instance["DBInstanceArn"])
            
            for tag in tags["TagList"]:

                      stopInstances.append(instance["DBInstanceIdentifier"])
                      rds.stop_db_instance(DBInstanceIdentifier=instance["DBInstanceIdentifier"])    
                    
                      pass

                      pass


     # print  all instances that will stop. 
    if len(stopInstances) > 0:
       print ("stopInstances")
    else:
       print ("No rds instances to shutdown.")

```