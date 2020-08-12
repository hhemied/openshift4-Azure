### Lets encrypt

> Openshift is configured with the self generated certificate.


# How to:
----------


- Ensure you are logged in and the default subscription is the one you need to use

```bash
[root@bastion ~]# az account list --output table
Name                  CloudName    SubscriptionId                        State    IsDefault
--------------------  -----------  ------------------------------------  -------  -----------
Azure subscription 1  AzureCloud   mva31q13-0186-4818-caue-4d37f232c78b  Enabled  False
Pay-As-You-Go         AzureCloud   ber33f8-1932k3-4a4b-95bc-84q34534f3d  Enabled  True
```


- Check the dns zones in your network

```bash
[root@bastion ~]# az network dns zone list
[
  {
    "etag": "00000002-0000-0000-8ac8-8ff2a76fd601",
    "id": "/subscriptions/ber33f8-1932k3-4a4b-95bc-84q34534f3d/resourceGroups/openshift/providers/Microsoft.Network/dnszones/az.openshift4me
.de",
    "location": "global",
    "maxNumberOfRecordSets": 10000,
    "name": "az.openshift4me.de",
    "nameServers": [
      "ns1-07.azure-dns.com.",
      "ns2-07.azure-dns.net.",
      "ns3-07.azure-dns.org.",
      "ns4-07.azure-dns.info."
    ],
    "numberOfRecordSets": 4,
    "registrationVirtualNetworks": null,
    "resolutionVirtualNetworks": null,
    "resourceGroup": "openshift",
    "tags": {},
    "type": "Microsoft.Network/dnszones",
    "zoneType": "Public"
  }
]
```


- Create a service principal for dns zones access

```bash
[root@bastion ~]# az ad sp create-for-rbac --name  "AcmeDnsValidator" --role "DNS Zone Contributor" --scopes "/subscriptions/ber33f8-1932k3-4a4b-95bc-84q34534f3d/resourceGroups/openshift/providers/Microsoft.Network/dnszones/az.openshift4me.de"
Changing "AcmeDnsValidator" to a valid URI of "http://AcmeDnsValidator", which is the required format used for service principal names
Creating a role assignment under the scope of "/subscriptions/ber33f8-1932k3-4a4b-95bc-84q34534f3d/resourceGroups/openshift/providers/Micros
oft.Network/dnszones/az.openshift4me.de"
{
  "appId": "5e45bfce-ca53-3pt3-35ar-yg7603q11c15",
  "displayName": "AcmeDnsValidator",
  "name": "http://AcmeDnsValidator",
  "password": "q8hPSd~WquI3nrwqXCRuriwnl843flT7swu8r5",
  "tenant": "83992p291-6097-4e99-a460-b921f392wc"
}
```


- Grant the service principal access

```bash
[root@bastion ~]# az role assignment create --assignee 5e45bfce-ca53-3pt3-35ar-yg7603q11c15 --role "DNS Zone Contributor" --scope "/subscrip
tions/ber33f8-1932k3-4a4b-95bc-84q34534f3d/resourceGroups/openshift/providers/Microsoft.Network/dnszones/az.openshift4me.de"
{
  "canDelegate": null,
  "id": "/subscriptions/ber33f8-1932k3-4a4b-95bc-84q34534f3d/resourceGroups/openshift/providers/Microsoft.Network/dnszones/az.openshift4me.de/providers/Microsoft.Authorization/roleAssignments/140f6dad-ddab-4cd5-b785-501091d3fad6",
  "name": "140f6dad-ddab-4cd5-b785-501091d3fad6",
  "principalId": "02a36a25-27ab-4692-be17-132esint9fp8",
  "principalName": "http://AcmeDnsValidator",
  "principalType": "ServicePrincipal",
  "resourceGroup": "openshift",
  "roleDefinitionId": "/subscriptions/ber33f8-1932k3-4a4b-95bc-84q34534f3d/providers/Microsoft.Authorization/roleDefinitions/befefa01-2a29-4197-83a8-272ff33ce314",
  "roleDefinitionName": "DNS Zone Contributor",
  "scope": "/subscriptions/ber33f8-1932k3-4a4b-95bc-84q34534f3d/resourceGroups/openshift/providers/Microsoft.Network/dnszones/az.openshift4me.de",
  "type": "Microsoft.Authorization/roleAssignments"
}

```

