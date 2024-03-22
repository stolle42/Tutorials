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
Choose an old application which you want to run on different machines. If you don't have an application, you can create a very simple one like this:
```js
console.log("Hello world!");
```
Save this as a javascript file (`greeting.js`) and - congratulations! You created your first simple application!

Ok, but that was a sidequest. We actually want to containerize the app. So how do we do that? Well, the first step is to create a file named `DOCKERFILE`. This file contains all the instructions necessary to build the application, like OS-version, dependencies etc. You can think of it as a release document, but one that is understandable to computers.

Let's talk about the most important DOCKERFILE-commands:
Normally, you won't create the entire container. instead, you'll reuse and extend some container(s) that already exist in order to simplify your workflow. This is done by the command `FROM`.

In order to include all the files you need from host system into the container you need to copy them there by the `COPY`-command.

If you want to run a command within the container, use the `CMD`-command.

These 3 commands are already enough to containerize our application:
```DOCKERFILE
#our starting point:linux alpine with node.js installed
FROM node:alpine 
#simple app: container only needs this one file
COPY greeing.js /home
#run the greeting script, so we can see the output
CMD node /home/greeting.js 
```
Note: In theory you can use any other linux distribution, but Linux Alpine is recommended for containers due to its small size.

Now we can create our first image by running
```bash
docker build .
```
Actually, that was not the best command. It works, but we can only access and run it by reffering to the imageID available in the command output. This is somewhat tedious. So instead, let's build and name it:
```bash
docker build -t helloworld .
```
Now we've created an image we can easily refer to by its name `helloworld`. You can see a whole list of all the containers from your machine by running
```bash
docker images
```
We can now run it like this:
```bash
docker run helloworld
```
This should now start the container and run the `greeting.js`-script, putting its output to the command line. Try it! Does your container greet the world?

Ok but so far we haven't actually accomplished much, this would have worked without docker as well. Let's now share it with another machine, so you can see its benefits firsthand!
## How to share containers
Ok, so now we will run the image on another machine (if you don't have one, you can use a virtual machine from [Play with Docker](https://www.docker.com/play-with-docker/)).

In order to share it, we first need to publish it. There are 2 main container sharing websites: [Dockerhub](https://hub.docker.com) and [Github](https://github.com). 

