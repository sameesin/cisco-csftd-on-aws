+++
title = "FTD and FMC - Deployment"
chapter = false
weight = 2
+++

# STEPS
Aim is to deploy Cisco FTDv and FMCv instances on AWS through terraform. 

Navigate to the Cloud9 terminal and copy **instances.tf** file from the Deployment_resources folder to the Deployment folder.

```
  cp ./Deployment_resources/instances.tf ./Deployment/instances.tf
  cd ./Deployment
``` 

>Note: We will deploy ```2``` FTD and ```1``` FMC instances

And then run the following set of commands.

1. **<ins>terraform validate**</ins>

    --Run *terraform validate* to check for any syntax error in the code.

    ![validate_fw](../IMAGES/VALIDATE_FW.png)

2. **<ins>terraform plan**</ins>

    --To understand what the code will reflect and do on your AWS account run *terraform plan --out awslab*, The resources shown with the '+' symbol are set to be created. It will show the additional number of resources to be added by Terraform.

    ![plan_fw](../IMAGES/PLAN_FW.png)

3. **<ins>terrafrom apply**</ins>

    --If you are satisfied with the plan of the configuration, run *terraform apply* to apply it.

Open your AWS Management Console to see if all the resources are correctly deployed. 

**EC2:**

--FMC and both the FTD are highlighted. 
![instances](../IMAGES/INSTANCE_FTD_FMC.png)

**FMC:** 

--Open the FMC instance to see the public IP and other details of FMC.
![fmc](../IMAGES/FMC_INS.png)

**FTD:**

-- FTD can also be expanded in the similar fashion
![ftd](../IMAGES/ftd0_exp.jpeg)