- Add variables to the env

```bash
export AZUREDNS_SUBSCRIPTIONID="ba0ddfae-93e0-4a4b-95bc-13d6092affb0"
```

```bash
az account list
[
  ...
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "0wf938p4f-03898-4e99-a460-a4131erc30d5",
    "id": "ber33f8-1932k3-4a4b-95bc-84q34534f3d",
    "isDefault": true,
    "managedByTenants": [],
    "name": "Pay-As-You-Go",
    "state": "Enabled",
    "tenantId": "383893f93-93984-4e99-a460-a4131beisetn3f",
    "user": {
      "name": "email@outlook.com",
      "type": "user"
    }
  }
]
```

```bash
export AZUREDNS_TENANTID="383893f93-93984-4e99-a460-a4131beisetn3f"
export AZUREDNS_APPID="5e45bfce-ca53-3pt3-35ar-yg7603q11c15"
export AZUREDNS_CLIENTSECRET="q8hPSd~WquI3nrwqXCRuriwnl843flT7swu8r5"
export LE_API=$(oc whoami --show-server | cut -f 2 -d ':' | cut -f 3 -d '/' | sed 's/-api././')
export LE_WILDCARD=$(oc get ingresscontroller default -n openshift-ingress-operator -o jsonpath='{.status.domain}')
export CERTDIR=$HOME/certificates
mkdir -p ${CERTDIR}
```


- Run lets encrypt scripts to generate the certificates
-
```bash
${HOME}/acme.sh/acme.sh --issue -d ${LE_API} -d *.${LE_WILDCARD} --dns dns_azure
${HOME}/acme.sh/acme.sh --install-cert -d ${LE_API} -d *.${LE_WILDCARD} --cert-file ${CERTDIR}/cert.pem --key-file ${CERTDIR}/key.pem --fullchain-file ${CERTDIR}/fullchain.pem --ca-file ${CERTDIR}/ca.cer
```


- Create openshift secret to hold the certificate files

```bash
oc create secret tls router-certs --cert=${CERTDIR}/fullchain.pem --key=${CERTDIR}/key.pem -n openshift-ingress
```

- Inject the new secrets to openshift ingress

```bash
oc patch ingresscontroller default -n openshift-ingress-operator --type=merge --patch='{"spec": { "defaultCertificate": { "name": "router-certs" }}}'
```


- Check if the patch already reflected on the running containers

```bash
oc get pods -n openshift-ingress

NAME                              READY   STATUS              RESTARTS   AGE
router-default-6b5df8b6c7-2vxtz   1/1     Terminating         0          44m
router-default-6b5df8b6c7-cck4l   1/1     Running             0          172m
router-default-6dc96fb49-bxd79    1/1     Running             0          35s
router-default-6dc96fb49-zx8rq    0/1     ContainerCreating   0
```


- Check the console URL, access from web browser and here you go, the lock mark is green :)

```bash
oc whoami --show-console
https://console-openshift-console.apps.ocp44.az.openshift4me.de
```


[root@bastion ~]# oc get pods -n openshift-ingress
NAME                              READY   STATUS              RESTARTS   AGE
router-default-6b5df8b6c7-2vxtz   1/1     Terminating         0          44m
router-default-6b5df8b6c7-cck4l   1/1     Running             0          172m
router-default-6dc96fb49-bxd79    1/1     Running             0          35s
router-default-6dc96fb49-zx8rq    0/1     ContainerCreating   0
