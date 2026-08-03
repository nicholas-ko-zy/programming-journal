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

- Press `Ctrl-C` to exit the program; Use terminal command below to confirm that the program status is `Exited`

```bash
docker ps -a
```
![](./img/container_of_cats_exited.png)

- Now we try doing `docker run` for `containerofcats` again, but detach the terminal, so that if we kill the process in the terminal, the docker container will still be running. To detach, we need to add the flag `-d`

```bash
# Run this command with the -d flag
docker run -p 8081:80 -d acantril/containerofcats

# Double check that your container is running
docker ps
```

- Copy down the container ID of `containerofcats` in the output of `docker ps`

```bash
# i.e. 8a2bba581eb2
```

- Run the following command to check the port mapping configuration.

```
docker port 8a2bba581eb2
```
Output: ![](./img/docker_port_id.png)

- Execute commands within the container
``` bash
# docker exec -it [YOUR_CONTAINER_ID] [YOUR_COMMAND]

# ps -aux: Command to see all processes running
docker exec -it 8a2bba581eb2 ps -aux
```
Note: If you get an error that `ps` executable file not found in $PATH, your WSL (assuming you're on windows) has no command called `ps`. So install the necessary packages.

``` bash
# Go into the container shell
docker exec -it 8a2bba581eb2 sh

# Install the necessary package
sh-4.4# dnf install -y procps-ng

# Exit the container's shell, bringing you back into your host OS's terminal
sh-4.4# exit

# Check if this command prints the running processes inside your container
docker exec -it 8a2bba581eb2 ps -aux
```

Output: ![](./img/docker_container_ps.png)


- Side note: If you run `df -k` inside the docker container shell, you can see the structure of your docker container's files.

- Other docker commands
```bash
# Restart the docker container
docker restart [YOUR_CONTAINER_ID]
docker stop [YOUR_CONTAINER_ID]
```

- To remove old containers that you've started up
```bash
# Remove container
docker rm [YOUR_OLD_CONTAINER_ID]

# Double check it's been removed
docker ps -a
```

- To remove old **images**
```bash
# Get the ID of your docker image
docker images

# Remove the image from your docker
docker rmi [YOUR_IMAGE_ID]
```

## Dockerfile Syntax

![](./img/docker_file_flowchart.png)

- `FROM`

    Sets the base image for a build (i.e. Alpine or Ubuntu)

- `LABEL`

    Adds metadata to an Image (e.e. description/maintainer)

- `RUN`

    Runs commands in a new layer (e.g. installs or configurations)

- `COPY`

    Copies NEW files/folders from src (client machine) to destination (new image layer)

- `ADD`

    Similar to `COPY` to add files.But has additional feature to add from a remote URL & do extraction etc (e.g. adding application/web files)

- `CMD`

    Sets the default executable of a container & arguments (e.g. web server)

    Can be override vai docker run parameters.

- `ENTRYPOINT`

    Similar to `CMD`, but **can't** be overridden.

    Creates single purpose image.

- `EXPOSE`

    Informs docker what port the container app is running on (metadata only! - no network configuration)


## [DEMO] Build and run a simple containerised application

Creating custom docker images using pre-prepared applications and Dockerfiles.

### App 1: `2048`
```dockerfile
FROM nginx:latest

LABEL maintainer="adrian@cantrill.io" 

COPY 2048 /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

- While inside the `./app1-2048` folder in your terminal, build a docker image.

```bash
# `-t` flag means tag: To assign a human-readable tag to your docker image
# `dockerized-2048` is the name of your image
# `.` Specifies the reference path for your `COPY` command in your Dockerfile
docker build -t dockerized-2048 .
```

- After the build finishes, check that your new image is inside your local images files

```bash
docker images
```

Output: You should see `dockerized-2048` added on top.

![](./img/docker_images_after_docker_build.png)

- Docker run your new image

    ```bash
    # -d: Detach terminal
    # -p: Specify port number
    docker run -d -p 8081:80 dockerized-2048

    # Verify your container is running
    docker -ps

    # Verify the port that `dockerized-2048` is running on.
    docker port [YOUR_CONTAINER_ID] # 80/tcp -> 0.0.0.0:8081
    ```

    Since the docker image is specified to run on port 80 `EXPOSE 80`, you need to docker run on that port. 

- Go to your web browser to check out the running game

Address: `localhost:8081` <- copy + paste into your web browser

![](./img/localhost_2048.png)

### App 2: `containerofcats`

Dockerfile (less efficient, uses a heavier image)

```dockerfile
FROM redhat/ubi8

LABEL maintainer="Animals4life"

RUN yum -y install httpd

COPY index.html /var/www/html/

COPY containerandcat*.jpg /var/www/html/

ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]

EXPOSE 80
```

- `RUN`: Installs the Apache 2 webserver
- `COPY`: The main web page `index.html`
- `COPY`: Wildcard to copy all the cat images.

^ Can copy one time only, but for educational purposes, two copies for less efficiency.

- Do a `docker build` while you're inside the app2 directory

```
docker build -t containerofcats .
```

Moral of the story: Select an appropriate base image to `RUN` for your application.

- Check that the build is completed
```bash
docker images
```

- `docker run`, match the exposed port
```bash
docker run -d -p 8081:80 containerofcats
```

- End the process

```bash
docker stop [YOUR_CONTAINER_ID]
docker rm [YOUR_CONTAINER_ID]
```

## Docker Storage - Writable Layer, Bind Mounts & Volumes









