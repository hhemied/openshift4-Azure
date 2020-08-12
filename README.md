### Openshift4-Azure
========

Openshift4-Azure is the exact steps you need to deploy **Openshift 4.x** on the Microsoft Azure Cloud

Content
--------
- [x] Prepare the azure account to be used by the openshift installer
- [x] Prepare the machine will be used as the installer to use **az** [azure command line]
- [x] Use the AWS route53 as the our main dns
- [x] Prepare the azure dns zone
- [x] Install required tools to install openshift 4
- [x] Create **Infra** machineset with keeping it in the worker role
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
    ```bash
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
    ```bash
    [root@bastion ~] az account set --subscription "Pay-As-You-Go"
    ```

    ```bash
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

    ```bash
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

    ```bash
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

    ```bash
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

    ```bash
    [root@bastion ocp44]# az ad app permission add --id <appId> --api 00000002-0000-0000-c000-000000000000 --api-permissions 824c81eb-e3f8-4ee6-8f6d-de7f50d565b7=Role
    Invoking "az ad app permission grant --id appId --api 00000002-0000-0000-c000-000000000000" is needed to make the change effective
    ```

    ```bash
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
---------------------------

- ensure you have enough quota for the cluster otherwise the installer will fail

  ```bash
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

### DNS Configuration

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

![AWS Route53](https://github.com/hhemied/openshift4-Azure/raw/master/azure_dns_zone.png?raw=true)


  - On the bastion node
    - Open https://cloud.redhat.com/openshift/install/azure/installer-provisioned
    - Download all the required tools [openshit-install, Pull secret, openshift client tools]
    - Add the binary files to your PATH
    - Create a directory for the installation files

      ```bash
      # for the first time will ask for the data we had, appId we created, app principal Id and the app password

      [root@bastion ocp443]# openshift-install create cluster
      ? SSH Public Key /root/.ssh/id_rsa.pub
      ? Platform azure
      INFO Credentials loaded from file "/root/.azure/osServicePrincipal.json"
      ? Region germanywestcentral
      ? Base Domain  [Use arrows to move, enter to select, type to filter, ? for more help]
      > az.openshift4me.de
      ```
    - Once the installer completes the process the message for kube admin and kube config will appear to be used by the **oc** cli
    - Now we can

      ```bash
      [root@bastion ocp4]# openshift-install create cluster
      INFO Credentials loaded from file "/root/.azure/osServicePrincipal.json"
      INFO Consuming Install Config from target directory
      INFO Creating infrastructure resources...
      INFO Waiting up to 20m0s for the Kubernetes API at https://api.ocp44.az.openshift4me.de:6443...
      INFO API v1.17.1+b83bc57 up
      INFO Waiting up to 40m0s for bootstrapping to complete...
      INFO Destroying the bootstrap resources...
      INFO Waiting up to 30m0s for the cluster at https://api.ocp44.az.openshift4me.de:6443 to initialize...
      INFO Waiting up to 10m0s for the openshift-console route to be created...
      INFO Install complete!
      INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/root/ocp4/auth/kubeconfig'
      INFO Access the OpenShift web-console here: https://console-openshift-console.apps.ocp44.az.openshift4me.de
      INFO Login to the console with user: kubeadmin, password: XXXXXXXXXX

      [root@bastion ocp443]# oc whoami
      system:admin
      ```

      ```bash
      [root@bastion ocp443]# oc get nodes
      NAME                                          STATUS   ROLES    AGE     VERSION
      ocp44-n5n7k-master-0                          Ready    master   10h     v1.17.1+b83bc57
      ocp44-n5n7k-master-1                          Ready    master   10h     v1.17.1+b83bc57
      ocp44-n5n7k-master-2                          Ready    master   10h     v1.17.1+b83bc57
      ocp44-n5n7k-worker-germanywestcentral-2bdnt   Ready    worker   6h28m   v1.17.1+b83bc57
      ocp44-n5n7k-worker-germanywestcentral-fbvlv   Ready    worker   10h     v1.17.1+b83bc57
      ocp44-n5n7k-worker-germanywestcentral-zjdhf   Ready    worker   10h     v1.17.1+b83bc57
      ```

      ```bash
      [root@bastion ocp443]# oc get machines -n openshift-machine-api
      NAME                                          PHASE     TYPE              REGION               ZONE   AGE
      ocp44-n5n7k-master-0                          Running   Standard_D8s_v3   germanywestcentral          11h
      ocp44-n5n7k-master-1                          Running   Standard_D8s_v3   germanywestcentral          11h
      ocp44-n5n7k-master-2                          Running   Standard_D8s_v3   germanywestcentral          11h
      ocp44-n5n7k-worker-germanywestcentral-2bdnt   Running   Standard_D2s_v3   germanywestcentral          6h37m
      ocp44-n5n7k-worker-germanywestcentral-fbvlv   Running   Standard_D2s_v3   germanywestcentral          10h
      ocp44-n5n7k-worker-germanywestcentral-zjdhf   Running   Standard_D2s_v3   germanywestcentral          10h
      ```

      ```bash
      [root@bastion ocp443]# oc get co
      NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
      authentication                             4.4.16    True        False         False      10h
      cloud-credential                           4.4.16    True        False         False      11h
      cluster-autoscaler                         4.4.16    True        False         False      10h
      console                                    4.4.16    True        False         False      10h
      csi-snapshot-controller                    4.4.16    True        False         False      10h
      dns                                        4.4.16    True        False         False      10h
      etcd                                       4.4.16    True        False         False      10h
      image-registry                             4.4.16    True        False         False      10h
      ingress                                    4.4.16    True        False         False      6h37m
      insights                                   4.4.16    True        False         False      10h
      kube-apiserver                             4.4.16    True        False         False      10h
      kube-controller-manager                    4.4.16    True        False         False      10h
      kube-scheduler                             4.4.16    True        False         False      10h
      kube-storage-version-migrator              4.4.16    True        False         False      10h
      machine-api                                4.4.16    True        False         False      10h
      machine-config                             4.4.16    True        False         False      10h
      marketplace                                4.4.16    True        False         False      10h
      monitoring                                 4.4.16    True        False         False      10h
      network                                    4.4.16    True        False         False      10h
      node-tuning                                4.4.16    True        False         False      10h
      openshift-apiserver                        4.4.16    True        False         False      10h
      openshift-controller-manager               4.4.16    True        False         False      10h
      openshift-samples                          4.4.16    True        False         False      10h
      operator-lifecycle-manager                 4.4.16    True        False         False      10h
      operator-lifecycle-manager-catalog         4.4.16    True        False         False      10h
      operator-lifecycle-manager-packageserver   4.4.16    True        False         False      10h
      service-ca                                 4.4.16    True        False         False      10h
      service-catalog-apiserver                  4.4.16    True        False         False      10h
      service-catalog-controller-manager         4.4.16    True        False         False      10h
      storage                                    4.4.16    True        False         False      10h
      ```
