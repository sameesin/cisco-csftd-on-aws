---
title: "Creating your environment"
weight: 1
---

In order for you to succeed in this workshop, you will need to run through a few steps to properly setup and configure your environment. These steps will include provisioning some services, installing some tools, and downloading some dependencies as well. We will begin with AWS Cloud9. Technically, you should be able to complete many of the steps in these modules if you have a properly configured terminal. However, in order to avoid the "works on my machine" response you've surely experienced at some point in your career, we strongly encourage you to proceed with launching Cloud9.

**AWS Cloud9** is a cloud-based integrated development environment (IDE) that lets you write, run, and debug your code with just a browser. It includes a code editor, debugger, and terminal. Cloud9 comes prepackaged with essential tools for popular programming languages, including JavaScript, Python, PHP, and more, so you don’t need to install files or configure your development machine to start new projects.

## **<ins>Starting AWS Cloud9 IDE</ins>**

Your Cloud9 environment will have access to the same AWS resources as the user with which you logged into the AWS Management Console. We strongly recommend using Cloud9 to complete this workshop.

**Cloud9 works best with Chrome or Firefox, not Safari. It cannot be used from a tablet.**

**Step-by-step Instructions**

1. From the AWS Management Console, Select **Services** then select **Cloud9** under Developer Tools. 

![Step 4](/static/Images/getting_started/c9-step4.png)

2. Select **Create environment**.

3. Enter `Cisco-IAC-Cloud9` into **Name** and optionally provide a **Description**.

4. Select the Environment type as New EC2 Instance

![Step 5](/static/Images/getting_started/c9-step5_new.png)

5. Under New EC2 Instance Set the *Instance type* to **t2.micro (1 GiB RAM + 1 vCPU)**. Other settings can be left with default values

6. under Network Settings click on Create VPC, 
  - select Create VPC in the new tab
  - select the option to create VPC and subnets 
  - provide a name ```IAC-vpc``` and CIDR ```10.0.0.0/16``` for the VPC
  - provide a name ```mgmt``` and CIDR ```10.0.0.0/24``` for the subnet
  - click on Create VPC
- Back in the cloud9 settings page, select the created VPC and the created subnet 
- Leave all other defaults unchanged.

![Step 6](/static/Images/getting_started/c9-step6-b.png)

7. Click on Create.<br>
It will take a couple of minutes for your Cloud9 environment to be provisioned and prepared.

## <ins>**Setting up Cloud9 IDE**</ins>

1. Once ready, click on the open option. your IDE will open to a welcome screen. Below that, you should see a terminal prompt. Close the Welcome tab and drag up the terminal window to give yourself more space to work in. 

![Step 6a](/static/Images/getting_started/c9-step6a.png)

![Step 7](/static/Images/getting_started/c9-step7.png)

- You can run AWS CLI commands in here just like you would on your local computer. Remember for this workshop to run all commands within the Cloud9 terminal window rather than on your local computer.
- Keep your AWS Cloud9 IDE opened in a browser tab throughout this workshop.

2. Verify that your user is logged in by running the command `aws sts get-caller-identity`. Copy and paste the command into the Cloud9 terminal window. 

```console
aws sts get-caller-identity
```

- You'll see output indicating your account and user information:

```json
{
    "Account": "123456789012",
    "UserId": "AKIAIOSFODNN7EXAMPLE",
    "Arn": "arn:aws:iam::123456789012:user/Alice"
}
```
3. **Install Terraform on Cloud9**
If Terraform is already installed you can skip this step. To check if Terraform is installed run the following command.

```console
terraform --version
```

- Run the following commands on the terminal to install Terraform

```console
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform
```

- Check the Terraform version to confirm Terraform commands are working.

4. **Clone the source repository for this workshop**

Now we want to clone the repository that contains all the content and files you need to complete this workshop.

```bash
cd ~/environment && \
git clone https://github.com/CiscoDevnet/secure-firewall.git
```

5. **Create a Key Pair**

To create a key pair go to AWS Management console, EC2 > Network & Security > Key Pairs

>Note: KeyPair will be helpful in the scenario where we need to access an instance via SSH

![key_pair](/static/Images/getting_started/KEY-PAIR.png)

### Tips

Keep an open scratch pad in Cloud9 or a text editor on your local computer for notes. When the step-by-step directions tell you to note something such as an ID or Amazon Resource Name (ARN), copy and paste that into your scratch pad.

### Recap

- This is your unique AWS account for this workshop. It will expire after you finish today.

- Use the same region for the entirety of this workshop.

- Keep your [AWS Cloud9 IDE](#aws-cloud9-ide) opened in a browser tab

>Note: Click [here](../20_Getting_Started/2_Subscribe_to_FMC_and_FTD_images.md) if you are using a personal account to subscribe to Cisco FTD and FMC Images.