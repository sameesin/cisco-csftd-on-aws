---
title: "Firewall Configuration"
weight: 2
---

## Introduction

The firewalls need to be configured with the required configurations to make sure that your firewall performs as expected.

In this lab we will make use of **Cisco FMC REST API** and **python** to programmatically configure the following components:  

1. Access control policy - **```1```**.
2. Security zones - **```2```**. (inside & outside)
3. Network & Host objects like LB & Gateway - **```1```** each.
4. NAT policy and rules - **```2```**
5. Onboard the 2 FTD to FMC
6. Interfaces  
7. Routes 
8. Associate NAT Policy with FTD

---
**Note: There are ways to set up an FMC:**
* Using FMC UI
* Using python script utilizing the **fmcpi** python module
* Using Terraform Provider for FMC

>We will be using Terraform Provider FMC for this lab


### <ins>Step 1:  Obtain ILB Private IP Addresses using AWS CLI</ins>

To find private IP addresses of the load balancer, run the command: 

```console
aws elbv2 describe-load-balancers --names External-LB
```
obtain the ELB id (e.g - net/test/a43050c9db0117) and use it in the next command.

```console
aws ec2 describe-network-interfaces --filters Name=description,Values="ELB <ELB ID>" --query 'NetworkInterfaces[*].PrivateIpAddresses[*].PrivateIpAddress'
```

This will give you all the private IPs of ILB, enter them when prompted. 

Update the terraform.tfvars file in AWS_Workshop_Code_EventEngine folder with the ELB Private IP address.

### <ins> Step 2: Tapping in FMC </ins>

```
terraform {
  required_providers {
    fmc = {
      source = "CiscoDevNet/fmc"
    }
  }
}

provider "fmc" {
  fmc_username = "admin"
  fmc_password = var.fmc_admin_password
  fmc_host = var.fmc_ip
  fmc_insecure_skip_verify = var.fmc_insecure_skip_verify
}
```

### <ins> Step 3: Creating Access Control Policy </ins>

**fmc_access_policies** resource is used to create an ACP, pass the the name of policy to deploy it and choose the default action that ACP must take, out of *Block all traffic, Intrusion Prevention & Network Discovery.

```
resource "fmc_access_policies" "access_policy" {
  name = "IAC-ACP"
  default_action = "BLOCK"
  default_action_send_events_to_fmc = "true"
  default_action_log_end = "true"
}
```

### <ins> Step 4: Adding Security Zones</ins>

Following security zones are created on FMC
- inside
- outside

**fmc_security_zone** resource is used to create two security zones. **Inside** and **Outside** SZs are created here with their interface mode set to *routed*.

```
resource "fmc_security_zone" "inside" {
  name            = "inside"
  interface_mode  = "ROUTED"
}
resource "fmc_security_zone" "outside" {
  name            = "outside"
  interface_mode  = "ROUTED"
}
```


### <ins> Step 5: Adding Hosts & Network Objects </ins>

Following host objects will be created on FMC:
- inside-gateway - `1` per AZ (inside subnet gateway)
- default-gateway - `1` per AZ (outside subnet gateway)
- ELB - `1` per AZ (private IP address of public LB)
- app-lb - `1` per AZ (IP address of internal lb)

Following network object is created on FMC:
APP - `1` per AZ (application instance subnet)

Following is an example of how the terraform code will look like.

```
resource "fmc_host_objects" "APP-LB" {
  count = 2
  name        = "app-lb${count.index+1}"
  value       = var.app_lb[count.index]
}

resource "fmc_network_objects" "APP" {
  count = 2
  name        = "APP${count.index+1}"
  value       = var.ftd_app_ip[count.index]
}
```


### <ins>Step 6: Creating ACP rules</ins>

The code below creates a rule to allows all outside traffic to the webserver.

**fmc_access_rules** resource is used to add source & destination zones and the destination network to make up the rule. 

We provide the name of security zones and name of the Load Balancer for the web server where needed. set *action=allow* to add the respective fields.

Access rule is a subset of ACP.

```
resource "fmc_access_rules" "access_rule_1" {
    acp = fmc_access_policies.access_policy.id
    section = "mandatory"
    name = "To Web Server"
    action = "allow"
    enabled = true
    send_events_to_fmc = true
    log_end = true
    source_zones {
        source_zone {
            id = fmc_security_zone.outside.id
            type = "SecurityZone"
        }
    }
    destination_zones {
        destination_zone {
            id = fmc_security_zone.inside.id
            type = "SecurityZone"
        }
    }
    destination_networks {
        destination_network {
            id = fmc_host_objects.APP-LB[0].id
            type =  fmc_host_objects.APP-LB[0].type
        }
        destination_network {
            id = fmc_host_objects.APP-LB[1].id
            type =  fmc_host_objects.APP-LB[1].type
        }
    }
    new_comments = [ "Traffic to Web Server" ]
}
```


### <ins> Step 7: Adding Static NAT</ins>

NAT rule is created to translate source IP address from external load balancer IP address to Inside interface IP address and destination IP address from outside interface IP address to Internal load balancer IP address.

`fmc_ftd_nat_policies` and `fmc_ftd_manualnat_rules` resources are used to create the NAT policy and NAT rules.


