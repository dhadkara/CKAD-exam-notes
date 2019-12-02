## STATE PERSISTANCE

### Volumes and Mounts

Volume is to persist data from the containers 

volume mount with empty dir yaml that exists till pod is available even if container restarts

```
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  volumes:
  - name: redis-storage
    emptydir: {}
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
```

or volume mount on host path if pod exits and restarts

```
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  volumes:
  - name: redis-storage
    hostPath: 
      path: /data
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
```

### Persistance Volume and Volume Claims

Above can work for single node cluster but with multiple nodes you need a persistance volume. However in production you will use GCE or AWS volumes or NFS share

Persistance Volume yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessMode: 
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```
```
k get pv task-pv-volume (get persistent volume)
```

Persistance Volume Claims yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
```
k get pvc task-pv-claim (get persistent volume claim)
```
Create Pod with pvc

```
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```


