---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(7) Unikernels
date: 2019-07-17 15:13:21
categories: Cloud Computing
tags:
- Unikernels
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Explain the concept of unikernels.
- Compare and contrast unikernels and containers.



<!-- more -->



## Concept

*“Unikernels are specialised, single-address-space machine images constructed by using library operating systems. With unikernels, we can also select the part of the kernel needed to run with the specific application. With unikernels, we can create a single address space executable, which has both application and kernel components. The image can be deployed on VMs or bare metal, based on the unikernel's type.”*

## Categories

- **Specialized and purpose-built** **unikernels** 
  They utilize all the modern features of software and hardware, without worrying about the backward compatibility. They are not POSIX-compliant. Some examples of specialized and purpose-built unikernels are ING, HalVM, MirageOS, and Clive.
- **Generalized 'fat' unikernels** They run unmodified applications, which make them fat. Some examples of generalized 'fat' unikernels are BSD Rump kernels, OSv, Drawbridge, etc.

## Components

The [Unikernel](http://unikernel.org/) goes one step further than other technologies, creating specialized virtual machine images with just:

- The application code
- The configuration files of the application
- The user-space libraries needed by the application
- The application runtime (like JVM)
- The system libraries of the unikernel, which allow back and forth communication with the hypervisor.

Unikernel images would run directly on top of a hypervisor like Xen or on bare metal, based on the unikernel types.

![Example_of_a_Unikernel_Architecture__as_Compared_to_a_Traditional_OS_Stack](/images/Example_of_a_Unikernel_Architecture__as_Compared_to_a_Traditional_OS_Stack.png)

## Unikernels and Constainers

- Both containers and unikernels can co-exist on the same host. They can be managed by the same Docker binary. 

- Unikernels helped Docker to run the Docker Engine on top of [Alpine Linux](http://www.alpinelinux.org/) on Mac and Windows with their default hypervisors, which are xhyve Virtual Machine and Hyper-V VM respectively.

![SharedKernel-vs-Unikernel](/images/SharedKernel-vs-Unikernel.png)