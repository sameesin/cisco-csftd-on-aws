+++
title = "Setting up network resources"
chapter = false
weight = 1
+++

### Introduction

Now in this section we are creating all the required network resources for creating our topology.Such as  
1. VPC
2. Subnets  
3. Interfaces
4. Security groups
5.  Route tables   

 Lets get started with our first resource that is **```VPC```**   

 ## **1. <ins>VPC**</ins>   (Virtual private cloud)
   
   For this model we are creating a vpc named ```Transit-Service-VPC1``` which will be expanded to two availability zones in ```ap-south-1``` region.

 <ins>***Code snippet to create ```VPC```***</ins>    (Basic code)
   
```
resource "aws_vpc" "vpc_name"{
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  instance_tenancy     = "default"
  tags = {
    Name = "First VPC"
  } 
}
```


This piece of code has many attributes:

1. A CIDR must be given to the VPC to help assign IP address to all the components inside it. 
2. Setting ```enable_dns_hostnames = true``` will simply enable DNS hostnames. 
> Note: DNS support is enabled by default; to disable it, add the attribute ```enable_dns_support = false``` 
3. Instance tenacy specifies the method by which VPC is distributed across physical servers at aws. AWS will charge you some amount if the value of ```instance_tenancy``` is changed from default to either *dedicated* or *host*.    
Check this [link](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-dedicated-instances.html) for more information. 
4. Count determines the number of particular resource to be created.  
   

![Created VPC](../images/vpc.jpeg)

## **2. <ins>Subnets**</ins>
  
  In this model we are creating the following subnets.  

* Inside subnets      - **```2```** (one in each AZ)  
* Outside subnets     - **```2```** (one in each AZ)  
* Mgmt subnets        - **```2```** (one in each AZ)  
* Diag subnets        - **```2```** (one in each AZ)
* Application subnets - **```2```** (one in each AZ)  
* Test subnet         - **```1```** (In any one of the AZ)  

To create one subnet the following is the code snippet (Basic code)


