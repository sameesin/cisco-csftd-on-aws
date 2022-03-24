+++
title = "Network resources - Deployment"
chapter = false
weight = 2
+++

# **STEPS**
Aim is to deploy all the resources created on AWS through terraform. 

Open a *terminal* in folder which contains the .tf file of the network configuration.

And then run the following sets of command.
1. **<ins>terraform init</ins>**
   
   -- cd into the folder 

   --Run ***terraform init***. This will download & install all the necessary packages needed, like the aws package. 

   ![init_nw](../IMAGES/INIT_NW.png)
2. **<ins>terraform validate</ins>**

    --Run *terraform validate* to check for any syntax error in the code.

    ![validate_nw](../IMAGES/VALIDATE_NW.png)
3. **<ins>terraform plan</ins>**

    --To understand what the code will reflect and do on your AWS account run *terraform plan*, The resources shown with the '+' symbol are set to be created. It will show the number(may be different for your topology) of resources to be added.

    ![plan_nw](../IMAGES/PLAN_NW.png)

4. **<ins>terrafrom apply</ins>**

    --If you are satisfied with the plan of the configuration, run *terraform apply* to apply it.

    ![apply_nw](../IMAGES/APPLY_NW.png)

Open your AWS Management Console to see if all the resources are correctly deployed. 

**VPC:** 
![vpc](../IMAGES/vpc.jpeg)
**Subnets:** 
Search for your VPC in subnets and see all of them.
![subnets](../IMAGES/subnets.jpeg)
**Interface:** 
![interfaces](../IMAGES/network_interfaces.jpeg)
**Route Tables:** 
![routes_tables](../IMAGES/route_tables.jpeg)

