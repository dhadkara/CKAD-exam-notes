## MULTI CONTAINER PODS

In some cases you want to run two applications running side by side e.g. app and log agent and thus you use multicontainer pod that has access to same network and storage volume. Thus they dont need any services to communicate between them.

multi container pod yaml

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
  - name: web-app
    image: nginx
    ports:
     - containerPort: 8080
  - name: log-agent
    image: log-agent
```

### Design patterns

* SideCar - e.g. Logging agent service with application
* Adaptor - e.g. A application converts different app logs to central logging system
* Ambassador - e.g. application needs to connect to different db for different envs

## Create and pod with commands

```
k run mypod --image=busybox --restart=Never -o yaml --dry-run -- /bin/sh -c "echo hello; sleep 3600" > pod.yaml

```
execute command in container 
```
k exec -it <pod_name> -c <container_name> -- command
k exec -it mypod -c busybox2 -- ls
```

## Logging architecture

Read about logging architecture 

https://kubernetes.io/docs/concepts/cluster-administration/logging/



