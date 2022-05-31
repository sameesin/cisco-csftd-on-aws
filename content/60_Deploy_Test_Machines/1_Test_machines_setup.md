---
title: "Test Machines Setup - Description"
weight: 1
---

## Introduction

To test the network setup we will create **2 Web servers** in different AZs. 
**1 bastion host** will be deployed to connect to the servers locally.

This document also consists of **USER Data** of both **application & bastion servers** to install ```apache web-server``` that will be used for out testing.

## <ins> **Deploying one Bastion server**</ins>

```console
resource "aws_instance" "bastion_machine" {
  ami           = "ami-055d15d9cfddf7bd3" 
  instance_type = "t2.micro"
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

The user data for bastion server is:

```conole
#!/bin/bash
sudo apt-get update
sudo apt install python3
sudo apt-get --print-uris --yes install apache2 | grep ^\' | cut -d\' -f2 > todownload.txt
mkdir web
cd ./web
sudo wget --input-file /todownload.txt
cd ..
tar -czvf archive.tar.gz /web
python3 -m http.server --bind 0.0.0.0 9000
```

>Note: Bastion machine is in bastion_subnet with it's own network interface & security group attached to it.

**Routes are also needed to be created for this machine**

```console
resource "aws_route_table" "bastion_route" {
  vpc_id = module.network.vpc_id
  tags = merge({
    Name = "bastion network Routing table"
  }, var.tags)
}

resource "aws_route_table_association" "bastion_association" {
  subnet_id      = aws_subnet.bastion_subnet.id
  route_table_id = aws_route_table.bastion_route.id
}

resource "aws_route" "bastion_default_route" {
  route_table_id         = aws_route_table.bastion_route.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = module.network.internet_gateway
}
``` 

## <ins> **Deploying two Web servers**</ins>

```console
resource "aws_instance" "Application_server" {
  count         = 2
  ami           = "ami-055d15d9cfddf7bd3" 
  instance_type = "t2.micro"
  key_name      = var.keyname

  network_interface {
    network_interface_id = element(aws_network_interface.app_interface[*].id,count.index)
    device_index         = 0
  }
  tags = {
    Name = "Application-Server ${count.index+1}"
  }
}
```
>Note: Both Web servers are in different AZ & in their own dedicated subnet with respective network interfaces with security group attached to them.  


We will also be creating an internal ***Network Load balancer*** with listeners & target for attaching both **app_subnets** to it.

```console
resource "aws_lb" "app-lb" {
  name                             = "App-LB"
  internal                         = true
  load_balancer_type               = "network"
  enable_cross_zone_load_balancing = "true"
  subnet_mapping {
    subnet_id            = aws_subnet.app_subnet[0].id
    private_ipv4_address = "<private ip address>"
  }

  subnet_mapping {
    subnet_id            = aws_subnet.app_subnet[1].id
    private_ipv4_address = "<private ip address>"
  }
}
```

* While mapping both subnets we also assign private_ip to each of them, this will help us while testing traffic. 

* Ensure that the IP address added belong to your application subnet.
  
>Note: Listner, Target group & Target Group attachment also needs to created for **app-lb**

**Creating Route table for web servers:**

```console
resource "aws_route_table" "ftd_app_route" {
  vpc_id = module.network.vpc_id
  tags = merge({
    Name = "App network routing table"
  }, var.tags)
}

resource "aws_route_table_association" "app_association" {
  count          = var.app_subnet_cidr != null ? length(var.app_subnet_cidr) : length(var.app_subnet_name)
  subnet_id      = aws_subnet.app_subnet[count.index].id
  route_table_id = aws_route_table.ftd_app_route.id
}
```

The user data for application server:
```
#!/bin/bash
echo 'test' > output.txt
curl 10.0.6.10:9000/archive.tar.gz -o kd.tar.gz
tar zxvf kd.tar.gz
cd ./web
sudo dpkg -i *.deb
```
  