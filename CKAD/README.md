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
#Exposing service on a port
kubectl expose pod <pod> --port=<port> --name <service-name>
# expose pod
kubectl run custom-nginx --image=nginx --port=8080
#create pod and service
kubectl run <pod-name> --image=<image> --port=80 --expose
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

## Containers
Containers run tasks and processes, at completion they do exit. 
```
CMD ["nginx"]
$ docker run ubuntu [COMMAND]
$ docker run ubuntu sleep 5
FROM Ubuntu
CMD sleep 5
CMD ["command","param"]
CMD ["sleep", "5"]
$ docker build -t ubuntu-sleeper .
$ docker run ubuntu-sleeper
$ docker run ubuntu-sleeper sleep 10
FROM Ubuntu
ENTRYPOINT ["sleep"]
$ docker run ubuntu-sleeper 10
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
$ docker run ubuntu-sleeper
$ docker run ubuntu-sleeper 10
$ docker run --entrypoint moreinterestingcommand ubuntu-sleeper 10
```

The entrypoint is overridden in the pod definition, entrypoint defined in image, gets overridden in pod definition.

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
# create with imperative commands then scale
kubectl create deployment webapp --image=<image>
kubectl scale deployment/webapp --replicas=3
#created with replicas
kubectl create deployment blue --image=nginx --replicas=6
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
kubectl create deployment <deployment-name> --namespace=<namespace> --image=<image> --replicas=2
```

## Namespaces
You can use namespace on yaml definitions or evenspecify `--namespace=<namespace>` when creating the pods.
You can even change context to avoid typing the name space every time:
```

kubect config set-context $(kubectl config current-context) --namespace=<namespace>
#following commands will refer to that namespace
```

Show all:
```
kubectl get pods --all-namespaces
kubectl get namespace
```

Create:
```
#create namespace
kubectl create ns <namespace-name>
#create pod
kubectl run <podname> --image=<image> --namespace=<namespace>
```

## Configuration
### Config maps
Enviroment variables can be set using the `env` tag of a pod definition file, the tag is an array and every variable can be created using name and value properties or name and valueFrom properties. Check some different configurationfiles here:
```
...
spec:
  containers:
  - name: nginx
    image: nginx
    env:
      - name: TEMPERATURE
        value: fahrenheit
```

```
#using valueFrom
env:
  - name: TEMPERATURE
    valueFrom: 
      configMapKeyRef:
        name: project-configmap
        key: TEMPERATURE
      
env:
  - name: TEMPERATURE
    valueFrom: 
      secretKeyRef:
      
#using volumes
volumes:
- name: project-volume
  configMap:
    name: project-configmap
```

ConfigMap is a key value pair map. A config map can be created by commands:
```
  kubectl create configmap  <name> --from-literal=TEMPERATURE=fahrenheit --from-literal=COLOR=blue
```
or files:
```
#from properties file
kubectl create configmap --from-file=configuration.properties

#from configmap definition file
kubectl create -f configuration.yaml
```

Definition:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: newconfig
data:
 TEMPERATURE: fahrenheit 
 COLOR: blue
```

Commands:
```
kubectl get configmaps
kubectl describe configmaps
```

Definitions:
```
...
spec:
  containers:
  - name: nginx
    image: nginx
    envFrom:
    - configMapRef:
        name: project-configmap
```

## Secrets
Config maps store data as playntext. Secrets do encode in base64.
```
kubectl create secret generic <name> --from-literal=<secret-name>=<value> --from-literal=<secret-name>=<value>
kubectl create secret generic <name> --from-file=<secrets>
kubectl create -f definition.yaml
kubectl get secrets
kubectl describe secrets
kubect get secret project-secrets -o yaml
```

Definitions:
```
apiVersion: v1
kind: secret
  name: project-secrets
data:
  USERNAME: rGDFXGDFSsv==
  PASSWORD:  esFDfreg==
```