```
resource "aws_subnet" "main" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"

  tags = {
    Name = "Main"
  }
}
```
><ins>**Note**</ins> : The following is the code snippet to create subnets according to our topology. We need to change the code accordingly.
```
resource "aws_subnet" "subnet_name" {
  count             = 2  //one in each az
  vpc_id            = aws_vpc.vpc_name.id
  cidr_block        = var.sub_cidr[count.index]
  availability_zone = data.aws_availability_zones.available.names[count.index]
  tags = {
    Name = "Subnet-$[count.index]"
  }
}
```
1. A subnet can't be created without a VPC, so we provide the id of our previously created VPC. In terraform we pass the id using ```aws_vpc.vpc_name.id``` 
2.  Make sure that the CIDR of subnet matches the range of the CIDR in VPC. Since 2 network interfaces are created here, we are passing two CIDRs by using the variable in conjunction with *count.index*.
```
variable "sub_cidr" {
  type    = list(string)
  default = ["10.0.1.0/24","10.0.10.0/24"]
}
```
3.  For creating 2 subnets in 2 different availabilty zones, you would need to fetch the data of all the zones present in required region. This can be done by using a dedicated [data source](https://www.terraform.io/language/data-sources)  block provided by terraform in the following manner.
```
data "aws_availability_zones" "available" {}
```
> Note: The value of just single availability zone can be passed like this: ```availability_zone = "ap-south-1a"```

![Created subnets](../images/subnets.jpeg)  
Subnets created

## **3. <ins>Network Interfaces** </ins>: 
  
  In this model we are creating the following network interfaces.  
  * ftd_inside   -  **```2```** (one in each AZ)
  * ftd_outside  - **```2```** (one in each AZ)
  * ftd_mgmt - **```2```** (one in each AZ)
  * ftd_diag - **```2```** (one in each AZ)
  * ftd_app - **```2```** (one in each AZ)
  * ftd_test - **```1```** (In the AZ where we created test subnet)  

  


To create one subnet the following is the code snippet (Basic code)  

```
resource "aws_network_interface" "interface_name" {
  subnet_id       = aws_subnet.subnet_name.id
  private_ips     = ["10.0.0.50"]
  security_groups = [aws_security_group.security_group_name.id]

  attachment {
    instance     = aws_instance.instance_name.id
    device_index = 1
  }
}
```
![Created network interfaces](../images/network_interfaces.jpeg)  
<br>

## 4. <ins>**Security Group :**</ins> 
<br>
 In this topology we are creating following two security groups.  

 * allow_all (allows all traffic)
 * default  (allows traffic according to the rules given)

  
A security group can be heavily customized according to the needs of particular network, the example given below focuses on allowing only TCP inbound request(*ingress*) and all Outbound requests(*egress*).

<br>
<ins>Code snippet </ins>:

```
resource "aws_security_group" "SG_name" {
  name        = "First_SG"
  description = "TCP inbound"
  vpc_id      = aws_vpc.vpc_name.id
  egress = [
    {
      cidr_blocks      = [ "0.0.0.0/0", ]
      description      = ""
      from_port        = 0
      ipv6_cidr_blocks = []
      prefix_list_ids  = []
      protocol         = "-1"
      security_groups  = []
      self             = false
      to_port          = 0
    }
  ]
 ingress                = [
   {
     cidr_blocks      = [ "0.0.0.0/0", ]
     description      = ""
     from_port        = 22
     ipv6_cidr_blocks = []
     prefix_list_ids  = []
     protocol         = "tcp"
     security_groups  = []
     self             = false
     to_port          = 22
  }
  ]
}
```
Setting CIDR to ```0.0.0.0/0``` will ensure that we all ips to connect with our server. Since we are only concerned with TCP connection thus we set the **from_port** & **to_port** attributes to port number 22 and set the protocol to TCP in the ingress.

This kind of egress part will allow all external connections, this can  be changed if more restrictions are required.  

To attach the security group to the network interface we need to use ```aws_network_interface_sg_attachment```

```
resource "aws_network_interface_sg_attachment" "attachment_name" {
  count                = 2
  security_group_id    = aws_security_group.SG_name.id
  network_interface_id = aws_network_interface.ftd_mgmt[count.index].id
}
```
Simply pass the SG id and network interface id to complete the attachment.
![Created security groups](../images/security_groups.jpeg)    

## **5. <ins>Route tables**</ins>:

Route tables or routes are important for VPC to keep a track of where the traffic from our subnets (or gateway) is directed to.

First create an [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html) for your VPC. Internet gateway is required for several reasons, without it the internet-routable traffic can't move. Gateway is also responisble to perform NAT on instances having public ip address.    
  

Code snippet to create internet gateway:

```
resource "aws_internet_gateway" "int_gw" {
  vpc_id = aws_vpc.vpc_name.id
  tags = {
    Name = "Internet Gateway"
  }
}
```
 ![internet_gateway](../images/internet_gateway.jpeg)   
 <br>


Now to create route tables and routes, we will use ```aws_route_table```.
```
resource "aws_route_table" "route_name" {
  vpc_id = aws_vpc.vpc_name.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.int_gw_name.id
  }
  tags = {
    Name = "Test route table"
  }
}
```  
  These are the route tables created 
![Created route tables](../images/route_tables.jpeg)    
  
> Note: Terraform also provides a dedicated resource block "[aws_route](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route)" for setting route. It can be used in the place of route attachment in ```aws_route_table```.

To associate a route table to any subnet we will use ```aws_route_table_association```

```
resource "aws_route_table_association" "route_association" {
  subnet_id      = aws_subnet.subnet_name.id
  route_table_id = aws_route_table.route_name.id
}
```  
We are also creating following route table associations in this model-

* inside_association
* outside_association
* mgmt_association
* diag_association
* app_association
* test_association

Pass the subnet id and the route table id to complete the creation of routes. 

