# CKAD-exam-notes
CKAD Exam Notes and Tips

* Course I used for prepration: https://www.udemy.com/course/certified-kubernetes-application-developer/
* Practice repo for prepration: https://github.com/dgkanatsios/CKAD-exercises (practice it 4-5 times)
* Another practice repo: https://github.com/bmuschko/ckad-prep
* Book I referenced : https://www.manning.com/books/kubernetes-in-action
* I used katacode heavily to practice : https://www.katacoda.com/courses/kubernetes/playground

# Time saving tips

* [vim shortcuts](vimtricks.md#section)

* Create shortcuts for kubectl commands
```
alias k=kubectl
complete -F __start_kubectl k (auto completes k)
```
* Set context for env

```
kubectl config set-context <context of qn> --namespace <namespace of qn>
kubectl config set-context $(kubectl config current-context) --namespace abc (switching context to ns abc)
kubectl config view | grep namespace (verify namespace context)
```
* Kubectl explain command

```
k explain pod.spec
k explain pod.spec.containers --recursive
k explain pod.spec | grep -i nodeselector (to confirm the attribute)
k explain pod.spec.nodeselector
```
* Kubectl help command

```
k annotate --help | head -30
```

* Grep from output

```
k describe pod <pod-name> | grep -i events -A 10
k describe pod <pod-name> | grep -i events -A 10 > events.txt (output it to file)
k describe pod <pod-name> | grep --context=10 Events: (output 10 lines above and below) 
```
The -A option essentially means 'after,' so you're saying give me the search results that start with 'events' and then the next 10 lines too.

```
k describe pods | grep -C 10 "author=John Doe" (grep surrounding 10 line of annotation)
k get pods -o yaml | grep -C 5 labels: (Print labels of all the pods in yaml output)
```

* Get root privileges if need to login into another node
```
sudo -i
```
* Bookmark kubernetes.io documentation important pages  

* [bash commands](bashcommands.md#section)

# Curriculam 

- [ ] __Core Concepts - 13%__
  - API Primitives
  - Basic Pods
  - Replica Sets
  - Deployments
  - Namespaces
- [ ] __Configuration - 18%__
  - Command and Arguments
  - ConfigMaps
  - Secrets
  - SecurityContexts
  - Resource Requirements
  - Service Accounts
  - Taints and Tolerations
- [ ] __Multi-Container Pods - 10%__
  - Design Patterns: Ambassador, Adapter, Sidecar
- [ ] __Pod Design - 20%__
  - Labels, Selectors, and Annotations
  - Deployments and Rolling Updates
  - Jobs and CronJobs
- [ ] - __State Persistence - 8%__
  - Volumes and Mounts
  - Persistent Volume and Claims
- [ ] __Observability - 18%__
  - Liveness and Readiness Probes
  - Container Logging and Debugging
- [ ] __Services and Networking - 13%__
  - Services
  - Ingress
  - Networking Policies

## Notes

* [core concepts](CoreConcepts/README.md#section)
* [configuration](Configurations/README.md#section)
* [multi container pods](Multi-ContainerPods/README.md#section)
* [pod design](PodDesign/README.md#section)
* [state persistence](StatePersistence/README.md#section)
* [observability](Observability/README.md#section)
* [services and networking](ServicesAndNetworking/README.md#section)