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


Running things in an isolated environment be it for testing, demo or simply build is such a neat thing. You dont have to worry about other processes
running on the system or ports being consumed by some one else or inconsistent libraries/packages on the system. So far the only option we had was
to create this heavy-weight Virtual machine that packages an entire Guest operating system with it. Due to its heavy weight nature, it was really
cumbersome to distribute this 10GB+ file. There was no good way to automate the complete configuration. Also the Host you would run this on needs to 
be powerful enough(RAM ~ 8GB+) to be able to run this Virtual machine which often was a challenge given the kind of laptops most field folks carry.
Running several such instance of VMs was impossible given the limited compute capacity of the host os. Come Docker....it addresses most of these problems
quite beautifully but also the developers can no more fix bugs by simply saying "It works on my machine" !! Read on to find out how the magic happens.

### What is Docker?

Docker is a container virtualization technology. It is a form of light weight container that allows you to run within a host operating system.
Unlike Virtual machines, you dont package a complete Guest operating system with it. There is a thin layer however that allows you to run your application
in an isolated manner over your choice of linux operating system that runs on top of the Host operating system. You can easily run several such 
containers on the Host Operating system. Since Docker uses underlying features(covered later) of an Linux operating system, 
the container needs to have Linux as their host operating system. 

Docker provides a platform that enables users to build, package, ship and run distributed applications. 
Docker users package up their applications, and any dependent libraries or files, into a Docker image. 
Docker images are portable artifacts that can be distributed across Linux environments. 
Images that have been distributed can be used to instantiate containers where applications can run in isolation from 
other applications running in other containers on the same host operating system. 

<center><img src="/assets/img/docker.png"></center>


### How it is better than Virtual Machine?
- Virtual machine is a heavy weight resource that takes up significant memory and cpu resources from the Host OS. It comes with a copy of the entire OS.
It runs on top of Hypervisor that takes 10-15% of resources on Host machine and even take several seconds to boot up.

#### Advantages as compared to Virtual Machine
More agile and light weight compute resource. It launches in sub-seconds. The entire application get shipped in a container which is isolated from 
the host OS just like the VM
The same container can be used by several teams such as QA and DevOps.
Also since these are light-weight, you can easily spawn several such containers on the Host and simulate a clustered environment if you need.
In a typical LAMP application, you can create one or several containers for the apache WebServer and say one for the MySQL server on same or different host and link them all together.


#### Layering System

### Good Usecases
- Continuous integration and continuous deployment
- Replicate production environment on their own laptops
- Run Acceptance tests on the production
- Since you can move these containers around, you can easily given this to other teams and repeat this build QA cycle
Since container are built in seconds, you can save so much time that now you can spend your time learning something else :)


#### Registry

#### Basic Operations


#### Useful Commands
Docker commands needs to be run as root(sudo) or you can add the user to the `docker` group.

# Search for images on dockerhub.
# Usage: docker search [image name]
$ docker search ubuntu

# Look at all the images downloaded locally.
# Usage: docker images
$ docker images

# Builds an image. 
# Usage: docker build -t <tag> -f <file>
$ docker build -t mysql56 .

# Run the docker container
$ docker run -i -t cfi_ubuntu_v1 /bin/bash

# List running and non running containers
$ sudo docker ps -l 

# Create a new container
# Usage: sudo docker run [image name] [command to run]
$ sudo docker run my_img echo "hello"

# To name a container instead of having long IDs
# Usage: sudo docker run -name [name] [image name] [command]
$ sudo docker run -name my_cont_1 my_img echo "hello"

# Run existing container
# Usage: sudo docker run [container ID]
$ sudo docker run c629b7d70666

# Delete Container
# Usage: sudo docker rm [container ID]
$ sudo docker rm c629b7d70666

# Delete Image
# Usage: sudo docker rm [image ID]
$ sudo docker rmi c629b7d70666

# Usage: sudo docker stop [container ID]
$ sudo docker stop c629b7d70666

# Saving (committing) a container: This command turns your container to an image.
# Usage: sudo docker commit [container ID] [image name]:[tag/version - default latest]
$ sudo docker commit 8dbd9e392a96 my_new_img:v2


#### Orchestration



#### Resources

#### Underlying Technology
Docker utilizes several linux features under the hood. These are 
**Namespaces**
This provides us isolation from other processes running on the host which is what Docker calls as the container.
Several namespaces are created for the container so that it can provide this isolated workspace.

** Control Groups**
This allows us to create containers with specific CPU and memory resources. This ensures that the containers behave as good citizens on the host.

**Union file systems**
This filesystem allows docker to create layers thereby making them fast and lightweight.


#### Working with Windows and Mac



#### New Tools with Docker

Fig


Compose
	What is does?
	Give example


Swarm 
	What is does? 
	Give example
	
What problem does it solve?
- No more need to create customized shell script to manage cluster of Docker nodes
- 

In a nutshell
- Orchestration tool to handle several Docker nodes
- Exposes several Docker Engines as a single virtual Engine
- You can still use the standard Docker API
- Very easy to 

Swarm 0.2.0
 - Supports 85% of Docker API as REST
 - Management of resources such as CPU, Memory and Networking
 - Multiple discovery backends
 - TLS Support

#### Example

Build a MySQL + Django Image



Deleted
- Takes tens of seconds to boot up 
Running more than one Virtual machine is close to impossible unless you run this on a really powerful host machine.


