##Docker container

**Commonly used docker commands**

`-p` : publish

**Create and start a docker container**
> map docker port 80 to local host 8080
> --name mydocker : the docker container's name is nginx
> last nginx: the image is nginx
> after create a container it will return you a container id

```
docker create --name mydocker -p 8080:80 nginx
docker start mydocker
```
or
> -d: start the container in the background
> -d nginx: nginx is the image

```
docker run --name mydocker -p 8080:80 -d nginx
```
**List conatiners**

list running containers

```
docker ps
```

list all containers

```
docker ps -a
```
**Stop / Restart / Remove a Docker Container**

• Restart a container> Syntax: docker restart [OPTIONS] CONTAINER [CONTAINER …]

```docker restart mydocker

```• Stop the container> Syntax: docker stop [OPTIONS] CONTAINER [CONTAINER …]

```docker stop mydocker
```
• Remove a non-running container> Syntax: docker rm [OPTIONS] CONTAINER [CONTAINER …]

```docker rm mydocker
```• Remove a running container

```docker rm -f mydocker
```
**Run a shell within a container**

Containers can be started and stopped, and behave like VMs.Instead of using SSH to access a VM, a container can be accessed with exec command.

(Not recommended, mainly for debug or testing purpose.)

> Syntax: docker exec [OPTIONS] CONTAINER COMMAND [ARG …]
> -t : terminal
> -i : interactive mode
> -w: set working directory
> /usr/share/nginx/html/ : html root of nginx
> 

```docker exec -ti -w /usr/share/nginx/html/ mydocker sh
```

**Create Volume in Docker**

• Create a volume> Syntax: docker volume create [OPTIONS] [VOLUME]

```docker volume create --name htdocs
```• *Volume mount* : Start a container with a volume attached
> -v : mount the volume htdocs volume to the container's root html file

```docker run --name nginx-volume -p 8080:80 \-v htdocs:/usr/share/nginx/html -d nginx
```• *Bind mount* : Start a container with bind mount attached
(more flexible, you can change file outsidee the container and it will update in the container)
> pwd : enter the path of your working directory, e.g. the path of demo1 folder in docker workshop

```docker run --name nginx-bind -p 8081:80 \-v $(pwd)/htdocs:/usr/share/nginx/html -d nginx
```
**Create image**

• Create an image> * Syntax: docker build [OPTIONS] PATH
> * "demo2 . ": with the " ." the docker will look at the all the file in current directory and interact them based on the dockerfile
> * Do not need to specify the dockerfile, it will look at a file called "dockerfile" in the current directory by default. However if it is not named as "dockerfile" you can add `-f` to specify the file path

```docker build -t demo2 .
```
• Create a container from the image
> -e: environmental variable

```docker run --name demo2 -e WELCOME_STRING="COMP90024" \-p 8081:80 -d demo2
```
##Docker Compose

• Start the containers> Syntax: docker-compose up [OPTIONS]

```docker-compose up -d
```
If the file name isn't "docker-compose", then do:

```
docker-compose -f <path of yoour composed file> up -d
```
• Stop the containers> Syntax: docker-compose stop [OPTIONS] [SERVICE …]

```docker-compose stop
```• Remove the containers> Syntax: docker-compose down [OPTIONS]
> However, this will not remove the volume of the docker

```docker-compose down
```
to remove the volume of the compose

```
docker-compose down -v
```
Example

```

version: '3.9'
services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: db
      #will link the db defined below
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_ROOT_PASSWORD: P@ssw0rd
    volumes:
      - ./mysql:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    restart: always
    ports:
      - "8081:80"
  
#it will automatically look for db
```
On terminal

```
docker run -e WORDPRESS_DB_HOST=db -e WORDPRESS_DB_USER=wordpress 
-e WORDPRESS_DB_PASSWORD=wordpress 
-e WORDPRESS_DB_NAME=wordpress 
-p 8080:80 -d --restart always wordpress

docker run -e MYSQL_DATABASE=wordpress
-e MYSQL_USER=wordpress -e MYSQL_PASSWORD=wordpress -e MYSQL_ROOT_PASSWORD=P@ssw0rd
-v $(pwd)/mysql:/var/lib/mysql -d --restart always mysql:5.7

docker run -p 8081:80 -d --restart always phpmyadmin/phpmyadmin

```


##Docker SWARM

To use docker SWARM, we need to have docker machine creates several nodes on the local compuuter

_**Docker Machine**_

create one manager and two workers

```
docker-machine create managerdocker-machine create worker1docker-machine create worker2
```
List all the running nodes

```
docker-machine ls
```
Get inside the manager node:
> The Docker SWARM operations below should be done inside the docker node

```
docker-machine ssh manager
```
**_Docker SWARM_**

* Create a Docker SWARM

>  --advertise-addr 192.168.99.100 : address set for the first manager node

```
docker swarm init --advertise-addr 192.168.99.100
```
It will then output a join tocken for other nodes to join the cluster. To check the join token of the manager:

```
docker-machine ssh manager docker swarm join-token manager
```
* Join a Docker SWARM via join tocken

```
docker-machine ssh worker1 docker swarm join --tokenSWMTKN-1-2uj8tpltiekyk1e5n4dcugcokjo8a2cfuvgby9s8ru8jwgg6q2-f3xw8888sfw0ayvgrcuv4efam 192.168.99.100:2377
```
* List all nodes IN the SWARM

```
docker-machine ssh manager docker node ls
```
**Create a service**

• Create a service> Syntax: docker service create [OPTIONS] IMAGE [COMMAND]
> -- replicas 3 : create three instances (3 containers, one for each node)```docker-machine ssh manager docker service create --replicas3 -p 8083:80 --name myswarm nginx:alpine
```
• List a service> Syntax: docker service ls [OPTIONS]

```docker-machine ssh manager docker service ls
```• Check a service> Syntax: docker service ps [OPTIONS] SERVICE [SERVICE …]

```docker-machine ssh manager docker service ps myswarm
```

**Scaling and Rolling Update**

• Scale up / down (but it will limited to the number of host machines)> Syntax: docker service scale SERVICE=REPLICAS
> The scalling is manual and not auto for Docker SWARM, but other orchestration tool may can do

```docker-machine ssh manager docker service scale nginx=6docker-machine ssh manager docker service scale nginx=2
```
• Rolling update> * Syntax: docker service update [OPTIONS] SERVICE
> * Update all the containers in the cluster's image to "alwynpan/comp90024:demo1" by sequences -- allows no downtime update (although users will sees different version while it updating)


```docker-machine ssh manager docker service update --imagealwynpan/comp90024:demo1 nginx
```






