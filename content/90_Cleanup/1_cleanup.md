---
title: "Cleanup"
weight: 10
---

::alert[In order to prevent charges to your account we recommend cleaning up the infrastructure that was created. If you plan to keep things running so you can examine the workshop a bit more, please remember to do the cleanup when you are done.]

>Note: ```terraform destroy``` will only destroy resources that are created & managed by terraform itself. 

**<ins>terrafrom destroy</ins>**
  
To destroy the whole configuration, cd into the folder from where you deployed the AWS resources plan (Development folder) and then  
- Run *terraform destroy* to terminate and delete all the resources created.

```console
terraform destroy
```

![delete](/static/Images/clean_up/DESTROY.png)

Resources like VPC and KeyPair, if created manually, needs to be destoryed in a similar manner via the console.