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

Let's get some basic commands:
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

TODO

```shell
kubectl create -f pod-definition.yml
# OR
# kubectl apply -f pod-definition.yml

kubectl get pods

kubectl describe pod myapp-pod
```

## rc and rs

TODO

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