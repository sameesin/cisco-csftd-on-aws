---
title: "Deploying Solution"
weight: 1
---

## **Introduction**
In this section user will deploy the AWS environment and then configure the cisco secure firewall which will be part of the target group of the gateway load balancer to enable traffic inspection from test VM to internet.

## Service VPC
The Service VPC is already deployed with a running FMC and user will reference that VPC in the code.

```terraform
data "aws_vpc" "ftd_vpc" {
  count = var.vpc_cidr == "" ? 1 : 0
  filter {
    name   = "tag:Name"
    values = [var.vpc_name]
  }
}
```

## Service VPC Subnets
The following subnets will be created in the service VPC per Availability zone:
- management
- Diagnostic
- Outside
- Inside
- Transit Gateway
- Gateway loadbalancer endpoint
- NAT Gateway

```terraform
resource "aws_subnet" "mgmt_subnet" {
  count                   = length(var.mgmt_subnet_cidr) != 0 ? length(var.mgmt_subnet_cidr) : 0
  vpc_id                  = local.con
  cidr_block              = var.mgmt_subnet_cidr[count.index]
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true
  tags = merge({
    Name = "${var.mgmt_subnet_name[count.index]}"
  }, var.tags)
}
```

## Gateway Load balancer
One Gateway load balancer will be deployed in the service VPC with Cisco secure firewalls as its target group to inspect traffic from the spoke network

```terraform
resource "aws_lb" "gwlb" {
  name               = var.gwlb_name
  load_balancer_type = "gateway"
  subnets            = var.gwlb_subnet

  tags = {
    Name = var.gwlb_name
  }
}
```

## NAT Gateway
One NAT Gateway in each AZ of Service VPC will be deployed to facilitate outbound traffic. The traffic from the nat gateway will point to the internet gateway associated with the vpc.

```terraform
resource "aws_nat_gateway" "natgw" {
  depends_on    = [data.aws_subnet.dngw_subnet]
  count         = var.availability_zone_count
  allocation_id = aws_eip.nat_eip[count.index].id
  subnet_id     = data.aws_subnet.dngw_subnet[count.index].id

  tags = {
    Name = "NAT-GW-${count.index + 1}"
  }
}
```

## Routes
The gateway load balancer endpoint subnets will have a default route pointing to the nat gateway of its respective AZ and routes to spoke VPC subnet pointing to the transit gateway
The transit gateway subnet will have a default route pointing to the gateway load balancer endpoint of its respective AZ
The spoke subnet will have a default route pointing to the transit gateway

## Cisco Secure Firewall
Two Cisco Secure Firewall Threat Defense will be deployed, one in each AZ with four interfaces and will be added in the target group of the gateway load balancer.
The firewalls will be added to the existing FMC

```terraform
resource "aws_instance" "ftdv" {
  count         = var.instances_per_az * var.availability_zone_count
  ami           = data.aws_ami.ftdv.id
  instance_type = var.ftd_size
  key_name      = var.keyname
  network_interface {
    network_interface_id = element(var.ftd_mgmt_interface, count.index)
    device_index         = 0
  }
  network_interface {
    network_interface_id = element(var.ftd_diag_interface, count.index)
    device_index         = 1
  }
  network_interface {
    network_interface_id = element(var.ftd_outside_interface, count.index)
    device_index         = 2
  }
  network_interface {
    network_interface_id = element(var.ftd_inside_interface, count.index)
    device_index         = 3
  }
  user_data = data.template_file.ftd_startup_file[count.index].rendered
  tags = merge({
    Name = "Cisco ftdv${count.index}"
  }, var.tags)
}
```

## Spoke VPC
A spoke VPC will be created where the test machine will be deployed. It will connect to the service VPC using the transit gateway.

```terraform
resource "aws_vpc" "ftd_vpc" {
  count                = var.vpc_cidr != "" ? 1 : 0
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true
  #enable_classiclink   = false
  instance_tenancy     = "default"
  tags = merge({
    Name = var.vpc_name
  }, var.tags)
}
```

## Spoke VPC Subnets
Two subnets will be created, one in each AZ in the spoke VPC and the test machines will be part of these subnets.

```terraform
resource "aws_subnet" "outside_subnet" {
  count             = length(var.outside_subnet_cidr) != 0 ? length(var.outside_subnet_cidr) : 0
  vpc_id            = local.con
  cidr_block        = var.outside_subnet_cidr[count.index]
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = merge({
    Name = var.outside_subnet_name[count.index]
  }, var.tags)
}
```

## Transit Gateway
One transit gateway will be deployed to bridge connection between the service and spoke VPC using transit gateway attachments. Routing table for each attachment will be configured.
Appliance mode will be enabled for service VPC Transit Gateway Attachment

### Transit Gateway

```terraform
resource "aws_ec2_transit_gateway" "tgw" {
  count = var.create_tgw ? 1 : 0
  tags = {
    Name = var.transit_gateway_name
  }
}
```

### Transit Gateway Attachments

```terraform
resource "aws_ec2_transit_gateway_vpc_attachment" "tgw_att_service_vpc" {
  count                                           = var.create_tgw ? 1 : 0
  subnet_ids                                      = local.tgw_subnet
  transit_gateway_id                              = local.tgw
  vpc_id                                          = var.vpc_service_id
  appliance_mode_support                          = "enable"
  transit_gateway_default_route_table_association = false
  transit_gateway_default_route_table_propagation = false
  tags = {
    Name = "tgw-att-service-vpc"
  }
}

resource "aws_ec2_transit_gateway_vpc_attachment" "tgw_att_spoke_vpc" {
  subnet_ids                                      = var.spoke_subnet_id
  transit_gateway_id                              = local.tgw
  vpc_id                                          = var.vpc_spoke_id
  transit_gateway_default_route_table_association = false
  transit_gateway_default_route_table_propagation = false
  tags = {
    Name = "tgw-att-spoke-vpc-${var.spoke_subnet_id[0]}"
  }
}
```

## Lambda Function
A lambda function will be deployed which will be invoked once deployed to configure the Cisco secure firewall management center. Using REST API the following will be configured:
  - FTD Device Registration
  - Interface Configuration
  - VTEP
  - VNI Interface
  - NAT rules for health check
  - Access Policy
  - Access Rule to allow health check probe traffic
  - Inside subnet Gateway Network object