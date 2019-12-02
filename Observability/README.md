## OBSERVABILITY


### Liveness and Readiness Probes

Many applications running for long period of time and cannot recover except restarting thus liveness probe helps to detect it and restart the container

'initialDelaySeconds' is used to delay before performing first probe
'periodSeconds' is used to perform probe every period seconds

LivenessProbe yaml with liveness command

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
or http liveness probe

```
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3

```
or tcp liveness probe

```
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

you can also use name container port for probe

```
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
```
Readiness probe is during start up of application that takes some time to start thus probe starts only when app is ready to serve the traffic
It is similar to Liveness probe but under readinessProbe tag

Readiness Probe yaml

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

### Container Logging and Debugging

```
k logs <pod_name>
k delete po busybox --force --grace-period=0 (delete the pod forcefully)
k top nodes (get nodes cpu/memory utilization)
```







