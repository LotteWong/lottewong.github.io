---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(2) Infrastructure as a Service (IaaS)
categories: Cloud Computing
tags:
- IaaS
- AWS
- Azure
- Google Cloud
- DigitalOcean
- OpenStack
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Explain the concept of Infrastructure as a Service (IaaS).
- Distinguish between different IaaS providers.
- Provision a virtual machine on top of different IaaS providers.



<!-- more -->



## Infrastructure as a Service (IaaS) 

### Concept

*“Infrastructure as a Service (IaaS) is a form of cloud computing which provides on-demand physical and virtual computing resources, storage, network, firewall, load balancers, etc. To provide virtual computing resources, IaaS uses some form of hypervisor, like Xen, KVM, VMware ESX/ESXi, Hyper-V, etc.ther than VMs, some IaaS providers offer bare metal machines for provisioning”.*

### Amazon EC2

#### Concept

*"[Amazo](https://aws.amazon.com/)[n](https://aws.amazon.com/)[ ](https://aws.amazon.com/)[Web](https://aws.amazon.com/)[ Ser](https://aws.amazon.com/)[vices](https://aws.amazon.com/) (AWS) is one of the leaders in providing different cloud services. With [Am](https://aws.amazon.com/ec2/)[azon](https://aws.amazon.com/ec2/)[ Elastic](https://aws.amazon.com/ec2/)[ ](https://aws.amazon.com/ec2/)[Compute](https://aws.amazon.com/ec2/), Amazon provides the IaaS infrastructure, on which most of the other services are built. We can manage compute resources from the Amazon EC2 web interface and can scale up or down, depending on the need. AWS also [offers a command line](https://aws.amazon.com/cli/) to manage the instances from the command line. Amazon EC2 uses XEN and KVM hypervisors to provision compute resources".*

#### Feature

- Amazon EC2 offers compute instances for different resources, which we can choose from depending on our need.
- Amazon EC2 provides some preconfigured images, called Amazon Machine Images (AMIs). These images can be used to quickly start instances. We can also create our own custom AMIs to boot our instances.
- One important aspect to note is that Amazon supports configuring security and network access to our instances.
- With Amazon Elastic Block Store (EBS) we can attach/detach persistent storage to our instances.
- EC2 supports the provisioning of dedicated hosts, which means we can get an entire physical machine for our use.
- Create an Elastic IP for remapping the Static IP address automatically.
- Provision a Virtual Private Cloud for isolation. Amazon Virtual Private Cloud provides secure and robust networking for Amazon EC2 instances.
- Use CloudWatch for monitoring resources and applications.
- Use Auto Scaling to dynamically resize your resources, etc.

### Azure Virtual Machine

#### Concept

*'"[Azure](https://azure.microsoft.com/) is Microsoft's cloud offering, which has products in different domains, such as compute, web and mobile, data and storage, Internet of Things, and many others. Through Azure Virtual Machine, Microsoft provides compute provisioning and management: We can manage Virtual Machines from Azure's web interface. Azure also provides a [command line utility](https://github.com/azure/azure-xplat-cli) to manage resources and applications on the Azure cloud".*

#### Feature

- Azure lets you choose between different tiers, based on the usage and the operating systems or the predefined application virtual machines (SharePoint, Oracle, etc.).
- Using Resource Manager templates, we can define the template for the virtual machine deployment.
- Azure offers other features as well, like making seamless hybrid connections, faster I/O in certain types of tiers, backups, etc.

### Google Compute Engine

#### Concept

*”[Google Cloud Platform](https://cloud.google.com/) is Google's Cloud offering, which has many products in different domains, like compute, storage, networking, big data, and others. [Google Compute Engine](https://cloud.google.com/compute/) provides the compute service. We can manage the instances through GUI, APIs or [command line](https://cloud.google.com/sdk/gcloud/). Access to the individual VM's console is also available“.*

#### Feature

- GCE supports different [machine types](https://cloud.google.com/compute/docs/machine-types), which we can choose from depending on our need.
- GCE has other features as well, like Persistent Disk, Local SSD, Global Load Balancing, Compliance and Security, Automatic Discount, etc.

### DigitalOcean

#### Concept

*"[DigitalOcean](https://www.digitalocean.com/) helps you create a simple cloud quickly, in as little as 55 seconds. All of the VMs are created on top of the KVM hypervisor and have SSD (Solid-State Drive) as the primary disk."*

#### Feature

- Based on your need, DigitalOcean offers [different plans](https://www.digitalocean.com/pricing/).
- DigitalOcean provides other features, like Floating IPs, Shared Private Networking, Load Balancers, Team Accounts, etc.
- It offers a one-click installation of a multitude of application stacks like LAMP, LEMP, MEAN, and Docker.

### OpenStack

#### Concept

*"With [OpenStack](https://www.openstack.org/)**,** we can offer a cloud computing platform for public and private clouds. Other than providing a IaaS solution, OpenStack has evolved over time to provide other services, like Database, Storage, etc".*

#### Feature

Due to the modular nature of OpenStack, anyone can add additional components to get specific features or functionality. Some of the major OpenStack components are

- [Keystone](http://docs.openstack.org/developer/keystone/) 
  Provides Identity, Token, Catalog, and Policy services to projects.

- [Nova](http://docs.openstack.org/developer/nova/) 
  Provides on-demand compute resources.
- [Horizon](http://docs.openstack.org/developer/horizon/) 
  Provides the Dashboard, which is a web-based user interface to manage the OpenStack service.
- [Neutron](http://docs.openstack.org/developer/neutron/) 
  Implements the network as a service and provides network capabilities to different OpenStack components.
- [Glance](http://docs.openstack.org/developer/glance/) 
  Provides a service where users can upload and discover data assets, like images and metadata.
- [Swift](http://docs.openstack.org/developer/swift/) 
  Provides a highly available, distributed, eventually consistent object/blob store.
- [Cinder](http://docs.openstack.org/developer/cinder/) 
  Provides block storage as a service.
- [Heat](http://docs.openstack.org/developer/heat/) 
  Provides a service to orchestrate composite cloud applications, using a declarative template format through an OpenStack-native REST API.
- [Ceilometer](http://docs.openstack.org/developer/ceilometer/) 
  It is part of the Telemetry project and provides data collection services for billing and other purposes.

Each of the OpenStack components is also modular by design. For example, with Nova we can select an underneath hypervisor depending on the requirement, which can be either libvirt (qemu/KVM), Hyper-V, VMware, XenServer, Xen via libvirt.