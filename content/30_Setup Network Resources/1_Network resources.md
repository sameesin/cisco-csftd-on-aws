---
title: "Setting up network resources"
chapter: false
weight: 1
---

### Introduction

Now in this section we are creating all the required network resources for creating our topology.Such as  
1. Subnets  
2. Interfaces
3. Security groups
4. Route tables  

 Lets get started with our first resource that is **```VPC```** 

 ## **1. <ins>VPC**</ins>   (Virtual private cloud)
   
   we are going to use VPC ```IAC-VPC``` created in the previous section.

   Open aws console to check the vpc you created

![Created subnets](//static/images/setup_network_resources/vpc.jpeg)  


 <ins>***Code snippet to refer the VPC created in previous section***</ins>
   
```
data "aws_vpc" "ftd_vpc" {
  count = var.vpc_cidr == "" ? 1 : 0
  filter {
    name   = "tag:Name"
    values = [var.vpc_name]
  }
}
``` 


## **2. <ins>Subnets**</ins>
  
In this lab we are creating the following subnets.  

* Inside subnets      - **```2```** (one in each AZ)  
* Outside subnets     - **```2```** (one in each AZ)  
* Mgmt subnets        - **```2```** (one in each AZ)  
* Diag subnets        - **```2```** (one in each AZ)

><ins>**Note**</ins> : The following is the snippet to create just the outside subnet. This block must be created for each subnet mentioned above.

```
resource "aws_subnet" "outside_subnet" {
  count             = 2  //one in each az
  vpc_id            = data.aws_vpc.ftd_vpc.id
  cidr_block        = var.outside_sub_cidr[count.index]
  availability_zone = data.aws_availability_zones.available.names[count.index]
  tags = {
    Name = "Subnet-$[count.index]"
  }
}
```

1. A subnet can't be created without a VPC, so we provide the id of our previously created VPC. In terraform we pass the id using ```data.aws_vpc.ftd_vpc.id``` 
2.  Make sure that the CIDR of subnet matches the range of the CIDR in VPC. Since 2 network interfaces are created here, we are passing two CIDRs by using the variable in conjunction with *count.index*.
```
variable "outside_sub_cidr" {
  type    = list(string)
  default = ["10.0.2.0/24","10.0.20.0/24"]
}
```
3.  For creating 2 subnets in 2 different availabilty zones, you would need to fetch the data of all the zones present in required region. This can be done by using a dedicated [data source](https://www.terraform.io/language/data-sources)  block provided by terraform in the following manner.
```
data "aws_availability_zones" "available" {}
```
> **Note**: If needed, the value of just single availability zone can be passed like this: ```availability_zone = "ap-south-1a"```


## **3. <ins>Network Interfaces** </ins>: 
  
  In this model we are creating the following network interfaces. 

  * ftd_inside   -  **```2```** (one in each AZ)
  * ftd_outside  - **```2```** (one in each AZ)
  * ftd_mgmt - **```2```** (one in each AZ)
  * ftd_diag - **```2```** (one in each AZ)
  * fmc_mgmt - **```1```**

>**Note** : The following is the snippet to create just the outside interface. This block must be created for each interface mentioned above.

```
resource "aws_network_interface" "ftd_outside" {
  count             = 2
  description       = "ftd${count.index}-outside"
  subnet_id         = local.mgmt_subnet[local.azs[count.index] - 1].id
  source_dest_check = false
  private_ips       = [var.ftd_outside_ip[count.index]]
}
```



## 4. <ins>**Security Groups :**</ins> 
<br>
 In this lab we are creating following two security groups.  

 * allow_all (allows all traffic)
 * default  (allows traffic according to the rules given)

  
A security group can be heavily customized according to the needs of particular network, the example given below focuses on allowing everything for our **allow_all** SG.

<br>
<ins>Code snippet </ins>:

```
resource "aws_security_group" "allow_all" {
  name        = "Allow All"
  description = "Allow all traffic"
  vpc_id      = aws_vpc.ftd_vpc.id
  dynamic "ingress" {
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/0"]
      description = null
    }
  }
  dynamic "egress" {
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/8"]
      description = null
    }
  }

```
Setting CIDR to ```0.0.0.0/0``` will ensure that we all ips to connect with our server. Since we are only concerned with TCP connection thus we set the **from_port** & **to_port** attributes to port number 22 and set the protocol to TCP in the ingress.

This kind of egress part will allow all external connections, this can  be changed if more restrictions are required.  

To attach the security group to the network interface we need to use ```aws_network_interface_sg_attachment```

```
resource "aws_network_interface_sg_attachment" "ftd_outside_attachment" {
  count                = 2
  security_group_id    = aws_security_group.allow_all.id
  network_interface_id = aws_network_interface.ftd_outside[count.index].id
}
```
Simply pass the SG id and network interface id to complete the attachment.
   

## **5. <ins>Route tables**</ins>:

Route tables or routes are important for VPC to keep a track of where the traffic from our subnets (or gateway) is directed to.

First create an [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html) for your VPC. Internet gateway is required for several reasons, without it the internet-routable traffic can't move. Gateway is also responisble to perform NAT on instances having public ip address.    
  

Code snippet to create internet gateway:

```
resource "aws_internet_gateway" "int_gw" {
  vpc_id = aws_vpc.ftd_vpc.id
  tags = {
    Name = "Internet Gateway"
  }
}
```
 ![internet_gateway](//static/images/setup_network_resources/igw.jpeg)   
 <br>

Now to create route tables and routes, we will use **aws_route_table** terraform resource.
```
resource "aws_route_table" "ftd_outside_route" {
  vpc_id = aws_vpc.ftd_vpc.id
  tags = {
    Name = "Outside route table"
  }
}
```  
We also need to associate the route using:
```
resource "aws_route_table_association" "outside_association" {
  count          = 2
  subnet_id      =  aws_subnet.outside_subnet[count.index].id
  route_table_id = aws_route_table.ftd_outside_route.id
}
```

A route table will be created for each of the subnets mentioned above.


## 6. <ins>**Elastic IP :**</ins> 

Next we create an Elastic IP for each of the **management** & the **fmcmgmt** interface created and associate them with the interfaces.


```
resource "aws_eip" "ftd_mgmt-EIP" {
  count = 2
  vpc   = true
  tags = merge({
    "Name" = "ftd-${count.index} management IP"
  }, var.tags)
}
```

```
resource "aws_eip_association" "ftd-mgmt-ip-assocation" {
  count                = 2
  network_interface_id = aws_network_interface.ftd_mgmt[count.index].id
  allocation_id        = aws_eip.ftd_mgmt-EIP[count.index].id
}
```

The same blocks will be created for FMC_management interface.