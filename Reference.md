# Docker

## Why Docker?

Code works on my system, but not on yours. This is a common problem in software development. Docker solves this problem by creating a container that can run on any system.

Contanerize application, run each service with its dependencies in a container. This makes it easy to run the application on any system.

## Container vs VM

Containers are lightweight and share the host system's kernel, while virtual machines (VMs) are isolated and have their own operating system. 

Containers are faster to start, use less resources, and are more portable compared to VMs. However, VMs provide stronger isolation and security.

![Container vs VM](SWTM-2060_Diagram_Containers_VirtualMachines_v03.png)

## Shared kernel

Docker containers share the host system's kernel. So lets say we have Ubuntu host and container needs alpine linux, the container will use the host's linux kernel.

When we have Windows host and we want to run a linux container, we need to use WSL2.

## Docker commads

- `docker run -d` - Run a container

- `docker run -it` - Run a container in interactive mode(We can pass input to the container)

- `docker run -v /opt/datadir:/var/lib/mysql mysql` - Use external dir for storing data. Data will remain even if container is deleted.

- `docker ps` - List running containers

- `docker ps -a` - List all containers

- `docker images` - List images

- `docker exec <container-id> <command>` - Execute a command in a running container

- `docker stop` - Stop a running container

- `docker rm` - Remove a container

- `docker rmi` - Remove an image

- `docker stop $(docker ps -a -q)` - Stop all containers

- `docker rm $(docker ps -a -q)` - Remove all containers

- `docker rmi $(docker images -q)` - Remove all images

- `docker logs <container-id>` - View logs of a container

- `docker inspect <container-id>` - View detailed information about a container(IP, environment variables, etc)

## ENTRYPOINT vs CMD

- `ENTRYPOINT` - Command that will be executed when the container starts. If we pass a command to the container, it will be appended to the entrypoint.

- `CMD` - **Default command** that will be executed when the container starts. If we pass a command to the container, it will override the CMD.

```Dockerfile
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["apache2-foreground"]
```
If we run the image with `docker run -it my-image /bin/bash`, it will run `docker-entrypoint.sh /bin/bash`.\
If we run the image with `docker run -it my-image`, it will run `docker-entrypoint.sh apache2-foreground`.

## Docker compose

```yaml
version: '3'
services:
  backend:
    build: backend
    restart: always
    secrets:
      - db-password
    environment:
      MYSQL_HOST: db
    networks:
      - react-spring
      - spring-mysql
    depends_on:
      db:
        condition: service_healthy
  db:
    image: mariadb:10.6.4-focal
    environment:
      - MYSQL_DATABASE=example
      - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db-password
    restart: always
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "--silent"]
      interval: 3s
      retries: 5
      start_period: 30s
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - spring-mysql
  frontend:
    build:
      context: frontend
      target: development
    ports:
      - 3000:3000
    volumes:
      - ./frontend/src:/code/src
      - /project/node_modules
    networks:
      - react-spring
    depends_on:
      - backend
    expose:
      - 3306
      - 33060
volumes:
  db-data: {}
secrets:
  db-password:
    file: db/password.txt
networks:
  react-spring: {}
  spring-mysql: {}
```

Use ```docker compose up -d``` to start all the services.

## Container orchestration

Multiple docker hosts can be managed using container orchestration tools like Kubernetes, Docker Swarm, Mesos, etc.

### Docker Swarm

One host is the manager and other hosts are workers. Manager distributes the work to workers.

```docker service create --replicas 3 -p 80:80 --name my-service my-image``` Run this on manager and it will create containers on 3 hosts.

### Kubernetes

Kubernetes is a container orchestration tool that manages containers in a cluster. It provides features like scaling, load balancing, self-healing, etc.