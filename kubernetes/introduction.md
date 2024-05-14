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
  - [Ressource overview](#ressource-overview)
  - [How to edit an existing ressource](#how-to-edit-an-existing-ressource)
  - [How to preview an ressource before creating it](#how-to-preview-an-ressource-before-creating-it)
  - [How to create a ressource from manifest](#how-to-create-a-ressource-from-manifest)
  - [How to work inside pods](#how-to-work-inside-pods)
  - [How to deploy busybox](#how-to-deploy-busybox)
  - [How to change specific settings without opening the manifest file](#how-to-change-specific-settings-without-opening-the-manifest-file)
  - [How to sort objects from kubectl get](#how-to-sort-objects-from-kubectl-get)
  - [How to edit a running ressource](#how-to-edit-a-running-ressource)
- [First steps](#first-steps)
  - [Volumes](#volumes)
    - [How to store data permanently](#how-to-store-data-permanently)
    - [How to share data in pods with multiple containers](#how-to-share-data-in-pods-with-multiple-containers)
  - [How to store confidential data](#how-to-store-confidential-data)
    - [How to create and use secretes](#how-to-create-and-use-secretes)
    - [Shortcomings of secrets](#shortcomings-of-secrets)
    - [Alternatives to secrets](#alternatives-to-secrets)
  - [How to group objects together](#how-to-group-objects-together)
    - [Namespaces](#namespaces)
    - [Labels](#labels)
  - [How to adapt deployment size to current demand](#how-to-adapt-deployment-size-to-current-demand)

# What is Kubernetes (K8s)
Kubernetes is a container orchestration tool. It is used to increase availability, scalability and performance of container applications (usually websites). With kubernetes, websites can keep running smoothly even during times of updates, maintenance and high load without the end user noticing a thing. Even all-out server failures can be brushed over with kubernetes.

Kubernetes is superior to traditional, monolithic web application in almost every regard. However it does add complexity to a system and requires skilled engineers to set up and maintain it, so it won't be suitable in every use case. Depending on the specifics, a monolith might still be the better option.
# Prerequisites
- Docker
# Tooling and preparation
Install:
- minikube
- docker
# Architecture
The entire kubernetes entity is called **cluster**. Each cluster consists of several **nodes** that might run on seperate machines. Each node consists of several **pods**. A pod is the smallest ressource in kubernetes and it consists of one or more Docker-containers (but ususally one).

Nodes and pods are essential for running an application, but kubernetes offers multiple other helpful tools for managing, exposing and securing our application. In Kubernetes, these tools are called **ressources**
# Hello World
## How to set up minikube
In this tutorial, we will use **minikube**, a lightweight open-source version of kubernetes missing some features. The command
```
    minikube start
```
launches a kubernetes cluster. This will contain one single node but no pods. Before we get our hands dirty, let's have a look at the minikube **dashboard**. This is a vizualization tool that comes in very handy whenever we want to get an overview over the current status of our cluster. Run
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

If you understand docker well enough, you can create a simple website, push it to dockerhub and replace `nginx` with your image. You don't need to do this, but using stuff built by your own is much more fun, isn't it?

You can get an overview of your pods by running 
```
kubectl get pods
```
Watch the status column. The status of your pod should be `pending` for the first few seconds, then switch to `running`. If should not show `failed` or any other errors.
## How to expose a pod
So far, the containers have been closed off in kubernets, unavailable to the ouside world. In order to actually benefit from the power of kubernetes, the containers need to be exposed, so they can act as web servers accessible with a browser.

In order to do this, we need to create a **service**. In Kubernetes, services group some pods together and set a common access policy to them. We also need to specify the service type as NodePort. We will dive deeper into services [later](intermediate.md#Services):
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

If we now delete the service with
```bash
kubectl delete service hellopod
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
kubectl create deployment hello-deployment  --replicas=3 --image=nginx
```
Check out the deployments again. Does the ready-column show `3/3` now?

Ok but still, all we did was create 3 instances of the same container. We could have done this with Docker. The great thing about kubernetes is its respawing capabilities. Delete one of the pods from `hello-deployment` (you already know how to do that).

Now check out the pods again. You should still see 3 pods of the form `hello-deployment-[id]`, but in the age-column, you should see that one of them is younger than the other two. This one should also have an id that wasn't there earlier.

That is the actual power of kubernetes. No matter what happens to the pod, the deployment will always be there and spawn a new one. This cannot be achieved with a simple tool like Docker. And you don't want to write time-consuming, error-prone script for this, do you? Especially once you learn that Kubernetes has many more features making our lives easier ready to explore.
# Kubectl tricks
Kubectl offers several features that simplify your work. Don't skip them, or you won't understand the following topics if you do.
## Ressource overview
We already got to know 4 kubernetes ressources: Nodes, pods, deployments and services. There a many more. You can list all the ressources by running
```sh
kubectl api-resources
```
Think of them as your toolbox. We will get to know the most important ones of the ressources during this tutorial.
## How to edit an existing ressource
You might have noticed that every time we wanted to change some settings in a ressource (e.g. a pod), we had to delete it and relaunch it with new settings. This is not only tedious, but unaccaptable in a production context. There is a better way: If you want to change settings, run
```bash
kubectl edit [ressource]
```
This will open a yaml-configuration file in vim (also called ressource **manifest**). Here, you can edit a plethora of settings that might seem intimidating at first. But don't worry, we'll get there.

Some of the settings might already seem familiar. Can you spot the `replicas`-setting in the deployment configuration? Try to change the number. Now close the file. Kubernetes should apply the new settings automatically. Check if it worked. Does `kubectl get deployments` show another number of replicas now?
## How to preview an ressource before creating it
What if we want to check if one of our commands has the right syntax without actually creating a ressource? Well, there is a flag for that: `dry-run=client`.

This is more useful when paired with the preview flag: `-o yaml`(change yaml to json if you prefer that). Try to get a grasp of it by running
```bash
kubectl run pod hello-pod --dry-run=client --image=nginx -o yaml
```
This command should not affact your cluster in any way.
## How to create a ressource from manifest
Working with manifest files might not seem appealing at first, but believe me - you'll have to learn it eventually. So in order to get a bit more used to them, well create an entirely new pod just from a manifest file. Create a file called `manifest.yaml` with the following contents:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manifest-pod
spec:
  containers:
  - image: nginx
    name: nginx-container
  restartPolicy: Always
```
Now we can create the pod:
```bash
kubectl apply -f manifest.yaml
```
Does this command return `pod/manifest-pod created`? And can you see your pod in the list of pods (`kubectl get pods`)? If you can answer both questions with yes, then you succeeded in creating a pod.
## How to work inside pods
You can run commands inside a pod with the `kubectl exec`-command. You can run any command in the container like this:
```bash
kubectl exec -it hello-pod [command]
```
We can use this command to enter the running container's console like this:
```bash
kubectl exec -it hello-pod -- /bin/sh
```
Try to mess around in the file system (try `ls`, `mkdir`, `rm`, etc.) Note that any changes you make will be lost after a restart.
## How to deploy busybox
BusyBox is a lightweight and versatile Unix-like toolset that combines several common command-line utilities into a single executable.

We will launch it now to get to know some pitfalls that might trip you up later. So let's create the busybox-pod:
```bash
kubectl run busypod --image=busybox
```
If we now check out the pods, you will see that `busypod` does not behave as expected. After some creation time it will show the state `COMPLETED` for some time and then `CrashLoopBackOff`. So what is going on?

Well, `nginx` was a somewhat special container. Since it's a web server, it will run forever unless it's stopped. But busybox is finite. It runs some commands and then finishes. That's why we saw the state `COMPLETED`.

Ok, but why did we see an error? Well, after it was completet, a kubernetes-feature called **restartPolicy** kicks in. Kubernetes assumes we want the container to be present all the time. When the container completes and disappears, kubernetes makes it reappear.

The restartPolicy offers 3 options:
- always: When the container fails or finishes, it is restarted
- onFailure: When the container fails, it is restarted. If it finishes, it is not restarted.
- never: When the container fails or finishes, it just stops

All we need to do is change it to `never` and the error should disappear. You can do this in the manifest file, or directly in the command line:
```bash
kubectl run busypod --image=busybox --restart=Never
```
Now we should see the status `COMPLETED` with no crash message.

Ok great, we solved it for pods, but what about deployments? They need to make sure a constant number of pods is present all the time, so by definition, their restart policy must be `always`. How can we use busybox in a deployment?

Well, we can still prevent the busybox from exiting by keeping it busy. The easiest way to do this is to make it sleep. This time, we won't be able to do it in the command line. Instead, create a deployment manifest file with the `busybox`-image. Then, edit `spec.template.spec.image` to look like this:
```yaml
containers:
  - image: busybox
    name: busybox
    command: [ "sh", "-c", "sleep 10h" ]
```
This will keep busybox busy for 10 hours, which should be enough to work with it. If it's not enough, you can increase the time.
## How to change specific settings without opening the manifest file
kubectl set
## How to sort objects from kubectl get
Let's say you want to sort your pods by their creation time. How would we do that?

Well, `kubectl get` offers the `--sort-by` flag to do that:
```bash
kubectl get pods --sort-by=STATUS
```
Ok that didn't work. Why? Well, you can't sort by one of the columns. Instead, you need to sort by the contents of the manifest file. And it so happens to contain the field `creationTimestamp` in its `metadata`-map. To make the sorting work, need to specify the yaml-path to the timestamp:
```bash
kubectl get pods --sort-by=.metadata.creationTimestamp
```
This should now work.
## How to edit a running ressource
Some ressources (e.g. pod) cannot be edited while they are live. Instead, we need to awkwardly delete and recreate them every time we want to change something. Is there a better way? Yes, there is. The process can be automated by running 
```sh
kubectl -f [sourcefile.yaml] replace 
```
But 
```sh
kubectl -f [sourcefile.yaml] replace --force --grace-period=0
```
# First steps
## Volumes
If a pod needs to access data outside the pod, you need to use **volumes**. There are several use cases for this which we'll discuss later, but let's have a look at the volume concept first. There are 2 volume categories:
- ephemeral: the volume's lifetime is equal to the pod lifetime and gets destroyed in case of a pod failure or restart
- durable: the volume will outlive the pod even in case of a pod crash or restart

There is a plethora of volume types, but each of them will fall in one of the 2 categories. So let's get our hands dirty with some actual use cases:
### How to store data permanently
Storing important data in a pod is unacceptable due to several reasons:
- if a pod fails, it will be restarted, but all its data will be lost
- pods are often replicated. If every pods stores different data, the website will be hillariously inconsistent.

Luckily, volumes can help us out here. We'll use a volume of the type **hostPath**.[^1] It mounts a dir from the host system into the pod.

[^1]: While it is not recommended to use hostPath in a production context it still serves demonstration purposes well. Once you understand hostPath, it will be easy to understand the other volume types.

In order to do that, we need to edit the manifest. Add a `volumes`-map to `spec` that contains the volume name and the path of the volume on the host system.

In `spec.containers` we need to add a `volumeMounts`-map containing the volume name and the mount path in the pod.

What??? Ok, I know, that was confusing. Let me give you a working example of a pod manifest file to make it more clear:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
spec:
  containers:
  - image: nginx
    name: hello-pod
    volumeMounts:
    - name: hello-volume
      mountPath: /app
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: hello-volume
    hostPath:
      path: /mnt/volume
```
Note that this will only work if the pod `hello-pod` does not exist yet. Seriouly, just create deployments. Why are we still creating pods directly?

Anyway, create the pod and enter its filesystem (`kubectl exec -it -- /bin/bash`). Change to the mounted directory (in our case `app`) and create a file called "hello.txt". Now destroy the pod, create it again and check the mounted directory. Is the file still there? Is its content still the same? If you made changes in any other directory, they should be gone, but those in `app` should still be there.

You can check out the mount on the host system as well. In order to enter the host, run
```bash
minikube ssh
```
and head over to the mounted directory (in our case `mnt/volume`). Here, you should be able to view your file as well.
### How to share data in pods with multiple containers
If you set up a multi-container pod, you probably want the containers to talk to each other. Otherwise, you could have just put the containers in different pods, right? So how do we enable inter-container-communication? It's volumes again.

A persistent volume is not suitable for this use case, though. Once the pod fails, it just uses up ressources with no benefit. So this time, we will use an ephemeral volume called **EmptyDir** instead of hostPath.

So let's create a double-container-pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: doublebox
spec:
  containers:
  - image: busybox
    name: box1
    command: [ "sh", "-c", "sleep 10h" ]
  - image: busybox
    name: box2
    command: [ "sh", "-c", "sleep 10h" ]
  restartPolicy: Always
```
And then add an empty-dir-volume to both pods:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: doublebox
spec:
  containers:
  - image: busybox
    name: box1
    command: [ "sh", "-c", "sleep 10h" ]
    volumeMounts:
    - mountPath: /mnt/shared
      name: shared-storage
  - image: busybox
    name: box2
    command: [ "sh", "-c", "sleep 10h" ]
    volumeMounts:
    - mountPath: /mnt/shared
      name: shared-storage
  restartPolicy: Always
  volumes:
  - name: shared-storage
    emptyDir:
      sizeLimit: 500Mi
```
Now exec into one container. We should specify which one:
```sh
kubectl exec -it doublebox --container=box1 -- /bin/sh
```
Create a file at `mnt/shared/`, then exec into `box2` and look at its mounted path. Were you sucessful in sharing data?

## How to store confidential data
Websites and other programs frequently need to handle and store data that should not be viewed by the public. In Kubernetes, this can be achieved with **secrets**.
### How to create and use secretes
### Shortcomings of secrets
### Alternatives to secrets
## How to group objects together
Just like in every other field of computer science (and in life), a high number of certain objects make it harder for the user to keep an overview and find single objects. That's why we group files together in folders, or code in classes and functions. Organization just makes our lives easier.

Kubernetes is no exception. In big projects, it is not uncommon to have hundrets of pods, deployments, services etc. And worse yet - several projects might have to share the same cluster to reduce cost. Obviously, managing a cluster like that with no grouping would be vastly chaotic.

Thankfully, kubernetes offers 2 ways to group objects together: **Namespaces** and **labels**.
### Namespaces
Ressources like pods, deployments and services need to be namespaced. This is by design. But wait - we created pods already without namespacing them, didn't we? Well, if we don't provide a namespace, kubernetes will automatically put them into the namespace called `default`. You can get all namespaces from your cluster by running
```sh
kubectl get namespaces 
```
This should return something like
```
NAME                   STATUS   AGE
default                Active   1d
[some other namespaces we don't care about]
```
Only one namespace is not helping us to organize, obviously. So let's create a second one. Let's say we want to group all tests together in a namespace, so we run
```sh
kubectl create namespace testing
```
Now that we have several namespaces, we need to be careful with our commands. By default, every command is run in the `default`-namespace (hence the name), but if we want a command to be run in another namespace, we need to use the `--namespace`-flag. Let's say we want to list all pods in the `testing`-namespace. How would we do that?
```sh
kubectl get pods --namespace=testing
```
Since we haven't created anything yet in `testing`, it should return
```sh
No resources found in testing namespace.
```
If you want your commands to run in a certain namespace by default, you can run
```sh
kubectl config set-context --current --namespace=testing
```
### Labels
Labels are pretty similar to namespaces, but they exist in form of key-value-pairs and are generally used for project-intern organization. Labels must be unique for every object.

If you created a pod via command line, it will already have a label. The `metadata`-map of the manifest should have a `label`-map containing a key-value pair, e.g. `app: ningx`. You can create new labels of any name and content you like (as long as you don't use special characters). Try, for example to add the label `env: prod` to one of your pods and `env: dev` to another.

Ok, but how do these labels help us in organization? That's where we need to learn about **selectors**. You can easily filter your objects according to the lables they have and their values. Selectors are very powerful, but we will only discuss the simple ones.

You can get all pods containing a certain label like this:
```bash
kubectl get pods -l [label]
```
Or all the pods where the label equals a specific value:

```bash
kubectl get pods -l [label]=[value]
```
You can select objects based on labels with multiple conditions as well. A comma serves as an `AND`-Operator. Multi-condition-selectors need to be enclosed in quotes like this:
```bash
kubectl get pods '[label1], [label2]'
```
There is no `OR`-operator, but you can still select objects with different labels, by using **set-based**-selectors. These work with the keywords `in`, `notin` and `exists`. If you want to select labels containing either value1 or value2, you can use
```bash
kubectl get pods '[label] in (value1, value2)'
```
Or you could invert the statement by chaning `in` to `notin`.

Excersize: List all the pods of a specifc deployment by making use of selectors!
## How to adapt deployment size to current demand
kubectl autoscale
