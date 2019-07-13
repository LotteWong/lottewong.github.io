---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(4) Containers
date: 2019-07-13 11:18:16
categories: Cloud Computing
tags:
- Docker
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Discuss about containers and their runtimes. 

- Describe the basic Docker operations.

- Understand how Project Moby helps create container platforms like Docker. 



<!-- more -->



## Concept

*"[Operating-System-level virtualization](https://en.wikipedia.org/wiki/Operating-system-level_virtualization) allows us to run multiple isolated user-space instances in parallel. These user-space instances have the application code, the required libraries, and the required runtime to run the application without any external dependencies. These user-space instances are referred to as **containers**".*

![Running_an_Application](/images/Running_an_Application.png)

![Docker_Container](/images/Docker_Container.jpg)

## Terms

### Images

- In the container world, this box (containing our application and all its dependencies) is referred to as an **image**.
- An image contains the application, its dependencies and the user-space libraries. User-space libraries like **glibc** enable switching from the user-space to the kernel-space. An image does not contain any kernel-space components.

### Container

- A running instance of this box is referred to as a **container**. We can spin multiple containers from the same image.
- When a container is created from an image, it runs as a process on the host's kernel. It is the host kernel's job to isolate and provide resources to each container.

## Building Blocks

### Namespaces

A namespace wraps a particular global system resource like network, process IDs in an abstraction, that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource. The following global resources are namespaced:

- pid - provides each namespace to have the same PIDs. Each container has its own PID 1.
- net - allows each namespace to have its network stack. Each container has its own IP address.
- mnt - allows each namespace to have its own view of the filesystem hierarchy.
- ipc - allows each namespace to have its own interprocess communication.
- uts - allows each namespace to have its own hostname and domainname.
- user - allows each namespace to have its own user and group ID number spaces. A *root*user inside a container is not the *root* user of the host on which the container is running.

### cgroups

*Control groups* are used to organize processes hierarchically and distribute system resources along the hierarchy in a controlled and configurable manner. The following cgroups are available for Linux:

- blkio
- cpu
- cpuacct
- cpuset
- devices
- freezer
- memory.

### Union filesystem

The Union filesystem allows files and directories of separate filesystems, known as layers, to be transparently overlaid on each other, to create a new virtual filesystem. An image used in Docker is made of multiple layers and, while starting a new container, we merge all those layers to create a read-only filesystem. On top of a read-only filesystem, a container gets a read-write layer, which is an ephemeral layer and it is local to the container.

## Runtimes

### runC

Over the past few years, we have seen a rapid growth in the interest and adoption for container technologies. Most of the cloud providers and IT vendors offer support for containers. To make sure there is no vendor locking and no inclination towards a particular company or project, IT companies came together and formed an open governance structure, called [The Open Container Initiative](https://www.opencontainers.org/), under the auspices of The Linux Foundation. The governance body came up with [specifications](https://github.com/opencontainers/specs) to create standards on Operating System process and application containers. [runC](https://github.com/opencontainers/runc/) is the CLI tool for spawning and running containers according to these specifications.

### containerd

containerd is an Open Container Initiative (OCI)-compliant container runtime with an emphasis on simplicity, robustness and portability. It runs as a daemon and manages the entire lifecycle of containers. It is available on Linux and Windows.

Docker, which is a containerization platform, uses containerd as a container runtime to manage runC containers.

### rkt

rkt (pronounced "rock-it") is an open source, Apache 2.0-licensed project from CoreOS. It implements the [App Container specification](https://github.com/coreos/rkt/blob/master/Documentation/app-container.md).

### CRI-O

CRI-O is an OCI-compatible runtime, which is an implementation of the Kubernetes Container Runtime Interface (CRI). It is a lightweight alternative to using Docker as the runtime for Kubernetes.

## Project Moby

### Problem

From the user perspective, we usually get a seamless experience, irrespective of the underlying platform. However, behind the scenes, someone connects different components like containers runtime, networking, storage, etc. to ensure this high quality experience. Open source projects like **containerd** and **libnetwork** are part of the container platform and have their own release cycle, governing model, etc. So, how can we take those individual components and build a container platform like Docker?

### Solution

[Project Moby](https://mobyproject.org/) is the answer. It is an open source project which provides a framework for assembling different container systems to build a container platform like Docker. Individual container systems provide features like image, container, secret management, etc.

### Application

Moby is particularly useful if you want to build your container-based system or just want to experiment with the latest container technologies. It is not recommended for application developers and newbies who are looking for an easy way to run containers.

LinuxKit, which is a tool to build minimal Linux distributions to run containers, uses Moby. You can find a few examples at its [GitHub repository](https://github.com/linuxkit/linuxkit).

## Docker

### Concept

*"Docker, Inc. is a company which provides Docker Containerization Platform to run applications using containers. Docker has a client-server architecture, in which a Docker client connects to a server (Docker Host) and executes the commands".*

### Commands

You can find a list of basic Docker operations below:

- Check version:

  ```powershell
  $ docker version
  ```

- About daemon:

  ```powershell
  $ docker info
  ```

- List images:

  ```powershell
  $ docker image ls
  ```

- Pulling an **alpine** image:

  ```powershell
  $ docker image pull alpine: latest
  ```

- Delete an image:

  ```powershell
  $ docker container rm  <image id/name>
  ```

- Run a container from a locally-available image:

  ```powershell
  $ docker container run -it alpine sh
  ```

- Run a container in the background (**-d** option) from an image:

  ```powershell
  $ docker container run -d --name web nginx
  ```

- List only running containers:

  ```powershell
  $ docker container ls 
  ```

- List all containers: 

  ```powershell
  $ docker container ls -a
  ```

- Inject a process inside a running container:

  ```powershell
  $ docker container exec -it <container_id/name> bash
  ```

- Stop a container:

  ```powershell
  $ docker container stop <container id/name> 
  ```

- Delete a container:

  ```powershell
  $ docker container rm  <container id/name>
  ```

## Containers vs VMs

- A virtual machine runs on top of a [hypervisor](https://en.wikipedia.org/wiki/Hypervisor), which emulates different hardware, like CPU, memory, etc., so that a guest OS can be installed on top of them. Different kinds of guest OSes can run on top of one hypervisor. Between an application running inside a guest OS and in the outside world, there are multiple layers: the guest OS, the hypervisor, and the host OS. 
- On the other hand, containers run directly as a process on top of the Host OS. There is no indirection as we see in VMs, which help containers to get near-native performance. Also, as the containers have very little footprint, we can pack a higher number of containers than VMs on the same physical machine. As containers run on the host OS, we need to make sure containers are compatible with the host OS.

![Docker_Containers_vs._VMs](/images/Docker_Containers_vs._VMs.png)