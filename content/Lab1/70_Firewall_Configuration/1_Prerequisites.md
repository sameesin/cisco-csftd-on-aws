---
title: "Prerequisites"
weight: 1
---

#### Following prerequisites need to be performed before configuring the Cisco FTD using Python

> Note: You might have to wait upto 30 min, for FMC to be up and running.
> Note: If there is an existing FMC up and running user can use that. Provide the public IP address of the FMC in the terraform.tfvars file in the AWS_Workshop folder. 

## **Enable Evaluation License**

1. Open a browser
2. Login to the Cisco FMC UI using the FMC Public IP Address and the admin credentials provided as values to the variables **fmc_username** and **fmc_password**

![fmc_login](/static/images/testing-traffic/FMC_LOGIN.png)

3. Click on Enable Evaluation License, make sure to do it when this pop-up shows. 

![smart_lic](/static/images/testing-traffic/SMART_LIC.png)
