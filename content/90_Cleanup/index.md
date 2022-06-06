---
title: "Cleanup"
weight: 90
---

::alert[In order to prevent charges to your account we recommend cleaning up the infrastructure that was created. If you plan to keep things running so you can examine the workshop a bit more, please remember to do the cleanup when you are done.]

>Note: ```terraform destroy``` will only destroy resources that are created & managed by terraform itself. 

**<ins>terrafrom destroy</ins>**
  
To destroy the whole configuration, cd into the folder from where you deployed the plan and then   
- Run *terraform destroy* to terminate and delete all the resources created.

```console
terraform destroy
```

![delete](/static/images/clean_up/DESTROY.png)

**<ins>Delete Cloud9</ins>**

Navigate to AWS Cloud9 > Your Environments

Select the environment you created in the getting started section

Click on Delete

![cloud9_delete](/static/images/clean_up/cloud9_delete.png)

::alert[Resources like VPC and KeyPair that were created manually, needs to be destoryed in a similar manner via the AWS console.]

