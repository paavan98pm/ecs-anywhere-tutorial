### Step 1: The regional deployment  

First of all, we will deploy the application in the aws region of your choice. During the [re:Invent demo](https://aws.amazon.com/blogs/containers/introducing-amazon-ecs-anywhere/) we have shown how to setup the regional deployment using [the new Docker Compose CLI that can deploy to Amazon ECS](https://docs.docker.com/cloud/ecs-integration/). For this tutorial you have two options: 

- using [AWS Copilot](../app/copilot/README.md) 
- using the [Docker Compose integration with ECS](../app/docker-compose/README.md) 

Please pick one and complete the steps. Deploying in the region is a mandatory step for this tutorial. Once you have deployed the application in the region, we can start testing that it works correctly. 


### Testing the regional deployment  

If you have a working in-region deployment, we will inspect the two folders the application has created on the EFS file system mounted under `/data`. In particular we will make sure they are empty and we will create 5 files (named `001`, `002`, `003`, `004` and `005`) in the `sourcefolder`:
```
# ls /data/sourcefolder     
# ls /data/destinationfolder
# cd /data/sourcefolder/ && touch 001 002 003 004 005 && cd /
# ls /data/sourcefolder    
001  002  003  004  005
# ls /data/destinationfolder
# 
```

---
**NOTE**

The example above assumes you have exec'ed into the worker container and you are manipulating the `/data` folder mount directly. If you are mounting the EFS files system from another environment your paths may be different.  

---

If you navigate in your SQS console and open the `main-queue` we have created, you can click on `Send and receive messages` and from there you can send 5 separate and distinct messages with a body that matches the 5 file names above. This is my first message: 

![sqs-message](../images/sqs-message.png) 


Once you have completed sending the 4 messages, after a few seconds, if you look at the folders above, you should see that the files have been processed (renamed and moved): 
```
# ls /data/sourcefolder
# ls /data/destinationfolder
001_has_been_processed	002_has_been_processed	003_has_been_processed	004_has_been_processed	005_has_been_processed
# 
```
If you look at the logs in the CloudWatch log group you will see all the details. The first part of the log is the application initializing while the remaining log entries are the files being processed. You can check the CW log group the application is using in the worker container in the ECS task. 

![cloudwatch-regional](../images/cloudwatch-regional.png) 

Congratulations! You have a working deployment in the AWS region of your choice!

### [Home](../README.md) <<< >>> [Step 2 - ECS Anywhere Deployment](./Step%202:%20ECS%20Anywhere%20Deployment/README.md)