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

• Restart a container

```

```

```
```


```
```

```
```
**Run a shell within a container**

Containers can be started and stopped, and behave like VMs.

(Not recommended, mainly for debug or testing purpose.)

> Syntax: docker exec [OPTIONS] CONTAINER COMMAND [ARG …]
> -t : terminal
> -i : interactive mode
> -w: set working directory
> /usr/share/nginx/html/ : html root of nginx
> 

```
```

**Create Volume in Docker**

• Create a volume

```
```
> -v : mount the volume htdocs volume to the container's root html file

```
```
(more flexible, you can change file outsidee the container and it will update in the container)
> pwd : enter the path of your working directory, e.g. the path of demo1 folder in docker workshop

```
```
**Create image**

• Create an image
> * "demo2 . ": with the " ." the docker will look at the all the file in current directory and interact them based on the dockerfile
> * Do not need to specify the dockerfile, it will look at a file called "dockerfile" in the current directory by default. However if it is not named as "dockerfile" you can add `-f` to specify the file path

```
```

> -e: environmental variable

```
```
##Docker Compose

• Start the containers

```
```
If the file name isn't "docker-compose", then do:

```
docker-compose -f <path of yoour composed file> up -d
```


```
```
> However, this will not remove the volume of the docker

```
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
docker-machine create manager
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
docker-machine ssh worker1 docker swarm join --token
```
* List all nodes IN the SWARM

```
docker-machine ssh manager docker node ls
```
**Create a service**

• Create a service
> -- replicas 3 : create three instances (3 containers, one for each node)
```


```
```

```
```

**Scaling and Rolling Update**

• Scale up / down (but it will limited to the number of host machines)
> The scalling is manual and not auto for Docker SWARM, but other orchestration tool may can do

```
```

> * Update all the containers in the cluster's image to "alwynpan/comp90024:demo1" by sequences -- allows no downtime update (although users will sees different version while it updating)


```
```





