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

cd hello
```

[Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/) is a tool built to provide best-practice for creating Kubernetes clusters.

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

## Deployments
```shell
kubectl create -f deployment-hello-nginx.yml
kubectl get deployments
kubectl get replicaset
kubectl get pods
# Or easier to see all with one command
kubectl get all
# Describe
kubectl describe deployment deployment-hello-nginx
# Delete deployment (and all underlying objects)
kubectl delete deployment deployment-hello-nginx
kubectl get all
```
Let's now see how Update & Rollback work:
```shell
# This will create 6 replicas
kubectl create -f deployment-hello-nginx.yml
# Checking status of the rollout (executing this immediately after the command above to see rolling out in progress)
kubectl rollout status deployment.apps/deployment-hello-nginx
# Check rollout history
kubectl rollout history deployment.apps/deployment-hello-nginx
# By default deployment creation doesn't record rollout cause, so let's delete and create it again with this enabled
kubectl delete deployment deployment-hello-nginx
kubectl create -f deployment-hello-nginx.yml --record
# Now, there is a change cause present
kubectl rollout history deployment.apps/deployment-hello-nginx
# Let's now edit deployment to change image from latest nginx to older nginx:1.18
kubectl edit deployment deployment-hello-nginx --record
# Checking status of the rollout (executing this immediately after the command above to see rolling out in progress)
# Now we can see that new replicas are being created, but old one are being destroyed (RollingUpdate (updating 1 by 1) is default strategy)
kubectl rollout status deployment.apps/deployment-hello-nginx
# Now we can see two revisions in the history
kubectl rollout history deployment.apps/deployment-hello-nginx
# We can also check the events section here, to see the rolling update
kubectl describe deployment deployment-hello-nginx
# Let's make one more change with another way, now to perl version of nginx
kubectl set image deployment deployment-hello-nginx hello-nginx-container=nginx:1.18-perl --record
kubectl rollout history deployment.apps/deployment-hello-nginx
# Let's now see how rollout works, we want to rollout to 2nd revision with nginx:1.18 (no perl)
kubectl rollout undo deployment/deployment-hello-nginx
kubectl rollout status deployment.apps/deployment-hello-nginx
# We will now see that 2nd revision now disappeared and became 4th revision
kubectl rollout history deployment.apps/deployment-hello-nginx
# And make sure we are actually using the correct image (nginx:1.18)
kubectl describe deployment deployment-hello-nginx
# Now, let's imagine that we update again the image to some non-existent 
# In that case rollout will start, but fail and Kubernetes terminated just 1 of 6 PODs and got stuck there
# But the app is still operational and we can rollback to desired revision to fix this and get all 6 replicas back
```

## Services
```shell
# We have deployment already in place
kubectl get deployments
# With 6 PODs with label app: hello-nginx
kubectl get pods
# Let's create NodePort service
kubectl create -f service-hello-nodeport.yml
kubectl get services
# OR
kubectl get svc
# To get the URL
# minikube service hello-nodeport-service --url
# But it doesn't work on M1
# https://github.com/kubernetes/minikube/issues/9016
# So:
kubectl port-forward service/hello-nodeport-service 30008:80
curl http://localhost:30008

# Let's now create ClusterIP service, after this, other layers (like frontend) could use "backend" layer
# ClusterIP is default type if not specified
kubectl create -f service-hello-clusterip.yml

# LoadBalancer service is utilizing supported cloud services (AWS, Azure, GCP...)
# If not on supported cloud, this type would behave exactly the same as NodePort
kubectl create -f service-hello-loadbalancer.yml

# We can describe services
kubectl describe service
kubectl describe service hello-loadbalancer-service

# Cleanup
kubectl delete deployment deployment-hello-nginx
delete service hello-nodeport-service
delete service backend
delete service hello-loadbalancer-service
```

## Microservices - Voting App
```shell
# First iteration of voting app with PODs + Services, just warming up
cd voting-app-1

# Port 80
kubectl create -f voting-app-pod.yaml
# NodePort 30004
kubectl create -f voting-app-service.yaml
# Checking
kubectl get pods,svc
# Not working on M1
minikube service voting-service --url
# So:
kubectl port-forward service/voting-service 30004:80
curl http://localhost:30004
^+C

# Port 6379
kubectl create -f redis-pod.yaml
# ClusterIP by default, internal redis
kubectl create -f redis-service.yaml
# Checking
kubectl get pods,svc

# Port 5432
kubectl create -f postgres-pod.yaml
# ClusterIP by default, internal db
kubectl create -f postgres-service.yaml
# Checking
kubectl get pods,svc

# No port, just a worker, no service..
kubectl create -f worker-app-pod.yaml
kubectl get pods

# Port 80
kubectl create -f result-app-pod.yaml
# NodePort 30005
kubectl create -f result-app-service.yaml
# Checking
kubectl get pods,svc
# Not working on M1
minikube service result-service --url
# So:
kubectl port-forward service/result-service 30005:80
curl http://localhost:30005
^+C

# Cleanup
kubectl delete pod postgres-pod
kubectl delete pod redis-po
kubectl delete pod result-app-pod
kubectl delete pod voting-app-pod
kubectl delete pod worker-app-pod
kubectl delete service db
kubectl delete service redis
kubectl delete service result-service
kubectl delete service voting-service
# Checking
kubectl get pods,svc
```

```shell
# Second iteration of voting app with Deployments + Services, this is the way to go
cd voting-app-2

# Port 6379
kubectl create -f redis-deploy.yaml
# ClusterIP by default, internal redis
kubectl create -f redis-service.yaml
# Checking
kubectl get all

# Port 5432
kubectl create -f postgres-deploy.yaml
# ClusterIP by default, internal db
kubectl create -f postgres-service.yaml
# Checking
kubectl get all

# Port 80
kubectl create -f voting-app-deploy.yaml
# NodePort 30004
kubectl create -f voting-app-service.yaml
# Checking
kubectl get all
# Not working on M1
minikube service voting-service --url
# So:
kubectl port-forward service/voting-service 30004:80
curl http://localhost:30004
^+C

# No port, just a worker, no service..
kubectl create -f worker-app-deploy.yaml
kubectl get all

# Port 80
kubectl create -f result-app-deploy.yaml
# NodePort 30005
kubectl create -f result-app-service.yaml
# Checking
kubectl get all
# Not working on M1
minikube service result-service --url
# So:
kubectl port-forward service/result-service 30005:80
curl http://localhost:30005
^+C

# Cleanup
kubectl delete deployment postgres-deploy
kubectl delete deployment redis-deploy
kubectl delete deployment result-app-deploy
kubectl delete deployment voting-app-deploy
kubectl delete deployment worker-app-deploy
kubectl delete service db
kubectl delete service redis
kubectl delete service result-service
kubectl delete service voting-service
# Checking
kubectl get all
```

## Amazon Elastic Kubernetes Service (AWS EKS)
- [EKS Docs](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Getting Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)
- [eksctl](https://eksctl.io)

There are two ways for creating Kubernetes cluster on AWS:
- [AWS Management Console & AWS CLI](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html)
- [Using eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)

We will use `eksctl` here.

```shell
brew tap weaveworks/tap
brew install weaveworks/tap/eksctlbrew install weaveworks/tap/eksctl
eksctl version
```





























