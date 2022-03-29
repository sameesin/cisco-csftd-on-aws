+++
title = "Deployment of Load balancers"
chapter = false
weight = 1
+++

# **Deployment of Load Balancers**
Load balancer is a key component of the network. It distributes all the incoming traffic across any set of components like EC2 instance, DB, etc. 

In this lab we will create ***one external network load balancer*** forwarding traffic to the Cisco FTD outside interface.

```aws_lb``` can be used to create the load balancer.  
```
resource "aws_lb" "external-lb" {
  name                      = "External-LB"
  load_balancer_type        = "network"
  enable_cross_zone_load_balancing = "true"
  subnets                   = var.outside_subnet_id
}
```

1. Notice that the external load balancer points towards outside subnet.
2. Enabling cross zone load balancing is crucial to make sure that requests are evenly split between all the availability zones used. 
   
  ![load_balancers](../IMAGES/load_balancers.jpeg) 

The variables created to set port and protocol on which the load balancer will listen to and use for checking health status of tartget instances 
```
variable "external_listener_ports" {
  default = [{
    protocol = "TCP"
    port = 22
    target_type = "ip"
  }]
}

variable "external_health_check" {
  default = {
    protocol = "TCP"
    port     = 80
  }
}
```

The load balancer will require some targets to route requests to, to create a target group we use ```aws_lb_target_group```.

```
resource "aws_lb_target_group" "external_front_end" {
  for_each    = {for k, v in var.external_listener_ports : k => v}
  name        = tostring("external-${each.key}")
  port        = each.value.port
  protocol    = each.value.protocol
  target_type = each.value.target_type
  vpc_id      = var.vpc_id

  health_check {
    interval = 30
    protocol = var.external_health_check.protocol
    port     = var.external_health_check.port
  }
}
```    
  ![target_groups](../IMAGES/target_groups.jpeg) 
  
We can change the attributes of ```heath_check ```attachment as per needed.

Attaching targets to the target group: 
```
resource "aws_lb_target_group_attachment" "target1" {
  count            = length(var.ftd_outside_ip)
  depends_on       = [aws_lb_target_group.external_front_end]
  target_group_arn = aws_lb_target_group.external_front_end[0].arn
  target_id        = var.ftd_outside_ip[count.index]
}
```

To configure the port and protocol the load balancer should listen on, we use ```aws_lb_listener```

```
resource "aws_lb_listener" "external_listener" {
  for_each  = { for k, v in var.external_listener_ports : k => v }
  load_balancer_arn = aws_lb.external-lb[0].arn
  port              = each.value.port
  protocol          = each.value.protocol
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.external_front_end[each.key].arn
  }
}
```
1. We add the ARN of load balancer and the ports and protocols desired. 
2. Target group is also attached as the part of ```default_action```.