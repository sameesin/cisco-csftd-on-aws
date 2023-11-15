---
title: "Solution Deployment"
weight: 2
---

## Steps to Execute

Open the Cloud9 terminal.

If you are using AWS Event Engine navigate to the folder named **secure-firewall/AWS_Workshop/AWS_Workshop_Code_Event_Engine/Working_Directory2/Resources**

If you are using personal AWS Account, navigate to the folder named **secure-firewall/AWS_Workshop/AWS_Workshop_Code/Working_Directory2/Resources**

- Update the terraform.tfvars file with the required input for the variables and rename the file as terraform.tfvars

- run ``terraform init`` command
- run ``terraform plan --out t`` command
- run ``terraform apply t`` command