# Kubernetes Playground

- Chip Apple **M1** Max
- Docker Desktop
- [Minikube](https://minikube.sigs.k8s.io/docs/start) (docs not fully compatible with M1)
- IntelliJ IDEA Ultimate with [Kubernetes plugin](https://www.jetbrains.com/help/idea/kubernetes.html)

## Getting started
We will use Minikube (with Docker) to install single-node Kubernetes cluster locally.
```shell
# This will install both minikube and kubectl as its dependency
brew install minikube
# This will start the Kubernetes cluster and configure kubectl to use it 
minikube start
# Let's test if everything works
kubectl create deployment hello-kubernetes --image=k8s.gcr.io/echoserver-arm:1.8
kubectl expose deployment hello-kubernetes --type=NodePort --port=8080
kubectl get deployments
kubectl get services hello-kubernetes
# After this, you should be able to open http://localhost:7080
kubectl port-forward service/hello-kubernetes 7080:8080
# Cleanup
^+C
kubectl delete services hello-kubernetes
kubectl delete deployment hello-kubernetes
```
One alternative to Minikube is single-node Kubernetes cluster via Docker Dekstop, but we will stick with Minikube.

Let's execute some basic commands:
```shell
minikube status
minikube stop
# Destroys everything
minikube delete
minikube start  	 
minikube dashboard
^+C
# Client and server versions
kubectl version
kubectl cluster-info
# Nodes in the cluster
kubectl get nodes
kubectl get nodes -o wide
# To see cluster components
kubectl get po -A 
```

## PODs

kubectl run nginx --image nginx
kubectl run nginx --image=nginx
^ komanda na ovu foru bi za nas kreirala POD i u taj pod instalirala nginx docker container koji je skinula sa docker hub repoa (--image nginx)
nginx posle run je pod name i moze biti bilo sta, ali nginx posle --image je ime image-a na docker registry koji gadjamo
Posto nismo specifirali neki drugi docker registry, default je Docker Hub
kubectl get pods
^ vidimo postojece PODove
kubectl get pods -o wide
kubectl describe pods
kubectl describe pod nginx
Btw rezultat komande: kubectl get pods - READY 1/2 na primer znaci:
  Running containers in POD / Total container in POD
  Znaci 1/2 znaci da 1 od dva kontejnera je running / ready...
kubectl delete pod POD_NAME # ovo brise pod POD_NAME
Ovaj primer je zanimljiv da vidimo a) --dry-run koji prakticno ne pravi stvarno nego puca u fajl i b) edit/apply:
  kubectl run redis --image=redis123 --dry-run=client -o yaml > redis-definition.yaml
  kubectl create -f redis-definition.yaml
  kubectl edit pod redis
  kubectl apply -f redis-definition.yaml

```shell



kubectl create -f pod-definition.yml
# OR
# kubectl apply -f pod-definition.yml
kubectl get pods
kubectl describe pod myapp-pod
```

## rc and rs

Controlers - oni su mozak Kubernetesa oni monitoruju objekte i reaguju odgovarajuce, imas vise vrsta..
Replication Controllers & ReplicaSets
  replication controler prakticno sluzi da omoguci HA, ako trokne POD, on ga zameni sam automatski!
  znaci on se brine da je u svakom trenutku broj POD-va aktivan koliko je specificirano!
  takodje radi load balancing and scaling, replication controller radi na celom clusteru i sa vise nodove bez problema..
  ReplicaSet je noviji nacin kako se radi replication, dok je Replicatin Controller stariji
  Tako da znamo da je to isto, ali je preporuceni i moderniji nacin ReplicaSets, tako da se drzimo toga!
  Labels and Selector - replica set monitors the pods to be able to maintain desired heatlhy number of PODS
  it knows which PODs to monitor with selector and labels
  so labels are this way a filter for replica set to scan only those PODS that we are interested in

```shell
kubectl create -f rc-definition.yml
kubectl get replicationcontrollers
kubectl get pods
# Delete replication controller (and all underlying PODs)
kubectl delete replicationcontrollers myapp-rc
kubectl get replicationcontrollers
kubectl get pods
```

```shell
kubectl create -f replicaset-definition.yml
kubectl get replicaset
kubectl get pods
# Delete one of the pods
kubectl delete pod myapp-replicaset-8nxxl
# And then we see that replicaset created new one to have 3 of them always
kubectl get pods
# And if we create POD automatically with that same label, replicaset will terminate it immediately to maintain desired numbers of PODs
# Describe also works for replicaset
kubectl describe replicaset myapp-replicaset
# After updating replicas from 3 to 6 in yml file (one way to scale)
kubectl replace -f replicaset-definition.yml
kubectl get replicaset
kubectl get pods
# Another way to scale (scaling back to 3)
kubectl scale --replicas=3 -f  replicaset-definition.yml
kubectl get replicaset
kubectl get pods
# Yet Another way to scale
kubectl scale --replicas=2 replicaset myapp-replicaset
kubectl get replicaset
kubectl get pods
# Also edit could be used
kubectl edit replicaset myapp-replicaset
# Delete replicaset (and all underlying PODs)
kubectl delete replicaset myapp-replicaset
kubectl get replicaset
kubectl get pods
```