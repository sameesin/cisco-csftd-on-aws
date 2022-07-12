---
title: "FTD and FMC - Deployment"
weight: 3
---

### STEPS
Lets proceed with the deployment. The aim here is to deploy Cisco Secure FTDv and FMCv instances on AWS through Terraform. 

Navigate to the Cloud9 terminal and copy **instances.tf** file from the Resources folder to the Development folder.

```console
  cp ./Resources/instances.tf ./Development/instances.tf
  cd ./Development
``` 

>Note: We will deploy ```2``` FTDs and ```1``` FMC instances


## When using Event Engine for this lab follow the below steps

- Navigate to the modules/FirewallServer/data.tf file inside AWS_Workshop_Code_event_engine folder.

- replace the < owner account id > with the AMI owner account ID. To ge this ID on the AWS console, select EC2 Service, under image section click on AMIs.
Select private images. Select one of the image, copy the Owner account ID under details 

And then run the following set of commands.

1. **<ins>terraform init</ins>**

   ```console 
   terraform init
   ``` 
   This will download & install all the necessary modules. 

![init_fw](/static/images/deploy_ftd_fmc/INIT_FW.png)

2. **<ins>terraform validate**</ins>

    ```console
    terraform validate
    ``` 
    to check for any syntax error in the code.

![validate_fw](/static/images/deploy_ftd_fmc/VALIDATE_FW.png)

3. **<ins>terraform plan**</ins>

    To understand what the code will reflect and do on your AWS account run 
    ```console
    terraform plan --out awslab
    ```
    The resources shown with the '+' symbol are set to be created. It will show the additional number of resources to be added by Terraform.

![plan_fw](/static/images/deploy_ftd_fmc/PLAN_FW.png)

4. **<ins>terraform apply**</ins>

    If you are satisfied with the plan of the configuration, run 
    ```console
    terraform apply awslab
    ```
    
![apply_fw](/static/images/deploy_ftd_fmc/APPLY_FW.png)
Open your AWS Management Console to see if all the resources are correctly deployed. 

**EC2:**

- FMC and both the FTD are highlighted. 
![instances](/static/images/deploy_ftd_fmc/INSTANCE_FTD_FMC.png)

**FMC:** 

- Open the FMC instance to see the public IP and other details of FMC.
![fmc](/static/images/deploy_ftd_fmc/fmc_detail.jpeg)

**FTD:**

- FTD can also be expanded in the similar fashion
![ftd](/static/images/deploy_ftd_fmc/ftd_detail.jpeg)



