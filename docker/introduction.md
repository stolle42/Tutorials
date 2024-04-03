# What is docker?
We all know the problem: Moving an application from one system to another sucks. There's a bunch of installers, dependencies and settings to adjust, leading to long and convoluted release documents that are tedious to write, tedious to read and in many cases incomplete. But we don't have to do this. Computers can do this for us. That's where docker comes into play.

Docker is a tool with which you can deploy and manage container applications. Containers are software packages containing the whole application code, dependencies and configuration settings. This allows you to easily run the application on different machines[^1] without the need to change any of the settings. This has the additional advantage that you won't need to update, install or remove any additional tools to run the software, since all additional tools are packaged into the container. 

Docker containers are lightweight, fast and completely independent from other applications. It can also be replicated multiple times, thus making your application easily scalable. Basically, it provides you with all the advantages of a VM, at a much greater flexibility and speed while using less ressources.

[^1]: The machines need to have a linux kernel for docker to work, so windows machines will need WSL
# Prerequisites
- Basic understanding of linux and the command line
# Tooling and preparation
- install docker
- create an account at dockerhub
# Architecture
Docker uses a client-server-architecture: The server (docker engine) manages different docker clients. Clients are running on the host OS as regular processes.
# Hello World
## How to package an application into a container
Now we will create an application which we will then share across different machines.

Create a javascript file called `greeting.js` with the following content:
```js
console.log("Hello world!");
```
Congratulations! You created your first web application!

Ok, but that was a sidequest. Our goal is containerizing the app. So how do we do that? Well, the first step is to create a file named `DOCKERFILE`. This file contains all the instructions necessary to build the application, like OS-version, dependencies etc. You can think of it as a release document, but one that is understandable to computers.

This tutorial will only cover about the most important DOCKERFILE-commands. A full list can be found [here](https://docs.docker.com/reference/dockerfile/).

Normally, you won't create the entire container from scratch. instead, you'll reuse and extend some container(s) that already exist in order to simplify your workflow. This is done by the command `FROM`.

In order to include all the files you need from host system into the container you need to copy them there by the `COPY`-command.

If you want to run a command within the container, use the `CMD`-command.

We only need 3 commands to containerize our application:
```DOCKERFILE
#starting point: linux alpine with node.js installed
FROM node:alpine 
#simple app: container only needs this one file
COPY greeting.js /home
#run the greeting script, so we can see the output
CMD node /home/greeting.js 
```
Note: In theory you can use any linux distribution, but Linux Alpine is recommended for containers due to its lightweightness.

Now we can create our first image by running
```bash
docker build .
```
Actually, that was not the best command. It works, but the only way to access and run it is by reffering to the imageID from the command output. This is somewhat tedious. So instead, let's give it a name during build:
```bash
docker build -t helloworld .
```
Now we've created an image we can easily refer to by its name `helloworld`. You can see a whole list of all the containers from your machine by running
```bash
docker images
```
This should show you name, tag, ID, age and size of your images.

If you previously ran `docker build .` without naming it, you're probably seeing 2 images: one of them named `helloworld`, the other one named `<none>`. We should remove the unnamed one to keep things organized. Images can be deleted  by referring to their ID like this:
```bash
docker image remove [Image ID]
```

Ok, that's enough talk, let's run it now:
```bash
docker run helloworld
```
This should now start the container and run the `greeting.js`-script, putting its output to the command line. Try it! Does your container greet the world?

Ok but to be fair - we haven't actually accomplished that much, have we? A js-script can be launched without docker as well. Let's now share it with another machine, so you can see its benefits firsthand!
## How to share containers
Let's share the image with another machine (if you don't have another machine, you can use a virtual machine from [Play with Docker](https://www.docker.com/play-with-docker/)).

The first step is publishing it to a container sharing website  (called docker registry). There are many options:
- [Dockerhub](https://hub.docker.com)
- [Github](https://github.com)
- pretty much every big cloud computing platform

In this tutorial, we will use Dockerhub as our docker registry. Sharing a container on Dockerhub is somewhat similar to sharing code on Github. Let's get started!

First of all, we need to create an account on Dockerhub. Then we need to login like this:
```bash
docker login
```
It will ask you for the username and password from the account you just created.

After being logged in, we cannot yet push our image (you'll get the error `denied: requested access to the resource is denied`). That's because Dockerhub can only understand images a format like this: `[username]/[imagename]`. Renaming images is done by the `tag`-command:
```bash
docker tag helloworld [username]/helloworld
```
Now you can push it:
```bash
docker push [username]/helloworld
```
Check it out on the dockerhub website. Was a new repository called `helloworld` created?
# First steps
## How to work inside a container
You can enter a container's terminal and make changes to the file system as if it were a virtual machine. Normally, this can be done by the command
```bash
docker run -it helloworld
```
However, linux alpine is a special case in which we need to add the entry point, so the command becomes:
```bash
docker run --entrypoint "/bin/sh" -it helloworld
```
Now you should have entered your container's terminal. You shell should show `/ # `. Try messing around in the file system. Most linux file commands should work here (e.g. list files, create a new dir, vi, cat,...)

Now try installing a new package with the alpine package manager (apk):
```bash
apk add git
```
Verify the installation worked. Does `git --version` work?

None of our changes are permanent. Type exit and relaunch the container with the integrated terminal like before. All the changes you made should be gone now and `git --version` should cause an error. That is because containers are by design stateless. They never store anything outside their lifecycle. 

But what if we wanted a package to exist in a container?
## How to install packages in the dockerfile
Let's say we wanted git to be installed in our container. How do we do that? You might remember we already needed a package earlier. To make our simple greeting application, we had to install node.js, right?

We did this by using the `node:alpine` image as our starting point. So for installing git, we could try
```DOCKERFILE
FROM git:alpine 
```
However this won't work, because an image like that has not been created yet. So instead, we need to install it ourselves. This can be done easily by appending
```DOCKERFILE
RUN apk add git
```
to the DOCKERFILE. Try it! Rebuild the docker image, then launch the integrated terminal of the container. Is git available in the container now without installing?
## How to containerize a website
So far, we only containerized a command line application. This won't do for most applications. Let's create a real html-based web application and containerize it.
TODO: finish section 
# Docker registry
# Docker compose