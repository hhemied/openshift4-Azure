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
  [root@bastion ocp443]# oc create -f infraMachineSet.yaml -n openshift-machine-api
  ```

- Scale up the machine set by the number you need

  ```
  [root@bastion ocp443]# oc scale machinesets.machine.openshift.io ocp44-n5n7k-infra-germanywestcentral -n openshift-machine-api --replicas=1
  machineset.machine.openshift.io/ocp44-n5n7k-infra-germanywestcentral scaled
  ```

- Check the machines

  ```
  [root@bastion ocp443]# oc get machines -n openshift-machine-api
  NAME                                          PHASE          TYPE              REGION               ZONE   AGE
  ocp44-n5n7k-infra-germanywestcentral-gcfkf    Provisioning   Standard_D2s_v3   germanywestcentral          59s
  ocp44-n5n7k-master-0                          Running        Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-master-1                          Running        Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-master-2                          Running        Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-worker-germanywestcentral-2bdnt   Running        Standard_D2s_v3   germanywestcentral          7h4m
  ocp44-n5n7k-worker-germanywestcentral-fbvlv   Running        Standard_D2s_v3   germanywestcentral          11h
  ocp44-n5n7k-worker-germanywestcentral-zjdhf   Running        Standard_D2s_v3   germanywestcentral          11h
  ```

  ```
  [root@bastion ocp443]# oc get machines -n openshift-machine-api
  NAME                                          PHASE     TYPE              REGION               ZONE   AGE
  ocp44-n5n7k-infra-germanywestcentral-gcfkf    Running   Standard_D2s_v3   germanywestcentral          5m4s
  ocp44-n5n7k-master-0                          Running   Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-master-1                          Running   Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-master-2                          Running   Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-worker-germanywestcentral-2bdnt   Running   Standard_D2s_v3   germanywestcentral          7h8m
  ocp44-n5n7k-worker-germanywestcentral-fbvlv   Running   Standard_D2s_v3   germanywestcentral          11h
  ocp44-n5n7k-worker-germanywestcentral-zjdhf   Running   Standard_D2s_v3   germanywestcentral          11h
  ```

  > you need to wait untill the machine becomes in **running** phase


- Finally check the cluster nodes state
  ```
  [root@bastion ocp443]# oc get nodes
  NAME                                          STATUS   ROLES          AGE     VERSION
  ocp44-n5n7k-infra-germanywestcentral-gcfkf    Ready    infra,worker   3m27s   v1.17.1+b83bc57
  ocp44-n5n7k-master-0                          Ready    master         11h     v1.17.1+b83bc57
  ocp44-n5n7k-master-1                          Ready    master         11h     v1.17.1+b83bc57
  ocp44-n5n7k-master-2                          Ready    master         11h     v1.17.1+b83bc57
  ocp44-n5n7k-worker-germanywestcentral-2bdnt   Ready    worker         7h5m    v1.17.1+b83bc57
  ocp44-n5n7k-worker-germanywestcentral-fbvlv   Ready    worker         11h     v1.17.1+b83bc57
  ocp44-n5n7k-worker-germanywestcentral-zjdhf   Ready    worker         11h     v1.17.1+b83bc57
  ```

- Scale down the machine set to 0

  ```
  [root@bastion ocp443]# oc scale machinesets.machine.openshift.io ocp44-n5n7k-infra-germanywestcentral -n openshift-machine-api --replicas=0
  machineset.machine.openshift.io/ocp44-n5n7k-infra-germanywestcentral scaled
  ```

  ```
  [root@bastion ocp443]# oc get machines -n openshift-machine-api
  NAME                                          PHASE      TYPE              REGION               ZONE   AGE
  ocp44-n5n7k-infra-germanywestcentral-gcfkf    Deleting   Standard_D2s_v3   germanywestcentral          10m
  ocp44-n5n7k-master-0                          Running    Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-master-1                          Running    Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-master-2                          Running    Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-worker-germanywestcentral-2bdnt   Running    Standard_D2s_v3   germanywestcentral          7h13m
  ocp44-n5n7k-worker-germanywestcentral-fbvlv   Running    Standard_D2s_v3   germanywestcentral          11h
  ocp44-n5n7k-worker-germanywestcentral-zjdhf   Running    Standard_D2s_v3   germanywestcentral          11h
  ```

  ```
  [root@bastion ocp443]# oc get machines -n openshift-machine-api
  NAME                                          PHASE     TYPE              REGION               ZONE   AGE
  ocp44-n5n7k-master-0                          Running   Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-master-1                          Running   Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-master-2                          Running   Standard_D8s_v3   germanywestcentral          11h
  ocp44-n5n7k-worker-germanywestcentral-2bdnt   Running   Standard_D2s_v3   germanywestcentral          7h16m
  ocp44-n5n7k-worker-germanywestcentral-fbvlv   Running   Standard_D2s_v3   germanywestcentral          11h
  ocp44-n5n7k-worker-germanywestcentral-zjdhf   Running   Standard_D2s_v3   germanywestcentral          11h
  ```

  ```
  [root@bastion ocp443]# oc get nodes
  NAME                                          STATUS   ROLES    AGE     VERSION
  ocp44-n5n7k-master-0                          Ready    master   11h     v1.17.1+b83bc57
  ocp44-n5n7k-master-1                          Ready    master   11h     v1.17.1+b83bc57
  ocp44-n5n7k-master-2                          Ready    master   11h     v1.17.1+b83bc57
  ocp44-n5n7k-worker-germanywestcentral-2bdnt   Ready    worker   7h10m   v1.17.1+b83bc57
  ocp44-n5n7k-worker-germanywestcentral-fbvlv   Ready    worker   11h     v1.17.1+b83bc57
  ocp44-n5n7k-worker-germanywestcentral-zjdhf   Ready    worker   11h     v1.17.1+b83bc57
  ```
