# CKA

## Misc

```
kubectl get nodes 
kubectl get pod -o wide
kubectl get deployments
kubectl describe pod 
kubectl edit deployment
kubectl get pods -n kube-system
#daemons
kubectl -n kube-system  get ds
kubectl get all
kubectl get all --all-namespaces
kubectl get <resource name> --all-namespaces
kubectl get deploy --all-namespaces
# gett all pods, services, deployments and replica sets from a namespace
kubectl -n <namespace> get all
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
Cluster nodes:
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

Ip allocation range of wave:
```
kubectl -n kube-system get pods
kubectl -n kube-system logs  <wave-pod-name> -c <wave-container-name>
```
Type of proxy the kube-proxy is configured to use:
```
kubectl -n kube-system logs  <proxy-pod-name> 
```

Deamons, check if kube-proxy is deployed ad daemon:
```
kubectl -n kube-system  get ds
```


### Service Networking
Ip addresses:
```
ip addr
```

IP range of the pods on the cluster: The network is configured with weave. Check the weave pods logs using 
```command kubectl logs <weave-pod-name> weave -n kube-system``` 
and look for ipalloc-range.

Cluster ip range:
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range
```

### CoreDNS in Kubernetes
Services:
```
kubectl get service -n kube-system
```

Check where is the configuration file located for configuring the CoreDNS service,
inspect the Args of the coredns deployment and check the file used, example: 
```
kubectl -n kube-system describe deployments.apps coredns | grep -A2 Args | grep Corefile
```

Get configmap:
```
kubectl get configmap -n kube-system
```

Describe configmap:
```
kubectl describe configmap coredns -n kube-system 
```

### Ingress Networking 
Check host configured, run the command and look at Host under the Rules section:
```
kubectl describe ingress --namespace <namespace>
```

Run the command and look under the Rules section to see backend is the path on the Ingress configured with:
```
kubectl describe ingress --namespace <namespace>
```

Change the path to the video  applications:
```
kubectl edit ingress --namespace <namespace>
```

Create an ingress from zero:
```
kubectl create namespace ingress-space
kubectl create configmap nginx-configuration --namespace ingress-space
kubectl create serviceaccount ingress-serviceaccount --namespace ingress-space
#someone created the Roles and RoleBindings for the ServiceAccount
kubectl get roles,rolebindings --namespace ingress-space
#create a deployment file then deploy
kubectl apply -f deployment.yaml
#create a service to make Ingress available to external users
kubectl apply -f service.yaml
kubectl expose deployment -n ingress-space ingress-controller --type=NodePort --port=80 --name=ingress --dry-run -o yaml >ingress.yaml
```


## Install
## Troubleshooting
### Application failures
```
controlplane $ kubectl -n <namespace> get svc
controlplane $ kubectl -n <namespace> get svc mysql -o yaml > mysql-service.yaml
controlplane $ kubectl -n <namespace> delete svc mysql
service "mysql" deleted
controlplane $ vi mysql-service.yaml
#do stuff
controlplane $ kubectl create -f mysql-service.yaml
```

```
controlplane $ kubectl -n <namespace> describe pod mysql
controlplane $ kubectl -n <namespace> delete  svc mysql-service
service "mysql-service" deleted
controlplane $ kubectl -n <namespace> expose pod mysql --name=mysql-serve
service/mysql-serve exposed
controlplane $ kubectl -n <namespace> get ep
NAME          ENDPOINTS         AGE
mysql-serve   10.244.1.8:3306   113s
web-service   10.244.1.9:8080   5m3s
```

```
controlplane $ kubectl -n delta describe deployments webapp-mysql
#check enviroment, you could find wrong sql credentials
controlplane $ kubectl -n delta get deployment webapp-mysql -o yaml >  web.yaml
controlplane $ kubectl delete deployments.webapp-mysql -n delta
vi web.yaml
#do stuff
controlplane $ kubectl create -f web.yaml
```

### Control plane failure

```
kubectl scale deployment app --replicas=2
```
## Other topics
## Lightning Labs
## Mock exams
