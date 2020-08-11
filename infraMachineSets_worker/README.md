### Creating Infra Machine Set

- This will build **infra** machine set, the nodes will be working as worker with special label as infra
- Can be used as **infra** node to host infra workload, but also will host normal workload automatically unless you add toleration to exclude it from being worker.

Steps
----
- get the infrastructure id|name

  ```
  [root@bastion ocp443]# oc get -o jsonpath='{.status.infrastructureName}{"\n"}' infrastructure cluster
  ocp44-n5n7k
  ```
- cat infraMachineSet.yaml
  ```
  apiVersion: machine.openshift.io/v1beta1
  kind: MachineSet
  metadata:
    labels:
      machine.openshift.io/cluster-api-cluster: [infrastructure_id]
      machine.openshift.io/cluster-api-machine-role: infra
      machine.openshift.io/cluster-api-machine-type: infra
    name: [infrastructure_id]-infra-germanywestcentral
    namespace: openshift-machine-api
  spec:
    replicas: 0
    selector:
      matchLabels:
        machine.openshift.io/cluster-api-cluster: [infrastructure_id]
        machine.openshift.io/cluster-api-machineset: [infrastructure_id]-infra-germanywestcentral
    template:
      metadata:
        creationTimestamp: null
        labels:
          machine.openshift.io/cluster-api-cluster: [infrastructure_id]
          machine.openshift.io/cluster-api-machine-role: infra
          machine.openshift.io/cluster-api-machine-type: infra
          machine.openshift.io/cluster-api-machineset: [infrastructure_id]-infra-germanywestcentral
      spec:
        metadata:
          creationTimestamp: null
          labels:
            node-role.kubernetes.io/infra: ""
        providerSpec:
          value:
            apiVersion: azureproviderconfig.openshift.io/v1beta1
            credentialsSecret:
              name: azure-cloud-credentials
              namespace: openshift-machine-api
            image:
              offer: ""
              publisher: ""
              resourceID: /resourceGroups/[infrastructure_id]-rg/providers/Microsoft.Compute/images/[infrastructure_id]
              sku: ""
              version: ""
            internalLoadBalancer: ""
            kind: AzureMachineProviderSpec
            location: germanywestcentral
            managedIdentity: [infrastructure_id]-identity
            metadata:
              creationTimestamp: null
            natRule: null
            networkResourceGroup: ""
            osDisk:
              diskSizeGB: 128
              managedDisk:
                storageAccountType: Premium_LRS
              osType: Linux
              publicIP: false
            publicLoadBalancer: ""
            resourceGroup: [infrastructure_id]-rg
            sshPrivateKey: ""
            sshPublicKey: ""
            subnet: [infrastructure_id]-worker-subnet
            userDataSecret:
              name: worker-user-data
            vmSize: Standard_D2s_v3
            vnet: [infrastructure_id]-vnet
  ```

- Create the machine set
  ```
  [root@bastion ocp443]# oc create -f infraMachineSet.yaml
  ```
