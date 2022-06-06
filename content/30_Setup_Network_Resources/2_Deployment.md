---
title: "Network resources - Deployment"
weight: 2
---

### <ins>**STEPS**</ins>
Aim is to deploy all the resources created on AWS through terraform. 

Open the Cloud9 terminal.
If you are using AWS Event Engine navigate to the folder named **secure-firewall/AWS_Workshop/AWS_Workshop_Code_EventEngine/Working_Directory**

If you are using AWS Account navigate to the folder named 
**secure-firewall/AWS_Workshop/AWS_Workshop_Code/Working_Directory**

Create a new folder inside Working_Directory named **Development**

```console
cd ./secure-firewall/AWS_Workshop/AWS_Workshop_Code/Working_Directory
mkdir Development
```

Copy **providers.tf**, **networks.tf**, **variables.tf**, **terraform.tfvars** files from the Resources folder to the Development folder.

1. Overview of thge **terraform.tfvars** file:
   - provide 'aws_access_key' of your user.
   - provide the 'aws_secret_key' of your user.
   - Change the region as per the region where you plan to deploy the resources.

![provider](/static/images/setup_network_resources/provider_var.png)

   - This section will use the same VPC that was created in the getting started section so **vpc_name** variable has the value **IAC-VPC** and variable **vpc_cidr** is empty as its not required.
   - provide the name of EC2 keypair that you created in the getting started section to the variable **keyname** 
   - Rest of the variables have been provided with a value already and require no change for this lab, however if you wish you can modify those values.

![keyname](/static/images/setup_network_resources/keyname.png)

**Note:** variable **create_igw** is set to false since the VPC created in the getting started section for deploying cloud9 will already have an Internet Gateway attached to it. If you plan to use a different VPC which does not already have an Internet Gateway attached, change the value of **create_igw** to true.


If you are not inside the Development folder, navigate to the **Development folder** And then run the following set of commands inside folder.

2. **<ins>terraform init</ins>**

```console
terraform init
```

This will download & install all the necessary packages needed, like the aws package. 
<br>  
<br>
   ![init_nw](/static/images/setup_network_resources/INIT_NW.png)
<br> 

3. **<ins>terraform validate</ins>**

```console
terraform validate
``` 

Run this to check for any syntax error in the code.

![validate_nw](/static/images/setup_network_resources/VALIDATE_NW.png)  
<br>  

4. **<ins>terraform plan</ins>**

To understand what the code will reflect and do on your AWS account run 
```console
terraform plan --out awslab
``` 
The resources shown with the '+' symbol are set to be created. It will show the number(may be different for your topology) of resources to be added.

![plan_nw](/static/images/setup_network_resources/PLAN_NW.png)

5. **<ins>terrafrom apply</ins>**

If you are satisfied with the plan of the configuration apply it.

```console
terraform apply awslab
```

![apply_nw](/static/images/setup_network_resources/APPLY_NW.png)

Open your AWS Management Console to see if all the resources are correctly deployed. 

**Subnets:** 
<br>
Search for your VPC in subnets and see all of them.
<br>

![subnets](/static/images/setup_network_resources/Subnets.png)

**Security Groups**
![Security Groups](/static/images/setup_network_resources/SecurityGroups.png)

**Interfaces:** 
![interfaces](/static/images/setup_network_resources/Network_interfaces.png)

**Route Tables:** 
![routes_tables](/static/images/setup_network_resources/routetables.png)

