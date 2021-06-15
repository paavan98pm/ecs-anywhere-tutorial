### Clean up 

- Delete the `ecsAnywhereCluster` (you need to stop existing services and deregister the external instances)
- Deregister the managed instances in SSM (Fleet Manager) and optionally delete the activation keys used for this tutorial (Hybrid Activations). If you already use SSM for other purposes make sure you deregister and delete the right objects
- Delete the `ecsAnywhereInstanceRole` IAM role 
- Delete any external infrastructure you may have deployed as part of this tutorial
- Delete the in-region deployment using the same tool you have used to deploy the stack
    - if you have used Copilot, you can run `copilot app delete` in the `/app/copilot` folder
    - if you have used Docker Compose, you can run `docker compose down` in the `/app/docker-compose` folder. In addition, you should delete the ECR repository and the EFS volume manually