# What is docker?
Docker is a tool with which you can deploy and manage container applications. Containers are software packages containing all application code, dependencies and configuration settings. This allows you to easily run the application on different machines[^1] without the need to change any of the settings. 

Docker containers are lightweight, fast and completely independent from other applications. Basically, it provides you with all the advantages of a VM, at a much greater flexibility and speed while using less ressources.

[^1]: The machines need to have a linux kernel, so windows machines will need WSL
# Prerequisites
Basic understanding of linux and the command line
# Tooling and preparation
- install docker
# Hello World
Create an application (or choose an old application) which you want to run on different machines. If you don't have an application, use this one:
<!-- TODO -->
Now how do we containerize it? Well, the first step is to create a `DOCKERFILE`. 