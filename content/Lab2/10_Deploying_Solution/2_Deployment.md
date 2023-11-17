---
title: "Solution Deployment"
weight: 2
---

## Steps to Execute

Open the Cloud9 terminal.

If you are using AWS Event Engine navigate to the folder named **secure-firewall/AWS_Workshop/AWS_Workshop_Code_Event_Engine/Working_Directory2/Resources**

If you are using personal AWS Account, navigate to the folder named **secure-firewall/AWS_Workshop/AWS_Workshop_Code/Working_Directory2/Resources**

- Update the terraform.tfvars file with the required input for the variables and run the following commands

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