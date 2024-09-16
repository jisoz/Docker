![alt text](<Screenshot (212).png>)

# Docker

Docker allows developers to package applications along with their dependencies into containers. These containers can run consistently across different computing environments, making it easier to develop, test, and deploy applications.


docker version
docker info 

# Key Components of Docker Architecture
1. Docker Daemon (dockerd):

The daemon is the core of Docker, responsible for managing containers, images, volumes, and networks. It listens for Docker API requests and processes them.
The daemon runs as a background process on the host machine.

2. Docker Client (CLI):

The Docker client (docker) is a command-line tool that interacts with the daemon using REST APIs. Commands like docker run, docker build, and docker pull are sent to the Docker daemon.

3. Images:

Docker images are immutable templates used to create containers. Images are made up of multiple layers. These layers can be shared across containers, making them lightweight.
The Union File System (UFS) is used to manage these image layers efficiently.
4. Containers:

Containers are runtime instances of Docker images. A container consists of an image, environment variables, configurations, and networking. Docker containers are isolated using Linux technologies like namespaces and cgroups.

5. Docker Registry:

A central location where Docker images are stored and distributed. Docker Hub is the default registry, but private registries can also be configured. Docker images are pulled from the registry using commands like docker pull.


containers is a running instance of an image 
we can have many containers running same image

docker have centralised docker repo => docker hub 

list running containers: docker container ls 
                         docker ps 


docker run -d -p 8080:8080 --name nginx  nginx 

docker container logs [container id / container name ]

docker container top [......]  see process running inside container

# container vs vm
containers virtualise the os / vm virtualise the hardware 


# container networking
Each container connect to vpn called bridge  (default network driver of Docker)

docker run -d -p 8080:8080 --name nginx1 nginx
docker inspect nginx1
2 containers with bridge network can communcate with each other via ip address


1-) bridge networking : by default contaienrs are connected to docker bridge network allowing them to communicate with each other on the same host 

2-) overlay networking : enable communication between container accross different hosts 
3-) container network interfaces (cni) : interface tht will help you to setup the communication between docker runtime environemetn and the network plugin that will be used between kubernetes and swarm 

show all networks: docker network ls 
filter network : docker network -f drive=bridge

crreate netwrok: docker netwrok create [name]
                 docker network inspect [name]
                 docker network rm [name] (if we want to remove it )


# data management 
data volumes assignemet : 
docker run -d --name=mysql-test -e "MYSQL_ROOT_PASSWORD=mypassword" --mount source=mysql-dbtest,target=/var/lib/mysql mysql:8.0

docker exec -it ....
mysql -u root -p
CREATE DATABASE testdb;
USE testdb;
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
SELECT * FROM users;

exit 

docker stop , docker rm ,

mount bind assignement: 
docker run -d  -p 80:80 --name nginx-test --mount type=bind,source="$(pwd)",target=/usr/share/nginx/html  nginx
