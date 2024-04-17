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
  - [How to create an object from manifest](#how-to-create-an-object-from-manifest)
  - [How to work inside pods](#how-to-work-inside-pods)
  - [How to deploy busybox](#how-to-deploy-busybox)
  - [How to change specific settings without opening the manifest file](#how-to-change-specific-settings-without-opening-the-manifest-file)
  - [How to sort objects from kubectl get](#how-to-sort-objects-from-kubectl-get)
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
- [Restricting ressource usage](#restricting-ressource-usage)
  - [How to view ressource usage](#how-to-view-ressource-usage)
  - [How to restrict memory usage](#how-to-restrict-memory-usage)
  - [How to restrict cpu usage](#how-to-restrict-cpu-usage)
- [Network](#network)
  - [Network policies](#network-policies)
    - [How to set up ingress](#how-to-set-up-ingress)
- [Security](#security)
  - [Role-based access control](#role-based-access-control)
  - [Security policies](#security-policies)
  - [How to schedule a command](#how-to-schedule-a-command)
    - [Jobs](#jobs)
  - [Cronjobs](#cronjobs)
  - [How to reuse and recreate a cluster (Helm)](#how-to-reuse-and-recreate-a-cluster-helm)
- [Advanced](#advanced)
  - [How to create a canary deployment](#how-to-create-a-canary-deployment)
  - [How to create a helm chart](#how-to-create-a-helm-chart)

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
kubectl create deployment hello-deployment  --replicas=3 --image=nginx
```
Check out the deployments again. Does the ready-column show `3/3` now?

Ok but still, all we did was create 3 instances of the same container. We could have done this with Docker. The great thing about kubernetes is its respawing capabilities. Delete one of the pods from `hello-deployment` (you already know how to do that).

Now check out the pods again. You should still see 3 pods of the form `hello-deployment-[id]`, but in the age-column, you should see that one of them is younger than the other two. This one should also have an id that wasn't there earlier.

That is the actual power of kubernetes. No matter what happens to the pod, the deployment will always be there and spawn a new one. This cannot be achieved with a simple tool like Docker. And you don't want to write time-consuming, error-prone script for this, do you? Especially once you learn that Kubernetes has many more features making our lives easier ready to explore.
# Kubectl tricks
Kubectl offers several features that simplify your work. Don't skip them, you won't understand the following topics if you do:
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
## How to create an object from manifest
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
# First steps
## Volumes
If you a pod to access data outside the pod, you need to use **volumes**. There are several use cases for this which we'll discuss later, but let's have a look at the volume concept first. There are 2 volume categories:
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

Ok that was confusing. Let me give you a working example of a pod manifest file to make it more clear:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: hello-pod
  name: hello-pod
spec:
  containers:
  - image: nginx
    name: hello-pod
    resources: {}
    volumeMounts:
    - name: hello-volume
      mountPath: /app
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: hello-volume
    hostPath:
      path: /mnt/volume
status: {}
```
Note that this will only work if the pod `hello-pod` does not exist yet. Seriouly, just create deployments. Why are we still creating pods directly?

Anyway, create the pod and enter its filesystem (`kubectl exec -it -- /bin/bash`). Change to the mounted directory (in our case `app`) and create a file called "hello.txt". Now destroy the pod, create it again and check the mounted directory. Is the file still there? Is its content still the same? All changes from every other directory should be gone, but those in `app` should still be there.

You can check out the mount on the host system as well. In order to enter the host, run
```bash
minikube ssh
```
and head over to the mounted directory (in our case `mnt/volume`). Here, you should be able to view your file as well.
### How to share data in pods with multiple containers
If you set up a multi-container pod, you probably want the containers to talk to each other. Otherwise, you could have just put the containers in different pods. So how do we enable inter-container-communication? It's volumes again.

A persistent volume is not suitable for this use case, though. Once the pod fails, there is no use in keeping it. So this time, we will use **EmptyDir** instead of hostPath.

So let's create a double-container-pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-debian
spec:
  containers:
  - image: nginx
    name: nginx-container
  - image: debian
    name: debian-container
  restartPolicy: Always
```
TODO: for some reason, doublepod container crashes unless one of the pods is echo-server. But echo-server is unsuitable because it can't run any linux commands. Try to find debugging methods that could help tracking down that problem, since now we're completely in the dark.

And then add an empty-dir-volume to both pods:

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
# Restricting ressource usage
High cpu or memory usage (e.g. due to memory leaks) can be a huge problem for traditonal web servers. If a program keeps allocating more and more cpu time or memory, several problems can occur that can be difficult to spot and hard to solve. You can buy more memory, but even that will get clogged after some time if you're dealing with poorly implemented applications.

Kubernetes offers a way to confine this problem and limit its repercussions: object ressource limits. An object (e.g. a pod) can be limited to a certain amount of memory or cpu usage. This way, the host system will not be affected making the solution process easier and in many cases limiting the scope of the problem to a single pod.
## How to view ressource usage
First, we need to install the `metrics-server`. In minikube, this can be done easily:
```bash
minikube addons enable metrics-server
```
Now you can view ressource usage of all pods:
```bash
 kubectl top pods
```
Or view ressource usage of the nodes:
```bash
 kubectl top nodes
```

## How to restrict memory usage
When changing a pod's config (`kubectl edit hello-deployment-[id]`), you'll find the limit settings in the `spec.containers[].resources`-section. It will look something like this:
```yaml
resources:
      limits:
        memory: 100Mi
      requests:
        memory: 100Mi
```
Ok, what do these settings mean?
- requests: The requested amount of memory is reserved for the pod, and therefore available it at any time. If the node does not have enough memory to allocate for a pod, it will keep the pod in `pending`-state until memory becomes available. Exceeding this amount is possible, but be careful - if the node runs out of memory, it will kill containers that exceed their memory request first. 
- limits: Maximum amount of memory the pod is allowed to use. Exceeding the limit is not possible and the attempt will cause an OOM-Error.

Since we never specified any limits, `100Mi` (100 Megabytes) was used as a default value. By the way - requests can never be greater than limits, because obviously.

Let's try to change the limits now. If you already tried changing the limit while editing, you should have gotten an error message. Thats because pod settings should never be edited directly. It would cause inconsistencies among replicated pods in your deployment. Also, pod setting changes will be lost after a pod replacement anyway.

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

You can try to occupy enough memory now to push the container to its limits, but be warned: the pod filesystem is rather confusing and won't be part of this introduction.
## How to restrict cpu usage
You can restrict cpu usage, too. It works about the same way like restricting memory usage:
- requests: CPU reservation. Unlike memory requests, exceeding cpu requests won't risk the pod to be killed
- limits: CPU usage ceiling. Attempting to exceed this limit will cause the pod to get throttled.

Of course we can't use the unit `bytes` here. Instead, kubernetes created a new unit called **Millicore**. 1000 Millicores equal 1 CPU core. As an example, limiting the cpu usage to 10% of the host machine's processing power and requesting 5% is done like this:
```yaml
      limits:
        cpu: 100m
      requests:
        cpu: 50m 
```

# Network
## Network policies
We [already established](#how-to-expose-a-pod) how we can expose a pod to the outside world using services. But there's much more to be considered when using network. We can restrict **ingress** (inbound) and **egress**(outbound) traffic to improve security. In Kubernetes, this is called **isolating** a pod, although traffic is not, as the word `isolating` would imply, cut off from the outside world completely. Only specific connections are forbidden.
### How to set up ingress
It does not make sense to set up ingress without a namesspace, so let's create one:
```bash
kubectl create namespace hello-ingress
```
Now we need to install the ingress controller:
```bash
helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace hello-ingress
```
TODO: finish
# Security
## Role-based access control
In order to protect sensitive data and prevent unskilled or malicious developpers from breaking parts of a system, user access should be restricted to only the ressources they need to get their job done, and no more. This is common practise in every area of IT, and kubernetes is no exception. You can use **RBAC** (Role-based access control) to restict user access depending on their role.

RBAC, as the name indicates, uses **roles** to restrict user access. You can define 2 types of roles:
- ClusterRole: the role will affect the whole cluster
- Role: the role will only affect a namespace

Todo: finish this
## Security policies
There are a [lot of options](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.29/#securitycontext-v1-core) to improve the security of a container or a pod. Pod security can be configured under `spec.securityContext` while container security is configured under `spec.containers.securityContext` in the manifest file. Container security takes precedence over Pod security.

Let's set the user to `1000` and the group to `3000`:
```yaml
securityContext:
    runAsUser: 1000
    runAsGroup: 3000
```
And in the container security, we could block priviledge escalation (i.e. prevent child processes from gaining more privileges than its parent process):
```yaml
securityContext:
  allowPrivilegeEscalation: false
```
If you now `exec` into the container and try to make any changes, you will be denied, since you're not root. Running `id` should return something like
```sh
uid=1000 gid=3000 groups=3000
```
## How to schedule a command
You might want a command to be run regularly. Let's say we want to create a pod every 5 seconds. How would we do that? Well, **cronjobs** have been written for this exact purpose. But in order to understand them, we must take a detour first and talk about **jobs**:
### Jobs
Let's say you need a certain number of pods to finish sucessfully. How do you do that? You could, of course, just launch them manually and keep track of their success, but the more completions you need, the more tedious this task will be.

Fortunately, jobs offer that very functionality. They keep track of sucessful completions and are considered "finished" when the specified number of completions is reached. Failures are not counted.

A pod manifest starting `busybox` 3 times could look like this:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: three-busyboxes
spec:
  completions: 3
  template:
    spec:
      containers:
      - name: busy
        image: busybox 
      restartPolicy: Never
```
So what we got here? Well, apart from the usual boilerplate, we need `spec.template` where we insert the manifest of the pod we like to see created. The `spec.completions`-setting defines the number of completions necessary for the job.

Create the job and then watch the pods getting created and then completing by running `kubectl get pods -l job-name=three-busyboxes` repeatedly (or by watching them in the dashboard).

After some time, the output should look something like this (with different IDs and ages, of course):
```
NAME                    READY   STATUS      RESTARTS   AGE
three-busyboxes-84sbh   0/1     Completed   0          74s
three-busyboxes-bfbkn   0/1     Completed   0          80s
three-busyboxes-l6t9k   0/1     Completed   0          68s
```
## Cronjobs
Jobs can be schedules by using cronjobs. These are kubernetes objects that are scheduled in the same way linux cronjobs are (if you don't know how that works, look [here](https://en.wikipedia.org/wiki/Cron)).

Just like every other kubernetes object, they too need a manifest file. The cronjob manifest contains the schedule and a job template. As an example, let's look at a cronjob that starts `busybox` once every minute:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cronjob
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: busy
            image: busybox
```
So let's have a quick repetition: A cronjob manifest consists of a `schedule` and a `job`. A `job` consists of a `completions`-setting and a `pod`. A `pod` consist of a `restart-policy` and a container. Anad a container is no kubernetes object, so we don't care what it consists of. This was, of course, grossly oversimplified.
## How to reuse and recreate a cluster (Helm)
If you have a cluster that's working well, you might get ideas of backing it up or even shipping it to another machine and reusing it. So how do we do that? We could, of course, create a script that runs all the commands necessary to set up the cluster. But creating and maintaining that script will be a lot of work. Is there a better way?

Of course there is. The tool we need to use is called **helm**. (What's a steerman without a helm?) With helm, you can create **charts** that allow you to easily ship, backup and recreate your old cluster. Actually, you don't normally handle your whole cluster. You could, but it is much better to divide your cluster into different packages (think of them as kubernetes libraries), so depending on the application you can reuse some of them. 

# Advanced
## How to create a canary deployment
Testing a software 100% is impossible. We have to expect some bugs even in well-tested applications. But bugs hurt the user experience. Kubernetes offers a way to test applications in production context while still reducing the impact of software bugs in new versions: **canary** deployments.

We are not talking about a kubernetes features, but a way to make use of the flexibility and agility of kubernetes. So how do canary deployments work? Well, we leave the old software deployment up and running but add another one with the new software version. Then we make the service route only a small percentage to the new software while routing the rest to the old one.

That way, most users won't be affected by the change, but we can still test it in production context and - hopefully - get some bug reports we can fix before updating the whole system.

Routing too many users to the old system will result in more bugs not being discorered. Routing too many to the new system will increase the impact of a bad customer experience for more users than necessary.

Ok, enough theory. How do we actually implement a canary?

## How to create a helm chart
You can create a new, empty helm chart by running
```bash
helm create hello-chart
```
This should create a folder called `hello-chart`. It contains a  folder called `templates`, which will be very useful once we understand how helm works. But since we haven't learned it yet, we now delete all the contents of `templates` and create our own template instead.

How do we create a template? Well for starters, all we need to do is to save a manifest-file in the template folder.

So let's get our hands dirty. Save a pod manifest in the `templates` folder, then create your first chart by running
```bash
helm install chart-name hello-chart
```
where `chart-name` is replacable by any ascii-string you like.

Let's check if it worked. First, you can print out the manifests of all chart ressources by running
```bash
helm get manifest chart-name
```
Does it show your pod manifest? Then you can check if the required pod was created. Does `kubectl get pods` show the new pod from the chart?

You can remove the pod again by running
```bash
helm uninstall chart-name
```
Ok great, we can collect different objects in a certain location and install or uninstall them simultanously. That somewhat simplifies our work, but is that all helm can do? No, there's much more. Templates are called that for a reason. You can name your pod after the chart name like this:
```yaml
metadata:
  name: {{.Release.Name}}-pod
```
Installing the helm chart now should create a pod named `chart-name-pod`. Ok but what is that weird `.Release.Name`? Well, that brings us to helm **objects**.

Objects are mostly data maps you can use to get certain information. They can contain other objects and even functions, but in most cases, they are just used in templates for fetching data. `Release` is one of the objects accessible to templates and contains data fields like `Name`, `NAMESPACE`, `Revision` and many more. You can view them by running
```bash
helm status chart-name
```
There are other objects, too, like `Values`, `Files` or `Capabilities`. 