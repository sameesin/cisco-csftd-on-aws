+++
title = "Firewall Configuration"
chapter = false
weight = 2
+++

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

>Note: There are two ways to set up an FMC:
* Using FMC UI
* Using python script utilizing the **fmcpi** python module

We are using python script.
 

### Step 0:  Install required module

Navigate to the cloud9 terminal and install fmcapi module with command ```pip3 install fmcapi```

### Step 1:  Obtain ILB Private IP Addresses using AWS CLI

To find private IP addresses of the load balancer, run the command: 
```
aws elbv2 describe-load-balancers --names External-LB
```
obtaine the ELB id (e.g - net/test/a43050c9db0117) and use it in the next command.
```
aws ec2 describe-network-interfaces --filters Name=description,Values="ELB <ELB ID>" --query 'NetworkInterfaces[*].PrivateIpAddresses[*].PrivateIpAddress'
```

This will give you all the private IPs of ILB, enter them when prompted. 

### Step 2: Tapping in FMC

```python
def main():
    with fmcapi.FMC(
        host=fmc_host_ip, 
        username=fmc_host_username,
        password=fmc_host_password, 
        autodeploy=False,
        file_logging='test.txt') as fmc:
```
This creates an instance of FMC class, we need to provide the *public ip, username, password*  of FMC.
Logs can be also be maintained in a .txt file by passing it as an argument.

>NOTE: The whole code from this point is also inside the class **main()** and on the same identation level.

### Step 3: Creating Access Control Policy
```python
acp = fmcapi.AccessPolicies(fmc=fmc, name= "IAC_ACP")
acp.defaultAction = "BLOCK"
acp.post()
```

**fmcapi.AccessPolicies** is used to create an ACP, pass the FMC instance and the name of policy to deploy it.

Choose the default action that ACP must take out of *Block all traffic, Intrusion Prevention & Network Discovery.

Writing **acp.post()** deploys this configuration

### Step 4: Adding Security Zones

```python
sz_inside = fmcapi.SecurityZones(fmc=fmc, name="inside", interfaceMode="ROUTED")
sz_inside.post()
sz_outside = fmcapi.SecurityZones(fmc=fmc, name="outside", interfaceMode="ROUTED")
sz_outside.post()
```

Use **fmcapi.SecurityZones** to create two(or more) security zones. **Inside** and **Outside** SZs are created here with their interface mode set to *routed*.

### Step 5: Adding Hosts & Network Objects

```python
#Gateway Host Object
dfgw_gateway = fmcapi.Hosts(fmc=fmc, name="default-gateway", value="10.0.2.1")
dfgw_gateway.post()

#ELB interface 1 IP
elb1 = fmcapi.Hosts(fmc=fmc, name="ELB1", value=elb_1)
elb1.post()

#ELB interface 2 IP
elb2 = fmcapi.Hosts(fmc=fmc, name="ELB2", value=elb_2)
elb2.post() 

#APP1 Network Object
 app1 = fmcapi.Networks(fmc=fmc, name="app15", value=app1_cidr)
app1.post()
        
#APP2 Network Object
app2 = fmcapi.Networks(fmc=fmc, name="app25", value=app2_cidr)
app2.post()

#APP LB IP Address 1
app_lb1 = fmcapi.Hosts(fmc=fmc, name="app-lb15", value=app1_lb_ip)
app_lb1.post()

#APP LB IP Address 2
app_lb2 = fmcapi.Hosts(fmc=fmc, name="app-lb25", value=app2_lb_ip)
 app_lb2.post()
```
* Provide an IP address for Internet Gateway, ensuring that it belongs to the outside_subnet.
* Priavte IPs that were obtained from Step 1 will be passed here in elb_1 & elb_2.
* Choose the CIDR which is used by your application server.
* Add the IP address of the loadbalancer attached to the application server. The IP address added must resonate with the CIDR of the webserver.

### Step 6: Creating ACP rules

The code below creates a rule to allows all outside traffic to the webserver.

ACP rules is a subset of ACP.

```python
acprule = fmcapi.AccessRules(fmc=fmc,acp_name=acp.name,name="To Web Server5",action="ALLOW",enabled=True)
acprule.source_zone(action="add", name=sz_outside.name)
acprule.destination_zone(action="add", name=sz_inside.name)
acprule.destination_network(action="add", name=app_lb1.name)
acprule.destination_network(action="add", name=app_lb2.name)
acprule.logEnd = False
acprule.post()
```
Using **fmcapi.AccessRules** we add source & destination zones and the destination network to make up the rule. 

We provide the name of security zones and name of the Load Balancer where needed, set *action=add* to add the respective fields.

### Step 7: Adding Static NAT

