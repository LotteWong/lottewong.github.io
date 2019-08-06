---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(12) Tools for Cloud Infrastructure I (Configuration Management)
date: 2019-07-21 15:13:21
categories: Cloud Computing
tags:
- CI/CD
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- List tools used to configure and manage the Cloud Infrastructure.
- Discuss and use configuration management tools: Ansible, Puppet, Chef and Salt Open.



<!-- more -->



## Concept

### Infrastructure as Code

*“When we have numerous systems (both Physical and VMs) to manage in different environments like Development, QA and Production, we want to do it in an automated way. At any point in time, we want to have consistent and desired state of systems and software installed on them. This is also referred as Infrastructure as Code.”*

### Configuration Management

*“Configuration Management tools allow us to define the desired state of the systems in an automated way. In this section, we will take a look at some of the Configuration Management tools available today, such as Ansible, Chef, Puppet, and Salt.”*

## Ansible

### Concept

*”Red Hat's [Ansible](http://www.ansible.com/) is an easy-to-use, open source configuration management tool. It is an agentless tool which works on top of SSH. Ansible also automates cloud provisioning, application deployment, orchestration, etc.“*

### Features

- It is an agentless tool.
- It provides role-based access control.

### Components

#### Nodes

The nodes can be grouped together as shown in the picture below. Ansible also supports dynamic inventory files for cloud providers like AWS and OpenStack. The management node connects to these nodes with a password or it can do passwordless login, using SSH keys.

![Ansible_Multi-Node_Deployment_Workflow](/images/Ansible_Multi-Node_Deployment_Workflow.png)

The Ansible management node connects to the nodes mentioned in the inventory file and runs the tasks mentioned in the playbook. A management node can be installed on any Unix-based system like Linux, Mac OS X, etc. It can manage any node which supports SSH and Python 2.4 or later.

#### Playbooks

