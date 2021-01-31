## datou-docker
---

#### Summary

Docker is a tool that allows you to package your applications in a virtual OS container that can be shared and deployed. Key benefits include:

- process isolation
- portability across platforms


This project will review the most common commands that you will use day to day while drill into a few commands to show some tricks for a select few commands that you will want to know about in-depth like `docker run`.

#### Pre-requisites

| Tool | Install |
| -- | -- |
| Docker | [Link](https://hub.docker.com/editions/community/docker-ce-desktop-mac/) |

#### Terminology

- `Image` - a packaged, versioned application
- `Container` - running instant of an image
- `Registry` - registers images for sharing publicly or privately

#### Docker Registries & Images

Docker containers can be hosted on public or private registries. 

The default public registry that Docker pulls from if you don't specify a specific registry is: docker.io

Popular private container registries include

- Docker Hub
- Elastic Container Registry (AWS ECR)
- Github Container Registry (GHCR)
- Google Container Registry (GCR)

Image Names follow the format for example:

```
    [REGISTRYHOST:PORT/][USERNAME/]NAME[:TAG]    
```

- `REGISTRYHOST` - host (ie, docker.io)
- `PORT` - host port
- `USERNAME` - or organization (ie, nginx)
- `NAME` - image name (ie, nginx)
- `TAG` - version (ie, latest)

#### Common Images
| Images | Description |
| -- | -- |
| nginx:latest | Popular web server |
| dockerinaction/helloworld | Just prints hello world and ends |
| busybox:latest | Small container with UNIX utilities |

#### Common Commands

**Help**
- `docker help` - retrieve help topics
- `docker help ps` - retrieve help topics specifically for command `ps`

**Images**
- `docker pull` - pulls an image from a registry
- `docker images` - shows the images you have stored locally on your computer
- `docker rmi` - removes an image (use with `docker images`)

**Containers**
- `docker ps` - shows the current running containers
- `docker ps -a` - shows all containers even if they are not running
- `docker run` - runs an image as a container
- `docker stop` - stops a running container
- `docker rm` - removes a container (use with `docker ps -a`)

**Inspection**
- `docker logs <container_name>` - view logs from this container
- `docker exec <container_name> <command>` - run a command against a container
- `docker top <container_name>` - view processes running in a container

#### Docker Run In Detail

##### A detached container 

```
    > docker run --detach --name web nginx:latest
```

Things to note:

- `--detach` specifies running the process in the background
- `--name` specifies a named reference to this container when it is running

##### An interactive container 

```
    > docker run --interactive --tty --link web:web --name web_test busybox:latest /bin/sh
```

Things to note:

- `--interactive` - stream in standard input
- `--tty` - virtual terminal
- `--link web:web` - links this container to another named web

##### Run container with a starting command

```
    > docker run -d --name busy busybox:latest /bin/sh -c "sleep 30000"
```

##### Run container as read-only with a mounted drive

```
    > docker run -d --name <name> --read-only -v /web --tmpfs /tmp <image>
```

Things to note:

- `--read-only` - sets the container volumes to read-only
- `-v` - mounts a writable drive at the location
- `--tmpfs` - mounts a writable temporary directory

##### Run container with environment variables passed in

```
    > docker run -d --name <name> --env SOME_VAR=<SOME_VALUE>
```

##### Run container and open a localport forward to the container port

```
    > docker run -d --name <name> -p 8000:80 <image>
```

##### Run container that will restart

```
    > docker run -d --name <name> --restart always <image>
```

#### Filesystems

Types of volumes to a container

- `bind mount` - mounts a location on the host system to the container at a specified location (ie, docker run --mount type=bind,src=<SRC>,dst=<DST>)
- `inmemory` - temporary storage in a container (ie, docker run --mount type=tmpfs,dst=/tmp,tmpfs-size=16k,tmpfs-mode=1770)
- `docker volume` - managed by docker on the host file system. You create the volume (ie, docker volume create --driver local --label name=value example-volume) and then mount it. (ie, docker run --mount type=volume,src=example-volume,dst=/data)

Some quick management commands:

- `docker volume list` - list all volumes
- `docker volume remove <id>` - remove a volume

#### Networking

Networking Basics

- `Host Network` - references the network that the server/computer is connected
- `Bridge Network` - connects all the containers on a server
- Each container has a network address that is attached to the bridge
- `docker network ls` - shows the interfaces
- `docker network create ... ` - create a bridge
- `docker network connect <network1> <network2>` connects 2 bridge networks together
- `docker attach <network>` - attaches the network to docker. it will create a new ethernet link on containers (ie, a new eth1 interface)

DNS Management

- `docker run --dns-search datou.local --dns=8.8.8.8 ...` - modify the /etc/resolv.conf file
- `docker run --add-host test:127.0.0.1 ...` - modify the /etc/hosts file

#### Resource Limits

MEMORY

- `docker run --memory 256m ...` - denotes allowable memory

CPU

- `docker run --cpu-share=1024 ...` - denotes the relative cpu on the server vs another container
- `docker run --cpu=0.75 ...` - denotes the raw cpu limit per calculated period
- `docker run --cpuset-cpus 0 ...` - for servers with multiple cores, to specify which core it can use

#### Building a Dockerfile

The difference directives of a Dockerfile that behave like a commandline building of a docker image

- `FROM` - the 
- `LABEL` - metadata (ie, --label on cli)
- `RUN` - runs a provided command
- `ENV` - sets environment variables (ie, --env on cli)
- `WORKDIR` - default working dir
- `COPY` - copies files from local filesystem into the container (ie, COPY ["./someFiles", "./someMoreFiles", /destination] - the last item is the destination in the container)
- `VOLUME` - create volume in the container - this is more limiting vs runtime mounting 
- `ENTRYPOINT` - the executable to run at container startup (default is bin/sh -c)
- `CMD` - arguments passed to the ENTRYPOINT
- `EXPOSE` - the port to expose when container is run
- `USER` - sets up the user on the container after the build is complete
- `HEALTHCHECK` - an instruction to run periodically to check health of container 

Multi-stage builds have multiple FROM lines that break up each stage of building. They are useful in separating build time and runtime concerns. 

#### Docker Compose

Similar to how Dockerfile declares the imperative approach of managing containers. Docker Compose is the declarative approach for the docker swarm command that orchestrates containers in an environment.

