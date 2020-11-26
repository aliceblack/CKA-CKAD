# CKA

## Misc

```
kubectl get nodes 
kubectl get pod -o wide
kubectl get deployments
kubectl describe pod 
kubectl edit deployment
kubectl get pods -n kube-system
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


## Storage
## Networking

### Explore Environment
Kluster nodes:
```
kubectl get nodes
```
Network interfaces:
```
ip link
```
MAC of the interface:
```
ip link show ens3
```
MAC of node01:
```
arp node01
```
Check ip:
```
kubectl get nodes -o wide
```
Look for a bridge interface created by docker:
```
ip link
```
State of interface:
```
ip link show interface0
```
Router thant pinging a site will take:
```
ip route show default
```
Ports listening:
```
netstat -nplt
```
ETCD listening on ports:
```
netstat -anp | grep etcd
```
### Explore CNI weave
Identify the network plugin configured for Kubernetes:
```
ps -aux | grep kubelet
```

Given  /opt/cni/bin is the cni-bin-dir, list the CNI plugins available:
```
ls /opt/cni/bin
```
CNI configured on this cluster:
```
ls /etc/cni/net.d/ 
```
Given /etc/cni/net.d/10-weave.conf is the CNI configured in the cluster, serach for what binary executable file will be run by kubelet after a container and its associated namespace are created:
```
nano /etc/cni/net.d/10-weave.conflist
```

### Deploy Network Solution 
Deploy weave net, when deployed, any further pods you create will be automatically attached to the Weave network:
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
### Networking Weave
Networking solution:
```
/etc/cni/net.d/
```
View names of the bridges created by weave on ech node:
```
ip link
```
IP ranges:
```
ip addr show weave
```

To get the routing ip, enter a pod and then run:
```
ip r
```
### Service Networking
### CoreDNS in Kubernetes
### Ingress Networking 


## Install
## Troubleshooting
## Other topics
## Lightning Labs
## Mock exams
