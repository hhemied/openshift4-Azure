### Openshift4-Azure
========

Openshift4-Azure is the exact steps you need to deploy **Openshift 4.x** on the Microsoft Azure Cloud

Content
--------
[ ] Prepare the azure account to be used by the openshift installer
[ ] Prepare the machine will be used as the installer to use **az** [azure command line]
[ ] Use the AWS route53 as the our main dns
[ ] Prepare the azure dns zone
[ ] Install required tools to install openshift 4
[ ] Create **Infra** machineset with keeping it in the worker role
[ ] Create **Infra** machineset without being a part of worker role
[ ] 2nd day operations


#### Technical Steps:
----------------
1. Download the **az** cli from [microsoft.com](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-yum?view=azure-cli-latest)
2. Configure **az** cli.
  - Login to azure
    NOTE:
    : If your web browser don't open URLs automatically, please copy and paste the URL from the command output
    : then paste the code given by the command line.
    ```
     [root@bastion ~]# az login
     To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code C82UQ5T5E to authenticate.

     [
      {
        "cloudName": "AzureCloud",
        "homeTenantId": "xxxxxxxx",
        "id": "yyyyyyy",
        "isDefault": true,
        "managedByTenants": [],
        "name": "Free Trial",
        "state": "Enabled",
        "tenantId": "zzzzzzzz",
        "user": {
          "name": "email@outlook.com",
          "type": "user"
        }
      }
     ]
    ```
  - Check the access you have
    ```bash
    az account list
    # OR
    az account list --output table
    ```
