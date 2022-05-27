---
title: "Event Engine: FTDv and FMCv - Description"
weight: 1
---

## **Introduction**

 When using the AWS Events Engine, the AMIs used to deploy Cisco secure FTDv and FMCv should have been deployed to your event engine platform prior to the event and are available in your AMI Owned Images. If you dont see the AMIs, please contact the event administrator.

### <ins>**FMC**</ins>
The code below is for creating one FMC in any one of the AZ which will host the 2 FTDv instances. 

Here is the data source to fetch private ami id of fmc:  

```
data "aws_ami" "fmcv" {

  owners      = ["self"]
  filter {
    name   = "name"
    values = ["${var.FMC_version}*"]
  }
  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}
```
The data source to render startup_file is: 

```
data "template_file" "fmc_startup_file" {
  template = file("${path.module}/fmc_startup_file.txt")
}
```

The AMI that we are using here is a private AMI.

<ins>**Creation of FMC**</ins>
```
resource "aws_instance" "fmcv" {
  ami                 = data.aws_ami.fmcv.id
  instance_type       = c5.4xlarge
  key_name            = var.keyname
    
  network_interface {
    network_interface_id = var.fmcmgmt_interface
    device_index         = 0
  }
  user_data = data.template_file.fmc_startup_file.rendered
  tags = {
    Name = "Cisco FMCv"
  }
}
```
We pass the user data and the network interface specific to FMC. The fmc_startup_file looks like this:
```
#FMC
{
"AdminPassword": "Password@123!",
"Hostname":      "FMC-01",
}
``` 
 
## <ins>**FTD</ins>**

The code below is deploying two FTD instances, each in a different availability zone with different network interfaces like *outside*, *inside*, *diagnostic* and *management* attached to it.

Here is the code snippet that fetches the private ami to create ftd:  

```
data "aws_ami" "ftdv" {

  owners      = ["self"]

 filter {
    name   = "name"
    values = ["${var.FTD_version}*"]
  }

 filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}
```

The data source to render startup_file is:   
```
data "template_file" "ftd_startup_file" {
    count     = 2
    template  = file("${path.module}/ftd_startup_file.txt")
    vars = {
      fmc_ip       = var.fmc_mgmt_ip
      fmc_nat_id   = var.fmc_nat_id
      reg_key      = var.reg_key
    }
}
data "aws_availability_zones" "available" {}
```
Template file is used to fetch the *"user data"* that is needed at the time of FTD creation. ```aws_availability_zones``` is used to get list of all the AZs in a region.  



**<ins>Creation of FTD instance </ins>** 

```
resource "aws_instance" "ftdv" {
  count               = 2
  ami                 = data.aws_ami.ftdv.id
  instance_type       = var.ftd_size
  key_name            = var.keyname

  network_interface {
    network_interface_id = element(var.ftd_mgmt_interface,count.index)
    device_index         = 0
  }
  network_interface {
    network_interface_id = element(var.ftd_diag_interface,count.index)
    device_index         = 1
  }
  network_interface {
    network_interface_id = element(var.ftd_outside_interface,count.index)
    device_index         = 2
  }
  network_interface {
    network_interface_id = element(var.ftd_inside_interface,count.index)
    device_index         = 3
  }
  user_data = data.template_file.ftd_startup_file[count.index].rendered

  tags ={
    Name = "Cisco ftdv${count.index}"
  }
}
```  
To attach various network interfaces we simply pass that network interface's ID, with a device index. 
>Note: Device index of each NIC must be different.

Following is the user data used.
```
  #Sensor
  {
      "AdminPassword": "Cisco123",
      "Hostname":       "FTD-01",
      "ManageLocally":  "No",
      "FmcIp":          "${fmc_mgmt_ip}",
      "FmcRegKey":      "${reg_key}",
      "FmcNatId":       "${fmc_nat_id}",
}
```

