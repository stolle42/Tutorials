# What is docker?
Docker is a tool with which you can deploy and manage container applications. Containers are software packages containing all application code, dependencies and configuration settings. This allows you to easily run the application on different machines[^1] without the need to change any of the settings. This has the additional advantage that you won't need to update, install or remove any additional tools to run the software, since all additional tools are packaged into the container. 

Docker containers are lightweight, fast and completely independent from other applications. Basically, it provides you with all the advantages of a VM, at a much greater flexibility and speed while using less ressources.

[^1]: The machines need to have a linux kernel for docker to work, so windows machines will need WSL
# Prerequisites
- Basic understanding of linux and the command line
# Tooling and preparation
- install docker
# Architecture
Docker uses a client-server-architecture: The server (docker engine) manages different docker clients. Clients are running on the host OS as regular processes.
# Hello World
Create an application (or choose an old application) which you want to run on different machines. If you don't have an application, you can create a very simple one like this:
```js
console.log("Hello world!");
```
save this as a javascript file (`greeting.js`) and - congratulations! You created your first app!

So how do we containerize it? Well, the first step is to create a `DOCKERFILE`. This is a plain text file containing all the instructions necessary to build the app, like OS-version, dependencies etc. You can think of it as a release document that is understandable to computers.

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
Now we can to create the container by running
```bash
docker build .
```
It will be hard to run it though, since we haven't named it yet. We can access it by reffering to the imageID which we can find in the command output, but this is somewhat tedious. So instead, let's run
```bash
docker build -t helloworld .
```
Now we've named it `helloworld`, thus simplifying access and management of the container. For instance, we can easily run it like this:
```bash
docker run helloworld
```
This should now start the container and run the `greeting.js`-script, putting its output to the command line. Try it! Does your container greet the world?