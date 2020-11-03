# CKA

## Misc

```
kubectl get nodes 
kubectl get pod -o wide
kubectl get deployments
kubectl describe pod 
kubectl edit deployment
```


## Cluster manteinance

### OS Upgrades
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

### Cluster upgrades
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

#### Upgrade additional control plane nodes

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

## Security

### Certificates
Show list of certificate signing requests (CSRs):
```
kubectl get csr
```

Approve a CSR:
```
kubectl certificate approve john
```

Inspect a CSR:
```
kubectl get csr john -o yaml
```

Deny and delete a CSR:
```
kubectl certificate deny john
kubectl delete csr agent-smith
```

In order to reduce the number of old CertificateSigningRequest resources left in a cluster, a garbage collection controller runs periodically. 

### KubeConfig

Get the clusters defined in the default kubeconfig file, you can see context and users too:
```
kubectl config view
kubectl config view --kubeconfig my-kube-config-file
```

Use context:
kubectl config --kubeconfig=/root/my-kube-config-file use-context mycontextname

### Roles
Get roles:
```
# on the default namespace
kubectl get roles
# on all namespaces
kubectl get roles --all-namespaces
# describe a role in a namespace
kubectl describe role kube-proxy -n kube-system
```

Run commands as users:
```
kubectl get pods --as developer
```

### Cluster roles
A role or a cluster role is an RBAC set of rules. There are no deny rules. They set permissions in a particular namespace. A role binding grants the permissions defined in a role to a user or set of users.

```
kubectl get clusterroles 
kubectl get clusterrolebindings
kubectl describe clusterrolebinding <name>
kubectl auth can-i list nodes --as jane
```
  
### Image security
Secret credentials:
```
# create
kubectl create secret docker-registry company-registry-credentials --docker-username=john --docker-password=password --docker-server=companyprivateregistry.com:5000 --docker-email=john.doe@companyprivateregistry.com
# inspect
kubectl get secret company-registry-credentials --output=yaml
```

Pod using the secret:
```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: company-registry-credentials
```


### Security context
The User ID defined in the securityContext of the container beats the User ID in the POD, the User ID defined in the securityContext of the POD is configured to all the PODs in the container. User that is running the container:
```
kubectl exec ubuntu -- whoami
```

### Network policies
By default, pods are non-isolated; they accept traffic from any source.

Pods become isolated by having a NetworkPolicy that selects them. Once there is any NetworkPolicy in a namespace selecting a particular pod, that pod will reject any connections that are not allowed by any NetworkPolicy.  
Network policies do not conflict; they are additive. Each NetworkPolicy may include a list of allowed ingress and egress rules.  
The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers:
* Other pods that are allowed (exception: a pod cannot block access to itself)
* Namespaces that are allowed
* IP blocks (exception: traffic to and from the node where a Pod is running is always allowed, regardless of the IP address of the Pod or the node)

```
kubectl get networkpolicy
kubectl describe networkpolicy
```
