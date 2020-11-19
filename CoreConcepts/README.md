## CORE CONCEPTS
The core concepts section covers the core K8s API and its primitives and resources. 

kubectl is command by which you interact with kubernetes master node via kube-apis

Shorthand for kubectl setup
```
$ alias k=kubectl
$ complete -F __start_kubectl k
```

Note: Going forward I will use k for kubectl

### Pods
It also covers the important concept of a POD. This is the basic unit of deployment for app developers and so this 'POD' concept is important to understand as well as how they are managed with kubectl. To me, this is embodied in the kubectl RUN command.

Pod Imperative commands
```
$ kubectl run mypod --image=nginx --restart=Never (Create and run the pod)
$ kubectl run mypod --image=nginx --restart=Never -l app=frontend (Create and run the pod with labels)
$ kubectl run mypod --image=nginx --restart=Never port=8080 (Create and run the pod with port exposed)
$ kubectl run redis --image=redis123 --restart=Never --dry-run -o yaml > redis.yaml (Create Pod yaml and then run create as below)
```
Basic pod yaml
```
apiVersion: v1
kind: Pod
metadata: 
  name: mypod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
  - name: nginx
    image: nginx

```
Create pod with yaml
```
$ kubectl create -f pod.yaml
```


Get and describe pod commands
```
$ kubectl get pods
$ kubectl get pod mypod
$ kubectl describe pod mypod
```
To get more information on pods e.g. ip and node
```
$ kubectl get pods -o wide
```
Get pod yaml
```
$ kubectl get pod mypod -o yaml > mypod.yaml
$ kubectl delete pod mypod (delete pod)
```


### Replica Sets

Replica Sets yaml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
  labels:
    app: myapp
    type: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      type: frontend
  template:
    metadata:
      name: mypod
      labels:
        app: myapp
        type: frontend
    spec:
      containers:
      - name: mypod
        image: nginx
    
```
Create rs with yaml
```
$ kubectl create -f rs.yaml
```

Replica Set commands
```
$ kubectl get rs (get replica sets)
$ kubectl get rs -o wide (get rs with containers, images, selectors)
$ k describe rs my-replicaset (describe replica sets)
$ k delete rs my-replicaset (delete replica sets)

```

### Deployments

Deployment creates replica sets and pods.

Deployment yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: myapp
    type: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      type: frontend
  template:
    metadata:
      name: mypod
      labels:
        app: myapp
        type: frontend
    spec:
      containers:
      - name: mypod
        image: nginx
    
```
Create deployment with yaml
```
$ kubectl create -f deploy.yaml
```

Deployment commands
```
$ k run nginx --image=nginx --replicas=3 (create deployment with 3 replicas )
$ k create deploy nginx --image=ngnix (create deployment with single instance)
$ k scale deploy nginx --replicas=3 (can scale deployment post creation)
$ k get deploy/deployment (get deployment)
$ k describe deploy my-deployment (describe deployment)
$ k delete deploy my-deployment (delete deployment)

```

### Namespaces

Namespaces provide a scope for resources and it needs to be unique within namespace. If no namespace is defined than resources are created in default namespace

commands

```
k create namespace/ns myns (create a namespace)
k get ns (list of namespaces)
k get pods --namespace/-n dev (get pods in namespace dev)
k get pods (get pods in default namespace)
k get pods --all-namespaces (get pods in all namespaces)
k config set-context $(k config current-context) -n dev (set context and namespace for it)

```
Resource quota

commands
```
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run -o yaml
```