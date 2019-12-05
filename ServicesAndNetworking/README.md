## SERVICES AND NETWORKING

### Services

Service enables access to group of pods as it has static ip e.g. connecting external users to frontend pods, connecting frontend pods to backend pods, connecting pods to external database etc. thus provides a loose coupling between our microservices

Service Types
1. Node Port
External Communication - Service will map kubernetes node port to pod container port.

machine -> k8s node (ip/node port) -> service -> pod (ip/port)

TargetPort - Port on Pod
Port - Port on the service
ClusterIp - IP address of service
NodePort - Port on the node use to access externally

service yaml 
Note: If targetPort is not provided then it assumed be same as port
      If nodePort is not provided then k8s automatically allocates
```
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
      app: frontend
      
```

2. Cluster IP

Service enables virtual ip to enable communication between frontend pods and backend pods

Front end Pods -> service -> backend Pods -> service -> db Pods

```
apiVersion: v1
kind: Service
metadata:
  name: my-backend-service
spec:
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
      app: backend
      
```


Services commands
```
k get service my-app-service (get service details)
k get services/svc (get list of services)
k run nginx --image=nginx --restart=Never --port=80 --expose (create a pod with service)
k expose pod nginx (create the service for pod)
kubectl expose deploy foo --port=6262 --target-port=8080 (create service for deployment)
k get endpoints/ep (get all the enpoints of service that would be pods ip/port)
k edit svc nginx (edit the service)

```
run a temp pod and run wget command to ip 
```
kubectl run busybox --rm --image=busybox -it --restart=Never -- sh
wget -O- IP:80 (O is caps)
exit
```
3. Load Balancer

Cloud service to distribue load across front end pods tier

### Ingress 

To expose your application externally you use service node port but that means you need know ip of node and node port as port
To solve this, you configure dns entry so you dont have to type ip address and use another proxy-server in between with port 80

www.myapp.com -> dns -> proxy-server (port 80) -> node (ip/node port) -> service (type: NodePort) -> pods (targetPort)

works fine if you are hosted on single server on-prem

For cloud env
instead of node port use load balancer service type. This creates node port as well create network level load balancer e.g. gcp load balancer

www.myapp.com -> dns -> gcp load balancer (port 80) -> node (ip/node port) -> service (type: LoadBalancer) -> pods (targetPort)

But if you have to deploy new apps of pods then you need to configure new load balancers each time that is expensive as well need configuration/deployment change every time you add a new url.

Thus introduce ingress, it can let you configure different url path as proxy server with ssl capabilities
So think it as a layer 7 load balancer built-in k8s cluster

Still Ingress needs to be expose externally with load balancer


www.myapp.com -> dns -> gcp load balancer (port 80) -> 
                  Ingress service(Load balancer) - SSL and url routing
                        |                                |
                 service app1 -> pods (targetPort)  service app2 -> pods (targetPort)

K8s doesn't provide you reverse proxy server so you have implement a solution like nginx, gcp load balancer or istio etc. called Ingress Controller
and configuration for it is called Ingress Resources (use yaml file to create)

Ingress Controller for nginx can be deployed as another deployment in k8s

For this you need

Service (to expose it) -> Nginx controller (deployment) - configMap for controller configs - Service Account to access other k8s resources

Ingress Resource is used to defined the url redirecting for ingress controller

www.my-app.com/watch goes to  serviceA whereas www.my-app.com/wear goes to serviceB.

ingress-resource.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: name-virtual-host-ingress
spec:
  rules:
  - host: first.bar.com
    http:
      paths:
      - backend:
          serviceName: service1
          servicePort: 80
  - host: second.foo.com
    http:
      paths:
      - backend:
          serviceName: service2
          servicePort: 80
  - http:
      paths:
       - path: /wear
         backend:
            serviceName: service3
            servicePort: 80
       - path: /watch   
         backend:
            serviceName: service4
            servicePort: 80
```

Ingress commands
```
k get ingress --all-namespaces (get list of ingress controllers)
k describe ingress -n app-space (describe the ingress controller)
k edit ingress -n app-space (edit the ingress controller)
```
### Network Policies

Ingress and Egress
Traffic coming to server is ingress and going out is egress, response of request doesn't matter

Network security
All the pods across nodes have vpn (virtual private network) create by k8s allow them to talk 
thus by default k8s is all-allow

web pod -> Api pod -> db pod (all of them can talk to each other)
but if we dont want web pod to communicate to db pod
Thus use network policy to allow traffic to db pod only from API pod -
so only allow ingress to db pod from API pod on port 3600

network policy yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
    ports:
    - protocol: TCP
      port: 5978
```
