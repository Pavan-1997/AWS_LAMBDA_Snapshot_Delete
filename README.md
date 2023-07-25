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
---
1. Now goto EC2 from AWS Console -> Click on Launch instance

    Give a name
    
    Use Ubuntu 22.04 LTS as an image
    
    Instance type as t2.micro
    
    Create a key pair if already present use the existing one
    
    Click on Launch instance

![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/02e08640-f00b-4f10-9a02-1d40c5b62e41)


2. Check in Volumes you should have a volume of the instance created

![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/0f905ac1-d3a8-4aa2-906f-98fbd1abcc52)


3. On the left pane under Elastic Block Storage select Snapshots -> Click on Create snapshot -> Select the Volume ID that is associated with the instance that has been created -> Give a Description -> Click on Create snaphot

![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/95144c87-c6b7-4d7d-994b-81e6fc2e1a5d)


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

![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/d65461bf-8087-4ed5-bde9-f3d856a46c52)


6. Now terminate the EC2 instance -> Check the instance and volume to be removed except the snapshot

![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/8b4b44cd-57c0-4971-9e39-1b2e4b0baa48)

![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/a15c16c0-50f1-4d18-89fe-bd876af641f9)


7. Perform the Step 5. again which should automatically delete the snaphot which is no longer associated with the volume 

![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/856fd6d0-a2ea-4abc-a376-ea5e26ff4914)


8. Manullay creating a volume -> Go to Volumes on the left pane in EC2 - Click on Create volume -> Size - 1 GiB -> Create Volume


9. Creating a snapshot -> Go to Snapshots on the left pane in EC2 - Click on Create snapshot -> Select the Volume ID that is associated with the instance that has been created -> Give a Description -> Click on Create snaphot

![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/8fb488cf-4c77-4def-978e-62af19e0413b)


10. Now again perform the Step 5. again which should automatically delete the snaphot whick is associated with the volume with no instance

![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/3c4f9e03-9d85-42a1-967c-0f587ca31587)


11. Triggering this with AWS CloudWatch -> Goto CloudWatch from AWS Console -> On the left pane click on Event and Rules -> Click on Create rule -> Give a Rule name -> Select 	Schedule -> Click on Continue in EventBridge Scheduler -> Now select Recurring schedule -> Click on Rate-based schedule -> Rate - 24 hours -> Flexible time windows - 15 minutes -> Select Target API - Templated targets - AWS Lambda -> Under Invoke - Lambda function select the Lambda function created -> Click on Next -> Click on Next -> Click on Create schedule
 
![image](https://github.com/Pavan-1997/AWS_Lambda_Stale_Snapshot_Delete/assets/32020205/0e42fd81-8dde-43fc-9a03-0eb5b8f221e2)

