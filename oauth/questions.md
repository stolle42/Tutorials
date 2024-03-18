<details><summary>  
Name 4 roles in oAuth and define them!
</summary>  
<details><summary>  
Client
</summary>  
The application asking for access to some ressource
</details>
<details><summary>  
Authorization server
</summary>  
The central entity where clients get permission to access a ressource
</details>
<details><summary>  
Ressource server
</summary>  
The place where the requested ressource is stored
</details>
<details><summary>  
Ressource owner
</summary>  
The user who owns the ressource and decides on application access permissions and their scope
</details>
</details>
&nbsp;
<details><summary>  
Authorization grant
</summary>  
Ressource usage permission given by the ressource owner
</details>
<details><summary>  
Access token (also called bearer token)
</summary>  
Authorization server hands over this token to the client, with which it can prove its access rights to the ressource server.
</details>
<details><summary>  
Explain all 7 steps of an oauth-autorization!
</summary>  
1. Client asks ressource owner for autorization
2. If ressource owner rejects, the process ends right here. If he allows, he sends the Authorization grant to the client
3. Client sends autorization grant to the autorization 
4. Authorization server sends an access token to the client
5. Client sends access token to the ressource server and asks for the ressource 
6. Ressource server checks acces token validity with the autorization server
7. If step 6 was successful, the ressource server shares the ressource with the client
</details><details><summary>  

</summary>  

</details>
