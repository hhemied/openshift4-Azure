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
      "homeTenantId": "HomeID",
      "id": "ID",
      "isDefault": true,
      "managedByTenants": [],
      "name": "Pay-As-You-Go",
      "state": "Enabled",
      "tenantId": "tenantID",
      "user": {
        "name": "email@outlook.com",
        "type": "user"
      }
    }
    ```
  - Create the service principal client id
    [Microsoft Docs](https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azps-4.5.0&viewFallbackFrom=azps-2.5.0)

    ```
    [root@bastion ocp44]# az ad sp create-for-rbac --name openshift4-sp
    Changing "openshift4-sp" to a valid URI of "http://openshift4-sp", which is the required format used for service principal names
    Creating a role assignment under the scope of "/subscriptions/ba0ddfae-93e0-4a4b-95bc-13d6092affb0"
      Retrying role assignment creation: 1/36
    {
      "appId": "service principal client id",
      "displayName": "openshift4-sp",
      "name": "http://openshift4-sp",
      "password": "service principal client id secret",
      "tenant": "tenantID"
    }
    ```
    ```
    [root@bastion ocp44]# az role assignment create --assignee <appId> --role Contributor
    {
      "canDelegate": null,
      "id": "/subscriptions/id/providers/Microsoft.Authorization/roleAssignments/xxx",
      "name": "name",
      "principalId": "principalId",
      "principalName": "http://openshift4-sp",
      "principalType": "ServicePrincipal",
      "roleDefinitionId": "/subscriptions/id/providers/Microsoft.Authorization/roleDefinitions/xxx",
      "roleDefinitionName": "Contributor",
      "scope": "/subscriptions/id",
      "type": "Microsoft.Authorization/roleAssignments"
    }
    ```

    ```
    [root@bastion ocp44]# az role assignment create --assignee <appId> --role "User Access Administrator"
    {
      "canDelegate": null,
      "id": "/subscriptions/id/providers/Microsoft.Authorization/roleAssignments/xxx",
      "name": "name",
      "principalId": "principalId",
      "principalType": "ServicePrincipal",
      "roleDefinitionId": "/subscriptions/id/providers/Microsoft.Authorization/roleDefinitions/xxx",
      "scope": "/subscriptions/id",
      "type": "Microsoft.Authorization/roleAssignments"
    }
    ```
  - Assign App Permission
    [Microsoft Docs](https://docs.microsoft.com/en-gb/archive/blogs/aaddevsup/guid-table-for-windows-azure-active-directory-permissions)

    ```
    [root@bastion ocp44]# az ad app permission add --id <appId> --api 00000002-0000-0000-c000-000000000000 --api-permissions 824c81eb-e3f8-4ee6-8f6d-de7f50d565b7=Role
    Invoking "az ad app permission grant --id appId --api 00000002-0000-0000-c000-000000000000" is needed to make the change effective
    ```
    ```
    [root@bastion ocp44]# az ad app permission grant --id <appId> --api 00000002-0000-0000-c000-000000000000
    {
      "clientId": "clientId",
      "consentType": "AllPrincipals",
      "expiryTime": "2021-08-08T19:44:37.054595",
      "objectId": "Th0YK9E7KUqWSRx1QBeISEb3AOvK4DpBnDz",
      "odata.metadata": "https://graph.windows.net/17235b28-6097-4q39-3123-a4131bea18d5/$metadata#oauth2PermissionGrants/@Element",
      "odatatype": null,
      "principalId": null,
      "resourceId": "eb00f746-e0ca-413a-9c3c-efw23213f",
      "scope": "user_impersonation",
      "startTime": "2020-08-08T19:44:37.054595"
    }
    ```
##### Important note here
    - ensure you have enough quota for the cluster otherwise the installer will fail
      ```
      [root@bastion ocp44]# az vm list-usage --location 'Germany West Central' --output table
      Name                               CurrentValue    Limit
      ---------------------------------  --------------  -------
      Availability Sets                  0               2500
      Total Regional vCPUs               0               12
      Virtual Machines                   0               25000
      Virtual Machine Scale Sets         0               2500
      Dedicated vCPUs                    0               3000
      Total Regional Low-priority vCPUs  0               10
      ...
      Standard Dv3 Family vCPUs          0               12 # this is the most important part
      ```

#### DNS Configuration
Azure
-----
- Go to https://portal.azure.com/
- search for DNS Zones
- Click [Add]
- Choose the pay-as-you-go subscription, and the name should be a subdomain from the domain you own in AWS route53
- Click [Review + create]

![azure dns zone](https://github.com/hhemied/openshift4-Azure/raw/master/aws_route53.png?raw=true)

AWS route53
- Go to https://aws.amazon.com/
- My Account -> AWS Management Console
- Click [Route 53]
- Click [Hosted Zones]
- Click the zone you have
- Click [Create Record Set]
- Name of the record should be the same as the one we created in Azure dns zones
- Type should be NS
- Value should be the name servers as mentioned in the previous pic.

[AWS Route53](https://github.com/hhemied/openshift4-Azure/raw/master/azure_dns_zone.png?raw=true)



