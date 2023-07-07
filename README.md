# docker-notes
Docker notes.

# Table of Contents
* **[Introduction](#introduction)**
  * **[What is Docker?](#what-is-docker)**
  * **[Docker Engine](#docker-engine)**
  * **[Docker Architecture](#docker-architecture)**
  * **[Docker Objects](#docker-objects)**
  * **[The underlying technology](#the-underlying-technology)**
* **[Installation](#installation)**
* **[Containers](#containers)**
  * **[Ports in Containers](#ports-in-containers)**
* **[Images](#images)**
  * **[Create our own images](#creat-our-own-images)**
* **[Volumes](#volumes)**
  * **[Choose the right type of mount](#choose-the-right-type-of-mount)**
    * **[Volumes](#volumes-1)**
    * **[Bind Mounts](#bind-mounts)**
    * **[tmpfs Mounts](#tmpgs-mounts)**
  * **[Good Use case for Volumes](#good-use-case-for-volumes)**
  * **[Good Use case for Bind Mounts](#good-use-case-for-bind-mounts)**
  * **[Good Use case for tmpfs Mounts](#good-use-case-for-tmpfs-mounts)**
  * **[Practical example](#practical-example)**
* **[Container life cycle (Create/Start/Stop/Kill/Remove)](#container-life-cycle-createstartstopkillremove)**
* **[Dockerfiles](#dockerfiles)**
  * **[How to start](#how-to-start)**
  * **[The Application](#the-application)**
  * **[Build the Dockerfile](#build-the-dockerfile)**
  * **[Run the App](#run-the-app)**
* **[Docker Compose: Linkar Containers](#docker-compose-linkar-containers)**
* **[Networking](#networking)**
  * **[Introduction](#introduction)**
  * **[Network Containers](#network-containers)**
    * **[1. Creat own "Bridge Network"](#1-creat-own-bridge-network)**
    * **[2. Add Containers to a Network](#2-add-containers-to-a-network)**
* **[Cleaning](#cleaning)**
  * **[Containers](#containers-1)**
  * **[Images](#images-1)**
  * **[Volumes](#volumes)**
* **[CheatSheets](#cheatsheets)**
  * **[Docker General Commands](#docker-general-commands)**
  * **[Dockerfile Commands](#dockerfile-commands)**

# Introduction

### What is Docker?

Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly.

### Docker Engine

Docker Engine is a *client-server* application with these major components:

* A server which is a type of long-running program called a daemon process (the dockerd command).

* A REST API which specifies interfaces that programs can use to talk to the daemon and instruct it what to do.

* A command line interface (CLI) client (the docker command).

<p align="center">
  <img src="images/engine-components-flow.png">
</p>


### Docker Architecture

As previously mentioned, Docker uses a **client-server** architecture.
* The **Docker client** talks to the **Docker daemon**, which does the heavy lifting of building, running, and distributing your Docker containers.
* The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon.
* The Docker client and daemon communicate using a **REST API**, over UNIX sockets or a network interface.

<p align="center">
  <img src="images/architecture.svg">
</p>

### Docker Objects

When you use Docker, you are creating and using **images**, **containers**, **networks**, **volumes**, **plugins**, and **other objects**. This section is a brief overview of some of those objects.

* **Image:**
  * An _image_ is a lightweight, stand-alone, executable package that includes everything needed to run a piece of software, including the code, a runtime, libraries, environment variables, and config files.

  * An image is a read-only template with instructions for creating a Docker container. Often, an image is based on another image, with some additional customization.

  * You might create your own images. To do that, you create a **Dockerfile** with a simple syntax for defining the steps needed to create the image and run it. Each instruction in a Dockerfile creates a layer in the image.

* **Container:**
  * A _container_ is a runtime instance of an image.
  * **Isolation:** It **runs completely isolated** from the host environment by default, only accessing host files and ports if configured to do so. By default, a container is relatively well isolated from other containers and its host machine. You can control how isolated a container’s network, storage, or other underlying subsystems are from other containers or from the host machine.
  * Containers **run apps natively** on the host machine’s kernel.
  * They have **better performance** characteristics than virtual machines that only get virtual access to host resources through a hypervisor.
  * Containers can get native access, each one running in a discrete process, taking no more memory than any other executable.

### The underlying technology

Docker is written in **Go** and takes advantage of several features of the Linux kernel to deliver its functionality.

* **Namespaces**

Docker uses a technology called **namespaces** to provide the isolated workspace called the container. When you run a container, Docker creates a set of namespaces for that container.

These namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace.

**Docker Engine** uses namespaces such as the following on Linux:

  * **The `pid` namespace:** Process isolation (PID: Process ID).
  * **The `net` namespace:** Managing network interfaces (NET: Networking).
  * **The `ipc` namespace:** Managing access to IPC resources (IPC: InterProcess Communication).
  * **The `mnt` namespace:** Managing filesystem mount points (MNT: Mount).
  * **The `uts` namespace:** Isolating kernel and version identifiers. (UTS: Unix Timesharing System).

* **Control Groups**

Docker Engine on Linux also relies on another technology called control groups (**cgroups**). A **cgroup** limits an application to a specific set of resources. Control groups allow Docker Engine to share available hardware resources to containers and optionally enforce limits and constraints. For example, you can limit the memory available to a specific container.

* **Union File Systems**

Union file systems, or **UnionFS**, are file systems that operate by **creating layers**, making them **very lightweight** and **fast**. Docker Engine uses UnionFS to provide the building blocks for containers. Docker Engine can use multiple UnionFS variants, including AUFS, btrfs, vfs, and DeviceMapper.

* **Container Format**

Docker Engine combines the namespaces, control groups, and UnionFS into a wrapper called a **container format**. The default container format is **libcontainer**. In the future, Docker may support other container formats by integrating with technologies such as BSD Jails or Solaris Zones.

# Installation

When we talk about "installing docker" we are referring to installing "Docker Engine". To do this, we will do it following the steps indicated in the [Official Documentation](https://docs.docker.com/engine/installation/).

# Containers

The following are the most common commands with "containers":

### `ps`

Shows running containers.

* See ripped containers: 
```shell
$ docker ps
```
* View "all history" of booted containers:
```shell
$ docker ps -a
```

* View the "ID" (-q) of the last (-l) container:
```shell
$ docker ps -l -q
```

### `exec`
To access the shell of a "container" that we had previously started:

Indicating your ID:
```shell
$ sudo docker exec -i -t 665b4a1e17b6 /bin/bash
```

o but also through his name:
```shell
$ sudo docker exec -i -t loving_heisenberg /bin/bash
```

### `run`
Create and start a "container".
By default, a container starts, executes the command that we tell it to, and stops.:
```shell
$ docker run busybox echo hello world
```

Usually `docker run` has the following structure:
```shell
$ docker run -p <puerto_host>:<puerto_container> username/repository:tag
```

We can pass various options to the `run` command, such as:

* `run` **interactivo**:
```shell
$ docker run -t -i ubuntu:16.04 /bin/bash
```
  * **-h:** configure a hostname to the "container".
  * **-t:** Assign a TTY.
  * **-i:** Communicate with the "container" interactively.

  **Nota:** When exiting interactive mode (-i) the "container" will stop.

* `run` **Detached Mode**:

  As we already know, after running a container interactively, it ends. If you want to make containers that run services (for example, a web server) the command is the following:
  ```shell
  $ docker run -d -p 1234:1234 python:2.7 python -m SimpleHTTPServer 1234
  ```
  * **Explanation:** This runs a server **Python** (SimpleHTTPServer module), on the port **1234**.
    * **-p 1234:1234** tells Docker to _port forward_ the container to port 1234 on the host machine. Now we can open a browser at the address http://localhost:1234.

    * **-d:** makes the "container" run in the background. This allows us to execute commands on it at any time while it is running. For example:
    ```shell
    $ docker exec -ti <container-id> /bin/bash
    ```

      * Here you simply open a **tty** in **interactive** mode. Other things could be done like changing the _working directory_, setting environment variables, etc.


### `inspect`
Display details about a container:

* Information about a container:
```shell
$ docker inspect
```

* Container IP address:
```shell
$ docker  inspect --format '{{ .NetworkSettings.IPAddress }}' <nombre_container>
```

### `logs`
* Show logs about a container:
```shell
$ docker logs
```

### `stats`

* Container statistics (CPU,MEM,etc.):
```shell
$ docker stats
```

### Ports in Containers
* Container public ports:
```shell
$ docker port
```
* Publish container port 80 to a random Host port:
```shell
$ docker run -p 80 nginx
```
* Publish container port 80 to host port 8080:
```shell
$ docker run -p 8080:80 nginx
```
* Publish all exposed ports of the container to random ports of the Host:
```shell
$ docker run -P nginx
```
* List all the port mappings of a container:
```shell
$ docker port <container_name>
```

# Images

* See list of images:
```shell
$ docker images
```

* View "all history" of ripped images:
```shell
$ docker images -a
```

* Delete an image:
```shell
$ docker rmi <images_name>
```

* To know the history and "layers" that an image has:
```shell
$ docker history <image_name>
```

  ### Creat our own Images:

  We can create our own images in different ways:

  A) `docker commit`: build an image from a container.

  Example:
  ```shell
   $ docker commit -m "Mensaje que queramos" -a "Nombre del que lo ha hecho" container-id NEW_NAME:TAG
   $ docker commit -m "MongoDB y Scrapy instalados" -a "Etxahun" 79869875807 etxahun/scrapy_mongodb:0.1
   ```
  B) `docker build`: create an image from a "Dockerfile" by executing the build steps given in the file.

  Within a Dockerfile the "instructions" that we can use are the following:

   * **FROM**: the base image for building the new docker image; provide "FROM scratch" if it is a base image itself.
   * **MAINTAINER**: the author of the Dockerfile and the email.
   * **RUN**: any OS command to build the image.
   * **CMD**: specify the command to be stated when the container is run; can be overriden by the explicit argument when providing docker run command.
   * **ADD**: copies files or directories from the host to the container in the given path.
   * **EXPOSE**: exposes the specified port to the host machine.


  Example:
  ```shell
  $ nano myimage/Dockerfile

  FROM ubuntu
  RUN echo "my first image" > /tmp/first.txt

  $ docker build -f myimage/Dockerfile   || o sino ||   docker build myimage
  Sending build context to Doker daemon 2.048 kB
  Step 1: FROM ubuntu
  ----> ac526a456ca4
  Step 2: RUN echo "my first image" > /temp/first.txt
  ----> Running in 18f62f47d2c8
  ----> 777f9424d24d
  Removing intermediate container 18f62f47d2c8
  Succesfully built 777f9424d24d

  $ docker images | grep 777f9424d24d

  <none>	<none>		777f9424d24d		4 minutes ago		125.2 MB

  $ docker run -it 777f9424d24d
  $ root@2dcd9d0caf6f:/#
  ```
  We can put a name or "tag" the image at the time we are doing the "build":
  ```shell
  $ docker build <dirname> -t "<imagename>:<tagname>"
  ```
  Example:
  ```shell
  $ docker build myimage -t "myfirstimage:latest"
  ```
### Tag the image

The notation commonly used to associate a local image with a "repository" within a "registry" is as follows: `username/repository:tag`. The "tag" part is optional, but recommended, since it is the way in which we will version the images in Docker

To assign the "tag":

```shell
$ docker tag image username/repository:tag
```

Example:

```shell
docker tag friendlyhello john/get-started:part2
```


To check the image that we have just tagged:

```shell
$ docker images_name

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
john/get-started         part2               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```

### Publish the image

To publish the image:
```shell
$ docker push username/repository:tagear
```

Once uploaded, we can see on the Web of [Docker Hub](https://hub.docker.com/ "Docker Hub").

# Volumes

**Referencias:**
* [Docker Volumes Tutorial](http://containertutorials.com/volumes.html "Docker Volumes Tutorial")

It is possible to store data within the writable layer of a container, but there are some downsides:

* The data won’t persist when that container is no longer running, and it can be difficult to get the data out of the container if another process needs it.

* A container’s writable layer is tightly coupled to the host machine where the container is running. You can’t easily move the data somewhere else.

* Writing into a container’s writable layer requires a storage driver to manage the filesystem. The storage driver provides a union filesystem, using the Linux kernel. This extra abstraction reduces performance as compared to using data volumes, which write directly to the host filesystem.


Docker offers **three different ways to mount data** into a container from the Docker host:
* Volumes
* Bind mounts
* Tmpfs volumes

When in doubt, volumes are almost always the right choice.

### Choose the right type of mount

No matter which type of mount you choose to use, the data looks the same from within the container. It is exposed as either a directory or an individual file in the container’s filesystem.

An easy way to visualize the difference among `volumes`, `bind mounts`, and `tmpfs mounts` is to think about where the data lives on the Docker host.

<p align="center">
  <img src="images/types-of-mounts.png">
</p>

* #### Volumes

  `Volumes` are stored in a part of the host filesystem which is managed by Docker (/var/lib/docker/volumes/ on Linux). Non-Docker processes should not modify this part of the filesystem. **Volumes are the best way to persist data in Docker**.

  `Volumes` are created and managed by Docker. You can create a volume explicitly using the `docker volume create` command, or Docker can create a volume during container or service creation.

  When you create a volume, it is **stored within a directory on the Docker host**. When you mount the volume into a container, this directory is what is mounted into the container. This is similar to the way that bind mounts work, except that volumes are managed by Docker and are isolated from the core functionality of the host machine.

  A given volume **can be mounted into multiple containers simultaneously**. When no running container is using a volume, the volume is still available to Docker and is not removed automatically. You can remove unused volumes using `docker volume prune`.

  When you mount a volume, it may be **named** or **anonymous**. Anonymous volumes are not given an explicit name when they are first mounted into a container, so Docker gives them a random name that is guaranteed to be unique within a given Docker host.

* #### Bind mounts

  `Bind mounts` may be stored anywhere on the host system. They may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time.

  `Bind mounts` have limited functionality compared to `volumes`. When you use a `bind mount`, a file or directory on the host machine is mounted into a container. The file or directory is **referenced by its full path** on the host machine.

  The file or directory does not need to exist on the Docker host already. It is created on demand if it does not yet exist. Bind mounts are very performant, but they rely on the host machine’s filesystem having a specific directory structure available.

  If you are developing new Docker applications, consider using named `volumes`.

    **Warning:** One side effect of using bind mounts, for better or for worse, is that you can change the host filesystem via processes running in a container, including creating, modifying, or deleting important system files or directories. This is a powerful ability which can have security implications, including impacting non-Docker processes on the host system.


* #### tmpfs mounts

  `tmpfs mounts` are stored in the host system’s memory only, and are never written to the host system’s filesystem.

  A `tmpfs mount` is **not persisted on disk**, either on the Docker host or within a container. It can be used by a container during the lifetime of the container, to store non-persistent state or sensitive information. For instance, internally, swarm services use tmpfs mounts to mount secrets into a service’s containers.

### Good use cases for Volumes

**Volumes are the preferred way to persist data** in Docker containers and services. Some use cases for volumes include:

* Sharing data among multiple running containers. If you don’t explicitly create it, a volume is created the first time it is mounted into a container. When that container stops or is removed, the **volume still exists**. Multiple containers can mount the same volume simultaneously, either read-write or read-only. Volumes are only removed when you explicitly remove them.

* When the Docker host is not guaranteed to have a given directory or file structure. Volumes help you decouple the configuration of the Docker host from the container runtime.

* When you want to store your container’s data on a remote host or a cloud provider, rather than locally.

* When you need to be able to **back up**, **restore**, or **migrate data** from one Docker host to another, volumes are a better choice. You can stop containers using the volume, then back up the volume’s directory (such as /var/lib/docker/volumes/<volume-name>).

### Good use cases for Bind Mounts

In general, you should use volumes where possible. Bind mounts are appropriate for the following types of use case:

* Sharing configuration files from the host machine to containers. This is how Docker provides DNS resolution to containers by default, by mounting `/etc/resolv.conf` from the host machine into each container.

* Sharing source code or build artifacts between a development environment on the Docker host and a container. For instance, you may mount a `Maven target/` directory into a container, and each time you build the Maven project on the Docker host, the container gets access to the rebuilt artifacts.

If you use Docker for development this way, your production Dockerfile would copy the production-ready artifacts directly into the image, rather than relying on a bind mount.

* When the file or directory structure of the Docker host is guaranteed to be consistent with the bind mounts the containers require.

### Good use cases for tmpfs mounts

`tmpfs mounts` are best used for cases when you do not want the data to persist either on the host machine or within the container. This may be for security reasons or to protect the performance of the container when your application needs to write a large volume of non-persistent state data.

### Practical Example

`Volumes` are the **preferred mechanism for persisting data** generated by and used by Docker containers. While `bind mounts` are dependent on the directory structure of the host machine, volumes are completely managed by Docker. Volumes have **several advantages** over bind mounts:

* Volumes are easier to back up or migrate than bind mounts.
* You can manage volumes using Docker CLI commands or the Docker API.
* Volumes work on both Linux and Windows containers.
* Volumes can be more safely shared among multiple containers.
* Volume drivers allow you to store volumes on remote hosts or cloud providers, to encrypt the contents of volumes, or to add other functionality.
* A new volume’s contents can be pre-populated by a container.

In addition, volumes are often a better choice than persisting data in a container’s writable layer, because using a volume does not increase the size of containers using it, and the volume’s contents exist outside the lifecycle of a given container.

<p align="center">
  <img src="images/types-of-mounts-volume.png">
</p>

If your container generates **non-persistent state data, consider using a tmpfs mount** to avoid storing the data anywhere permanently, and to increase the container’s performance by avoiding writing into the container’s writable layer.

Prior to Docker 17.06 version, the `-v` or `--volume` flag was used for **standalone containers** and the `--mount` flag was used for **swarm services**. Starting with Docker 17.06 we can use `--mount`with standalone containers.

Differences between `-v (--volumen)` and `--mount`:

* `-v` or `--volume`: it combines all the options together in one field.

  * Consists of **three fields**, separated by colon characters (`:`):
    1. The first field is the name of the volume, and is unique on a given host machine.
    2. The second field is the path where the file or directory will be mounted in the container.
    3. The third field is optional, and is comma-separated list of options.

* `--mount`: is more explicit and verbose. `--mount` syntax seperates all the options.

  * Consists of **multiple key-value pairs**, separated by commas and each consisting of a **<key>=<value>** tuple. The `--mount syntax` is more verbose than `-v` or `--volume`, but the order of the keys is not significant, and the value of the flag is easier to understand.

  * The type of the mount, which can be bind, volume, or tmpfs. This topic discusses volumes, so the type will always be volume.
  * The source of the mount. For named volumes, this is the name of the volume. For anonymous volumes, this field is omitted. May be specified as source or src.
  * The destination takes as its value the path where the file or directory will be mounted in the container. May be specified as destination, dst, or target.
  * The readonly option, if present, causes the bind mount to be mounted into the container as read-only.
  * The volume-opt option, which can be specified more than once, takes a key-value pair consisting of the option name and its value.


* **Tip:** New users should  use the `--mount` syntax; it is easier to use.

#### Create and Manage Volumes

Unlike a `bind mount`, you can create and manage volumes outside the scope of any container:

  * **Create a volume:**

    ```shell
    $ docker volume create my-vol
    ```

  * **List volumes:**

    ```shell
    $ docker volume ls

    local               my-vol
    ```

  * **Inspect a volume:**

    ```shell
      $ docker volume inspect my-vol
      [
          {
              "Driver": "local",
              "Labels": {},
              "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
              "Name": "my-vol",
              "Options": {},
              "Scope": "local"
          }
      ]
    ```

  * **Remove a volume:**

    ```shell
    $ docker volume rm my-vol
    ```

  * **Start a container with a volume:**

    If you start a container with a volume that does not yet exist, Docker creates the volume for you. The following example mounts the volume `myvol2` into `/app/` in the container.

    ```shell
    $ docker run -d \
      -it \
      --name devtest \
      --mount source=myvol2,target=/app \
      nginx:latest
    ```
    Use `docker inspect devtest` to verify that the volume was created and mounted correctly. Look for the Mounts section:

    ```shell
    $ docker inspect devtest

      "Mounts": [
          {
              "Type": "volume",
              "Name": "myvol2",
              "Source": "/var/lib/docker/volumes/myvol2/_data",
              "Destination": "/app",
              "Driver": "local",
              "Mode": "",
              "RW": true,
              "Propagation": ""
          }
      ],
    ```

# Container life cycle (Create/Start/Stop/Kill/Remove)

So far we've seen how to run a container in both *foreground* and *background* (detached). Now we will see how to handle the complete life cycle of a container. Docker provides commands like `create` , `start` , `stop` , `kill` , and `rm` . In all of them the "-h" argument could be passed to see the available options.

Example:
```shell
$ docker create -h
```

Above we saw how to run a container in the background (detached). Now we will see in the same example, but with the `create` command. The only difference is that this time we will not specify the "-d" option. Once ready, we will need to launch the container with `docker start`.

Ejemplo:
```shell
$ docker create -P --expose=8001 python:2.7 python -m SimpleHTTPServer 8001
  a842945e2414132011ae704b0c4a4184acc4016d199dfd4e7181c9b89092de13

$ docker ps -a
  CONTAINER ID IMAGE      COMMAND              CREATED       ... NAMES
  a842945e2414 python:2.7 "python -m SimpleHTT 8 seconds ago ... fervent_hodgkin

$ docker start a842945e2414
  a842945e2414

$ docker ps
  CONTAINER ID IMAGE      COMMAND              ... NAMES
  a842945e2414 python:2.7 "python -m SimpleHTT ... fervent_hodgkin
```

### Detener Containers:

Following the example, to **stop** the container, you can execute any of the following `kill` or `stop` commands:
```shell
$ docker kill a842945e2414 (sends SIGKILL)
$ docker stop a842945e2414 (sends SIGTERM).
```

They can also be restarted (do a `docker stop a842945e2414` and then a `docker start a842945e2414`):

```shell
$ docker restart a842945e2414
```

o To remove:

```shell
$ docker rm a842945e2414
```

# Dockerfiles

* **Problem:**

  Running containers in interactive mode (-ti), making some changes, and then committing them to a new image works fine. But in most cases, you may want to automate this process of creating your own image and share these steps with others.

* **Solution:**

  To automate the Docker image creation process, we will create the **Dockerfile**. This text file is composed of::
  * A series of instructions describing what **base image** the new container is based on.
  * The **steps/instructions** that need to be performed to install the application's dependencies.
  * Files that need to be present in the image.
  * The **ports** will be exposed by the container.
  * The **command(s) to run** when the container runs.

### How to start

First we will create an empty directory and enter that directory
```shell
 $ mkdir pruebadockerfile
 $ cd pruebadockerfile/
 ```
Once inside we will create a file called "dockerfile" and we will copy the following code:

```shell
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```
Within the "dockerfile" reference is made to a couple of files that we have not created yet:

**app.py** y **requirements.txt**.

### The Application

We will create both files inside the same directory where the "dockerfile" file is located:

`requirements.txt`
```shell
Flask
Redis
```

`app.py`
```shell
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

As we can see, the "requirements.txt" file specifies the python packages **Flask** and **Redis** that are going to be installed.

### Build the Dockerfile

We are now ready to make the "build" of the application. We will make sure we are in the same directory where the "dockerfile", "app.py" and "requirements.txt" files are:

```shell
$ ls
Dockerfile		app.py			requirements.txt
```
And then we perform the "build". This will create a "Docker image" that we will "tag" with "-t" so that it has a "friendly name":

```shell
$ docker build -t friendlyhello .
```
To see that the image has been created correctly we will do the following:
```shell
$ docker images (o sino docker image ls)

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```
### Run the app

We will start the application by mapping the port **4000** of our host to the port **80** of the container using the "-p" parameter:

```shell
$ docker run -p 4000:80 friendlyhello
```

If all went well we should see a Python Flask Web server loading at: `http://0.0.0.0:80`. Said message is indicated by the Web server that is running inside the container, however since we have mapped the port **4000** with the port **80**, we will open the browser and access through: `http://localhost:4000 `.

If we want the container to work in *background* (detached mode) we will do the following:

```shell
$ docker run -d -p 4000:80 friendlyhello
```
We have used the "-d" option to start it in "detached mode".

# Docker Compose: Linkar containers

When we are designing a "distributed application", each of the pieces is known as a "service". For example, if we think of a "Video Sharing site" application, on the one hand we will have to have a service that allows us to store all the multimedia content in a database, on the other hand we will have a service to perform the "transcoding" in background each time a user uploads a video, we will also have a service for the front-end part, etc.

We call the "containers" that we put into production "services". A service is made up of a single image, with everything necessary for it to provide the functions for which it was created. In Docker, the way we will define these "images" is with "Docker Compose", by writing what are known as **docker-compose.yml** files.

To work with Compose we will follow the following steps:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.

2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.

3. Lastly, run `docker-compose up` and Compose will start and run your entire app.

Compose tiene comandos para poder gestionar el ciclo de vida completo de nuestra aplicación.

* Start, stop, and rebuild services
* View the status of running services
* Stream the log output of running services
* Run a one-off command on a service

Reference:
  * [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/ "Compose file version 3 reference")

### Practical Example: Wordpress + MySQL

When we want to "link" two or more containers we will have to establish their relationship in a YAML file. Below is an example of a file that "links" a "Web" container (Wordpress) and a "MySQL" database container:

* File content **docker-compose.yml**:

  ```YAML
  version: '3'

  services:
     db:
       image: mysql:5.7
       volumes:
         - db_data:/var/lib/mysql
       restart: always
       environment:
         MYSQL_ROOT_PASSWORD: somewordpress
         MYSQL_DATABASE: wordpress
         MYSQL_USER: wordpress
         MYSQL_PASSWORD: wordpress

     wordpress:
       depends_on:
         - db
       image: wordpress:latest
       ports:
         - "8000:80"
       restart: always
       environment:
         WORDPRESS_DB_HOST: db:3306
         WORDPRESS_DB_USER: wordpress
         WORDPRESS_DB_PASSWORD: wordpress
  volumes:
      db_data:

  ```
  And to execute it, being in the same directory where the **docker-compose.yml** file is, we will do the following:

  ```shell
  $ docker-compose up
  ```
  And to check that everything has gone well, we will open the url http://localhost:8000 to access the Wordpress page.

  To stop we have two options:

  * `docker-compose down` deletes the containers, the default network but keeps the Wordpress database.

  * `docker-compose down --volumes` deletes the containers, the default network and the databases.

# Docker Stack

**Reference:**
* [Docker Stacks and why we need them](https://blog.nimbleci.com/2016/09/14/docker-stacks-and-why-we-need-them/ "Docker Stacks and why we need them")

Conceptually, `docker-compose` and `docker stack` files serve the same purpose - **deployment and configuration of your containers on docker engines**.

* **Docker-compose** tool was created first and its purpose is "for defining and running multi-container Docker applications" on a single docker engine.

* **Docker Stack** is used in **Docker Swarm** (Docker's orchestration and scheduling tool) and, therefore, it has additional configuration parameters (i.e. replicas, deploy, roles) that are not needed on a single docker engine. **This command can be invoked from a docker swarm manager only**. Stacks are very similar to docker-compose except they define services while docker-compose defines containers. Stacks allow us to tell the docker engine the definition of the services that should be running, so the engine can monitor and orchestrate the services.

# Networking

### Introduction

  **Referencias:**
  * [Docker Networking Hands-on Lab](http://training.play-with-docker.com/docker-networking-hol/ "Docker Networking Hands-on Lab")



When you install Docker, it creates **three networks** automatically: `bridge`, `none` and `host`. You can list these networks using the `docker network ls` command:

```shell
$ docker network ls

NETWORK ID          NAME                DRIVER
7fca4eb8c647        bridge              bridge
9f904ee27bf5        none                null
cf03ee007fb4        host                host
```

When you run a container, you can use the `--network` flag to specify which networks your container should connect to.

* **Bridge:** The bridge network represents the `docker0` network present in all Docker installations. Unless you specify otherwise with the `docker run --network=<NETWORK>` option, the Docker daemon connects containers to this network by default.

  We can see this bridge as part of a host’s network stack by using the `ip addr show` command:

  ```shell
  $ ip addr show

  docker0   Link encap:Ethernet  HWaddr 02:42:47:bc:3a:eb
            inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
            inet6 addr: fe80::42:47ff:febc:3aeb/64 Scope:Link
            UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
            RX packets:17 errors:0 dropped:0 overruns:0 frame:0
            TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
            collisions:0 txqueuelen:0
            RX bytes:1100 (1.1 KB)  TX bytes:648 (648.0 B)
  ```

* **None:** The `none` network adds a container to a container-specific network stack. That container lacks a network interface. Attaching to such a container and looking at its stack you see this:

  ```shell
  $ docker attach nonenetcontainer

  root@0cb243cd1293:/# cat /etc/hosts
  127.0.0.1	localhost
  ::1	localhost ip6-localhost ip6-loopback
  fe00::0	ip6-localnet
  ff00::0	ip6-mcastprefix
  ff02::1	ip6-allnodes
  ff02::2	ip6-allrouters

  root@0cb243cd1293:/# ip -4 addr
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever

  root@0cb243cd1293:/#
  ```
  * **Note:** You can detach from the container and leave it running with `CTRL-p CTRL-q`.

* **Host:** The `host` network adds a container on the host’s network stack. As far as the network is concerned, **there is no isolation between the host machine and the container**. For instance, if you run a container that runs a web server on port 80 using host networking, the web server is available on port 80 of the host machine.

  The `none` and `host` networks are not directly configurable in Docker. However, you can configure the default `bridge` network, as well as your own user-defined bridge networks.

### Network Containers

Docker allows "container networking" thanks to the use of its **network drivers**. By default, Docker provides two **drivers**: `bridge` and `overlay`.

Every installation of **Docker Engine** automatically includes the following three networks:

```shell
$ docker network ls

NETWORK ID          NAME                DRIVER
18a2866682b8        none                null
c288470c46f6        host                host
7b369448dccb        bridge              bridge
```
* **Bridge:** It is a special network. Unless we specify otherwise, Docker will always start containers on this network. We can try to do the following:
```shell
$ docker run -itd --name=networktest ubuntu

74695c9cea6d9810718fddadc01a727a5dd3ce6a69d09752239736c030599741
```
<p align="center">
  <img src="images/bridge1.png">
</p>

Para comprobar la IP del container, haremos lo siguiente:

```shell
$ docker network inspect bridge

[
    {
        "Name": "bridge",
        "Id": "f7ab26d71dbd6f557852c7156ae0574bbf62c42f539b50c8ebde0f728a253b6f",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.1/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "3386a527aa08b37ea9232cbcace2d2458d49f44bb05a6b775fba7ddd40d8f92c": {
                "Name": "networktest",
                "EndpointID": "647c12443e91faf0fd508b6edfe59c30b642abb60dfab890b4bdccee38750bc1",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "9001"
        },
        "Labels": {}
    }
]
```

To disconnect a container from a network we will have to indicate the network in which it is connected as well as the name of the container:

```shell
$ docker network disconnect bridge networktest
```

**Important:** Networks are natural ways to isolate containers from other containers or other networks.

#### 1. Creat own "Bridge Network"

As we have already mentioned, **Docker Engine** supports two types of networks: *bridge* and *overlay*:

* **Bridge:** It is limited to a "single host" where "Docker Engine" is running
* **Overlay:** It can include multiple hosts with "Docker Engine" installed.

Next we will create a "bridge network":

```shell
$ docker network create -d bridge my_bridge
```

The "-d" flag tells Docker to load the "bridge" network driver. It is optional since Docker by default loads "bridge".

If we list the network drivers again, we will see the one we just created:

```shell
$ docker network ls

NETWORK ID          NAME                DRIVER
7b369448dccb        bridge              bridge
615d565d498c        my_bridge           bridge
18a2866682b8        none                null
c288470c46f6        host                host
```

And if we do an "inspect" of the network we will see that it does not have any information:

```shell
$ docker network inspect my_bridge

[
    {
        "Name": "my_bridge",
        "Id": "5a8afc6364bccb199540e133e63adb76a557906dd9ff82b94183fc48c40857ac",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1"
                }
            ]
        },
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

#### 2. Add Containers to a Network

When we build Web applications that must work together, for security, we will create a network. Networks, by definition, provide complete isolation to containers. When we go to start a container we can add it to a network.

In the following example we will start a PostgreSQL database container by passing it the "--net=my_bridge" flag:

```shell
$ docker run -d --net=my_bridge --name db training/postgres
```

If we now perform an "inspect" of the "my_bridge" network we will see that it has an associated container. We can also inspect the container to see what network it is connected to:

```shell
$ docker inspect --format='{{json .NetworkSettings.Networks}}'  db

{"my_bridge":{"NetworkID":"7d86d31b1478e7cca9ebed7e73aa0fdeec46c5ca29497431d3007d2d9e15ed99",
"EndpointID":"508b170d56b2ac9e4ef86694b0a76a22dd3df1983404f7321da5649645bf7043","Gateway":"10.0.0.1","IPAddress":"10.0.0.254","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:11:00:02"}}
```

If we now start the Web image we will see that the network is as follows:

```shell
$ docker run -d --name web training/webapp python app.py
```

<p align="center">
  <img src="images/bridge2.png">
</p>

# Cleaning
### Containers:
* Borrar container:
```shell
$ docker rm <container_ID>
```

* Delete container and its associated volumes:
```shell
$ docker rm -v <container_ID>
```

* Delete ALL containers:
```shell
$ docker rm $(docker ps -a -q)
```

### Images:
* Delete images:
```shell
$ docker rmi <container_ID>
```

* Delete ALL images:
```shell
$ docker rmi $(docker images -q)
```
* List dangling <none> images:
```shell
$ docker images -f "dangling=true"
```

* Delete dangling <none> images:
```shell
$ docker rmi $(docker images -f "dangling=true" -q)
```

### Volumes:
* Delete all "volumes" that are not being used:
```shell
$ docker volume rm $(docker volumels -q)
```

# CheatSheets
### Docker General Commands

Below is a list of the *basic* commands of Docker:

```shell
docker build -t friendlyname .                   # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyname               # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyname            # Same thing, but in detached mode
docker container ls                              # List all running containers
docker container ls -a                           # List all containers, even those not running
docker container stop <hash>                     # Gracefully stop the specified container
docker container kill <hash>                     # Force shutdown of the specified container
docker container rm <hash>                       # Remove specified container from this machine
docker container rm $(docker container ls -a -q) # Remove all containers
docker image ls -a                               # List all images on this machine
docker image rm <image id>                       # Remove specified image from this machine
docker image rm $(docker image ls -a -q)         # Remove all images from this machine
docker login                                     # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag       # Tag <image> for upload to registry
docker push username/repository:tag              # Upload tagged image to registry
docker run username/repository:tag               # Run image from a registry
```

### Dockerfile Commands

#### ADD

The ADD command gets two arguments: a source and a destination. It basically copies the files from the source on the host into the container's own filesystem at the set destination. If, however, the source is a URL (e.g. http://github.com/user/file/), then the contents of the URL are downloaded and placed at the destination.

Example:
```shell
# Usage: ADD [source directory or URL] [destination directory]
ADD /my_app_folder /my_app_folder
```

#### CMD

The command `CMD`, similarly to `RUN`, can be used for executing a specific command. However, unlike `RUN` it is **not executed during build**, but **when a container is instantiated** using the image being built. Therefore, it should be considered as an initial, default command that gets executed (i.e. run) with the creation of containers based on the image.

To clarify: an example for `CMD` would be running an application upon creation of a container which is already installed using RUN (e.g. RUN apt-get install …) inside the image. This default application execution command that is set with CMD becomes the default and replaces any command which is passed during the creation.

Example:
```shell
# Usage 1: CMD application "argument", "argument", ..
CMD "echo" "Hello docker!"
```
#### ENTRYPOINT

`ENTRYPOINT` argument sets the concrete default application that is used every time a container is created using the image. For example, if you have installed a specific application inside an image and you will use this image to only run that application, you can state it with ENTRYPOINT and whenever a container is created from that image, your application will be the target.

If you couple `ENTRYPOINT` with `CMD`, you can remove "application" from CMD and just leave "arguments" which will be passed to the ENTRYPOINT.

* **Example:**

  ```shell
  # Usage: ENTRYPOINT application "argument", "argument", ..
  # Remember: arguments are optional. They can be provided by CMD
  #           or during the creation of a container.
  ENTRYPOINT echo

  # Usage example with CMD:
  # Arguments set with CMD can be overridden during *run*
  CMD "Hello docker!"
  ENTRYPOINT echo
  ```

#### ENV

The `ENV` command is used to set the environment variables (one or more). These variables consist of “key = value” pairs which can be accessed within the container by scripts and applications alike. This functionality of docker offers an enormous amount of flexibility for running programs.

* **Example:**

  ```shell
  # Usage: ENV key value
  ENV SERVER_WORKS 4
  ```

#### EXPOSE

The `EXPOSE` command is used to associate a specified port to enable networking between the running process inside the container and the outside world (i.e. the host).

* **Example:**

  ```shell
  # Usage: EXPOSE [port]
  EXPOSE 8080
  ```

#### FROM

`FROM` directive is probably **the most crucial command** amongst all others for Dockerfiles. It **defines the base image to use to start the build process**. It can be any image, including the ones you have created previously. If a `FROM` image is not found on the host, docker will try to find it (and download) from the docker image index. It needs to be the first command declared inside a Dockerfile.

* **Example:**

  ```shell
  # Usage: FROM [image name]
  FROM ubuntu
  ```

#### MAINTAINER

One of the commands that can be set anywhere in the file - although it would be better if it was declared on top - is `MAINTAINER`. This non-executing command **declares the author**, hence setting the author field of the images. It should come nonetheless after `FROM`.

* **Example:**

  ```shell
  # Usage: MAINTAINER [name]
  MAINTAINER authors_name
  ```

#### RUN

The `RUN` command is the central executing directive for Dockerfiles. It takes a command as its argument and runs it to form the image. Unlike `CMD`, it actually is used to build the image (forming another layer on top of the previous one which is committed).

* **Example:**

  ```shell
  # Usage: RUN [command]
  RUN aptitude install -y riak
  ```

### USER

The `USER` directive is used to set the UID (or username) which is to run the container based on the image being built.

* **Example:**

  ```shell
  # Usage: USER [UID]
  USER 751
  ```

#### VOLUME

The `VOLUME` command is used to enable access from your container to a directory on the host machine (i.e. mounting it).

* **Example:**

  ```shell
  # Usage: VOLUME ["/dir_1", "/dir_2" ..]
  VOLUME ["/my_files"]
  ```

#### WORKDIR

The `WORKDIR` directive is used to set where the command defined with CMD is to be executed.

* **Example:**

  ```shell
  # Usage: WORKDIR /path
  WORKDIR ~/
  ```

#### EJEMPLO: Create an Image to Install MongoDB

I will create a Dockerfile document and populate it step-by-step with the end result of having a Dockerfile, which can be used to create a docker image to run MongoDB containers.

#### 1. Create the empty Dockerfile

Using the nano text editor, let's start editing our Dockerfile:
```shell
$ sudo nano Dockerfile
```

#### 2. Defining our file and its purpose

Although optional, it is always a good practice to let yourself and everybody figure out (when necessary) what this file is and what it is intended to do.

For this, we will begin our Dockerfile with fancy comments (i#) to describe it.
```shell
############################################################
# Dockerfile to build MongoDB container images
# Based on Ubuntu
############################################################
```

#### 3. Setting the base image to use

```shell
# Set the base image to Ubuntu
FROM ubuntu
```


#### 4. Defining the Maintainer (Author)

```shell
# File Author / Maintainer
MAINTAINER Example McAuthor
```

#### 5. Updating the application repository list

**Note:** This step is not necessary, given that we are not using the repository right afterwards. However, it can be considered good practice.

```shell
# Update the repository sources list
RUN apt-get update
```

#### 6. Setting arguments and commands for downloading MongoDB

```shell
################## BEGIN INSTALLATION ######################
# Install MongoDB Following the Instructions at MongoDB Docs
# Ref: http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/

# Add the package verification key
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

# Add MongoDB to the repository sources list
RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/mongodb.list

# Update the repository sources list once more
RUN apt-get update

# Install MongoDB package (.deb)
RUN apt-get install -y mongodb-10gen

# Create the default data directory
RUN mkdir -p /data/db

##################### INSTALLATION END #####################
```

#### 7. Setting the default port MongoDB
```shell
# Expose the default port
EXPOSE 27017

# Default port to execute the entrypoint (MongoDB)
CMD ["--port 27017"]

# Set default container command
ENTRYPOINT usr/bin/mongod
```

#### 8. Saving the Dockerfiles

After you have appended everything to the file, it is time to save and exit. Press `CTRL+X` and then "Y" to confirm and save the Dockerfile.

```shell
############################################################
# Dockerfile to build MongoDB container images
# Based on Ubuntu
############################################################

# Set the base image to Ubuntu
FROM ubuntu

# File Author / Maintainer
MAINTAINER Example McAuthor

# Update the repository sources list
RUN apt-get update

################## BEGIN INSTALLATION ######################
# Install MongoDB Following the Instructions at MongoDB Docs
# Ref: http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/

# Add the package verification key
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

# Add MongoDB to the repository sources list
RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | tee /etc/apt/sources.list.d/mongodb.list

# Update the repository sources list once more
RUN apt-get update

# Install MongoDB package (.deb)
RUN apt-get install -y mongodb-10gen

# Create the default data directory
RUN mkdir -p /data/db

##################### INSTALLATION END #####################

# Expose the default port
EXPOSE 27017

# Default port to execute the entrypoint (MongoDB)
CMD ["--port 27017"]

# Set default container command
ENTRYPOINT usr/bin/mongod
```

#### 9. Building our first image

Using the explanations from before, we are ready to create our first MongoDB image with docker!

```shell
$ sudo docker build -t my_mongodb .
```

* **Note:** The `-t [name]` flag here is used to **tag the image**. To learn more about what else you can do during build, run sudo `docker build --help`.


#### 10. Running a MongoDB instance

Using the image we have build, we can now proceed to the final step: creating a container running a MongoDB instance inside, using a name of our choice (if desired with `-name [name]`).

```shell
sudo docker run -name my_first_mdb_instance -i -t my_mongodb
```

* **Note:** If a name is not set, we will need to deal with complex, alphanumeric IDs which can be obtained by listing all the containers using sudo `docker ps -l`.

* **Note:** To detach yourself from the container, use the **escape sequence** `CTRL+P` followed by `CTRL+Q`.
