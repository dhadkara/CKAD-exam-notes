## CONFIGURATIONS 


### Command and Arguments

Docker containers are build to run a command/job and then exit. So the CMD is the default command when container starts and if it is webserver or db start command it won't exit.
If you run a container with external command it will override the CMD default
e.g. docker run ubuntu sleep 5
CMD syntax in dockerfile:
CMD command param1 or CMD ["command", "param1"]
CMD sleep 5 or CMD ["sleep","5"]

But if you want always run a default command on container startup and pass params to it we use ENTRYPOINT

Below dockerfile example container will start and sleep for 5 seconds
```
ENTRYPOINT ["sleep']
CMD ["5"]
```
To override default sleep time to 10 seconds
```
docker run sleeper 10
```
In case to override entrypoint of container
```
docker run --entrypoint sleep2.0  sleeper 10
```

In pods defination below is to achieve same

| __Docker__  | __K8s__ | 
|-------------|------------|
| ENTRYPOINT  | command  |
| CMD         | args    |

Kubernetes pod definition 
```
apiVersion: v1
kind: Pod
metaData: 
  name: myPod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
  - name: nginx
    image: nginx
    command: ["sleep2.0"]
    args: ["10"]
```

### ConfigMap

Env variables can be assigned to pod as name value pairs

```
env:
  - key1: value1
  - key2: value2
```

If you have large set of variables then k8s prefer to create ConfigMap and can use env variable from there

Create configmap
```
k create configmap/cm mymap --from-literal=APP_COLOR=blue --from-literal=APP_MODE=Dev
or 
k create configmap/cm mymap --from-file=app.properties

k get configmaps (get list of configmaps)
k describe configmaps (describe configmap)

```
configmap yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```
Use configmap in pod defination file

```
apiVersion: v1
kind: Pod
metaData: 
  name: myPod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
  - name: nginx
    image: nginx
    ports: 
      - containerPort: 8080
    envFrom:
      - configMapRef:
          name: app-config 
```

or single env variable from configmap

``` 
    env:
     - name: APP_COLOR
         valueFrom:
           configMapKeyRef:
             name: app-config 
             key: APP_COLOR
```

or volumes

``` 
    volumes:
     - name: app-config-volume
       configMap:
          name: app-config 
```

### Secrets

Similar as configmap but for sensitive information that you don't want to store in plain text e.g. db passwords, key or tokens etc.

```
k create secret generic app-secret --from-literal=DB_HOST=sys \
                                    --from-literal=DB_PASS=jsaioek

k create secret generic app-secret --from-file=app_secret.properties

k get secrets

k describe secrets

#For secret you need to pass hash value i.e.
echo -n 'mysql' | base64

#To decode secret hash value i.e.
echo -n 'bXlzcWw=' | base64 --decode

k edit secrets mysecret (Edit the current secret)
```

secrets yaml

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```
Use Secrets in Pod 

as volume mount
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```
or as env variable

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
  ```
  ### Security Context



  ### Resources

  ### Service Accounts

  Service Accounts are needed by external applications to access kube-api in k8s cluster
  e.g. dashboard might show list of pods via kube-api needs access
  To provide correct access to ServiceAccount use RBAC (Role based access control) but this is out of scope for CKAD exam

  When service account is created, a secret is also created that has token to access the k8s apis and linked to service account
  
  Token can exported and used in third party application but if you are running an application on k8s itself then can mount that secret as volume.

  FYI: There is a service account named default in every namespace that got mounted when any pod is created.

  Service Account Commands

  ```
  k create serviceaccount/sa my-sa (create service account)
  k get serviceaccount/sa my-sa (get service account)
  k describe sa my-sa (describe service account)
  ``` 
  Service Account yaml - You can optionally decide not mount secret token on service account automatically by using automountServiceAccountToken property as false
  ```
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: my-dashboard
  automountServiceAccountToken: false
  ```

  Using service account in pod
  Note: Service Account is used at Pod level for all the containers
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
      - image: nginx
        name: nginx
    serviceAccount: my-dashboard
  ```
  using command
  ```
  k run nginx --image=nginx --restart=Never --serviceaccount=myuser -o yaml --dry-run > pod.yaml
  k apply -f pod.yaml
  ```
  ### Taints and Tolerations

  The taint has key key, value value, and taint effect NoSchedule. This means that no pod will be able to schedule onto node1 unless it has a matching toleration.

  Add a taint to node 
  Other values for taint-effect are NoSchedule, NoExecute and PreferNoSchedule
  ```
  k taint nodes <node-name> key=value:taint-effect
  k taint nodes node1 app=blue:NoSchedule
  ```
  Remove the taint
  ```
  k taint nodes node1 app:NoSchedule-
  ```
  Tolerations are added to pod
  ```
  