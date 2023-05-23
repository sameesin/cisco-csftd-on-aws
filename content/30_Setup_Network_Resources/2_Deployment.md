---
title: "Network resources - Deployment"
weight: 2
---

### <ins>**STEPS**</ins>
Aim is to deploy all the resources created on AWS through terraform. 

Open the Cloud9 terminal.
If you are using AWS Event Engine navigate to the folder named **AWS_Workshop_Code_event_engine/Working_Directory/Resources**

If you are using personal AWS Account, navigate to the folder named **AWS_Workshop_Code_aws_account/Working_Directory/Resources**

Copy **providers.tf**, **networks.tf**, **variables.tf**, **terraform.tfvars** files from the Resources folder to the Development folder.

1. **<ins>Enter values for Terraform variables</ins>**
   - provide the 'aws_access_key', 'aws_secret_key', 'session_token' of your user.
   - provide the name of the key created in previous section
   - provide the FMC IP address
   - Rest of the variables have been provided with a value already, however if you wish you can modify those values

And then run the following set of commands.

2. **<ins>terraform init</ins>**

```console
terraform init
```

This will download & install all the necessary packages needed, like the aws package. 
<br>  
<br>
   ![init_nw](/static/Images/setup_network_resources/INIT_NW.png)
<br> 

3. **<ins>terraform validate</ins>**

```console
terraform validate
``` 

Run this to check for any syntax error in the code.

![validate_nw](/static/Images/setup_network_resources/VALIDATE_NW.png)  
<br>  

4. **<ins>terraform plan</ins>**

To understand what the code will reflect and do on your AWS account run 
```console
terraform plan --out awslab
``` 
The resources shown with the '+' symbol are set to be created. It will show the number(may be different for your topology) of resources to be added.

![plan_nw](/static/Images/setup_network_resources/PLAN_NW.png)

5. **<ins>terrafrom apply</ins>**

If you are satisfied with the plan of the configuration apply it.

```console
terraform apply awslab
```

![apply_nw](/static/Images/setup_network_resources/APPLY_NW.png)

Open your AWS Management Console to see if all the resources are correctly deployed. 

**Subnets:** 
Search for your VPC in subnets and see all of them.
<br>
 

![subnets](/static/Images/setup_network_resources/subnets.jpeg)

**Interface:** 
![interfaces](/static/Images/setup_network_resources/network_interfaces.jpeg)

**Internet_Gateway:**
 ![internet_gateway](/static/Images/setup_network_resources/igw.jpeg)
 
**Security Group:**
![SG](/static/Images/setup_network_resources/security_groups.jpeg)

**Route Tables:** 
![routes_tables](/static/Images/setup_network_resources/routetables.jpeg)

