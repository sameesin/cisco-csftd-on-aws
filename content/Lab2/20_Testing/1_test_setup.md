---
title: "Test Setup"
weight: 1
---

## Test Setup

To test this setup we will deploy 2 linux machines. One will act as a bastion server to which the user will SSH to. The other machine will be the test machine (without direct internet connection) from which the test traffic to the internet will be generated.

### **1. Deploy the Bastion** 

Terraform will be used to deploy the bastion server and attach an Elastic IP to it for SSH access

```terraform
resource "aws_instance" "bastion_machine" {
  ami           = "<AMI ID"
  instance_type = "t3.micro"
  key_name      = var.keyname
  network_interface {
    network_interface_id = aws_network_interface.bastion_interface.id
    device_index         = 0
  }
  tags = {
    Name = "bastion"
  }
}
```

![bastion](/static/images/deploy_test_machines/bastion_instance.jpeg)

### **2. Deploy the Test machine** 

Terraform will be used to deploy the test machine from which internet access will be tested. User will SSH into the test machine from the bastion server.

```terraform
resource "aws_instance" "app_machine" {
  ami           = "<AMI ID>"
  instance_type = "t3.micro"
  key_name      = var.keyname
  network_interface {
    network_interface_id = aws_network_interface.app_interface.id
    device_index         = 0
  }
  tags = {
    Name = "application"
  }
}
```

### Deployment

Run the following set of commands to deploy the 2 instances in AWS

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
