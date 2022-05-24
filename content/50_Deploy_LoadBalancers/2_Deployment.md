---
title: "Load Balancer - Deployment"
weight: 2
---

### STEPS
Aim is to deploy the Load balancer on AWS through terraform. 

Navigate to the Cloud9 terminal and copy **loadbalancers.tf** file from the Deployment_resources folder to the Deployment folder.

```console
  cp ./Deployment_resources/loadbalancers.tf ./Deployment/loadbalancers.tf
  cd ./Deployment
``` 

And then run the following set of commands.

1. **<ins>terraform init</ins>**

```conslole
terraform init
``` 
Running this will download & install all the necessary modules. 

![init_fw](/static/images/deploy_loadbalancers/LOADBALANCER_INIT.png)

2. **<ins>terraform validate**</ins>

```console
terraform validate
``` 
Run this to check for any syntax error in the code.

![validate_lb](/static/images/deploy_loadbalancers/LOADBALANCER_VALIDATE.png)

3. **<ins>terraform plan**</ins>

--To understand what the code will reflect and do on your AWS account run 
```console
terraform plan --out awslab
```
The resources shown with the '+' symbol are set to be created. It will show the number of additional resources to be added.

![plan_lb](/static/images/deploy_loadbalancers/PLAN_LB.png)

4. **<ins>terrafrom apply**</ins>

--If you are satisfied with the plan of the configuration, run the following to apply it.

```console
terraform apply awslab
```

![apply_lb](/static/images/deploy_loadbalancers/APPLY_LB.png)

Open your AWS Management Console to see if all the resources are correctly deployed. 

**Load Balancer:** 
![lb](/static/images/deploy_loadbalancers/lb.jpeg)

You can click on any one to see detailed info like this:
![ext_lb](/static/images/deploy_loadbalancers/ext_lb.png)  
<br>    

![int_lb](/static/images/deploy_loadbalancers/app_lb.png)  

**Target Groups:**  

![target_group](/static/images/deploy_loadbalancers/target_groups.jpeg)

