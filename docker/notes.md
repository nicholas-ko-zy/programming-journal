# Docker
- Resource: https://learn.cantrill.io/courses/docker-fundamentals/lectures/44151189

## Introduction

**Physical Servers vs Virtual Machines**

From top to bottomw

<u>Physical Servers</u>
- Application Layer
- Runtime Environment, Dependencies & Libraries
- Operating System (Windows, Linux, MacOS)

<u>Virtual Machines</u>
- VM Layer
    - VM 1: OS/Runtime/App
    - VM 2: OS/Runtime/App
    - ...
    - VM $n$: OS/Runtime/App
- Hypervisor <- referee to allocate hardware resources for virtual machines

Containers -> How virtual machines are isolated.

## Containers

Types of containers I know: 
- Docker containers
- Windows containers
 
What's inside a (generic)container? (From top-to-bottom)

- App
- Libs
- Guest OS
- Hypervisor
- Host OS
- Hardware

What's inside a Docker container?

- App Libs Layer
    - App/Libs 1
    - App/Libs 2
    - ...
    - App/Libs $n$
- Docker (Container) Engine <- replaces the hypervisor
- Host OS
- Hardware

The 'app libs' layer in a docker container architecture is more lightweight since there is no duplication of guest OS(s), all it is sharing the host OS, which I assume is a some Linux instance?

Summary:
- Containers run on a container host
- ...via container (Docker) Engine
- Containers only run an APP & libraries / Runtime Env
- Share the container HOST OS (run as a process on it)
- Lightweight - Can be densely packed & started/restarted quickly
- Can be impaced by other containers

## Docker Architecture

Think of it as a client-server application. 

- Docker Host
    - Running the docker daemon
    - Server part of the docker engine's 'client-server' architecture
    - Provides API access used by docker clients, in the form of 
        + CLI tool
        + Docker Desktop
    - Is composed of
        + Containers
        + Images
- Docker Engine 
- Registry (HUB)
    - A public / private storage for images e.g. Docker Hub
    - Images are used to run containers, i.e. (A base Linux image.) You can either 
        + download an image: In your CLI, do `docker pull` to download an image. This will pull images from the Registry into the daemon's image pool.
        + create an image: In your CLI do `docker build`, which takes a dockerfile as an input. The dockerfile is passed onto the Docker Daemon, to create an image.

Once you have one or more docker images running on your Docker Daemon, you can do `docker run` to run containers using docker images. Containers turns docker images, read-only, into read-write, which allows the container to be run on a docker host. 

Once you've created an image, you can also do `docker push` to move that image into a registry, either publicly or privately.

Summary of commands:

- `docker pull`: Pull an image from a registry into your Docker Host's image pool.
- `docker build`: Create an image using a Dockerfile, and add it to your Docker Host's image pool.
- `docker run`: Turn images into read-write containers.
- `docker push`: Add images to the Registry (public/private)

## Demo: Interacting with Docker Engine

ENSURE DOCKER-DESKTOP IS RUNNING IN THE BACKGROUND

Running a "hello-world" container.

High-level steps
1. Download a docker image
2. Use that docker image to spin up a docker container
3. Interact with the container

Actual Steps

- Check docker is running in your command line with
    ```
    docker ps
    ``
- Check if you have any docker images

    ```
    docker images
    ```

- Run `hello-world`

    ```
    docker run hello-world
    ```

    Message in my terminal (It will try to find the image locally first then download from the web):
    > Unable to find image 'hello-world:latest' locally
    > latest: Pulling from library/hello-world
    > 4f55086f7dd0: Pull complete
    > d5e71e642bf5: Download complete
    > Digest: sha256:c3cbe1cc1aa588a64951ac6286e0df7b27fe2e6324b1001c619bb358770c0178
    > Status: Downloaded newer image for hello-world:latest

    > Hello from Docker!
    > This message shows that your installation appears to be working correctly.

    > To generate this message, Docker took the following steps:
    > 1. The Docker client contacted the Docker daemon.
    > 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    >     (amd64)
    > 3. The Docker daemon created a new container from that image which runs the
    >     executable that produces the output you are currently reading.
    > 4. The Docker daemon streamed that output to the Docker client, which sent it
    >     to your terminal.

    > To try something more ambitious, you can run an Ubuntu container with:
    > $ docker run -it ubuntu bash

    > Share images, automate workflows, and more with a free Docker ID:
    > https://hub.docker.com/

    > For more examples and ideas, visit:
    > https://docs.docker.com/get-started/
 
 The docker image stops running after the `docker run` command above. To view your docker run history do

 ```
 docker ps -a
 ```

 ![](./img/docker_ps_-a.png)

 Now after we've pull `hello-world` from the Docker Hub Registry, if we do 

 ```
 docker images
 ```

 We'll see that hello-world is now localled stored on our computer.

 ![](./img/docker_images.png)

## Container and Image Architecture

**Docker Images**: A read-only immutable template. Any change will be creating a new image.

- A collection of file system layers. Layers are mutually exclusive, they only contain the differences of other layers inside the image. Example
    - Top Layer: application
    - Middle layer: env/libs
    - Bottom layer: a base Linux
- Separate layers are independent and can be reused.


**Docker Containers**: A writeable layer added to the image. Comes with container storage, i.e. local storage in the container.

Each container has a unique writable layer. This keeps containers isolated while using the same image.

## Demo: Working with existing docker images

- Pull `containerofcats` from Docker Hub

```bash
docker pull acantril/containerofcats
```

- Inspect the image to see
    - When it was created
    - Who created it
    - Description
    - Volumes
    - Entry point
    - Architecture/OS
    - Layers

```bash
# Last bit is the docker image, you can get it from running docker images
docker inspect 3ffa9b0efe79
```

- Run `containerofcats`

```bash
# -p: Specifies the to docker host 80 (image) to 8081 (local)
docker run -p 8081:80 acantril/containerofcats
```

Output:
![](./img/container_of_cats.png)

Stopped at 08:32 of the video.