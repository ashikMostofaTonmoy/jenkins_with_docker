## Kubernatis (K8S)

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
kubectl get service
kubectl edit deployment <name-of-deployment>
kubectl describe pod <pod-name>
kubectl logs <pod-name> # for debugging
kubectl exec -it <pod name> -- / bin/bash # to get into the container
kubectl delete deployment <deployment-name> # to delete deployment 

# but mainly we use to apply all this 
kubectl apply -f <filename.yaml> #to up  the deployment from file
kubectl delete -f <filename.yaml> #to delete the deployment from file
```

* To keep secret text / db credential / other password in config file 1st change the text in `base64` formate from terminal. Then put the text in the config file. so that other people gets confused :D.

```sh
echo -n 'ashik' | base64
```

The secretss should be up before dploying other things. otherwise they won't find where to reference.

`---` this shows that `yaml` file is seperated . it's possible to put two dirrerent file in a monolith file. but they are treated as different files 

* Default `Namespace` should be kept clean. 
* It's better to group different services and tasks in a seperate namespace.
* a `Namespace` can be used by multiple namespaces.
* Each `Namespace` has their own configmap, secrets.

To create namespace:
```sh
kubectl create namespace my-ns
```
> By default everything is installed or configured in `default `namespace.

To change default namespace you may need `kubectx`. To install kubectx
```sh
sudo add-apt-repository 'deb [trusted=yes] http://ftp.de.debian.org/debian buster main' # adding repository
sudo apt update
sudo apt install kubectx
```
If public key is missinng when `sudo apt update`, add public key by 
```sh
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys DCC9EFBF77E11517 \
# change `DCC9EFBF77E11517` value according to terminal.
```
use `kubens` to list all available namespace
```sh
kubens
kubens <custome-namespace>
```` 
This will change default name space to custome namespace. If you need to check resource from other namespace  use commands like this:
```sh
kubectl get pod -n <default/name-space>
```
### setup `Ingress`

install ingress controller in miniKube

```sh
minikube addons enable ingress
```
this automatically implements k8s `nginx` implementation of ingress controller.