[Playbooks](http://docs.ansible.com/ansible/playbooks.html) are Ansible’s configuration, deployment, and orchestration language.

Below we provide an example of a playbook which performs different tasks based on roles:

```yaml
---
# This playbook deploys the whole application stack in this site. 
- name: apply common configuration to all nodes
  hosts: all
  remote_user: root

  roles:
    - common

- name: configure and deploy the webservers and application code
  hosts: webservers
  remote_user: root

  roles:
    - web

- name: deploy MySQL and configure the databases
  hosts: dbservers
  remote_user: root

  roles:
    - db

The sample tasks mentioned in the playbook are:

---
# These tasks install http and the php modules.

- name: Install http and php etc
  yum: name={{ item }} state=present
  with_items:
   - httpd
   - php
   - php-mysql
   - git
   - libsemanage-python
   - libselinux-python

- name: insert iptables rule for httpd
  lineinfile: dest=/etc/sysconfig/iptables create=yes state=present regexp="{{ httpd_port }}" insertafter="^:OUTPUT "
              line="-A INPUT -p tcp  --dport {{ httpd_port }} -j  ACCEPT"
  notify: restart iptables

- name: http service state
  service: name=httpd state=started enabled=yes
```

Ansible ships with a [default set of modules](https://github.com/ansible/ansible-modules-core), like packaging, network, etc., which can be executed directly or via [Playbooks](http://docs.ansible.com/ansible/playbooks.html). We can also create custom modules.

#### Galaxy

[Ansible Galaxy](https://galaxy.ansible.com/) is a free site for finding, downloading, and sharing community-developed Ansible roles.

#### Tower

Ansible also has an enterprise product called [Ansible Tower](https://www.ansible.com/tower), which has a GUI interface, access control, central management, etc.

## Puppet

### Concept

*”Puppet is an open source configuration management tool. It mostly uses the agent/master (client/server) model to configure the systems. The agent is referred to as the Puppet Agent and the master is referred to as the Puppet Master. The Puppet Agent can also work locally and is then referred to as Puppet Apply.“*

### Features

- It provides scalability, automation, and centralized reporting.
- It provides role-based access control.

### Components

#### Puppet Agent

We need to install Puppet Agent on each system we want to manage/configure with Puppet. 

Each agent:

- Connects securely to the Puppet Master to get the series of instructions in a file referred to as the Catalog File.
- Performs operations from the Catalog File to get to the desired state.
- Sends back the status to the Puppet Master.

Puppet Agent can be installed on the following platforms:

- Linux
- Windows
- Mac OSX.

#### Puppet Master

Puppet Master can be installed only on Unix-based system. It:

- Compiles the Catalog File for hosts based on the system, configuration, manifest file, etc.
- Sends the Catalog File to agents when they query the master.
- Has information about the entire environment, such as host information, metadata (like authentication keys), etc.
- Gathers reports from each agent and then prepares the overall report.

#### The Catalog File

Puppet prepares a Catalog File based on the manifest file. A manifest file is created using the Puppet Code:

```json
user { 'nkhare':
  ensure     => present,
  uid        => '1001',
  gid        => '1001',
  shell      => '/bin/bash',
  home       => '/home/nkhare'
}
```

A manifest file can have one or more sections of code, like we exemplified above, and each of these sections of code can have a signature like the following:

```json
resource_type { 'resource_name':
  attribute => value
  ...
}
```

Puppet defines resources on a system as Type which can be file, user, package, service, etc. They are well-documented in their [documentation](https://docs.puppet.com/puppet/latest/reference/type.html).

After processing the manifest file, Puppet Master prepares the Catalog File based on the target platform.

#### Others

- Centralized reporting (PuppetDB), which helps us generate reports, search a system, etc.
- Live system management
- [Puppet Forge](https://forge.puppet.com/), which has ready-to-use modules for manifest files from the community.

## Chef

### Concept

*”[Chef](https://www.chef.io/) uses the client/server model to do the configuration management. The client is installed on each host which we want to manage and is referred to as Chef Client. The server is referred to as Chef Server.“*

![chef_overview](/images/chef_overview.svg)

### Features

- It provides real-time visibility with [Chef Analytics](https://docs.chef.io/analytics.html).
- It provides role-based access control.

### Components

Additionally, there is another component called Chef Workstation, which is used to:

- Develop *cookbooks* and *recipes.*
- Synchronize **chef-repo** with the version control system.
- Run command line tools.
- Configure policy, roles, etc.
- Interact with nodes to do a one-off configuration.

Chef supports the following [platforms ](https://downloads.chef.io/chef-client/)for *Chef Client:*

- Unix-based systems
- Mac OS X
- Windows
- Cisco IOS XR and NX-OS.

*Chef Server* is supported on the following platforms:

- Red Hat Enterprise Linux
- Ubuntu Linux.

Chef also has a GUI built on top of *Chef Server*, which can help ups running the cookbook from the browser, prepare reports, etc.

#### Chef Cookbook

A [Chef Cookbook](https://docs.chef.io/cookbooks.html) is the basic unit of configuration which defines a scenario and contains everything that supports the scenario. Two of its most important components are:

- [Recipes](https://docs.chef.io/recipes.html) 
  A recipe is the most fundamental unit for configuration, which mostly contains resources, resource names, attribute-value pairs, and actions.

  ```yaml
  package "apache2" do
    action :install
  end
  
  service "apache2" do
    action [:enable, :start]
  end
  ```

- [Attributes](https://docs.chef.io/attributes.html) 
  An attribute helps us define the state of the node. After each *chef-client* run, the node's state is updated on the *Chef Server*.

#### Chef Knife

[Knife](https://docs.chef.io/knife.html) provides an interface between a local **chef-repo** and the *Chef Server*.

## Salt Open

### Concept

*”[Salt Open](https://saltstack.com/salt-open-source/) is an open source configuration management system built on top of a remote execution framework. It uses the client/server model, where the server sends commands and configurations to all the clients in a parallel manner, which the clients run, returning back the status.“*

### Features

- It supports agent and agentless deployments.
- It provides role-based access control.

### Components

#### Salt Minions

Each client is referred to as a **Salt minion**.

![Salt_Minions](/images/Salt_Minions.png)

Minions can be installed on:

- Unix-based systems
- Windows
- Mac OS X.

#### Salt Masters

A server is referred to as a **Salt master**. Multi-master is also supported.

![Salt_Master](/images/Salt_Master.png)

In a default setup, the Salt master and minions communicate over a high speed data bus, [ZeroMQ](http://zeromq.org/), which requires an agent to be installed on each minion. Salt also supports an agentless setup on top of SSH. The Salt master and minions communicate over a secure and encrypted channel.

#### Modules and Returners

Remote execution is based on [Modules](https://docs.saltstack.com/en/latest/ref/modules/all/index.html) and [Returners](https://docs.saltstack.com/en/latest/ref/returners/all/index.html).

Modules provide basic functionality, like installing packages, managing files, managing containers, etc. All support modules are listed in the [Salt documentation](https://docs.saltstack.com/en/latest/ref/modules/all/index.html). We can also write custom modules.

With Returners, we can save a minion's response on the master or other locations. We can use default [Returners](https://docs.saltstack.com/en/latest/ref/returners/all/index.html) or write custom ones.

Salt has different [State Modules](https://docs.saltstack.com/en/latest/ref/states/all/index.html) to manage a state.

#### Grains and Pillar Data

All the information collected from minions is saved on the master. The collected information is referred to as Grains. Private information like cryptographic keys and other specific information about each minion which the master has is referred to as Pillar Data. Pillar Data is shared between the master and the individual minion.

By combining Grains and Pillar Data, the master can target specific minions to run commands on them.