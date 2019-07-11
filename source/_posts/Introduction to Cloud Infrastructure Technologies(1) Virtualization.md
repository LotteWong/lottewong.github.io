---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(1) Virtualization
categories: Cloud Computing
tags:
- Virtualization
- KVM
- VirtualBox
- Vagrant
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Describe the different types of virtualization.

- Explain how hypervisors can be used to create virtual machines. 
- Create and configure virtual machines automatically, using KVM, VirtualBox and Vagrant.



<!-- more -->



## Virtualization

### Concept

*"In computing, virtualization refers to the act of creating a virtual (rather than actual) version of something, including virtual computer hardware platforms, operating systems, storage devices, and computer resources".*

### KVM

![Kernel-based_Virtual_Machine](/images/Kernel-based_Virtual_Machine.png)

### VirtualBox

- VirtualBox is distributed under the [GNU General Public License (GPL) version 2](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html).

### Vagrant

#### Concept

*"Configuring and sharing one VM is easy, but, when we have to deal with multiple VMs for the same project, doing everything manually can be tiresome. [Vagrant](https://www.vagrantup.com/) by HashiCorp helps us automate the setup of one or more VMs by providing an end-to-end lifecycle using the vagrant command line. Vagrant is a cross-platform tool. It can be installed on Linux, Mac OSX, and Windows. We have to use different providers, depending on the OS. It has recently added support for Docker, which can help us manage Docker containers.“*

#### Feature

- **Boxes**

  We need to provide an image in the Vagrantfile, which we can use to instantiate machines. In the example above, we have used **centos/7** as the base image. If the image is not available locally, then it can be downloaded from a central repository like [Atlas](https://atlas.hashicorp.com/), which is the image repository provided by HashiCorp. We can version these images and use them depending on our need, by updating the Vagrantfile accordingly.

- **Vagrant Providers** 

  [Providers](https://www.vagrantup.com/docs/providers/) are the underlying engine/hypervisor used to provision a machine. By default, Vagrant supports VirtualBox, Hyper-V and Docker. We also have custom providers, like KVM, AWS, etc. VirtualBox is the default provider.

- **Synced Folders** 

  With the *Synced Folder* feature, we can sync a directory on the host system with a VM, which helps the user manage shared files/directories easily. For example, in the above example, if we un-comment the line below from Vagrantfile, then the **../data** folder from the current working directory of the host system would be shared with the **/vagrant_data** file on the VM.

  ```config
  config.vm.synced_folder "../data", "vagrant_data"
  ```

- **Provisioning**
  [Provisioners](https://www.vagrantup.com/docs/provisioning/) allow us to automatically install software, make configuration changes, etc. after the machine is booted. It is a part of the **vagrant up** process. There are many types of provisioners available, such as File, Shell, Ansible, Puppet, Chef, Docker, etc. In the example below, we used Shell as the provisioner to install the **vim** package.

  ```config
  config.vm.provision "shell", inline: <<-SHELL
  	yum install vim -y
  SHELL
  ```

- **Plugins**
  We can use [plugins](https://www.vagrantup.com/docs/plugins/) to extend the functionality of Vagrant.

#### VagrantFile

![VagrantFile](/images/VagrantFile.png)

#### VagrantCmd

- `vagrant up`: start VM
- `vagrant halt`: stop VM
- `vagrant destory`: delete VM
- `vagrant status`: check status
- `vagrant user`: login in as user