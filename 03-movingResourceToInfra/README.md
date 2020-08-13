### Moving resources to infrastructure MachineSets


#### Moving the Router

- Get the ingress controller configuration
```bash
oc get ingresscontroller default -n openshift-ingress-operator -o yaml
```

- Edit the ingress controller
| Append the following lines to the ingress controller
|   spec:
|    nodePlacement:
|      nodeSelector:
|        matchLabels:
|          node-role.kubernetes.io/infra: ""
```yaml
apiVersion: operator.openshift.io/v1
kind: IngressController
metadata:
  creationTimestamp: 2019-04-18T12:35:39Z
  finalizers:
  - ingresscontroller.operator.openshift.io/finalizer-ingresscontroller
  generation: 1
  name: default
  namespace: openshift-ingress-operator
  resourceVersion: "11341"
  selfLink: /apis/operator.openshift.io/v1/namespaces/openshift-ingress-operator/ingresscontrollers/default
  uid: 79509e05-61d6-11e9-bc55-02ce4781844a

# This part
spec:
  nodePlacement:
    nodeSelector:
      matchLabels:
        node-role.kubernetes.io/infra: ""
###
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: 2019-04-18T12:36:15Z
    status: "True"
    type: Available
  domain: apps.<cluster>.example.com
  endpointPublishingStrategy:
    type: LoadBalancerService
  selector: ingresscontroller.operator.openshift.io/deployment-ingresscontroller=default
```

- Ensure the pods are moving to the infra nodes
```bash
[root@bastion yamls]# oc get pod -n openshift-ingress -o wide
NAME                              READY   STATUS        RESTARTS   AGE    IP            NODE                                          NOMINATED NODE   READINESS GATES
router-default-5bc7ccb9f9-g2gx6   0/1     Running       0          15s    10.128.4.4    ocp44-klk2k-infra-germanywestcentral-pph59    <none>           <none>
router-default-5bc7ccb9f9-rsbst   1/1     Running       0          41s    10.129.4.4    ocp44-klk2k-infra-germanywestcentral-d4c9x    <none>           <none>
router-default-6dc96fb49-ktlsg    1/1     Terminating   0          103m   10.128.2.14   ocp44-klk2k-worker-germanywestcentral-sgvld   <none>           <none>
router-default-6dc96fb49-tcrsn    1/1     Running       0          104m   10.131.0.23   ocp44-klk2k-worker-germanywestcentral-jpk72   <none>           <none>
```

#### Moving the default Registry

- Get the current configuration
```bash
oc get config/cluster -o yaml
```

- Append this part to the configuratio
>  nodeSelector:
>    node-role.kubernetes.io/infra: ""
 ```yaml
 ...
 spec:
  nodeSelector:
    node-role.kubernetes.io/infra: ""
  logging: 2
  managementState: Managed
  proxy: {}
...

```

- Check pods are moving
```bash
[root@bastion yamls]# oc get pods -n openshift-image-registry -o wide
NAME                                               READY   STATUS        RESTARTS   AGE    IP            NODE                                          NOMINATED NODE   READINESS GATES
cluster-image-registry-operator-7f4d7f7cd5-tk7rg   2/2     Running       0          164m   10.130.0.23   ocp44-klk2k-master-2                          <none>           <none>
image-registry-75c5b56d8b-4mn9q                    0/1     Terminating   0          152m   10.128.2.6    ocp44-klk2k-worker-germanywestcentral-sgvld   <none>           <none>
image-registry-75c5b56d8b-xdzpb                    1/1     Terminating   0          160m   10.131.0.18   ocp44-klk2k-worker-germanywestcentral-jpk72   <none>           <none>
image-registry-7f86688d89-nk89b                    1/1     Running       0          5s     10.128.4.5    ocp44-klk2k-infra-germanywestcentral-pph59    <none>           <none>
image-registry-7f86688d89-x6lwn                    1/1     Running       0          17s    10.130.4.5    ocp44-klk2k-infra-germanywestcentral-kt8mt    <none>           <none>
node-ca-68nqf                                      1/1     Running       0          160m   10.128.0.35   ocp44-klk2k-master-1                          <none>           <none>
node-ca-bs6xl                                      1/1     Running       0          154m   10.129.2.2    ocp44-klk2k-worker-germanywestcentral-xh5z4   <none>           <none>
node-ca-cnmqg                                      1/1     Running       0          93m    10.128.4.3    ocp44-klk2k-infra-germanywestcentral-pph59    <none>           <none>
node-ca-dk7x4                                      1/1     Running       0          160m   10.129.0.23   ocp44-klk2k-master-0                          <none>           <none>
node-ca-jhzrg                                      1/1     Running       0          93m    10.129.4.2    ocp44-klk2k-infra-germanywestcentral-d4c9x    <none>           <none>
node-ca-lb748                                      1/1     Running       0          160m   10.130.0.30   ocp44-klk2k-master-2                          <none>           <none>
node-ca-lxhjs                                      1/1     Running       0          154m   10.128.2.2    ocp44-klk2k-worker-germanywestcentral-sgvld   <none>           <none>
node-ca-n278c                                      1/1     Running       0          93m    10.130.4.2    ocp44-klk2k-infra-germanywestcentral-kt8mt    <none>           <none>
node-ca-r2hg9                                      1/1     Running       0          155m   10.131.0.3    ocp44-klk2k-worker-germanywestcentral-jpk72   <none>           <none>
```
