---
title: "Cisco Secure Firewall on AWS Workshop"
chapter: true
weight: 1
---

# Cisco Secure Firewall on AWS Workshop

### Welcome

In this lab, users will programmatically deploy Cisco Secure Firewall Threat Defence (FTD) and Firewall Management Center (FMC) using Infrastructure as Code (Terraform). The firewalls will be placed behind a network load balancer. User will also programmatically configure the firewalls once onboarded to ensure it allows required traffic flow from internet to the test machine setup in the AWS environment.

### Learning Objectives

1. Exercise 1 - Deploy FTDv and FMCv in an AWS VPC with terraform 
2. Exercise 2 - Gateway LB and Transit Gateway: Connect two workload VPCs to Security VPC using transit transit gateway and AWS GWLB
3. Exercise 3 - Configure Geneve on FTDv and deploy service and endpoint in VPCs, add firewalls in GWLB. 
4. Exercise 4 - Enable E/W protection through FTD and GWLB
5. Exercise 5 - Enable N/S protection through FTD and GWLB 
6. Exercise 6 (optional) - Add firewall in second AZ in security VPC and use appliance mode attachment to maintain traffic symmetry. 

{{% notice warning %}}
<p style='text-align: left;'>
The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how various AWS and Cisco services can be architected to build a solution while demonstrating best practices along the way. These examples are not intended for use in production environments.
</p>
{{% /notice %}}