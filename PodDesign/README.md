## POD Design

### Labels, selector and Annotations

Labels are added as key/value pairs to the k8s object to identify them. 
Annotations used to provide more information on objects e.g. buildversion

Commands for labels
```
k run nginx --image=nginx --restart=Never --labels=app=v1/ -l app=v1 (create pods with labels)
k get pods --show-labels (all pods with labels)
k label po nginx2 app=v2 --overwrite (change label of the pod)
k label po nginx1 nginx2 nginx3 app- (remove the label from pods)
```

pod yaml with labels
```
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```
command to get pods with label selectors

a. equality based 
```
kubectl get pods -l environment=production,tier=frontend
```
b. set based

```
kubectl get pods -l 'environment,environment notin (frontend)'
```

Configure a pod with node selector
```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

Annotations and label selector for replica sets 

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
  labels:
    app: myapp
    type: frontend
  annotations:
    buildVersion: 1.34
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
Commands for annotations
```
k annotate pod nginx1 nginx2 nginx3 description=mydesc (create annotation to the pods)
k annotate pod nginx1 nginx2 nginx3 description- (remove annotations from pods)
```

### Deployment Updates

Rollout commands
```
k rollout status deploy nginx (get deployment rollout status)

```