Using in pods:
```
envFrom: 
   - secretRef:
        name: project-secret
#or        
env:
  - name: TEMPERATURE
    valueFrom: 
      secretKeyRef:
        name: project-secret
        key: TEMP
        
#or using volumes
volumes:
- name: project-secret-volume
  secret:
    secretName: project-secret
#notethat they will be stored as separate files in the container, the file name will be the secret variable key and the content of the file will be the secret variable value 
```

### Users
Run as user:
```
docker run --user=1000 ubuntu sleep 5000
```

Or user id in file:
```
FROM ubuntu
USER 1000
```

In a pod definition file it can be set a securityContext tag either in `spec` or in `containers` tags on a specific container.
Container settings will overwrite pod settings.
You can use `--cap-add` and `--cap-drop` to manage Linux capabilities. Use `--privileged` to give extended privileges and `--device=[]` to run devices insede teh container without the `--privileged` flag.

### Service Accounts
Accounts: 
* user accounts
* service accounts

When creating a a service account, an account is created and a token si generated, then K8s creates a secret object, stores the token in the secret object and link the secret object to the account. The token is an authetnication bearer for the Kuberntes APIs.
```
kubectl create serviceaccount <name>
kubectl get serviceaccount
kubectl describe serviceaccount <name>
kubectl describe secret <name>
```

Each namespace has a service account. Any pod created has the secret automatically mounted as a volume.
You must delete and reacreate the pod if you plan to change service account, if you edit a deplyment it will do it automatically instead.

```
...
spec:
  containers:
    - name: nginx
      image: nginx
    serviceAccount: nginxServiceAccount
```

```
...
spec:
  containers:
    - name: nginx
      image: nginx
    automountServiceAccountToken: false
```

### Memory
When a pod tries to surpass the CPU limit, the CPU does get throttled, so it does not surpass the limit, but when memory limit is surpassed constantly (it can!), it gest terminated. 
```
...
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
         requests:
           memory: "1Gi"
           cpu: 1
         limits:
           memory: "2Gi"
           cpu: 2
```

### Taints and tolerations
Pod placement in nodes by the scheduler.
Taint is placed on nodes, it prevents pod placement. Natively any created pod can not be placed where a taint has be applied. A toleration allow pods to be blaced where a taint has be placed. Toleration is placed on pods. Use node affinity to place pods on nodes. Master has an automatic taint.
```
kubectl taint nodes <node> <key>=<value>:<taint-effect>
kubectl taint nodes <node> key=value:NoSchedule
kubectl describe node kubemaster | grep taint
```
* NoSchedule
* PreferNoSchedule, not guerantee
* NoExecute

```
...
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
  tolerations:
  - key: "temperature"
    operator: "Equal"
    value: "centigrades"
    effect: "NoSchedule"
```
### Node selectors
Node selectors and node affinities. Selectors are not suited for more than one label, use node affinity and anti affinity.
```
kubectl label nodes <node> <key>=<value>
kubectl label node node01 color=blue

...
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
  nodeSelector:
    size: <lable>          # is a lable assigned to a node, you must fisrt lable a node
```

Node affinity types:
* requiredDuringSchedulingIgnoredDuringExecution 
* preferredDuringSchedulingIgnoredDuringExecution
* (more to come in the future)

## Config mpas
```
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=blue
```

## Security context
Check the user running the container:
```
kubectl exec ubuntu-sleeper -- whoami
```

Pod definition:
```
spec:
  securityContext:
    runAsUser: 1010
  containers:
    ...
```

Pod definition:
```
  image: ubuntu
  name: ubuntu-sleeper
  securityContext:
    capabilities:
    add: ["SYS_TIME"]
```

Open shell on the pod:

```
kubectl exec --stdin --tty ubuntu-sleeper -- /bin/bash
```

## Service account
```
kubectl create serviceaccount <name>
```

## Taints and tolerations
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```

Remove taint from master:
```
kubectl taint nodes master/controlplane node-role.kubernetes.io/master:NoSchedule-
```

```
containers:
 ...
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution
    nodeSelectorTerms:
    - matchExpression:
      - key: color
        operator: In
        values:
        - blue
```
