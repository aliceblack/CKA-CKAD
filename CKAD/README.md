# CKAD

## Kubernetes clusters
Master node:
* kube-apiserver
* etcd
* controller
* scheduler

Worker node:
* kubectls
* container runtime

Useful commands:
```
kubectl get cluster-info
kubectl get nodes
kubectl get all
```

Kubectl advanced commands:
```
# Format using -o json, -o wide, -o yaml
kubectl get pods -o wide
#--dry-run to immediately create the resource, --dry-run=client to see if the resource can be created, -o yaml to generate a resource defionition file
#generate pod maniffest but does not create it
kubectl run nginx --image=nginx  --dry-run=client -o yaml
```

## Pods
Pods can talk to eachother on localhost, they share the same network space. Usually one pod contains one container and optionally its helpers.

When running a container, Kubernetes creates a pod and pulls and image:
```
kubectl run nginx --image nginx
```

A definition file is written in yaml and contains the following root tags, they are required tags:
* apiVersion
* kind
* metadata
* spec

Useful commands:
```
kubectl get pods
kubectl describe pod <pod-name>
controlplane $ kubectl create -f definition.yaml
#get a pod definition, to reacreate the pod you must delete the current pod
kubectl get pod <pod-name> -o yaml > pod-definition.yaml
#use 'kubectl apply' command with a file or use 'kubectl edit pod <pod>' to edit a pod
controlplane $ kubectl edit pod <pod>
#edit pod properties
kubectl edit pod <pod-name>
#delete
controlplane $ kubectl delete pod webapp
```

Creating a pod:
```
controlplane $ kubectl run nginx --image=nginx --dry-run=client -o yaml
apiVersion: v1kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
controlplane $ kubectl run nginx --image=nginx
pod/nginx created
```

## Replica sets
Replica sets can create pods or manager existingr pods.
```
kubectl get replicaset
kubectl describe replicaset <replicaset>
kubectl delete replicaset <replicaset>
kubectl create -f definition.yml
#delete and re-create the ReplicaSet or update the existing ReplicSet and then delete all PODs, so new ones with the correct image will be created
kubectl edit replicaset <replica-set>
kubectl get replicationcontroller
# pods of the replication controller be named with the replication controller name as prefix
kubectl get pods
```

Update replica sets, use 'kubectl scale' command or edit the replicaset using 'kubectl edit replicaset':
```
kubectl replace -f repilicaset.yml
#or
kubectl scale --replicas=3 -f repilicaset.yml
#or
kubectl scale --replicas=3 replicaset <replicaset-name>
```

If you delete a replicaset, all pods will be deleted.

## Deployments
```
kubectl get deployment
kubectl describe deployment <deployment>
kubectl get deployment <deployment> -o yaml
```

## Namespaces
You can use namespace on yaml definitions or evenspecify --namespace=<namespace> when creating the pods.
You can even change context to avoid typing the name space every time:
```
kubect config set-context $(kubectl config current-context) --namespace=<namespace>
#following commands will refer to that namespace
```
Show all:
```
kubectl get pods --all-namespace
```
