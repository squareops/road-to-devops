



##### Below is the Boto script to stop all EC2 instances in the mentioned regions
```
import boto3

def lambda_handler(event, context):
   
    # Provide regions here (eg ['us-west-2', 'ap-southeast-2'], or use [] for all regions
    regions_to_check = []

    # Check all regions?
    if not regions_to_check:
        regions_to_check = [r['RegionName'] for r in boto3.client('ec2').describe_regions()['Regions']]

    for region in regions_to_check:
        print('Region: ', region)
    
        ec2_resource = boto3.resource('ec2', region_name = region)
        
        running_filter = {'Name':'instance-state-name', 'Values':['running']}
        instances = ec2_resource.instances.filter(Filters=[running_filter])
        
        for instance in instances:
            action = 'stop' # Default action
            
            # Extract action from 'Auto-Stop' tag (if present)
            if instance.tags != None:
                for tag in instance.tags:
                    if tag['Key'].lower() == 'auto-stop':
                        action = tag['Value'].lower()
            
            try:
                if action == 'stop':
                    print(f'Stopping instance {instance.id}')
                    instance.stop()
                else:
                    print(f'Ignoring instance {instance.id}')
            except Exception as e:
                print(e)
                
```