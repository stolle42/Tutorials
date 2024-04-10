- [What is Kubernetes (K8s)](#what-is-kubernetes-k8s)
- [Prerequisites](#prerequisites)
- [Tooling and preparation](#tooling-and-preparation)
- [Architecture](#architecture)
- [Hello World](#hello-world)
  - [How to set up minikube](#how-to-set-up-minikube)
  - [Create a pod](#create-a-pod)
  - [How to expose a pod](#how-to-expose-a-pod)
  - [How to create a deployment](#how-to-create-a-deployment)
- [Kubectl tricks](#kubectl-tricks)
  - [How to edit an existing ressource](#how-to-edit-an-existing-ressource)
  - [How to preview an object before creating it](#how-to-preview-an-object-before-creating-it)
- [First steps](#first-steps)
  - [How to store data permanently](#how-to-store-data-permanently)
  - [How to store confidential data](#how-to-store-confidential-data)
    - [How to create and use secretes](#how-to-create-and-use-secretes)
    - [Shortcomings of secrets](#shortcomings-of-secrets)
    - [Alternatives to secrets](#alternatives-to-secrets)
  - [How to group objects together](#how-to-group-objects-together)
    - [Namespaces](#namespaces)
    - [Labels](#labels)
  - [Ressource usage](#ressource-usage)
  - [How to limit memory usage](#how-to-limit-memory-usage)

# What is Kubernetes (K8s)
Kubernetes is a container orchestration tool. It is used to increase availability, scalability and performance of container applications (usually websites). With kubernetes, websites can keep running smoothly even during times of updates, maintenance and high load without the end user noticing a thing. Even all-out server failures can be brushed over with kubernetes.

Kubernetes is superior to traditional, monolithic web application in almost every regard. However it does add complexity to a system and requires skilled engineers to set up and maintain it, so it won't be suitable in every use case. Depending on the specifics, a monolith might still be the better option.
# Prerequisites
- Docker
# Tooling and preparation
Install:
- minikube (a lightweight open-source version of kubernetes missing some features)
# Architecture
The largest Kubernetes ressource is called **cluster**. Each cluster consists of several **nodes** that might run on seperate machines. Each node consists of several **pods**. A pod is the smallest ressource in kubernetes and it consists of one or more Docker-containers (but ususally one).
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
`kubectl` is our kubernetes handle. Get used to it, we need it for basically every kubernetes command. 

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

In order to do this, we need to create a **service**. In Kubernetes, services group some pods together and set a common access policy to them. There are 4 types of services:
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

If now delete the service with
```bash

```
the application should not be available anymore in your browser.
## How to create a deployment
Ok, switching on and off a webservice with a single command is convenient, but we don't need a heavy tool like kubernetes for that. You can achieve the same thing much easiser with a tool like docker. Therefore, the above method of creating pods is rarely used in practise (Nevertheless it is vital for understanding the following concepts).

So how do we create pods in real life? Well, we don't create them directly. Instead, we'll create a **deployment**. 

A deployment is something like a pod supervisor. It creates the pod and ensures its continued existence and availability. 

If the pod terminates, the deployment starts a new one in its place. You can set the deployment to create replicas of the pod. This way, we can increase pod availability and performance due to redundancy (if one of the pods terminates, we can use its replica. If all of them are available, they can share the workload).

Ok, enough theory. Let's get our hands dirty and create a deployment:
```bash
kubectl create deployment hello-deployment --image=nginx
```
Running `kubectl get pods` should show a new pod called `hello-deployment-[id]`. We can check the deployment directly by running
```bash
kubectl get deployments
```
It should show `hello-deployment` and ind the READY-column it should show `1/1`. That means the pod has been created once, without replicas. Let's delete our pod again (`kubectl delete deployment hello-deployment`) and increase the number of pod replicas, thus creating redundancy:
```bash
kubectl create deployment hello-deployment  --replicas=3
```
Check out the deployments again. Does the ready-column show `3/3` now?

Ok but still, all we did was create 3 instances of the same container. We could have done this with Docker. The great thing about kubernetes is its respawing capabilities. Delete one of the pods from `hello-deployment` (you already know how to do that).

Now check out the pods again. You should still see 3 pods of the form `hello-deployment-[id]`, but in the age-column, you should see that one of them is younger than the other two. This one should also have an id that wasn't there earlier.

That is the actual power of kubernetes. No matter what happens to the pod, the deployment will always be there and spawn a new one. This cannot be achieved with a simple tool like Docker. And you don't want to write time-consuming, error-prone script for this, do you? Especially once you learn that Kubernetes has many more features making our lives easier ready to explore.
# Kubectl tricks
## How to edit an existing ressource
You might have noticed that every time we wanted to change some settings in a ressource (e.g. a pod), we had to delete it and relaunch it with new settings. This is not only tedious, but unaccaptable in a production context. There is a better way: If you want to change settings, run
```bash
kubectl edit [ressource]
```
This will open a yaml-configuration file in vim (also called object **manifest**). Here, you can edit a plethora of settings that might seem intimidating at first. But don't worry, we'll get there.

Some of the settings might already seem familiar. Can you spot the `replicas`-setting in the deployment configuration? Try to change the number. Now close the file. Kubernetes should apply the new settings automatically. Check if it worked. Does `kubectl get deployments` show another number of replicas now?
## How to preview an object before creating it
What if we want to check if one of our commands has the right syntax without actually creating an object? Well, there is a flag for that: `dry-run=client`.

This is more useful when paired with the object preview flag: `-o yaml`(change yaml to json if you prefer that). Try to get a grasp of it by running
```bash
kubectl run pod hello-pod --dry-run=client --image=nginx -o yaml
```
This command should not affact your cluster in any way.

# First steps
## How to store data permanently
Websites need to store data permanently. A website without a database is pretty useless in this day and age. But how do we do that in kubernetes? Storing it in a pod is unacceptable due to several reasons:
- if a pod fails, it will be restarted, but all its data will be lost
- pods are mostly replicated. But if every pods stores different data, the website will be hillariously inconsistent. Depending on what pod you happen to land on, you'll get different results.

Luckily, kubernetes offers an object for this use case: **volumes**.
## How to store confidential data
Websites and other programs frequently need to handle and store data that should not be viewed by the public. In Kubernetes, this can be achieved with **secrets**.
### How to create and use secretes
### Shortcomings of secrets
### Alternatives to secrets
## How to group objects together
Just like in every other field of computer science (and in life), a high number of certain objects make it harder for the user to keep an overview and find single objects. That's why we group files together in folders, or code in classes and functions. Organization just makes our lives easier.

Kubernetes is no exception. In big projects, it is not uncommon to have hundrets of pods, deployments, services etc. And worse yet - several projects might have to share the same cluster to reduce cost. Obviously, managing a cluster like that with no grouping would be vastly chaotic.
Luckily, kubernetes offers 2 ways to group objects together: **Namespaces** and **labels**.
### Namespaces
### Labels

## Ressource usage
High cpu or memory usage (e.g. due to memory leaks) can be a huge problem for traditonal web servers. If a program keeps allocating more and more storage, several problems can occur that are often hard to spot and to solve.

Kubernetes offers a way to confine this problem and limit its repercussions: object ressource limits. An object (e.g. a pod) can be limited to a certain amount of memory or cpu usage. That way, the host system will not be affected making the solution process easier and in many cases limiting the scope of the problem to a single pod.
## How to limit memory usage
When changing a pod's config (`kubectl edit hello-deployment-[id]`), you'll find the limit settings in the `spec.containers[].resources`-section. It will look something like this:
```yaml
resources:
      limits:
        memory: 100Mi
      requests:
        memory: 100Mi
```
Ok, what do these settings mean?
- limits: Maximum amount of memory the pod is allowed to use. Exceeding the limit will cause an OOM-Error.
- requests: Minimum memory amount reservation. Kubectl guarantees that at least this amount will be available for the pod at any time.

 Since we never specified any limits, `100Mi` (100 Megabytes) was used as a default value. By the way - requests can never be greater than limits, because obviously.

Let's try to change the limits now. If you already tried changing the limit while editing, you should have gotten an error message. Thats because pod settings should never be edited directly. It would cause inconsistencies among replicated pods. Also all pod setting changes will be lost after a pod replacement anyway.

So let's edit the deployment settings directly:
```bash
kubectl edit deployment hello-deployment 
```
Look for the `spec.templates.spec.containers[].resources`-map. If you haven't edited it yet, it should be empty. Now we can insert our own limits and requests, e.g.
```yaml
      limits:
        memory: 10Mi
      requests:
        memory: 5Mi
```
Have a look into the pod settings. Did they get updated?

You can try to occupy enough storage now to push the container to its limits, but be warned - the pod filesystem is rather confusing and won't be part of this introduction.

You can limit cpu usage, too. There is a special unit for this called `Millicore`. 1000 Millicores equal 1 CPU core. Limiting the cpu usage to 10% of the host machine's processing power is done like this:
```yaml
      limits:
        cpu: 100m
      requests:
        memory: 100m 
```
Todo: Check object status to get ressource usage