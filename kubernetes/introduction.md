- [What is Kubernetes (K8s)](#what-is-kubernetes-k8s)
- [Prerequisites](#prerequisites)
- [Tooling and preparation](#tooling-and-preparation)
- [Architecture](#architecture)
- [Hello World](#hello-world)
  - [How to set up minikube](#how-to-set-up-minikube)
  - [Create a pod](#create-a-pod)
  - [How to expose a pod](#how-to-expose-a-pod)
  - [How to create a deployment](#how-to-create-a-deployment)

# What is Kubernetes (K8s)
Kubernetes is a container orchestration tool. It is used to increase availability, scalability and performance of container applications (usually websites). With kubernetes, websites can keep running smoothly even during times of updates, maintenance and high load without the end user noticing a thing. Even all-out server failures can be brushed over with kubernetes.
# Prerequisites
- Docker
# Tooling and preparation
Install:
- minikube (a lightweight open-source version of kubernetes missing some features)
# Architecture
The largest Kubernetes unit is called **cluster**. Each cluster consists of several **nodes** that might run on seperate machines. Each node consists of several **pods**. A pod is the smallest unit in kubernetes and it consists of one or more Docker-containers (but ususally one).
# Hello World
## How to set up minikube
The command
```
    minikube start
```
launches a kubernetes cluster with one single node. Before we get our hands dirty, let's have a look at the minikube dashboard. This is a vizualization tool that comes in very handy whenever we want to get an overview over the current status of our cluster. Run
```
minikube dashboard
```
In theory, you don't need the dashboard. This tutorial will use the command line for everything, but feel free to check out the dashboard from time to time. It can help a lot in understanding what is going on.
## Create a pod
Currently the dashboard will be empty, because we have not yet created a pod. Let's do this now! Run 
```
kubectl run hellopod --image=nginx
```
`kubectl` is our kubernetes handle. We need it for basically every kubernetes command. 

`run` is the command we need to launch a pod.

`hellopod` is the name of our pod.

`nginx` is the image for our pod. Why do we need that? Well a pod has no benefit without a running container in it, which is why we cannot create empty pods. So kubernetes requires us to provide some image which it then pulls from Dockerhub, builds a container, and runs said container inside the pod.

If you understand docker well enough, create a simple website, push it to dockerhub and replace `nginx` with your image. Otherwise, this tutorial will work, too, but it won't be as much fun, since your not using your stuff built by your own.

You can get an overview of your pods by running 
```
kubectl get pods
```
Watch the status column. The status of your pod should be `pending` for the first few seconds, then switch to `running`. If should not show `failed` or any other errors.
## How to expose a pod
So far, the containers have been closed of in kubernets, unavailable to the ouside world. In order to actually benefit from the power of kubernetes, the containers need to be exposed, so they can act as web servers we can access with a browser.

In order to do this, we need to create a service. In Kubernetes, services group some pods together and set a common access policy to them. There are 4 types of services:
- ClusterIP: Service is only accessible from inside the cluster
- NodePort: Service is accessible from outside the cluster on a specific port
- LoadBalancer: Service is accessible from outside the cluster via an external program that tries to divide the traffic to the endpoints equally
- ExternalName: Service is accessible on a DNS-name

In our case, we will expose one port of a pod with a NodePort-service:
```bash
kubectl expose pod hellopod --port=80 --type=NodePort
```
Let's verify this worked. For this, we need 2 things: The cluster ip-adress and the port our pod is accessible at. First, let's check out the ip-adress of our cluster:
```bash
minikube ip
```
Then we check the port. This can be done by running
```bash
kubectl get service
```
This shows every currently running service. The service we created should have the name "hellopod". The port we are looking for is found in the PORT(S)-column behind the colon. 

Head over to your browser and open
```
[cluster ip]:[exposed port]
```
If it shows "Welcome to nginx" (or your own application if you used it), great! You have sucessfully integrated an application in kubernetes!
## How to create a deployment
The above method of creating pods is the most simple one, but it is rarely used in practice. It is much more common to create a **deployment**. 

A deployment can be imagined as a pod supervisor. It creates the pod and makes sure it keeps running. If the pod terminates, the deployment restarts it. You can set the deployment to create replicas of the pod. This way, we can  increase pod availability due to redundancy (if one of the pods terminates, we can use its replica).

Let's create a deployment:
```bash
kubectl create deployment hello-deployment --image=kicbase/echo-server
```
`kubectl get pods` should now show a new running pod called something like `hello-deployment-7dd5559ccf-mnqj6`. We can check the deployment directly by running
```
kubectl get deployments
```
It should show hello-deployment as ready with a marker 1/1. That means the pod has been created only once, with no replicas. We will later see (which chapter?) how we use replicaSets to increase the number of pods, thus creating redundancy.
