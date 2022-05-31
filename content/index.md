---
title: "Cisco Secure Firewall on AWS Workshop"
weight: 1
---

## Welcome

In this lab, users will programmatically deploy Cisco Secure Firewall Threat Defence (FTDv) and Firewall Management Center (FMC) using Infrastructure as Code (Terraform). The firewalls will be placed behind a network load balancer. Users will also programmatically configure the firewalls once onboarded to ensure it allows required traffic flow from internet to the test machine setup in the AWS environment. 

The expected duration for this lab is 2-3 hours.

The target audience for this lab is anyone that has an interest in learning about how Cisco FTDv and FMCv firewall solutions can be deployed on AWS with terraform automation.

### Learning Objectives

1. Deploy Cisco Secure Firewall Threat Defence and Management Center using Terraform
2. Enable N/S protection through Cisco Secure Firewall

### Lab Topology

![Topology](/static/images/topology.png)

> The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how various AWS and Cisco services can be architected to build a solution while demonstrating best practices along the way. These examples are not intended for use in production environments.