```python
nat = fmcapi.FTDNatPolicies(fmc=fmc, name="NAT Policy5")
nat.post()
#NAT Rule 1
manualnat1 = fmcapi.ManualNatRules(fmc=fmc)
manualnat1.natType = "STATIC"
manualnat1.original_source(elb1.name)
manualnat1.original_destination_port("HTTP")
manualnat1.translated_destination_port("HTTP")
manualnat1.translated_destination(app_lb1.name)
manualnat1.interfaceInOriginalDestination = True
manualnat1.interfaceInTranslatedSource = True
manualnat1.source_intf(name=sz_outside.name)
manualnat1.destination_intf(name=sz_inside.name)
manualnat1.nat_policy(name=nat.name)
manualnat1.enabled = True
manualnat1.post()
```

The NAT Rule above is for only one application server. 2nd NAT Rule will be the same with ```original_source``` as "elb2.name" & ```translated_destination``` as "app_lb1.name" 


### Step 8: Adding FTD as a device

```python
#Register 1st FTD
ftd1 = fmcapi.DeviceRecords(fmc=fmc)
ftd1.hostName = host1_ip
ftd1.regKey = reg_key
ftd1.acp(name="IAC-ACP")
ftd1.name = "ftd1"
ftd1.licensing(action="add", name="BASE")
ftd1.post(post_wait_time=300)
        
#Register 2nd FTD
ftd2 = fmcapi.DeviceRecords(fmc=fmc)
ftd2.hostName = host2_ip
ftd2.regKey = reg_key
ftd2.acp(name="IAC-ACP5")
ftd2.name = "ftd2"
ftd2.licensing(action="add", name="BASE")
ftd2.post(post_wait_time=300)
```
1. Provide the IP address of your deployed FTD for hostName field. Can be found in the AWS EC2 console.
2. Provide the same registration key used to create the FTD, make sure that this key must not have any special characters apart from '-'. 
3. Attach the ACP to the FTD .
4. **post_wait_time** takes values in seconds and will wait for that amount of time until the FTD is added.  

For interfaces of FTD the code snippet will be:
```python
#Interface of FTD-1
ftd1_g00 = fmcapi.PhysicalInterfaces(fmc=fmc, device_name=ftd1.name)
ftd1_g00.get(name="TenGigabitEthernet0/0")
ftd1_g00.enabled = True
ftd1_g00.ifname = "inside"
ftd1_g00.dhcp(True, 1)
ftd1_g00.sz(name="inside")
ftd1_g00.put()
ftd1_g01 = fmcapi.PhysicalInterfaces(fmc=fmc, device_name=ftd1.name)
ftd1_g01.get(name="TenGigabitEthernet0/1")
ftd1_g01.enabled = True
ftd1_g01.ifname = "outside"
ftd1_g01.dhcp(True, 1)
ftd1_g01.sz(name="outside")
ftd1_g01.put()
```

### Step 9: Adding static routes on FTD to Webserver

Adding a static default route for FTD
```python
#static route - ftd1
web_route1 = fmcapi.IPv4StaticRoutes(fmc=fmc, name="app_route1")
web_route1.device(device_name=ftd1.name)
web_route1.networks(action="add", networks=["app1"])
web_route1.gw(name=app_lb1.name)
web_route1.interfaceName = ftd1_g00.ifname
web_route1.metricValue = 1
web_route1.post()
        
#static route - ftd2
web_route2 = fmcapi.IPv4StaticRoutes(fmc=fmc, name="app_route2")
web_route2.device(device_name=ftd2.name)
web_route2.networks(action="add", networks=["app2"])
web_route2.gw(name=app_lb2.name)
web_route2.interfaceName = ftd2_g00.ifname
web_route2.metricValue = 1
web_route2.post()
```
All static routes can be created using **fmcapi.IPv4StaticRoutes**.  

### Step 10: Associate NAT policy with FTD.

```python
# Associate NAT policy with ftd1.
devices = [{"name": ftd1.name, "type": "device"}]
assign_nat_policy = fmcapi.PolicyAssignments(fmc=fmc)
assign_nat_policy.ftd_natpolicy(name=nat.name, devices=devices)
assign_nat_policy.post()

# Associate NAT policy with ftd2.
devices = [{"name": ftd2.name, "type": "device"}]
assign_nat_policy = fmcapi.PolicyAssignments(fmc=fmc)
assign_nat_policy.ftd_natpolicy(name=nat.name, devices=devices)
assign_nat_policy.post()
```

Use **fmcapi.PolicyAssignments** and pass the name of FTD and NAT Policy to connect them together.

### Final step: Running the Script

Open the terminal and *cd* into the folder which has your python script in it.

Run the file by typing the command. 

```
python3 fmc.py --addr <fmc ip> --username <username> --password <password> --elb1 <ELB IP 1> --elb2 <ELB IP 2>

```
(Ensure that you have python 3 installed on your system)

![script](/images/firewall_configuration/FMC_SCR_1.png)

In the terminal itself you would be able to see all the successful tasks and errors (if any). 

To ensure that everything took place as per your requirment open the FMC-UI on your browser and check things there. 

For eg: 

**ACP:**
![fmcapi_policy](/images/firewall_configuration/FMCAPI_POLICY.png) 

It also shows that this was created using fmcapi

**Devices:**
![device](/images/firewall_configuration/FMC_DEVICE_SCR.png)

Check the policy name attached to the FTD.

