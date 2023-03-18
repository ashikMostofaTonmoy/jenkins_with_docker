* to check everythin locally , install `minikube` , a choosen hypervisor according to the os you are on
* install `kubectl` if not installed by default
* you may need to restart/logout the pc to take the change effect
  
  >In this case we are using `kvm2` hypervisor   

```sh
minikube start --driver=kvm2
```
```sh
kubectl get nodes # this gets status of the nodes
minikube status 
```

> `kubectl cli` for configuring minikube cluster
> `minikube cli` for up/deletiog cluster

### Some basic commands for K8S

```sh
kubectl get pod
kubectl get services
kubectl create # ... to to create something
kubectl create deployment nginx-demo --image=nginx # to create deployment
kubectl get deployment 
kubectl get replicaset
kubectl edit deployment <name-of-deployment>
kubectl describe pod <pod-name>
kubectl logs <pod-name> # for debugging
kubectl exec -it <pod name> -- / bin/bash # to get into the container
kubectl delete deployment <deployment-name> # to delete deployment 

# but mainly we use to apply all this 
kubectl apply -f <filename.yaml>
```
