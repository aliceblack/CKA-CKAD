```
kubectl get nodes 
kubectl get pod -o wide
kubectl get deployments
```

## OS Upgrades
Empty the node of all applications, mark the node unschedulable:
* Node node01 becomes unschedulable
* Pods are evicted from node01
``` 
kubectl drain node01 --ignore-daemonsets
```


Node becomes schedulable again (note that only when pods are created they will be back again, for now they remain live on the pods they where moved to):
```
kubectl uncordon node01
```

You mustforce drain for Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet, they can not be deleted unless the command is forced.
Pod not part of replica set will be lost forever tho.

Make node03 unschedulable, no more app will be scheduled, only the running ones will remain.
* Unschedulable
* Mantains its current apps
```
kubectl cordon node03
```

## Cluster upgrades
Stable version available for upgrade:
```
kubeadm upgrade plan
```

Drain the master node of workloads and mark it UnSchedulable:
```
kubectl drain master/controlplane --ignore-daemonsets
```

Upgrade:
* Upgrade kubeadm tool 
* upgrade then the master components
* upgrade the kubelet on the master node
```
apt install kubeadm=1.18.0-00 
kubeadm upgrade apply v1.18.0 
apt install kubelet=1.18.0-00
```
Uncordon the control plane node, master will be shedulable again:
```
kubectl uncordon master
```

### Upgrade additional control plane nodes

Note that `kubeadm upgrade plan` is not needed to upgrade additional control plane nodes.  

Drain and make unschedulable:
* drain worker node
* make node unschedulable
```
kubectl drain node01 --ignore-daemonsets
```

Upgrade worker node:
* Worker Node Upgraded to v1.18.0
* Worker Node Ready
```
apt install kubeadm=1.18.0-00
kubeadm upgrade node                 # upgrade node not upgrade apply!
apt install kubelet=1.18.0-00
```

