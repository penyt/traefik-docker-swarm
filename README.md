# traefik-docker-swarm

## 🐧 Main purpose
Set traefik on the docker swarm overlay network in order to enable other docker hosts to use the traefik docker labels.
### 1. Easy requirements
*Only one public IP is needed for all of the docker hosts, which is the one that hosts traefik.*
### 2. Easy settings
*Simply change the "labels" section to be under the "deploy" section in docker-composes.yml file of any apps.*

## 🐧 Method
$${\color{red}host1}$$ :  the host that owns a public IP, and will have traefik running.  
$${\color{blue}host2}$$ :  the host that **doesn't** own a public IP, and will have apps running.
<br>
<br>
### 1. Initialize docker swarm
Run on $${\color{red}host1}$$, 
```
docker swarm init
```
it will give an output like `docker swarm join --token THISIS-7-89456123789aworker456987token....... IP_ADDR:2377`, this is a token for **setting a worker**.  

And also, a line `To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.`, this one is the command we would like to use.

<br>

Then follow the instruction, run again on $${\color{red}host1}$$, 
```
docker swarm join-token manager
```
it will give an output like `docker swarm join --token MAGENA-7-89456123789r456987token....... IP_ADDR:2377`, this is a token for **setting a manager**.


<br>
<br>

### 2. Set host2 to be a manager instead of a worker (the worker CAN'T deploy apps by itself)
Switch to $${\color{blue}host2}$$, run the **MANAGER** join command
```
docker swarm join --token MAGENA-7-8943789r487token....... IP_ADDR:2377 (copy your own one, don't copy this)
```
Then, run the command below to check nodes
```
docker node ls
```
Output :  
<blockquote>
ID                            &emsp;&emsp;&emsp;&emsp;
HOSTNAME                      &emsp;&emsp;
STATUS    &emsp;&emsp;
AVAILABILITY   &emsp;&emsp;&emsp;
MANAGER STATUS   &emsp;&emsp;&emsp;
ENGINE VERSION    
  
idofhostt2     &emsp;&emsp;
host2     &emsp;&emsp;&emsp;&emsp;
Ready     &emsp;&emsp;&emsp;
Active         &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
Reachable        &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
27.3.1
    
idofhostt1 *   &emsp;
host1   &emsp;&emsp;&emsp;&emsp;
Ready     &emsp;&emsp;&emsp;
Active         &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
Leader           &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
27.3.1  
</blockquote>
<br>
Now, we have finished connecting hosts by using docker swarm.

<br>
<br>

### 3. Create traefik docker-compose.yml & configuration files [^1]
Create folder and needed files on $${\color{red}host1}$$,  
* create folder : `mkdir traefik`  
* get into folder : `cd traefik`  
* create files : `touch traefik.yml config.yml acme.json docker-compose.yml`  
* set permission : `chmod 600 acme.json`
 
**traefik**  
├ docker-compose.yml  
├ traefik.yml  
├ config.yml  
└ acme.json  

edit docker-compose.yml `nano docker-compose.yml`, paste in and edit;  

edit traefik.yml `nano traefik.yml`, paste in and edit.  

---

**(OPTIONAL but RECOMMENDED) Add hashed password**
```
sudo apt-get install apache2-utils
```
Set "username" & "yourpassword"
```
htpasswd -nb username yourpassword
```
it will give an output like `username:$apr1$8VzK7EwL$4Z9T.HqxGkJpAqVnqp4Ol1`.  

Change all `$` to `$$` (prevent docker compose interpreting it as an env variable), then replace the `user:hashed_password` in the docker-compose.yml of traefik.  

<br>
<br>

### 4. Deploy traefik
Run on $${\color{red}host1}$$
```
docker stack deploy -c docker-compose.yml traefik
```
or any stack name `docker stack deploy -c docker-compose.yml traefik_stack_name`.

<br>

Check on both  $${\color{red}host1}$$ & $${\color{blue}host2}$$ :
```
docker service ls
```
Output :   
<blockquote>
ID             &emsp;&emsp;&emsp;&emsp;&emsp;
NAME              &emsp;&emsp;&emsp;&emsp;&emsp;
MODE         &emsp;&emsp;
REPLICAS   &emsp;&emsp;
IMAGE                  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
PORTS  
  
rtafeiksid     &emsp;&emsp;
traefik_traefik   &emsp;&emsp;
replicated   &emsp;
1/1        &emsp;&emsp;&emsp;&emsp;
traefik:latest
</blockquote>

<br>
<br>

### 5. Create whoami docker-compose.yml
Switch to $${\color{blue}host2}$$, create folder and needed files on $${\color{blue}host2}$$,  
* create folder : `mkdir whoami`  
* get in folder : `cd whoami`  
* create files : `touch docker-compose.yml`  
 
**whoami**  
└ docker-compose.yml  

edit docker-compose.yml `nano docker-compose.yml`, paste in and edit.  

<br>
<br>

### 6. Deploy apps (whoami)
Run on $${\color{blue}host2}$$
```
docker stack deploy -c docker-compose.yml whoami
```
or any stack name `docker stack deploy -c docker-compose.yml whoami_stack_name`

<br>

Check on both  $${\color{red}host1}$$ & $${\color{blue}host2}$$
```
docker service ls
```
Output : 
<blockquote>
ID             &emsp;&emsp;&emsp;&emsp;&emsp;
NAME              &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
MODE         &emsp;&emsp;
REPLICAS   &emsp;&emsp;
IMAGE                  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
PORTS  

rtafeiksid     &emsp;&emsp;
traefik_traefik   &emsp;&emsp;&emsp;&emsp;
replicated   &emsp;
1/1        &emsp;&emsp;&emsp;&emsp;
traefik:latest  

whoidami   &emsp;&emsp;
whoami2_whoami2   &emsp;&emsp;
replicated   &emsp;
1/1        &emsp;&emsp;&emsp;&emsp;
traefik/whoami:v1.10   
</blockquote>

<br>
<br>

### 7. Finish
Now, you should be able to visit traefik dashboard and whoami by the domain https://your.subdomain.com/, which was set in each of your docker-compose.yml.  

If there's an error, remember to check:
* your DNS record point to the public IP of $${\color{red}host1}$$
* check traefik logs `docker service logs traefik_traefik`
* check docker swarm overlay network `docker network inspect proxy`

<br>
<br>



[^1]:traefik deployment reference: [JimsGarage](https://github.com/JamesTurland/JimsGarage/blob/main/Traefik/docker-compose.yml)
