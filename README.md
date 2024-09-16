![alt text](<Screenshot (212).png>)

# Docker

Docker allows developers to package applications along with their dependencies into containers. These containers can run consistently across different computing environments, making it easier to develop, test, and deploy applications.


docker version
docker info 
docker stop , docker rm ,
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


# Docker Daemon Socket
By default, the Docker daemon communicates via a Unix socket /var/run/docker.sock. This socket is used for API communication between the client and the daemon. Securing this socket (e.g., with TLS) is crucial in production environments to prevent unauthorized access.

# container vs vm
containers virtualise the os / vm virtualise the hardware 

#  Multi-Stage Builds
Multi-stage builds allow you to separate build dependencies from runtime dependencies, keeping final images smaller. It’s particularly useful for reducing image size when building complex applications.

## Example of Multi-Stage Build

 Stage 1: Builder
FROM golang:1.16 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

Stage 2: Runtime
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/myapp .
CMD ["./myapp"]

In this example, only the final binary (myapp) is included in the final image, making it much smaller than the image that includes all build tools.

# Docker Networking Overview
Docker networking is how containers communicate with each other, the host system, and the outside world. Each container has a network interface connected to a virtual network.

Each container connect to vpn called bridge  (default network driver of Docker)
## Types of Docker Networks:
Bridge Network: Default network for standalone containers. Containers can communicate with each other using IP addresses or container names.
Host Network: Removes network isolation between the Docker container and the host. The container shares the host’s networking stack.
Overlay Network: Used in Docker Swarm or across multiple Docker hosts. Enables communication between containers running on different Docker daemons.
Macvlan Network: Assigns MAC addresses to containers, giving them direct access to the network like a physical device. Often used for legacy applications.

1-) bridge netwok : docker network create --driver bridge my_bridge_network
By default, containers are attached to the default bridge network. However, for custom setups, you can specify a network: docker run -d --name web1 --network my_bridge_network nginx
Connecting an Existing Container to a New Network: You can attach a running container to a network.=> docker network connect my_bridge_network web1
Inspecting Network Details: To view network configuration and container connections:docker network inspect my_bridge_network

2-) Host Network : docker run --network host nginx
Use Cases for Host Network:

High-performance, low-latency network access.
Containers that require access to services running on the host machine.
Containers that need to bind directly to the host's IP address.


# Volumes and Data Management in Docker
Containers are usually ephemeral, meaning that any data stored inside the container is lost once the container is deleted. However, Docker provides a way to persist data through volumes and bind mounts, which allow you to store and manage data outside the lifecycle of individual containers.
## volumes :
Volumes are Docker’s preferred mechanism for persisting data. Volumes are stored outside the container's filesystem and are managed directly by Docker.

Creating a Volume: docker volume create my_volume
                   docker volume inspect my_volume
                   docker run -d -v my_volume:/var/lib/mysql mysql
                   docker volume rm my_volume
Note: You cannot remove a volume that is currently in use by a container.


Backup a Volume: docker run --rm -v my_volume:/data -v $(pwd):/backup busybox tar cvf /backup/backup.tar /data
This creates a tar archive of the my_volume and saves it in the current directory on the host.
Restore from a Backup: docker run --rm -v my_volume:/data -v $(pwd):/backup busybox tar xvf /backup/backup.tar -C /data

## bind mounts
 bind mounts allow you to use a directory on the host filesystem directly. Bind mounts are useful when you need to have direct access to the filesystem of the host.

 Direct Host Access: You can directly mount a host directory into the container.
Use Case: Useful for development environments where you want to share code between your host and the container, allowing live updates without needing to rebuild the container.

docker run -d --name my_app -v /host/path:/container/path nginx


## volumes vs bind mounts 
Comparison:
Volumes: Managed by Docker, better for long-term data storage and sharing between containers.
Bind Mounts: Direct mapping to host filesystem, useful for development and testing, but not as flexible or optimized as volumes.


# Docker compose 
## Multi-Container Applications
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "8080:80"
    networks:
      - frontend
    volumes:
      - web_data:/usr/share/nginx/html

  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: example
    networks:
      - backend
    volumes:
      - db_data:/var/lib/postgresql/data

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  web_data:
  db_data:



## Service Dependencies
Docker Compose allows you to specify dependencies between services using depends_on. This ensures that services start in a specific order.
version: '3.8'

services:
  web:
    image: nginx
    depends_on:
      - db
    ports:
      - "8080:80"

  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: example


Here, the web service depends on the db service, ensuring that the db container is started before the web container.


## Environment Variables and Secrets
Docker Compose allows you to manage environment variables and secrets for your services.

- Using .env File: Create a .env file: POSTGRES_PASSWORD=example
                 version: '3.8'
                  
                  services:
                    db:
                      image: postgres
                      environment:
                        POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

- Using Docker Secrets:
  For more sensitive data, Docker Secrets can be used (requires Docker Swarm).
  echo "example_password" | docker secret create db_password -


version: '3.8'

services:
  db:
    image: postgres
    secrets:
      - db_password

secrets:
  db_password:
    external: true




## Scaling Services

docker-compose up --scale web=3

Define Replicas in docker-compose.yml:
version: '3.8'

services:
  web:
    image: nginx
    deploy:
      replicas: 3


## Multi-Stage Builds in Docker Compose
Multi-stage builds optimize Docker images by separating build and runtime environments.

Example Dockerfile:
Build Stage
FROM node:14 AS build
WORKDIR /app
COPY . .
RUN npm install && npm run build

 Runtime Stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html



Using Multi-Stage Builds in docker-compose.yml:
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"

