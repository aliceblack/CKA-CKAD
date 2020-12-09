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
kubectls get nodes
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
#get a pod definition, to reacreate the pod you must delete the current pod
kubectl get pod <pod-name> -o yaml > pod-definition.yaml
#edit pod properties
kubectl edit pod <pod-name>
```

## Replica sets
Replica tests can create pods or manager existing rpods.
```
kubectl create -f definition.yml
kubectl get replicationcontroller
# pods of the replication controller be named with the replication controller name as prefix
kubectl get pods
```

Update replica sets:
```
kubectl replace -f repilicaset.yml
#or
kubectl scale --replicas=3 -f repilicaset.yml
#or
kubectl scale --replicas=3 replicaset <replicaset-name>
```

If you delete a replicaset, all pods will be deleted.

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
