<details><summary>  
What is the control plane?
</summary>  

</details>
<details><summary>  
What is the difference between init containers and sidecar containers?
</summary>  
initcontainers start and complete before the main container is started. If there are many init containers, they will be run as a queue. The main container can only start after all of them are completed.

On the other hand, Sidecars start before the main container and continue to run alongside the main container.
</details>
<details><summary>  
What will happen to a pod when an init-container fails?
</summary>  
That depends on the restartPolicy. If it is `never`, then the entire pod is treated as a failed pod. Otherwise, the initContainer is restarted over and over again until it succeeds.
</details>
<details><summary>  

</summary>  

</details>
<details><summary>  

</summary>  

</details>
