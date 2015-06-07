---
layout: post
title: Docker Introduction and Tools
category: tech
tags: docker
year: 2015
month: 5
day: 20
published: true
summary: Introduction to Docker and its tools
---


Running things in an isolated environment be it for testing, demo or simply build is such a neat thing. You don't have to worry about other processes
running on the system or ports being consumed by someone else or inconsistent libraries/packages installed. So far the only option we had was
to create this heavy-weight Virtual machine that packages an entire Guest operating system with it. Due to its heavy weight nature, it was really
cumbersome to distribute this 10GB+ file. There was no good way to automate the complete configuration. Also the Host you would run this on needs to 
be powerful enough(RAM ~ 8GB+) to be able to run this Virtual machine which often was a challenge given the kind of laptops most field folks carry.
Running several such instance of VMs was impossible given the limited compute capacity of the host OS. Come `Docker` :whale:

Docker addresses most of these problems quite beautifully and also the developers can no more close bugs by simply saying "It works on my machine" !! 

<center><img src="/assets/img/docker.png"></center>

### What is Docker?

Docker is a container virtualization technology. It is a form of light weight container that allows you to run within a host operating system.
Unlike Virtual machines, you don't package a complete Guest operating system with it. There is a thin layer however that allows you to run your application
in an isolated manner over your choice of linux operating system that runs on top of the Host operating system. You can easily run several such 
containers on the Host Operating system. Since Docker uses underlying features(covered later) of an `Linux` operating system, 
the container needs to have **Linux as their host operating system**. 

In a nutshell:

 - Docker provides a platform that enables users to build, package, ship and run distributed applications. 
 - Docker users package up their applications, and any dependent libraries or files, into a Docker image. 
 - Docker images are portable artifacts that can be distributed across Linux environments. 
 - Images that have been distributed can be used to instantiate containers where applications can run in isolation from other applications running in other containers on the same host operating system. 


##### How it is better than Virtual Machine?
- Virtual machine is a heavy weight resource that takes up significant memory and cpu resources from the Host OS.
- It comes with a copy of the entire OS.
- It runs on top of Hypervisor that takes 10-15% of resources on Host machine and even take several seconds to boot up.

##### Advantages as compared to Virtual Machine
 - More agile and light weight compute resource. 
 - It launches in sub-seconds. 
 - The entire application get shipped in a container which is isolated from the host OS just like the VM.
 - The same container can be used by several teams such as QA and DevOps.
 - Since these are light-weight, you can easily spawn several such containers on the Host and simulate a clustered environment if you need.
 
In a typical `LAMP` application, you can create one or several containers for the Apache WebServer and say one for the MySQL server on same or different host and link them all together.


#### Usecases for Docker
- Continuous integration and continuous deployment
- Replicate production environment on developer's own laptops
- Run Acceptance tests on the production image.
- Since you can move these containers around, you can easily given this to other teams and repeat this build QA cycle

Since container are built in seconds, you can save so much time that now you can spend your time learning something else :golf: :video_game: :surfer:

For installation, refer [link](https://docs.docker.com/installation/)

#### Registry
You can find most of the Docker images on a public repository hosted at [Docker Hub](https://hub.docker.com/).
This allows you to quickly download images that are precreated or contributed by the community. e.g. ubuntu:14.04 image

The Docker client allows to search for published images from Docker Hub and download them locally that you can build containers for.

A private registry can also be created and hosted locally in order to serve images within your company.

#### Basic Operations
<center><img src="/assets/img/basic.png"></center>

Typically you would perform the following operations:

- Search the Docker Hub or your registry for an existing image. e.g `ubuntu:14.04`
- Pull the specific image from the registry if not available locally
- Create Dockerfile that does additional configuration on this image like downloading more packages and some configuration.
- Create a running environment (container) from this image using the Dockerfile.
- Stop the container (optional)
- Save the container as an image 
- Push the modified image back to the registry

#### Useful Commands
Docker commands needs to be run as root(sudo) or you can add the user to the `docker` group.

:point_right: Search for images on dockerhub.

    # Usage: docker search [image name]
    $ docker search ubuntu

:point_right: Look at all the images downloaded locally.

    # Usage: docker images
    $ docker images

:point_right: Builds an image. 

    # Usage: docker build -t <tag> -f <file>
    $ docker build -t mysql56 -f Dockerfile_mysql .

:point_right: Create/Run the docker container

    # Usage: docker run <options> [image name] [command to run]
    $ docker run -it cfi_ubuntu:14.04 /bin/bash
    $ sudo docker run my_img echo "hello"

:point_right: List running and non running containers

    $ docker ps -l 

:point_right: Run a named container instead of having long IDs

    # Usage: sudo docker run -name [name] [image name] [command]
    $ sudo docker run -name my_cont_1 my_img echo "hello"

:point_right: Run existing container

    # Usage: sudo docker run [container ID]
    $ sudo docker run c629b7d70666

:point_right: Stop running container

    # Usage: sudo docker stop [container ID]
    $ sudo docker stop c629b7d70666

:point_right: Delete Container

    # Usage: sudo docker rm [container ID]
    $ sudo docker rm c629b7d70666

:point_right: Delete Image

    # Usage: sudo docker rm [image ID]
    $ sudo docker rmi c629b7d70666

:point_right: Saving (committing) a container: This command turns your container to an image.

    # Usage: sudo docker commit [container ID] [image name]:[tag/version - default latest]
    $ sudo docker commit 8dbd9e392a96 my_new_img:v2


#### Underlying Technology
Docker utilizes several linux features under the hood. These are:

##### Namespaces
This provides us isolation from other processes running on the host which is what Docker calls as the container.
Several namespaces are created for the container so that it can provide this isolated workspace.

##### Control Groups
This allows us to create containers with specific CPU and memory resources. This ensures that the containers behave as good citizens on the host.

##### Union file systems
This filesystem allows docker to create layers thereby making them fast and lightweight.


#### Working with Windows and Mac
The Docker daemon uses linux-specific kernel feature so you need to use Docker from a Linux machine. 
If you are using Windows or Mac, you can use `Boot2Docker`. Boot2Docker is composed of

 - VirtualBox Virtual Machine (VM)
 - Docker and 
 - Boot2Docker management tool (Lightweight Linux VM)


#### New Tools with Docker

##### Compose 
There used to be a tool called [fig](http://www.fig.sh/) which allowed you to manage Docker containers using a single file.
Its now replaced with inbuilt tool called `Compose`.

Compose allows you to specify your entire application in a single `yml` file which could be composed of multiple containers.
You can expose the correct port and create links between the containers in this file.
Now you can start/stop your application using `docker-compose` commands instead of worrying about individual containers.

Example of `docker-compose.yml`

    db:
      image: mysql
      expose:
        - "3306"
      volumes_from:
        - DBDATA

    web:
      build: .
      working_dir: /app
      command: python manage.py runserver 0.0.0.0:8000
      volumes:
        - .:/app
      ports:
        - "8000:8000"
      links:
        - db
  
Commands:
> Start your application
    # This will build the containers and run them
    $ docker-compose up 

> Stop application

    $ docker-compose down

> Check status of your application (containers)
    $ docker-compose ps

##### Swarm 
Currently, if you run your containers across multiple hosts, most create their own customized shell script to manage cluster of Docker nodes.
Instead now you could use `Swarm`. It is a native clustering tool for Docker. It allows your to manage your Docker nodes as a single virtual host.

It supports the following in a nutshell:

- Orchestration tool to handle several Docker nodes
- Exposes several Docker Engines as a single virtual Engine
- You can still use the standard Docker API

Swarm 0.2.0

 - Supports 85% of Docker API as REST
 - Management of resources such as CPU, Memory and Networking
 - Multiple discovery backends
 - TLS Support

#### Sample Docker file to build a MySQL + Django Image

    FROM ubuntu:14.04

    MAINTAINER John Doe

    RUN apt-get update

    RUN apt-get install -y build-essential git wget supervisor

    # Pre-requisites for a python/pip environment
    RUN apt-get install -y python python-dev python-setuptools python-pip libmysqlclient-dev
    RUN easy_install pip

    # Copy our runtime install components file
    COPY requirements.txt /tmp

    # Install all runtime packages
    RUN cd /tmp; pip install -r requirements.txt

:whale2:

Till then... :metal:
