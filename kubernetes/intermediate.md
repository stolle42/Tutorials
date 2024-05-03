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
  - [How to create helm charts](#how-to-create-helm-charts)
- [Service accounts](#service-accounts)
- [Automated container monitoring using probes](#automated-container-monitoring-using-probes)
  - [Liveness probes](#liveness-probes)
  - [Startup probes](#startup-probes)
  - [Readyness probes](#readyness-probes)
- [How to create a canary deployment](#how-to-create-a-canary-deployment)

# Restricting ressource usage
High cpu or memory usage (e.g. due to memory leaks) can be a huge problem for traditonal web servers. If a program keeps allocating more and more cpu time or memory, several problems can occur that can be difficult to spot and hard to solve. You can buy more memory, but even that will get clogged after some time if you're dealing with poorly implemented applications.

Kubernetes offers a way to confine this problem and limit its repercussions: object ressource limits. a ressource (e.g. a pod) can be limited to a certain amount of memory or cpu usage. This way, the host system will not be affected making the solution process easier and in many cases limiting the scope of the problem to a single pod.
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

You can try to occupy enough memory now to push the container to its limits, but be warned: the pod filesystem is rather confusing and won't be part of this tutorial.
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
Jobs can be schedules by using cronjobs. These are kubernetes ressources that are scheduled in the same way linux cronjobs are (if you don't know how that works, look [here](https://en.wikipedia.org/wiki/Cron)).

Just like every other kubernetes ressource, they too need a manifest file. The cronjob manifest contains the schedule and a job template. As an example, let's look at a cronjob that starts `busybox` once every minute:
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
So let's have a quick repetition: A cronjob manifest consists of a `schedule` and a `job`. A `job` consists of a `completions`-setting and a `pod`. A `pod` consist of a `restart-policy` and a container. Anad a container is no kubernetes ressource, so we don't care what it consists of. This was, of course, grossly oversimplified.
# How to reuse and recreate a cluster (Helm)
If you have a cluster that's working well, you might get ideas of backing it up or even shipping it to another machine and reusing it. So how do we do that? We could, of course, copy and paste all the yaml-files or create a script that runs all the commands necessary to set up the cluster. But creating and maintaining that script will be a lot of work. Is there a better way?

Of course there is. The tool we need to use is called **helm**. (What's a steerman without a helm?) With helm, you can create **charts** that allow you to easily ship, backup and recreate your old cluster. Actually, you don't normally handle your whole cluster. You could, but it is much better to divide your cluster into different packages (think of them as kubernetes libraries), so depending on the application you can reuse some of them. 


## How to create helm charts
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

Let's check if it worked. First, let's check if the chart exists in our namespace:
```sh
helm list
```
It should show `hellochart`.

Secondly, you can print out the manifests of all chart ressources by running
```bash
helm get manifest chart-name
```
Does it show your pod manifest? Then you can check if the required pod was created. Does `kubectl get pods` show the new pod from the chart?

You can remove the pod again by running
```bash
helm uninstall chart-name
```
Ok great, we can collect different ressources in a certain location and install or uninstall them simultanously. That somewhat simplifies our work, but is that all helm can do? No, there's much more. Templates are called that for a reason. You can name your pod after the chart name like this:
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
# Service accounts
We already learned how we can use `RBAC` to create a role-based authentication system. Non-human entities like a deployment can get accounts too. These are called **serviceAccounts**. You unwittingly have created serviceAccounts already. Run 
```sh
kubectl get serviceaccounts
```
and you'll see them. You might already have noticed that they have the same name like your namespaces. That is not a coincidence: every time a namespace is created, a serviceAccount is created along with it. There cannot be a namespace without a service account. If you try to delete it, it will be replaced by a new one. Automatically created serviceAccounts have the least possible permissions initially.

You can create a new service account by running
```sh
kubectl create serviceaccount hello-account
```
# Automated container monitoring using probes
Kubernetes offers mechanisms to determine the health and status of a running container. These are called **probes** and can help a great deal in reducing error impact.

Probes are **not** ressources, but settings you can integrate in ressources like pods. There's 3 different types of probes:
## Liveness probes
Applications can quickly get deadlocked and therefore deny service. It's a common problem in monolithic web servers, and it probably won't suprise you to learn that kubernetes has a solution for that, too. It's called **liveness probe**.

It works by monitoring a part of the container, e.g. an http-endpoint for availability. If the endpoint is not available a specified number of times, the pod is killed (and normally replaced). Setting the threshold too low can lead to unnessessary container kills, but setting it too high will increase the probe's response time.

Developpers don't necessarily need to write applications in a way that runs forever, which can ease their work.

Ok enough theory. Let's create a liveness probe!
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-probes
spec:
  containers:
  - name: busy
    image: busybox
    args:
      - /bin/sh
      - -c
      - touch /tmp/healthy; sleep 20; rm -f /tmp/healthy; sleep 100
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 2
      periodSeconds: 4
```
The probe runs every 4 seconds and checks if the file `tmp/healthy` is still there. We deliberately ran a command in the container making that file disappear after 20 seconds. So after 20 seconds, the probe will fail and cause the pod to be restarted or deleted, depending on the pod's restart policy.

Watch the pod failing and getting restarted again and again in `kubectl get pods` or `kubect describe pod`. Find out, how many probe failures will cause a restart? Spoiler alert: It's 3. You can change it to x times by adding `failureThreshold: x` to the probe.

Ok, but I promised we will do it by watching endpoints, not by watching a file. Ok fine. Let's watch endpoints:

TODO: find out if nginx can be configured to change return code after some time

Liveness probes are a great tool. But they can be very harmful when not used correctly. So the designers of kubenretes decided to introduce 2 more probes to mitigate the problems caused by liveness probes:
## Startup probes
You may have noticed the liveness probe contained an option called `initialDelaySeconds`. This was introduced to prevent the probe from failing at startup. Usually, containers take some time to be up and running, and we don't want the probe to kill the container at startup, leading to an endless restarting loop. That's why we have `initialDelaySeconds`. Set it to your container's startup time, and the restarting loop is prevented.

Ok, but what if we don't know the container's startup time? What if our container might start instantly, but might also take several minutes to start? We don't want an extremely high initial delay, we want the probe to be active as long as possible. So what do we do?

Well, that's where **startup probes** save us. They monitor the startup process in regular intervals. The liveness probe won't run as long as the startup probe is still there. Once  the startup is finished, it hand the baton over to the liveness probe. If the startup probe takes longer than a specified time, the probe fails and the container is restarted.

Todo: add example
## Readyness probes
Liveness probes can be dangerous. A container might appear to be hung up but, in reality, just have a lot on its hand. The liveness probe, though, doesn't know this and might decide to kill the container. This would be a disaster. So be careful when writing liveness probes - they should trigger in cases of real hang-ups, but not in the case of a high load.

However, we should do something about the high load. We should stop sending requests to it and wait until it finished (or at least reduced) its stack. Thats where **Readyness probes** help us out. They work exactely the same like liveness probes and they have the same syntax, but when triggered, they shut the container out from any traffic until the probe finds the high load reduced again.

# How to create a canary deployment
Testing a software 100% is impossible. We have to expect some bugs even in well-tested applications. But bugs hurt the user experience. Kubernetes offers a way to test applications in production context while still reducing the impact of software bugs in new versions: **canary** deployments.

We are not talking about a kubernetes features, but a way to make use of the flexibility and agility of kubernetes. So how do canary deployments work? Well, we leave the old software deployment up and running but add another one with the new software version. Then we make the service route only a small percentage to the new software while routing the rest to the old one.

That way, most users won't be affected by the change, but we can still test it in production context and - hopefully - get some bug reports we can fix before updating the whole system.

Routing too many users to the old system will result in more bugs not being discorered. Routing too many to the new system will increase the impact of a bad customer experience for more users than necessary.

Ok, enough theory. How do we actually implement a canary?

**Todo:** Read this and do all examples: https://kubernetes.io/docs/concepts/cluster-administration/logging

Understand Rolling Update Deployment including maxSurge and maxUnavailable