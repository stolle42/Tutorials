- [What is Kubernetes (K8s)](#what-is-kubernetes-k8s)
- [Prerequisites](#prerequisites)
- [Tooling and preparation](#tooling-and-preparation)
- [Architecture](#architecture)
- [Hello World](#hello-world)
  - [Hello Pod](#hello-pod)
  - [Hello Deployment](#hello-deployment)
  - [How to expose a service](#how-to-expose-a-service)

# What is Kubernetes (K8s)
Kubernetes is a container orchestration tool. It is used to increase availability, scalability and performance of container applications (usually websites). With kubernetes, websites can keep running smoothly even during times of updates, maintanence and or load without the end user noticing a thing.
# Prerequisites
- Docker
# Tooling and preparation
Install:
- minikube (a lightweight open-source version of kubernetes missing some features)
# Architecture
A Kubernetes entity is called **cluster**. Each cluster consists of several **nodes** that might run on seperate machines. Each node consists of several **pods**. Each pod consists of one or more Docker-containers (but ususally one).
# Hello World
## Hello Pod
The command
```
    minikube start
```
launches a kubernetes cluster with one single node. We won't need more than that for this tutorial.

Minikube offers a nice visualization dashboard. In order to view it, run
```
minikube dashboard
```
The dashboard comes in very handy whenever we want to get an overview over the current status of our cluster. This tutorial uses the command line to achieve the same thing, but feel free to check out the dashboard from time to time.

Currently the dashboard will be empty, because we have not yet created a pod. Let's do this now! Run 
```
kubectl run helloPod --image=kicbase/echo-server
```
`kubectl` is our kubernetes handle. We need it for basically every kubernetes command. 

`run` is the command we need to launch a pod.

`helloPod` is the name of our pod.

`kicbase/echo-server` is the image of our pod. It is a webserver returning everything we sent to it. It has no purpose outside a debugging/demonstation context.

You can check the status by running 
```
kubectl get pods
```
The status should be `pending` for the first few seconds, then switch to `running`. If should not show `failed` or any other errors.

Let't check if the echo-server is working. https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/
## Hello Deployment
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
## How to expose a service
So far, the containers have been closed of in kubernets, unavailable to the ouside world. In order to actually benefit from the power of kubernetes, the containers need to be exposed, so they can act as web servers. 