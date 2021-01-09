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
apiVersion: v1
kind: Pod
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

```
apiVersion: v1
kind: Pod
metadata:
  name: colors
spec:
  containers:
  - image: nginx
    name: blue
  - image: nginx
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
kubectl describe pod <pod> --namespace=<namespace>
kubectl describe pod <pod> --n=<namespace>
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

## Status and conditions
Statuses:
* Pending (waiting to be scheduled on a node)
* ContainerCreating
* Running

Conditions:
* PodScheduled
* Initialized
* ContainersReady
* Ready

## Readiness probes

Pod definitions with readiness probes:
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    name: webapp
spec:
  containers:
  - name: webapp
    image: webapp
    ports:
      - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
    
```

```
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 20 #additional deleya
      periodSeconds: 5 #how often to probe
      failureThreshold: 10 #default is 3
```

```
    readinessProbe:
      tcpSocket:
        port: 3306
```

```
    readinessProbe:
      exec:
        command: 
        - cat
        - /app/isready
```
## Liveness probes
Liveness probes are defined just like the readiness probes:
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    name: webapp
spec:
  conta8iners:
  - name: webapp
    image: webapp
    ports:
      - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /api/ready
        port: 8080
    
```

```
    livenessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 20 #additional deleya
      periodSeconds: 5 #how often to probe
      failureThreshold: 10 #default is 3
```

```
    livenessProbe:
      tcpSocket:
        port: 3306
```

```
    livenessProbe:
      exec:
        command: 
        - cat
        - /app/isready
```

## Logs
Multiple containers pod require you to specify the name of the contianer:
```
#single container pod
kubectl logs -f <pod-name>
#multy container pod
kubectl logs -f <pod-name> <container-of-the-pod>
```

## Labels
Pod definition:
```
apiVersion: v1 
kind: Pod
metadata:
  name: webap
  labels:
    application: gui
    mode: mobile
    
```
Get the pod using the lables:
```
kubectl get pods --selector application=gui
```

Replica set definition:
```
apiVersion: apps/v1 
kind: ReplicaSet
metadata: #lables of the replica sets itself, we are not talking about these
  name: webapp
  labels:
    application: gui
    mode: mobile
spec:
  replicas: 5
  selector:
    matchLabels:
      application: gui
  template:
    metadata:
      labels:
        application: webapp
        mode: mobile
    spec:
      containers:
      - name: webapp
        image: webapp
```

## Monitoring
Metrics server is required for CKAD, but other solutions are available.
The Metrics server store info in memory, there is no historic data. Container advisor exposes metrics through the Kubelet API, used by the Metrics server.

If you use minikube, do `minikube addons enable metric-server`, otherwise clone from git and deploy using kubectl.

## Labels and selectors

## Rolling updates
Rolling updates is the default update strategy. Recreate will shutdown every instance and then create all the new sitances, Rolling update will do it one by one.

When you ask kubectl to describe the deployments, with recreate the old replica gets scaled to 0, and the the new replica kicks in, with rolling the replicas are seen to be decreased by one (the old replica) and increased by one (the new replica) at the same time. 

```
kubectl rollout status deployment/webapp-deployment
#edit image or use apply command using a definiton file 
kubectl apply -f deployment.yaml
kubectl set image deployment/webapp-deployment nginx=nginx:latest
#to bring up the old pods do a rollback
kubectl rollout undo deployment/webapp-deployment
#status
kubectl rollout status deployment/webapp-deployment
kubectl rollout history deployment/webapp-deployment
```

If you edit the container image (version) and then use the apply command, a new rollout is triggered. Otherwise you can use 
the command  `kubectl set image deployment/webapp-deployment nginx=nginx:latest`.

Use the `--revision=<revision-number>` flag to see the status of a revision, `kubectl rollout status deployment/webapp-deployment --revision=3`, use the `--record` flag to add the command used to update a delployment to teh revision number, `kubectl set image deployment nginx nginx=nginx:latest --record`.

  
## Jobs
Creates pods until the desired numberof succesful completed jobs is reached. 

```
apiVersion: batch/v1
kind: Job
metadata:
  name: thejob
spec:
  template:
    spec:
      containers:
        - name: thejob
          image: ubuntu
          command: ['date']
          
      restartPolicy: Never
```

```
kubectl get jobs
kubectl get pods
kubectl logs thejob # will print the date
kubectl delete thejob
```

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: thejob
spec:
  shedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 5
      parallelism: 5
      template:
        spec:
          containers:
          - name: ...
            image: ...
          
          
          restartPolicy: Never
```

## Volumes

Mounting a volume in a pod and adding it to the containers, but a folder on multiple containers has the problem of data allineation, you need a a cluster storage replicationn strategy (GlusterFS for example):
```
apiVersion: v1
kind: Pod
metadata:
 name: application
spec:
 containers:
 - image: alpine
   name: alpine
   command: ["/bin/sh","-c"]
   args: ...
   volumeMounts:
   - mountPath: /opt
     name: volume
  
  volumes:
  - name: volume
  hostPath:
    path: /data
    type: Directory
```

## PV and PVCs

PV are cluster wide, PVCs and PVs are bond with a one-to-one relationship. PersistentVolumeclaim tag can be used in the volum tag  of pods, replica sets, deployments. 

For a PVC, a PV with sufficient volume is found, but you can always use labels and selectors to get the PV you want. When a PV is bigger than its PVC, the reaining space remaints unused and can not be claimed. Once a PVC is deleted, behaviour depends upon `persistentVolumeReclaimPolicy`:
* Retain, PV must be manually deleted
* Delete, PV gets automatically deleted
* Recycle, the PV is recycled (data wiped, available again)

```
kubectl get pvc
```

NOT IN THE EXAM:
* storage classes
* stateful sets


```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvol
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 6Gi
  hostPath:
    path: /tmp/data
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvolclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

PV:
```
labels: 
  name: pv-n
```

PVC:
```
selector:
  matchLabels:
    name: pv-n
```

Pod:
```
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app_0
    image: app_0
	env:
	- name: logs
	  value: file
	volumeMounts:
	- mounthPath: /log
	  name: myvolume
	
  volumes:
  - name: myvolume
    persistentVolumeClaim:
	  claimName: myclaimname
```

## Storage classes
What we saw until now was static provisioning. There are provisioners capable of provision volumes dinamically (Dinamic provisioning). When a PVC is created, a storage class will create a PV automatically. A storage class definition will define the provisioner to use.

## Stateful sets
Stateful sets are similar to deployments, they create pods in sequantial order (up and running), assign unique newtwork ids,assign names and numbers (not random names), but you can still ask thme to create pods in parallel. Deletion is done in reverse order. 
  
Use a volume claim template when you need to define automatically a PVC for ach pod of a statefull set.

## Services
A service is a DNS entry to a pod name. A service is not a load balancer.
DNS entry to each pod has the following structure: `<pod-name>.<headless-service-name>.<namespace>.<svc>.<cluster-domain>.<app>`.
A Service definition file creates a headless service.

## Services
Ports:
* Node Port
* Service Port
* Target Port (Port of a pod)

Services can be createsi via definition files.
It does use a random algorithm to load balance of the labels/selectors identify multiple istances of the same pod.

## Metrics server
Use just for testing:
```
#deploy metric server
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
cd kubernetes-metrics-server/
kubectl create -f .
#check how it is going
kubectl top node 
kubectl top pod
```
