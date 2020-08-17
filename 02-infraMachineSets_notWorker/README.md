### Creating Infra Machine Configuration


- Create machine configuration whether from the command line or the web console, you can copy the configuration from exist ones withe updating it to match the one you need
```bash
[root@bastion ~]# oc get mc
NAME                                                        GENERATEDBYCONTROLLER                      IGNITIONVERSION   AGE
00-infra                                                                                               2.2.0             35h
00-master                                                   807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             4d1h
00-worker                                                   807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             4d1h
01-infra-container-runtime                                                                             2.2.0             35h
01-infra-kubelet                                                                                       2.2.0             35h
01-master-container-runtime                                 807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             4d1h
01-master-kubelet                                           807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             4d1h
01-worker-container-runtime                                 807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             4d1h
01-worker-kubelet                                           807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             4d1h
99-infra-13dfe340-f798-42d7-baa8-67d3dd4b8144-registries    807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             36h
99-infra-ssh                                                                                           2.2.0             35h
99-master-79efe988-aeaf-4cc1-b644-80ed917f7571-registries   807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             4d1h
99-master-ssh                                                                                          2.2.0             4d1h
99-worker-66d3551c-2d40-4158-9823-b1597d1f1d43-registries   807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             4d1h
99-worker-ssh                                                                                          2.2.0             4d1h
rendered-infra-ec440848aa2220601f4eb9e772ffa819             807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             35h
rendered-master-63a83ba8ba39442f815cb5bc982cd05d            807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             3d13h
rendered-master-f2a2d42c4469b90da750a5af28506d52            601c2285f497bf7c73d84737b9977a0e697cb86a   2.2.0             4d1h
rendered-worker-1cebd98f33aea85a5047da8696b09761            601c2285f497bf7c73d84737b9977a0e697cb86a   2.2.0             4d1h
rendered-worker-f7bd0ca9528f2d0a370002621f5ccf3b            807abb900cf9976a1baad66eab17c6d76016e7b7   2.2.0             3d13h
```

- Create the Machine Config Pool
```bash
oc create -f infra-mcp.yaml
```

- Check the mcp
```bash
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
infra    rendered-infra-b4ac791ae751cae7f248e353280ae344    True      False      False      3              3                   3                     0                      35h
master   rendered-master-63a83ba8ba39442f815cb5bc982cd05d   True      False      False      3              3                   3                     0                      4d2h
worker   rendered-worker-f7bd0ca9528f2d0a370002621f5ccf3b   True      False      False      3              3                   3                     0                      4d2h
```

> Ensure to update the infra nodes to have the correct label and remove any other labels if not needed
