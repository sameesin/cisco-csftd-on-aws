+++
title = "Load Balancer - Deployment"
chapter = false
weight = 2
+++

# Deployment of the Load Balancer.
Aim is to deploy the Load balancer on AWS through terraform. 

Navigate to the Cloud9 terminal and copy **loadbalancers.tf** file from the Deployment_resources folder to the Deployment folder.

```
  cp ./Deployment_resources/loadbalancers.tf ./Deployment/loadbalancers.tf
  cd ./Deployment
``` 

And then run the following set of commands.

1. **<ins>terraform validate**</ins>

    --Run *terraform validate* to check for any syntax error in the code.

    ![validate_lb](../IMAGES/LOADBALANCER_VALIDATE.png)

2. **<ins>terraform plan**</ins>

    --To understand what the code will reflect and do on your AWS account run *terraform plan --out awslab*, The resources shown with the '+' symbol are set to be created. It will show the number of additional resources to be added.

    ![plan_lb](../IMAGES/PLAN_LB.png)

3. **<ins>terrafrom apply**</ins>

    --If you are satisfied with the plan of the configuration, run *terraform apply* to apply it.

Open your AWS Management Console to see if all the resources are correctly deployed. 

**Load Balancer:** 
![lb](../IMAGES/load_balancers.jpeg)

You can click on any one to see detailed info like this:
![ext_lb](../IMAGES/ext_lb_exp.jpeg)
**Target Groups:**
![target_group](../IMAGES/target_groups.jpeg)

