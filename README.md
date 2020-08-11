### Openshift4-Azure
========

Openshift4-Azure is the exact steps you need to deploy **Openshift 4.x** on the Microsoft Azure Cloud

Content
--------
- [x] Prepare the azure account to be used by the openshift installer
- [x] Prepare the machine will be used as the installer to use **az** [azure command line]
- [ ] Use the AWS route53 as the our main dns
- [ ] Prepare the azure dns zone
- [ ] Install required tools to install openshift 4
- [ ] Create **Infra** machineset with keeping it in the worker role
- [ ] Create **Infra** machineset without being a part of worker role
- [ ] 2nd day operations


#### Technical Steps:
----------------
1. Download the **az** cli from [microsoft.com](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-yum?view=azure-cli-latest)
2. Configure **az** cli.
  - Login to azure
    NOTE
    > If your web browser don't open URLs automatically, please copy and paste the URL from the command output
      then paste the code given by the command line.
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
    Note:
    > If your new subscription is not yet reflected on the command line, try to relogin.
    > Be sure to subscripe to the pay-as-you-go as the default subscription is not enough for openshift cluster

  - Change the default subscription to the pay-as-you-go
    ```
    [root@bastion ~] az account set --subscription "Pay-As-You-Go"
    ```

    ```
    [root@bastion ~]# az account show
    {
      "environmentName": "AzureCloud",
      "homeTenantId": "17235b28-6097-4e99-a460-a4131bea18d5",
      "id": "ba0ddfae-93e0-4a4b-95bc-13d6092affb0",
      "isDefault": true,
      "managedByTenants": [],
      "name": "Pay-As-You-Go",
      "state": "Enabled",
      "tenantId": "17235b28-6097-4e99-a460-a4131bea18d5",
      "user": {
        "name": "hhemied@outlook.com",
        "type": "user"
      }
    }
    ```
  - Create the service principal client id