```
resource "fmc_ftd_nat_policies" "nat_policy" {
    count = 2
    name = "NAT_Policy${count.index}"
    description = "Nat policy by terraform"
}

resource "fmc_ftd_manualnat_rules" "new_rule1" {
    count = 2
    nat_policy = fmc_ftd_nat_policies.nat_policy[count.index].id
    nat_type = "static"
    original_source{
        id = fmc_host_objects.ELB[0].id
        type = fmc_host_objects.ELB[0].type
    }
    source_interface {
        id = fmc_security_zone.outside.id
        type = "SecurityZone"
    }
    destination_interface {
        id = fmc_security_zone.inside.id
        type = "SecurityZone"
    }
    original_destination_port {
        id = data.fmc_port_objects.http.id
        type = data.fmc_port_objects.http.type
    }
    translated_destination_port {
        id = data.fmc_port_objects.http.id
        type = data.fmc_port_objects.http.type
    }
    translated_destination {
        id = fmc_host_objects.APP-LB[0].id
        type = fmc_host_objects.APP-LB[0].type
    }
    interface_in_original_destination = true
    interface_in_translated_source = true
}
```

The NAT Rule above is for only one application server. 2nd NAT Rule will be the same with ```original_source``` as "ELB[1]" & ```translated_destination``` as "APP-LB[1]" 


### <ins> Step 8: Register FTD to FMC </ins>

`fmc_devices` resource is used to register FTD devices to FMC.

1. Provide the IP address of your deployed FTD for hostName field. Can be found in the AWS EC2 console.
2. Provide the same registration key used while creating the FTD, make sure that this key must not have any special characters apart from '-'. 
3. Attach the ACP to the FTD .

```
resource "fmc_devices" "device1"{
  depends_on = [fmc_ftd_nat_policies.nat_policy, fmc_security_zone.inside, fmc_security_zone.outside]
  name = "FTD1"
  hostname = var.ftd_mgmt_ip[0]
  regkey = "cisco"
  type = "Device"
  #license_caps = [ "MALWARE"]
  #nat_id = "cisco"
  access_policy {
      id = fmc_access_policies.access_policy.id
      type = fmc_access_policies.access_policy.type
  }
}
```
The same needs to be done for FTD2
 
### <ins> Step 9: Physical Interfaces </ins>

2 interfaces with logical name `inside` and `outside` will be created on FTD using `fmc_device_physical_interfaces` resource.

Following is example of creating `outside` interface on FTD
```
data "fmc_devices" "device" {
  depends_on = [fmc_devices.device2]
  count = 2
  name = "FTD${count.index+1}"
}

data "fmc_device_physical_interfaces" "zero_physical_interface" {
    count = 2
    device_id = data.fmc_devices.device[count.index].id
    name = "TenGigabitEthernet0/0"
}

resource "fmc_device_physical_interfaces" "physical_interfaces00" {
    count = 2
    enabled = true
    device_id = data.fmc_devices.device[count.index].id
    physical_interface_id= data.fmc_device_physical_interfaces.zero_physical_interface[count.index].id
    name =   data.fmc_device_physical_interfaces.zero_physical_interface[count.index].name
    security_zone_id= fmc_security_zone.outside.id
    if_name = "outside"
    mode = "NONE"
    ipv4_dhcp_enabled = true
    ipv4_dhcp_route_metric = 1
}
```

### <ins> Step 10: Adding static routes on FTD to Webserver </ins>

A static route will be created on FTD to webserver with next hop as inside gateway IP address.
`fmc_staticIPv4_route` resource will be used to create the static route.

```
resource "fmc_staticIPv4_route" "route" {
  depends_on = [data.fmc_devices.device, fmc_device_physical_interfaces.physical_interfaces00,fmc_device_physical_interfaces.physical_interfaces01]
  count = 2
  metric_value = 1
  device_id  = data.fmc_devices.device[count.index].id
  interface_name = "inside"
  selected_networks {
      id = fmc_network_objects.APP[count.index].id
      type = fmc_network_objects.APP[count.index].type
      name = fmc_network_objects.APP[count.index].name
  }
  gateway {
    object {
      id   = fmc_host_objects.inside-gw[count.index].id
      type = fmc_host_objects.inside-gw[count.index].type
      name = fmc_host_objects.inside-gw[count.index].name
    }
  }
}
```

### <ins> Step 11: Associate NAT policy with FTD. </ins>

`fmc_policy_devices_assignments` resource is used to assign  the created NAT Policy to FTD instances.

```
resource "fmc_policy_devices_assignments" "policy_assignment" {
  depends_on = [fmc_staticIPv4_route.route]
  count = 2
  policy {
      id = fmc_ftd_nat_policies.nat_policy[count.index].id
      type = fmc_ftd_nat_policies.nat_policy[count.index].type
  }
  target_devices {
      id = data.fmc_devices.device[count.index].id
      type = data.fmc_devices.device[count.index].type
  }
}
```

### <ins> Step 11: Deploy changes to FTD. </ins>

`fmc_ftd_deploy` resource is used to deploy the changes made on FMC to the FTD devices.

```
resource "fmc_ftd_deploy" "ftd" {
    depends_on = [fmc_policy_devices_assignments.policy_assignment]
    count = 2
    device = data.fmc_devices.device[count.index].id
    ignore_warning = true
    force_deploy = false
}
```

## <ins>Running the Terraform code</ins>

In cloud9 IDE navigate to the `AWS_Workshop` folder and ensure following files are available.
**fmc_config_terraform.tf**, **providers.tf**, **variables.tf** and **terraform.tfvars**

Run the following commands:
```
terraform init
```

```
terraform validate
```

```
terraform plan --out awslab
```

```
terraform apply awslab
```

>Note: FMC Script can take upto 10 min to properly execute based on the delay added in the code.

To ensure that everything took place as per your requirment open the FMC-UI on your browser and check things there. 

For eg: 
1. **ACP:**
![fmcapi_policy](/static/images/firewall_configuration/FMCAPI_POLICY.png) 

It also shows that this was created using fmcapi

2. **Devices:**
![device](/static/images/firewall_configuration/FMC_DEVICE_SCR.png)

Check the policy name attached to the FTD.

