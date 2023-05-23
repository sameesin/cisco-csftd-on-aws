---
title: "Testing Traffic"
weight: 10
---

To test the traffic, ensure that all the resources including FMC are running successfully on AWS.

>Note: You might have to wait upto 30 min, for FMC to be up and running.   


## <ins>**Step 1 :**</ins> Platform Settings - SSH Access
After executing the python script, log-in back into the FMC UI.

Go to Devices > Platform Settings and add a new policy.

![dev_PS](/static/Images/testing-traffic/DEV-PS.png)

Make sure that you add both devices to the new policy. 

![policy](/static/Images/testing-traffic/PLATFORM_SETT.png)

We need to give SSH-Access to our FTD, so that health check doesn't fail.

![ssh_acc](/static/Images/testing-traffic/SSH_ACCESS.png)

![ps_SSH](/static/Images/testing-traffic/PS_SSH.png)

Save all the new conifguration before exiting the platform settings page.

![save](/static/Images/testing-traffic/SAVE.png)

## <ins>**Step 2 :**</ins> Deploy Configuration
Now deploy both the FTDs to complete the network topology.

![Deploy](/static/Images/testing-traffic/DEPLOY.png)

## <ins>**Step 3 :**</ins> Test connectivity
After the deployment is complete check on the AWS Console that the target group instances are in **healthy** state.<br>
Go to your AWS console > EC2 > Load Balancer. 

Find the ***external loadbalancer*** and look for it's DNS Name.

![DNS](/static/Images/testing-traffic/ELB-DNS.png)

Copy the DNS name and paste it on a new tab of your browser.

![APP_Server](/static/Images/testing-traffic/APP-SERVER.png)

If everything goes right, the browser should be able to reach our application server (Apache2 is used in our case). 

**This step concludes testing traffic**