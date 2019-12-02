## POD DESIGN

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

Revisiting deployment command and more
```
k create deploy nginx --image=nginx (create a deployment)
k scale deploy nginx --replicas=3 (scale a deployment)
k set image deploy nginx nginx=nginx:1.7.9 (update the deployment image) or
k edit deploy nginx (update the deploy yaml, automatically updates the deployment)
k autoscale deploy nginx --min=5 --max=10 --cpu-percent=80 (autoscale the deployment based on cpu usage)
k delete deploy nginx (delete deployment)
k delete hpa nginx (delete horizontal pod autoscalar)
```

Rollout commands
```
k rollout status deploy nginx (get deployment rollout status)
k rollout history deploy nginx (get deployment rollout history)
k rollout undo deploy nginx (undo the last rollout)
k rollout undo deploy nginx --to-revision=2 (undo to particular revision)
k rollout history deploy nginx --revision=3 (check history of particular revision)
k rollout pause deploy nginx (pause the rollout of deployment)
k rollout resume deploy nginx (resume the rollout of deployment)

```
### Jobs and CronJobs

Job create one or more pods and successfully terminate them on execution of job.
By default restartPolicy for k8s pods is 'always' but for jobs it should be overridden to 'Never' or 'OnFailure' so that container doesn't restart once job is finished.

'backoffLimit' is number of retries after which job is considered failed. By default its value is 6 but one can override it.

job yaml

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

job commands

```
k create job pi --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)' (create job with command)
k get job pi (get job status)
k get jobs --watch (get jobs status )
k describe job pi (describe pod)
k logs jobs/pi (get logs from job)
k delete job pi (delete a job)
```

By default jobs runs sequentially if we define 'completion' with multiple values till it gets those completed . If one wants to run them in parallel than define 'parallerism'

'activeDeadlineSeconds' is to terminate the job if take more than that time.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  activeDeadlineSeconds: 30
  template:
    completions: 3
    parallelism: 3
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

CronJob runs job on scheduled time interval

TimeInterval define as below
*/min(0-59) hr(0-23) day(1-31) month(1-12) dayOfWeek (0-6/Sunday-Saturday) 

cronjob yaml (three spec sections below)
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
cronjob commands
```
k create cronjob busy --image=busybox --schedule=*/1 * * * * -- /bin/sh -c 'date; echo Hello' (creates a cronjob)
k get jobs --watch (get jobs status )
k delete cronjob/cj busy (delete the cronjob)
```
