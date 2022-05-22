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
```shell
# Create hello-nginx POD from nginx Docker image
kubectl run hello-nginx --image=nginx
# It's there (READY X/Y means that there are X running containers of total Y containers in the pod)
kubectl get pods
# Extended info
kubectl get pods -o wide
# Even more info
kubectl describe pods
# If if want only specific POD
kubectl describe pod hello-nginx
# Cleanup
kubectl delete pod hello-nginx

# Create pod-hello-nginx POD from YAML file 
kubectl create -f pod-hello-nginx.yml
# OR
# https://www.theserverside.com/answer/Kubectl-apply-vs-create-Whats-the-difference
# https://www.containiq.com/post/kubectl-apply-vs-create
# https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create
# kubectl apply -f pod-hello-nginx.yml
kubectl get pods
kubectl describe pod pod-hello-nginx
# Cleanup
kubectl delete pod pod-hello-nginx

# With dry run we can just generate YAML file (purposely wrong redis123 image here)
kubectl run hello-redis --image=redis123 --dry-run=client -o yaml > pod-hello-redis.yml
# Then we create POD from that file
kubectl create -f pod-hello-redis.yml
# We can see it's not running and ErrImagePull status (redis123 image doesn't exist)
kubectl get pods
# Edit and fix redis123 to redis correct image
kubectl edit pod hello-redis
# It's now running correctly
kubectl get pods
# Cleanup
kubectl delete pod hello-redis
```

## Replication Controllers & ReplicaSets
```shell
# Replication Controllers is older way of controlling replicas
kubectl create -f rc-hello-nginx.yml
kubectl get replicationcontrollers
kubectl get pods
# Delete replication controller (and all underlying PODs)
kubectl delete replicationcontrollers rc-hello-nginx
kubectl get replicationcontrollers
kubectl get pods

# ReplicaSets is newer way of controlling replicas
kubectl create -f rs-hello-nginx.yml
kubectl get replicaset
kubectl get pods
# Delete one of the pods
kubectl delete pod rs-hello-nginx-7zz2n
# And then we see that replicaset created new one to have 3 of them always
kubectl get pods
# And if we create POD automatically with that same label, replicaset will terminate it immediately to maintain desired numbers of PODs
# Describe also works for replicaset
kubectl describe replicaset rs-hello-nginx
# After updating replicas from 3 to 6 in yml file (one way to scale)
kubectl replace -f rs-hello-nginx.yml
kubectl get replicaset
kubectl get pods
# Another way to scale (scaling back to 3)
kubectl scale --replicas=3 -f rs-hello-nginx.yml
kubectl get replicaset
kubectl get pods
# Yet Another way to scale
kubectl scale --replicas=2 replicaset rs-hello-nginx
kubectl get replicaset
kubectl get pods
# Also edit could be used
kubectl edit replicaset rs-hello-nginx
# Delete replicaset (and all underlying PODs)
kubectl delete replicaset rs-hello-nginx
kubectl get replicaset
kubectl get pods
```

## Deploymentds
TODO