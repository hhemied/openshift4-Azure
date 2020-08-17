### Cluster Upgrade



- Check the current version
```bash
oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.5.5     True        False         3d13h   Cluster version is 4.5.5
```

#### Minor Version
For example upgrading from 4.4.8 to 4.4.13
---------
> Can also be done from the cluster configuration in the web console


##### From CLI

- Can check there is any update in the channel
```bash
oc adm upgrade
```

```bash
oc adm upgrade to [the new version]
```

#### Major Version
For example upgrading from 4.4 to 4.5
------
> Can also be done from web console by changing the upgrade channel to stable 4.5

##### From CLI

- Change the upgrade channel in the cluster version configuration

```bash
oc edit clusterversion
```
```yaml
apiVersion: config.openshift.io/v1
kind: ClusterVersion
metadata:
  creationTimestamp: "2020-08-13T08:05:22Z"
  generation: 5
  name: version
  resourceVersion: "1611234"
  selfLink: /apis/config.openshift.io/v1/clusterversions/version
  uid: 866b7164-2b6b-418b-83f3-303e975662a9
spec:
  channel: stable-4.4   ### <- change this part from stable-4.4 to stable-4.5
  clusterID: 6789ea61-ad9f-478f-ae7c-c58978d68596
  desiredUpdate:
    force: false
    image: quay.io/openshift-release-dev/ocp-release@sha256:a58573e1c92f5258219022ec104ec254ded0a70370ee8ed2aceea52525639bd4
    version: 4.5.5
  upstream: https://api.openshift.com/api/upgrades_info/v1/graph
status:
  availableUpdates: null
```

- Check the cluster operators if anything is failing
```bash
oc get co

NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.5.5     True        False         False      4d1h
cloud-credential                           4.5.5     True        False         False      4d2h
cluster-autoscaler                         4.5.5     True        False         False      4d2h
config-operator                            4.5.5     True        False         False      3d14h
console                                    4.5.5     True        False         False      3d13h
csi-snapshot-controller                    4.5.5     True        False         False      43h
dns                                        4.5.5     True        False         False      4d2h
etcd                                       4.5.5     True        False         False      4d2h
image-registry                             4.5.5     True        False         False      35h
ingress                                    4.5.5     True        False         False      4d1h
insights                                   4.5.5     True        False         False      4d2h
kube-apiserver                             4.5.5     True        False         False      4d2h
kube-controller-manager                    4.5.5     True        False         False      4d2h
kube-scheduler                             4.5.5     True        False         False      4d2h
kube-storage-version-migrator              4.5.5     True        False         False      39h
machine-api                                4.5.5     True        False         False      4d2h
machine-approver                           4.5.5     True        False         False      3d14h
machine-config                             4.5.5     True        False         False      39h
marketplace                                4.5.5     True        False         False      3d13h
monitoring                                 4.5.5     True        False         False      35h
network                                    4.5.5     True        False         False      4d2h
node-tuning                                4.5.5     True        False         False      3d14h
openshift-apiserver                        4.5.5     True        False         False      4d2h
openshift-controller-manager               4.5.5     True        False         False      3d14h
openshift-samples                          4.5.5     True        False         False      3d14h
operator-lifecycle-manager                 4.5.5     True        False         False      4d2h
operator-lifecycle-manager-catalog         4.5.5     True        False         False      4d2h
operator-lifecycle-manager-packageserver   4.5.5     True        False         False      3d13h
service-ca                                 4.5.5     True        False         False      4d2h
storage                                    4.5.5     True        False         False      3d14h
```

> Don't forget to change the version for the monitoring, cluster logging and elasticSearch to match the cluster version
