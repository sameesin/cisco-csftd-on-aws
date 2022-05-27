---
title: "Test Machines Setup - Deployment"
weight: 2
---

### STEPS
Lets proceed with the deployment. The aim here is to deploy the Test Machines on AWS through terraform. 


>**IMPORTANT**: We will first run our Bastion terraform plan and then run terraform plan for application server.  



## <ins>**Bastion Server**:</ins>
Navigate to the Cloud9 terminal and copy **bastion.tf**, **bastion_install.tpl** file from the Resources folder to the Development folder.

```console
  cp ./Resources/bastion.tf ./Development/bastion.tf
  cp ./Resources/bastion_install.tpl ./Development/bastion_install.tpl
  cd ./Development
``` 

Run the following set of commands.

1. **<ins>terraform init</ins>**

```console
terraform init
``` 
This will download & install all the necessary modules. 

![init_fw](/static/images/deploy_test_machines/INIT_BASTION.png)

1. **<ins>terraform validate**</ins>

```console
terraform validate
```
This will check for any syntax error in the code.

![validate_lb](/static/images/deploy_test_machines/VALIDATE_BASTION.png)

3. **<ins>terraform plan**</ins>

To understand what the code will reflect and do on your AWS account run 

```console
terraform plan --out awslab
```
The resources shown with the '+' symbol are set to be created. It will show the number of additional resources to be added.

![plan_lb](/static/images/deploy_test_machines/PLAN_BASTION.png)

4. **<ins>terrafrom apply**</ins>

If you are satisfied with the plan of the configuration apply it. 

```console
terraform apply awslab
```

Open the AWS console to see bastion server in the EC2 Instance.

Bastion:
![bastion](/static//images/deploy_test_machines/bastion_instance.jpeg)  

## <ins>**Application Server**:</ins>
Navigate to the Cloud9 terminal and copy **apps.tf**, **apache_install.tpl** files from the Resources folder to the Development folder.

```console
  cp ./Resources/apps.tf ./Development/apps.tf
  cp ./Resources/apache_install.tpl ./Development/apache_install.tpl
  cd ./Development
``` 

Run the following set of commands.

1. **<ins>terraform init</ins>**

```console
terraform init
``` 
This will download & install all the necessary modules.

![init_fw](/static/images/deploy_test_machines/INIT_BASTION.png)

1. **<ins>terraform validate**</ins>

```console
terraform validate
```
This will check for any syntax error in the code.

![validate_lb](/static/images/deploy_test_machines/VALIDATE_BASTION.png)

3. **<ins>terraform plan**</ins>

To understand what the code will reflect and do on your AWS account run 

```console
terraform plan --out awslab
```
The resources shown with the '+' symbol are set to be created. It will show the number of additional resources to be added.

![plan_lb](/static/images/deploy_test_machines/PLAN_APPS.png)

4. **<ins>terrafrom apply**</ins>

If you are satisfied with the plan of the configuration apply it. 

```console
terraform apply awslab
```

![apply_lb](/static/images/deploy_test_machines/APPLY_APPS.png)


After the deployment go to the AWS Console and check the instance is AWS Console for your application machine(EC2-ubuntu here) and bastion server.

![application_server](/static//images/deploy_test_machines/instances.jpeg)

You can further expand the machine to see details specific to it.

Application server:
![web](/static//images/deploy_test_machines/ec2_detail.jpeg)   
  

