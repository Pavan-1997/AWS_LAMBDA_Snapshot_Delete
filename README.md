# AWS Lambda Stale Snapshot Delete 

As a DevOps Engineer we create a Lambda function that identifies EBS snapshots that are no longer associated with any active EC2 instance and deletes them to save on storage costs. The Lambda function fetches all EBS snapshots owned by the same account ('self') and also retrieves a list of active EC2 instances (running and stopped). For each snapshot, it checks if the associated volume (if exists) is not associated with any active instance. If it finds a stale snapshot, it deletes it, effectively optimizing storage costs.

---
Why do we switch from On-Prem to Cloud
```
1. Reduce overhead of Infra
2. Optimize the Infra cost
```

- Snapshot is like taking a backup of the EC2 instance volume, it's like a image of the volume

- If volume and snapshots are not deleted after deleting the instance then AWS charges for it

- AWS as ~200 resources

How to optimize? - Look for stale resources:
```
- Send out notifications
- Delete the resources
```

1. Now goto EC2 from AWS Console -> Click on Launch instance

Give a name

Use Ubuntu 22.04 LTS as an image

Instance type as t2.micro

Create a key pair if already present use the existing one

Click on Launch instance



2. Check in Volumes you should have volume of the instance created


3. On the left pane under Elastic Block Storage select Snapshots -> Click on Create snapshot -> Select the Volume ID that is associated with the instance that has been created -> Give a Description -> Click on Create snaphot


4. a) Now goto Lambda from AWS console ->  Give a Function name ->  Runtime - Python 3.10 -> Click on Create function -> Click on Deploy -> Use the code from the file in repo and click on Test -> Give a Event name -> Click on Save > Click on Test

You will get error 

Default execution time for AWS Lambda is 3 seconds

b) Go to Configuration tab in AWS lambda function -> Under General configuration -> Click on Edit -> Change the timeout from 3 to 10 seconds -> Click on Save

AWS charges are directly propotional to AWS Lambda function execution (timeout) time

c) Go to Configuration tab in AWS lambda function -> Under Permissions -> Click on the Role name -> Click on Add permissions and Attach policies 

d) Click on Create policy -> Click on EC2 -> Search and select DeleteSnapshot, DescribeSnapshots, DescribeInstances, DescribeVolumes  -> Reseouces select All -> Click on Next -> Give a Policy name -> Click on Create policy

Now go back to the Step 4.c) -> Select the policy created in Step 4.d) -> Click on Add permissions 


5. Now go back to the Step 4.a) 

Now the script executes


6. Now terminate the EC2 instance -> Check the instance and volume to be removed except the snapshot


7. Perform the Step 5. again which should automatically delete the snaphot whick is no longer associated with the volume 


8. Manullay creating a volume -> Go to Volumes on the left pane in EC2 - Click on Create volume -> Size - 1 GiB -> Create Volume


9. Creating a snapshot -> Go to Snapshots on the left pane in EC2 - Click on Create snapshot -> Select the Volume ID that is associated with the instance that has been created -> Give a Description -> Click on Create snaphot


10. Now again perform the Step 5. again which should automatically delete the snaphot whick is associated with the volume with no instance


11. Triggering this with AWS CloudWatch -> Goto CloudWatch from AWS Console -> On the left pane click on Event and Rules -> Click on Create rule -> Give a Rule name -> Select 	Schedule -> Click on Continue in EventBridge Scheduler -> Now select Recurring schedule -> Click on Rate-based schedule -> Rate - 24 hours -> Flexible time windows - 15 minutes -> Select Target API - Templated targets - AWS Lambda -> Under Invoke - Lambda function select the Lambda function created -> Click on Next -> Click on Next -> Click on Create schedule
