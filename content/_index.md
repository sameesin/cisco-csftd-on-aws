---
title: "Cisco Secure Firewall on AWS Workshop"
chapter: true
weight: 1
---

# Cisco Secure Firewall on AWS Workshop

### Welcome

In this lab, users will programmatically deploy Cisco Secure Firewall Threat Defence (FTD) and Firewall Management Center (FMC) using Infrastructure as Code (Terraform). The firewalls will be placed behind a network load balancer. User will also programmatically configure the firewalls once onboarded to ensure it allows required traffic flow from internet to the test machine setup in the AWS environment.

### Learning Objectives

1. Deploy Cisco Secure Firewall Threat Defence and Management Center using Terraform
2. Enable N/S protection through Cisco Secure Firewall

### Lab Topology

![Topology](content/IMAGES/topology.png)

{{% notice warning %}}
<p style='text-align: left;'>
The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how various AWS and Cisco services can be architected to build a solution while demonstrating best practices along the way. These examples are not intended for use in production environments.
</p>
{{% /notice %}}
