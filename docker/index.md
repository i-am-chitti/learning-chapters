# Docker

### What is Docker?
Docker is a tool that lets you package an application and everything it needs to run. An application needs many dependencies to run - libraries, code, config etc. It packages all into a single unit and can be run the same on any machine. With docker, we have same config, same dependencies and same libraries version.

> Docker fixes classic dev problem => It works on my machine!!

### Key Concepts

#### Image
A blueprint of how to run an application. Example - Node JS v20, application code, installed dependencies. Images are immutable.

#### Container
An image in running state. We can run many containers from a single image.

#### Dockerfile
A text file that says - how to build the image.

#### Docker Engine
A platform that builds the image and runs containers.

#### Volume
Containers are ephemeral. Without volume, container deleted -> data gone. With volumes, data lives outside container. Used for DB, file, logs.

#### Network
Containers need to talk to each other. This is facilitate via virtual network created by docker.

#### Image Registry
To distribute images, registry is used. So, registry hosts/stores the image. Images live in docker hub, private registries or cloud registries.
> Build -> Push -> Pull -> Run

#### Docker Lifecycle
![Docker Lifecycle](./images/docker-lifecycle.jpg)

### Docker vs Virtual Machine
Docker
- Lightweight
- Shares host OS
- Boots in seconds
- Perfect for microservices

VM
- Heavyweight
- Has its own OS
- Boots in minutes
- Better for OS-isolation

### Docker CLI

#### Check docker is installed & running

```
docker --version
docker info
```

#### Docker Images

##### List Images

```
docker images
```

##### Pull an image

```
docker pull node:20
```

##### Remove

```
docker rmi node:20
```