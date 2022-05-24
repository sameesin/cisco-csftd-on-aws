---
title: "Network resources - Deployment"
<<<<<<< HEAD:content/30_Setup_Network_Resources/2_Deployment.md
=======
chapter: false
>>>>>>> f75898c75d08fd68a38aaa8372166c9192ec4371:content/30_Setup Network Resources/2_Deployment.md
weight: 2
---

### <ins>**STEPS**</ins>
Aim is to deploy all the resources created on AWS through terraform. 

Open the Cloud9 terminal and navigate to the folder named **Deployment**
Ensure the following files are present in the folder.

- networks.tf
- providers.tf
- variables.tf
- terraform.tfvars

Ensure the following folder is present in the folder
- modules
- deployment folder

Navigate to the Cloud9 terminal and copy **network.tf** file from the Deployment_resources folder to the Deployment folder.

1. **<ins>Enter values for Terraform variables</ins>**
   - provide the 'aws_access_key' and 'aws_secret_key' of your user.
   - provide the name of the key created in previous section
   - Rest of the variables have been provided with a value already, however if you wish you can modify those values

And then run the following sets of command.

2. **<ins>terraform init</ins>**

   --Run ```terraform init```. 
   This will download & install all the necessary packages needed, like the aws package. 
<br>  
<br>
<<<<<<< HEAD:content/30_Setup_Network_Resources/2_Deployment.md
   ![init_nw](/static/images/setup_network_resources/INIT_NW.png)
=======
   ![init_nw](//static/images/setup_network_resources/terraform_init.jpeg)
>>>>>>> f75898c75d08fd68a38aaa8372166c9192ec4371:content/30_Setup Network Resources/2_Deployment.md
<br> 

3. **<ins>terraform validate</ins>**

    --Run ```terraform validate``` to check for any syntax error in the code.

<<<<<<< HEAD:content/30_Setup_Network_Resources/2_Deployment.md
    ![validate_nw](/static/images/setup_network_resources/VALIDATE_NW.png)  
=======
    ![validate_nw](//static/images/setup_network_resources/terraform_validate.jpeg)  
>>>>>>> f75898c75d08fd68a38aaa8372166c9192ec4371:content/30_Setup Network Resources/2_Deployment.md
  <br>  

4. **<ins>terraform plan</ins>**

    --To understand what the code will reflect and do on your AWS account run ```terraform plan --out awslab```, The resources shown with the '+' symbol are set to be created. It will show the number(may be different for your topology) of resources to be added.

<<<<<<< HEAD:content/30_Setup_Network_Resources/2_Deployment.md
    ![plan_nw](/static/images/setup_network_resources/PLAN_NW.png)
=======
    ![plan_nw](//static/images/setup_network_resources/plan_complete.jpeg)
>>>>>>> f75898c75d08fd68a38aaa8372166c9192ec4371:content/30_Setup Network Resources/2_Deployment.md

5. **<ins>terrafrom apply</ins>**

    --If you are satisfied with the plan of the configuration, run ```terraform apply awslab``` to apply it.

<<<<<<< HEAD:content/30_Setup_Network_Resources/2_Deployment.md
    ![apply_nw](/static/images/setup_network_resources/APPLY_NW.png)

Open your AWS Management Console to see if all the resources are correctly deployed. 

=======
    ![apply_nw](//static/images/setup_network_resources/apply_complete.jpeg)

Open your AWS Management Console to see if all the resources are correctly deployed. 

**VPC:** 
![vpc](//static/images/setup_network_resources/vpc.jpeg)
<br> 

>>>>>>> f75898c75d08fd68a38aaa8372166c9192ec4371:content/30_Setup Network Resources/2_Deployment.md
**Subnets:** 
Search for your VPC in subnets and see all of them.
<br>
 

<<<<<<< HEAD:content/30_Setup_Network_Resources/2_Deployment.md
![subnets](/static/images/setup_network_resources/subnets.jpeg)

**Interface:** 
![interfaces](/static/images/setup_network_resources/network_interfaces.jpeg)

**Internet_Gateway:**
 ![internet_gateway](/static/images/setup_network_resources/igw.jpeg)
 
**Security Group:**
![SG](/static/Images/setup_network_resources/security_groups.jpeg)

**Route Tables:** 
![routes_tables](/static/images/setup_network_resources/routetables.jpeg)
=======
![subnets](//static/images/setup_network_resources/subnets.jpeg)

**Interface:** 
![interfaces](//static/images/setup_network_resources/network_interfaces.jpeg)

**Internet_Gateway:**
 ![internet_gateway](//static/images/setup_network_resources/igw.jpeg)
 
**Security Group:**
![SG](//static/images/setup_network_resources/security_groups.jpeg)

**Route Tables:** 
![routes_tables](//static/images/setup_network_resources/routetables.jpeg)
>>>>>>> f75898c75d08fd68a38aaa8372166c9192ec4371:content/30_Setup Network Resources/2_Deployment.md

