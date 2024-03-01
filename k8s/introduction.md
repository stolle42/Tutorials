- [What is Kubernetes (K8s)](#what-is-kubernetes-k8s)
- [Prerequisites](#prerequisites)
- [Tooling and preparation](#tooling-and-preparation)
- [Architecture](#architecture)
- [Hello World](#hello-world)
  - [Hello Pod](#hello-pod)

# What is Kubernetes (K8s)
Kubernetes is a container orchestration tool. It is used to increase availability, scalability and performance of container applications (usually websites).
# Prerequisites
- Docker
# Tooling and preparation
Install:
- minikube (a lightweight open-source version of kubernetes missing some features)
# Architecture
A Kubernetes entity is called **cluster**. Each cluster consists of several **nodes**. Each node consists of several **pods**. Each pod can be filled with one or more Docker-containers (but ususally one).
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
Let't break down this command:

`kubectl` is our kubernetes handle. We need it for every kubernetes command. 

`run` is the command we need to launch a pod.

`helloPod` is the name of our pod.

`kicbase/echo-server` is the image of our pod. It is a  server returning every command we send to it.

You can check the status by running 
```
kubectl get pods
```
The status should be `pending` for the first few seconds, then switch to `running`. If it shows `failed` or any other errors, it was not successful.