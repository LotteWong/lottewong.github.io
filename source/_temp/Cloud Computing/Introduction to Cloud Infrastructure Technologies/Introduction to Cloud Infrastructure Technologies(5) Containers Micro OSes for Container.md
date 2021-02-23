---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(5) Micro OSes for Containers
date: 2019-07-15 15:13:21
categories: Cloud Computing
tags:
- Micro OSes
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Discuss the characteristics and functionality of Micro OSes, which are specially designed to run containers.

- Describe different Micro OSes designed to run containers: Project Atomic and Red Hat CoreOS, VMware Photon, and RancherOS.

- Deploy containers and containerized applications on Micro OSes.

  

<!-- more -->



## Concept

*“Once we remove the packages which are not required to boot the base OS and run container-related services, we are left with specialized OSes, which are referred to as Micro OSes for containers”.*

![Micro_OSes_for_Containers_updated](/images/Micro_OSes_for_Containers_updated.png)

## Atomic Host

### Concept

*"[Atomic Host](http://www.projectatomic.io/) is a lightweight operating system, assembled out of a specific RPM content. It allows us to run just containerized applications in a quick and reliable manner".*

### Features

- Atomic Host can be based on Fedora, CentOS, or Red Hat Enterprise Linux (RHEL).
- Atomic Host is a sub-project of **Project Atomic**, which includes other sub-projects, such as [Buildah](https://www.projectatomic.io/blog/2017/06/introducing-buildah/), [Cockpit](https://www.projectatomic.io/docs/cockpit/) and [skopeo](https://www.projectatomic.io/blog/2016/07/working-with-containers-image-made-easy/).
- Atomic Host comes out-of-the-box with Kubernetes installed. It also includes several Kubernetes utilities, such as **etcd** and **flannel**.

### Components

Atomic Host has a very minimal base OS, but it includes components like **systemd**and **journald** to help its users and administrators. It is built on top of the following:

- **rpm-ostree** 
One cannot manage individual packages on Atomic Host, as there is no **rpm** or other related commands. To get any required service, you would have to start a respective container. Atomic Host has two bootable, immutable, and versioned filesystems; one is used to boot the system and the other is used to fetch updates from upstream. **rpm-ostree** is the tool to manage these two versioned filesystems.
- **systemd** 
It is used to manage system services for Atomic Host.
- **Docker** 
Atomic Host currently supports Docker as a container runtime.
- **Kubernetes** 
With Kubernetes, we can create a cluster of Atomic Hosts to run applications at scale.

Atomic Host also comes with a command called **atomic.** This command provides a high-level, coherent entry point to the system, and fills in the gaps that are not filled by Linux container implementations, such as upgrading the system to the new **rpm-ostree**, running containers with pre-defined **docker run** options using [labels](https://github.com/docker/docker/pull/9882/), verifying an image, etc.

## Red Hat CoreOS

### Concept

*"According to the "Bringing CoreOS Technology to Red Hat OpenShift to Deliver a Next-Generation Automated Kubernetes Platform" article posted on the CoreOS blog, Red Hat CoreOS will be based on Fedora and Red Hat Enterprise Linux sources, and is expected to ultimately supersede Atomic Host as Red Hat’s immutable, container-centric operating system".*

### Others

- Atomic Host can be managed using tools such as [Cockpit](http://www.projectatomic.io/docs/cockpit/). Cockpit is a server manager that makes it easy to administer your GNU/Linux servers via a web browser.

## VMware Photon

### Concept

*"[Photon OS™](https://vmware.github.io/photon/) is a technology preview of a minimal Linux container host provided by VMware. It is designed to have a small footprint and boot extremely quickly on VMware platforms".*

### Features

- It supports Docker, rkt, and the Pivotal Garden container specifications.
- It also has a new, open source, yum-compatible package manager (**tdnf**).
- Photon OS™ is a security-hardened Linux. The kernel and other aspects of the Photon OS™ are built with an emphasis on security recommendations given by the [Kernel Self-Protection Project](https://kernsec.org/wiki/index.php/Kernel_Self_Protection_Project) (KSPP).
- It can be easily managed, patched, and updated. It also provides support for persistent volumes to store the data of cloud-native applications on VMware vSAN™ .

## RancherOS

### Concept

*"[RancherOS](http://rancher.com/rancher-os/) is a 20 MB Linux distribution that runs Docker containers. It has the least footprint of all the Micro OSes available nowadays. This is possible because it runs directly on top of the Linux kernel. RancherOS is a product provided by [Rancher](http://rancher.com/), which is an end-to-end platform used to deploy and run private container services".*

### Features

- It automates OS configuration with **cloud-init**.
- It can be customized to add custom system Docker containers using the **cloud-init** file or Docker Compose.

### Components

RancherOS runs two instances of the Docker daemon. Just after booting, it starts the first instance of the Docker daemon with **PID 1** to run system containers like **dhcp**, **udev***,* etc. To run user-level containers, the System Docker daemon creates a service to start other Docker daemons.

![RancherOS_Architecture](/images/RancherOS_Architecture.png)

RancherOS also includes a CLI utility called **ros**, which can be used to control and configure the system.