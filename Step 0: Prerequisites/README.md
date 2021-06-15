
### Step 0: Prerequisites  

To get started with the deployment you need an [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) and the [latest CLI installed](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) and [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html). Make sure that the region is configured properly.

You would also need `jq` and `vagrant` (optional) installed in your command path.

Also be sure that your shells throughout the tutorial have the following variables available. We will use them extensively. 
```
export ACCOUNTNUMBER=<your account>
export MYREGION=<your region>
```

Before you start, clone this repository locally and move into it:
```
git clone https://github.com/aws-containers/ecs-anywhere-tutorial
cd ecs-anywhere-tutorial
``` 

Depending on the path you will take in this tutorial, there may be additional pre-requisites that will be called out. 



### [Home](../README.md) <<<  >>> [Step 1 - Regional Deployment](./Step%201:%20Regional%20Deployment/README.md)