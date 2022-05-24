---
title: "Test Machines - Deployment"
chapter: false
weight: 2
---

### STEPS
Aim is to deploy the Test Machines on AWS through terraform. 

Navigate to the Cloud9 terminal and copy **application.tf** file from the Deployment_resources folder to the Deployment folder.

```
  cp ./Deployment_resources/application.tf ./Deployment/application.tf
  cd ./Deployment
``` 

And then run the following set of commands.

1. **<ins>terraform init</ins>**

--Run ```terraform init``` This will download & install all the necessary modules. 

![init_fw](//static/images/deploy_loadbalancers/LOADBALANCER_INIT.png)

2. **<ins>terraform validate**</ins>

--Run ```terraform validate``` to check for any syntax error in the code.

![validate_lb](//static/images/deploy_loadbalancers/LOADBALANCER_VALIDATE.png)

3. **<ins>terraform plan**</ins>

--To understand what the code will reflect and do on your AWS account run ```terraform plan --out awslab```, The resources shown with the '+' symbol are set to be created. It will show the number of additional resources to be added.

![plan_lb](//static/images/deploy_loadbalancers/PLAN_LB.png)

4. **<ins>terrafrom apply**</ins>

--If you are satisfied with the plan of the configuration, run ```terraform apply``` to apply it.

After the deployment go to the AWS Console and check the instance is AWS Console for your application machine(EC2-ubuntu here) and bastion server.

![application_server](//static/images/deploy_test_machines/instances.jpeg)

You can further expand the machine to see details specefic to it.

Application server:
![web](//static/images/deploy_test_machines/ec2_detail.jpeg)   
  
Bastion
![bastion](//static/images/deploy_test_machines/bastion_instance.jpeg)  
