<details><summary>  
Name 3 reasons, why your software might not work on your clients' systems (without using docker) despite running on yours!
</summary>  
- clients have different software installed or use different OS
- you forgot to put all files into yoru software package
- client machine is configured differently
</details>
<details><summary>  
How does Docker solve these issues?
</summary>  
todo
</details>
<details><summary>  
What is the simples way to share Docker containers with other machines?
</summary>  
Push it to Dockerhub, then everyone can pull it from there.
</details>
<details><summary>  
What is the difference between horizontal scaling and vertical scaling?
</summary>  
Horizontal: Increasing the hardware capabilities of a server
Vertical: Increase the number of servers
</details>
<details><summary>  
Microservices
</summary>  
is an architecture style where an application is divided into several microservices. Each one of these is independently deployable (executable on its own) and loosely coupled (each service's development and functioniality should depend as little as possible on the availability of other services). Normally, microservices have their own repository and are tested in isolation with their own pipeline before being deployed.
</details>
<details><summary>  
Distributed system
</summary>  
A system that runs on several machines
</details>
<details><summary>  
Docker containers are stateless. What does that mean, and why is it a good thing?
</summary>  
"Stateless" means changes are not saved when shut down. This simplifies scalability (no need to sync state when adding a new container), portability (no need to convert data to a different format compatible with a new system) and makes them more resilient (container failure can't corrupt data, since rebooting will reset it to a working state)
</details>
<details><summary>  
How can data be stored persistently in a docker container?
</summary>  
Since docker containers are stateless, it isn't possible to persistently store data on them. Instead, a **bind mount** or a **volume** can be mounted in the docker file system, thus permanently saving data in a persistent location, e.g. on the host machine.
</details>
<details><summary>  

</summary>  

</details>
<details><summary>  

</summary>  

</details>
