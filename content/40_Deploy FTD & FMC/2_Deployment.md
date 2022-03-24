+++
title = "FTD and FMC - Deployment"
chapter = false
weight = 2
+++

# STEPS
Aim is to deploy all the instances on AWS through terraform. 

Open a *terminal* in folder which contains the .tf file of the firewall configuration.

>Note: We have ```2``` FTD, ```1``` FMC & ```2``` Application server deployed here

And then run the following sets of command.
1. **<ins>terraform init**</ins>
   
   -- cd into the folder 
  
   --Run *terraform init*. This will download & install all the necessary packages needed, like the aws package. 

   ![init_fw](../IMAGES/INIT_FW.png)
2. **<ins>terraform validate**</ins>

    --Run *terraform validate* to check for any syntax error in the code.

    ![validate_fw](../IMAGES/VALIDATE_FW.png)
3. **<ins>terraform plan**</ins>

    --To understand what the code will reflect and do on your AWS account run *terraform plan*, The resources shown with the '+' symbol are set to be created. It will show the number of resources to be added.

    ![plan_fw](../IMAGES/PLAN_FW.png)

4. **<ins>terrafrom apply**</ins>

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

5. **<ins>terrafrom destroy</ins>**

     --Run *terraform destroy* to terminate and delete all the components created.



