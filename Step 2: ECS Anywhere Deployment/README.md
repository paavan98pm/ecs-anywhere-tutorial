### Selecting the infrastructure for the stretched deployment 

Now that we have a working regional deployment we will work to separate what will remain in the region (the SQS queue, the ECR repository, the CloudWatch log group) from what will be re-deployed outside of the region. The first thing we need to do is to identify where we want to run our tasks outside of the region. For the [demo at re:Invent](https://aws.amazon.com/blogs/containers/introducing-amazon-ecs-anywhere/) we have used two Raspberry Pi's. This could absolutely be done again (with some additional considerations due to different CPU architectures). It is possible (and likely) that you may want to use some bare metal or virtual machines in your data center. That is also fine (and expected). 

In reality you can use any infrastructure provided it meets these characteristics: 
- the operating systems have outbound Internet connectivity (they need to reach AWS cloud services endpoints)
- the operating systems are included in the list of [operating systems supported by ECS Anywhere](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-anywhere.html)
- the systems have at least 1 vCPU and 512MB of memory to run the SSM agent, the ECS agent and the Python application in this tutorial  
- for this tutorial specifically, all instances need to map a shared folder under the `/data` mountpoint to mimic the in-region mount of the EFS file system  

In this tutorial we have included the following alternative(s): 
- [Vagrant with Virtualbox](./instances-preparation/vagrant/README.md)

Again, this was just a convenient way for this demo to provision non EC2 capacity. Feel free to use any other type of infrastructure (e.g. VMware vSphere) that can run any of the [operating systems supported by ECS Anywhere](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-anywhere.html).

### Deploying ECS Anywhere on M1 Mac Mini (or other ARM architectures like Graviton or Raspberry-Pi)

Since the Dockerfile provided in the application pulls the x86 base image, the stretched deployment works on stretched deployment using the x86 architecture.

```Dockerfile
FROM public.ecr.aws/bitnami/python:3.8.8-prod
```

For deployment on M1 Macs and ARM architectures, you will need to replace the above with:

```Dockerfile
FROM public.ecr.aws/fincompare/python:3-slim
```

Since Virtualbox is not supported in M1 Macs, you may use the [UTM](https://mac.getutm.app/gallery/) utility based on QEMU emulation software and select the Debian 10.4 Minimal image from the UTM gallery.

### Preparing for the stretched deployment 

Before we can start to deploy the ECS task on the infrastructure you have created outside of AWS, we need to prepare that infrastructure. This consists of two things: 

- preparing the AWS resources 
- preparing the infrastructure nodes   

#### Preparing AWS resources  

In this step we will create a separate ECS cluster that will host the remote nodes, we will create the `instance role` that we will use to register the SSM agent and we will generate the SSM keys to register these nodes in AWS Systems Manager.

Move into the `/anywhere-preparation` folder in the repo. Then run the `pre-instance-preparation-setup-external.sh` script and follow the instructions. Feel free to inspect the script to investigate what it does. Note that the script informs you about the names it is using by default to create the resources above. You should accept them unless you have a burning need/desire to change them in the script (note if you do so you will later need to change the other scripts). 

#### Preparing the infrastructure nodes 

The script above, at the end of the execution, will suggest a number of exports (the SSM Keys, the ECS cluster to connect to and the region you set with the `MYREGION` variable). Please gain a shell in each of the nodes in the infrastructure you provisioned (through Vagrant or anything you opted to use) and execute those exports. Once you have executed them, run these commands to download and execute the ECS Anywhere install script on each of the nodes:    

```
curl -o "ecs-anywhere-install.sh" "https://amazon-ecs-agent.s3.amazonaws.com/ecs-anywhere-install-latest.sh"
chmod +x ecs-anywhere-install.sh 
sudo ./ecs-anywhere-install.sh --cluster $ECS_ANYWHERE_CLUSTER_NAME --activation-id $ACTIVATION_ID --activation-code $ACTIVATION_CODE --region $MYREGION
```

#### Confirming external infrastructure is operational  

The instances have autoregistered into both SSM and ECS.

This is how they show up in SSM (Fleet Manager):

![ssm-registered-instances](../images/ssm-registered-instances.png)

This is how they show up in the ECS cluster:

![ecs-cluster-registered-instances](../images/ecs-cluster-registered-instances.png)

Alternatively, in the `anywhere-preparation` directory, there is an additional (idempotent) script called `post-instance-preparation-setup-external.sh`. After you registered all the nodes run this script to verify that all instances have been registered correctly into both SSM and ECS. The information in the output of this script will match what you see in the ECS and SSM console.

You can now move to the next step and deploy the `worker` task on this external infrastructure. 


### The external deployment  

We are now ready to deploy our worker application on the stretched infrastructure. In the fullness of time we envision this process will use the same tooling we use today to deploy in the region but, for now, we are going to manually craft a task definition and deploy the workload manually. 

Before we can do so you need to explore the in-region deployment you have done at the beginning of the tutorial and extract two things: the ECS cluster name and the ID of the ECS Task that contains the worker application. For example, if you used Copilot to do the in-region deployment, you would see something like this: 

![in-region-cluster-name](../images/in-region-cluster-name.png) 

![in-region-task-id](../images/in-region-task-id.png) 

First you will need to export these two variables and set them with the values you have obtained above:
```
export IN_REGION_CLUSTER_NAME=<cluster_name>  
export IN_REGION_TASK_ID=<task_id>
```

Next, move into the `anywhere-deployment` directory and locate the script `external-deployment.sh`. You can explore this script but this is what it will do for us: 
- validate that the cluster and the task ID have been entered correctly by checking their existence 
- extract all relevant setup information from the in-region task (e.g. ECR image being used, IAM roles associated with the task, CW log group, SQS queue endpoint)
- customize the task definition template json with these parameters 
- register the resulting customized json into a task definition
- create a service that launches this task definition 

At this point you should have a new task running on your external infrastructure as part of the service that the `external-deployment.sh` script has created. The way you'd test the application is functioning properly is similar to how you tested the in-region deployment with the only difference that now the worker task and the files are running and hosted on the external infrastructure. Everything else, the container image in ECR, the SQS queue, the IAM roles and the CW loggroup are still hosted in the region. 

### [Home](../README.md) <<<  >>> [Step 3 - Cleanup](../Step%203:%20Cleanup/README.md)