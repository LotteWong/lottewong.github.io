---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(6) Container Orchestration
date: 2019-07-16 15:13:21
categories: Cloud Computing
tags:
- Kubernetes
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Describe different container orchestration tools: Docker Swarm, Kubernetes, Mesos, Nomad, Amazon ECS.
- Describe different Kubernetes hosted services like AWS Elastic Kubernetes Service (EKS), Azure Kubernetes Services (AKS) and Google Kubernetes Engine (GKE).
- Deploy sample applications using various container orchestration tools: Docker Swarm, Kubernetes, Mesos, Nomad, Amazon ECS.



<!-- more -->



## Concept

*"Container orchestration is an umbrella term which encompasses container scheduling and cluster management. Container scheduling allows us to decide on which host a container or a group of containers should be deployed. With the cluster management orchestrator we can manage the existing nodes, add or delete nodes, etc".*

## Docker Swarm

### Concept

*"[Docker Swarm](https://docs.docker.com/engine/swarm/) is a native container orchestration tool from Docker, Inc. It logically groups multiple Docker engines to create a virtual engine, on which we can deploy and scale applications".*

### Features

- It is compatible with Docker tools and API, so that the existing workflow does not change much.
- It provides native support to Docker networking and volumes.
- It can scale up to large numbers of nodes.
- It supports failover and High Availability for the cluster manager.
- It uses a declarative approach to define the desired state of the various services of the application stack.
- For each service, you can declare the number of tasks you want to run. When you scale up or down, the Swarm manager automatically adapts by adding or removing tasks to maintain the desired state.
- The Docker Swarm manager node constantly monitors the cluster state and reconciles any differences between the actual state and your expressed desired state.
- The communication between the nodes of Docker Swarm is enforced with [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security) (TLS), which makes it secure by default.
- It supports rolling updates, using which we can control the delay between service deployment to different sets of nodes. If anything goes wrong, you can roll back a task to a previous version of the service.

### Components

- **Swarm Manager Nodes**

  There can be one or more manager nodes. They accept commands on behalf of the cluster and make scheduling decisions. They also store the cluster state using the *Internal Distributed State Store*, which uses the [Raft](https://raft.github.io/raft.pdf) consensus algorithm. One or more nodes can be configured as managers, but they work in *active*/*passive* modes.

- **Swarm Worker Nodes**
  They run the Docker Engine and the sole purpose of the worker nodes is to run the container workload given by the manager node(s).

- ![Swarm_Cluster_Components](/images/Swarm_Cluster_Components.png)

### Docker Machine

#### Concept

*"Docker Machine helps us configure and manage one or more Docker engines running locally or on cloud environments. With Docker Machine we can start, inspect, stop, and restart a managed host, upgrade the Docker client and daemon, and configure a Docker client to talk to our host. We can also use Docker Machine to configure a Swarm cluster".*

![provision-use-case](/images/provision-use-case.png)

#### Commands

Docker Machine has drivers for Amazon EC2, Google Cloud, Digital Ocean, Vagrant, etc., to set up Docker engines. You can also add already running instances of the Docker engine to the Docker Machine:

- Setting up the Docker engine using the VirtualBox driver:

  ```powershell
  $ docker-machine create -d virtualbox dev1
  ```

- Setting up the Docker engine using DigitalOcean:

  ```powershell
  $ docker-machine create --driver digitalocean --digitalocean-access-token=<TOKEN> dev2
  ```

### Docker Compose

#### Concept

*"[Docker Compose](https://docs.docker.com/compose/overview/) allows us to define and run multi-container applications through a configuration file. In a configuration file, we can define services, images or Dockerfiles to use, network, etc. Docker Swarm uses [Docker Stack](https://docs.docker.com/get-started/part5/) to deploy the distributed applications. We can use Docker Compose to generate the stack file, which can be used to deploy applications on Docker Swarm".*

#### DockerFile

![DockerFile](/images/DockerFile.png)

### Docker Datacenter

#### Concept

*"Docker has another project called [Docker Datacenter](https://www.docker.com/products/docker-datacenter), which is built on top of UCP and Docker Trusted Registry. Docker Datacenter is hosted completely behind the firewall. With Docker Datacenter we can build an enterprise-class CaaS platform on-premises, as it is built on top of Docker Swarm and integrates well with Docker tools, Docker Registry. It also has other features, such as LDAP/AD integration, monitoring, logging, network and storage plugins".*

![Docker_Datacenter](/images/Docker_Datacenter.png)

### Docker Enterprise Edition

#### Concept

*"Docker Enterprise Edition (EE) 2.0 is the Container-as-a-Service (CaaS) platform that manages the entire lifecycle of the applications on enterprise Linux or Windows operating systems and Cloud providers. It supports Docker Swarm and Kubernetes as container orchestrators."*

#### Features

- It is a multi-Linux, multi-OS, multi-Cloud solution.
- It supports Docker Swarm and Kubernetes as container orchestrators.
- It provides centralized cluster management.
- It has a built-in authentication mechanism with role-based access control (RBAC).

#### Components

- **Docker EE Engine**

  It is a commercially supported Docker Engine for creating images and running Docker containers.

- **Docker Trusted Registry** (**DTR**)
  It is a production-grade image registry designed to store images, from Docker, Inc.
  
- **Universal Control Plane** (**UCP**)It manages the Kubernetes and Swarm orchestrators, deploys applications using the CLI and GUI, and provides High Availability. UCP also provides role-based access control to ensure that only authorized users can make changes and deploy applications to your cluster.

![Docker_EE_Architecture](/images/Docker_EE_Architecture.png)

## Kubernetes

### Concept

*“[Kubernetes](https://kubernetes.io/) is an Apache 2.0-licensed open source project for automating deployment, operations, and scaling of containerized applications. It was started by Google, but many other companies like Docker, Red Hat, and VMware contributed to it. Kubernetes supports container runtimes like Docker, CRI-O, etc., to run containers”.*

### Features

- It automatically places containers based on resource requirements and other constraints.
- It supports horizontal scaling through the CLI and GUI. It can auto-scale based on the CPU load as well.
- It supports rolling updates and rollbacks.
- It supports multiple volume plugins like the GCP/AWS disk, NFS, iSCSI, Ceph, Gluster, Cinder, Flocker, etc. to attach volumes to pods.
- It automatically self-heals by restarting failed pods, rescheduling pods from failed nodes, etc.
- It deploys and updates secrets for an application without rebuilding the image.
- It supports batch execution.
- It support High Availability cluster.
- It eliminates infrastructure lock-in by providing core capabilities for containers without imposing restrictions.
- We can deploy and update the application at scale.

### Components

- **Cluster** 
  The cluster is a group of systems (physical or virtual) and other infrastructure resources used by Kubernetes to run containerized applications.
  
- **Master Node** 
  The master is a system that takes pod scheduling decisions and manages the replication and manager nodes. It has three main components: API Server, Scheduler, and Controller. There can be more than one master node. 
  
- **Worker Node** 
  A system on which pods are scheduled and run. The node runs a daemon called **kubelet** to communicate with the master node. **kube-proxy**, which runs on all nodes, allows applications from the external world.
  
- **Key-Value Store**
  The Kubernetes cluster state is saved in a *key-value store*, like **etcd**. It can be either part of the same Kubernetes cluster or it can resides outside.
  
- **Pod** 
  The pod is a co-located group of containers with shared volumes. It is the smallest deployment unit in Kubernetes. A pod can be created independently, but it is recommended to use the Replica Set, even if only a single pod is being deployed.
  
- **Replica Set**

  The Replica Set manages the lifecycle of pods. It makes sure that the desired numbers of pods is running at any given point in time. 

- **Deployments**

  Deployments allow us to provide declarative updates for pods and Replica Sets. We can define Deployments to create new resources, or replace existing ones with new ones. 

- **Service** 
  The service groups sets of pods together and provides a way to refer to them from a single static IP address and the corresponding DNS name. 

- **Label** 
  The label is an arbitrary key-value pair which is attached to a resource like pod, Replica Set, etc. In the example above, we defined labels as **app** and **tier**.

- **Selector** 
  Selectors enable us to group resources based on labels. In the above example, the **frontend** service will select all pods which have the labels **app==dockchat** and **tier==frontend**.

- **Volume** 
  The volume is an external filesystem or storage which is available to pods. They are built on top of [Docker volumes](https://docs.docker.com/userguide/dockervolumes/).

- **Namespace** 
  The namespace allows us to partition the cluster into sub-clusters.

![kubernetes_architecture](/images/kubernetes_architecture.png)

### KubernetesFile

Some typical use cases are presented below:

1. Create a Deployment to bring up a Replica Set and pods.

2. Check the status of a Deployment to see if it succeeds or not.

3. Later, update that Deployment to recreate the pods (for example, to use a new image).

4. Roll back to an earlier Deployment revision if the current Deployment isn’t stable.

5. Pause and resume a Deployment. Below we provide a [sample deployment](http://kubernetes.io/docs/user-guide/deployments/):

   ![KubernetesFile1](/images/KubernetesFile1.png)

   ![KubernetesFile2](/images/KubernetesFile2.png)

### Kubernetes Hosted Solutions

#### Concept

*"Kubernetes can be deployed anywhere, be it on-premise or on-cloud. If we are deploying on-premise, then our Kubernetes administrators would have to perform all the Kubernetes management tasks like upgrading, back up, etc. With an on-cloud setup, we have different options. For instance, we can manage our own on-premise or opt for hosted Kubernetes services in which all the management tasks would be performed by the service providers. "*

There are many hosted solutions available for Kubernetes, including:

- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/)
Offers managed Kubernetes clusters on Google Cloud Platform.
- [Amazon Elastic Container Service for Kubernetes](https://aws.amazon.com/eks/) (Amazon EKS) 
Offers a managed Kubernetes service on AWS.
- [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS)
Offers a managed Kubernetes clusters Microsoft Azure.
- [Stackpoint.io](https://stackpoint.io/)
Provides Kubernetes infrastructure automation and management for multiple public clouds.
- [OpenShift Dedicated](https://www.openshift.com/products/dedicated/)
Offers managed Kubernetes clusters powered by Red Hat.

#### Google Kubernetes Engine (GKE)

##### Concept

*“[Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) is a fully-managed solution for running Kubernetes on Google Cloud. As we have learned earlier, Kubernetes is used for automating deployment, operations, and scaling of containerized applications. In GKE, Kubernetes can be integrated with all Google Cloud Platform's services, like Stackdriver monitoring, diagnostics, and logging, identity and access management, etc”.*

##### Features

- It has all of Kubernetes' features.
- It runs on a container-optimized OS built and managed by Google.
- It is a fully-managed service, so the users do not have to worry about managing and scaling the cluster.
- We can store images privately, using the private container registry.
- Logging can be enabled easily using Google [Cloud Logging](https://cloud.google.com/logging/docs/).
- It supports Hybrid Networking to reserve an IP address range for the container cluster.
- It enables a fast setup of managed clusters.
- It facilitates increased productivity for Dev and Ops teams.
- It is Highly Available in multiple zones and SLA promises 99.5% of availability.
- It has Google-grade managed infrastructure.
- It can be seamlessly integrated with all GCP services.
- It provides a feature called *Auto Repair*, which initiates a repair process for unhealthy nodes.

#### Amazon Elastic Container Service for Kubernetes (EKS)

##### Concept

*"[Amazon Elastic Container Service for Kubernetes](https://aws.amazon.com/eks/) is a hosted Kubernetes service offered by AWS. With Amazon EKS, users don't need to worry about the infrastructure management, deployment and maintenance of the Kubernetes control plane. EKS provides a scalable and highly-available control plane that runs across multiple AWS availability zones. It can automatically detect the unhealthy Kubernetes control plane nodes and replace them. EKS also supports cluster autoscaling, using which it can dynamically add worker nodes, based on the workload. It also integrates with Kubernetes RBAC (Role-Based Control Access) to support AWS IAM authentication".*

![Amazon_Elastic_Container_Service_for_Kubernetes](/images/Amazon_Elastic_Container_Service_for_Kubernetes.png)

##### Features

- No need to manage the Kubernetes Control Plane.
- Provides secure communication between the worker nodes and the control plane.
- Supports auto scaling in response to changes in load.
- Well integrated with various AWS services, like IAM and CloudTrail.
- Certified hosted Kubernetes platform.

#### Azure Kubernetes Service (AKS)

##### Concept

*"[Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) is a hosted Kubernetes service offered by Microsoft Azure. AKS offers a fully-managed Kubernetes container orchestration service, which reduces the complexity and operational overhead of managing Kubernetes. AKS handles all of the cluster management tasks, health monitoring, upgrades, scaling, etc. AKS also supports cluster autoscaling using which it can dynamically add worker nodes, based on the workload. It supports Kubernetes RBAC (Role-Based Control Access) and can integrate with Azure Active Directory for identity and security management."*

##### Features

- No need to manage the Kubernetes Control Plane.
- Supports GUI and CLI-based deployment.
- Integrates well with other Azure services.
- Certified hosted Kubernetes platform.
- Compliant with SOC and ISO/HIPAA/HITRUST.

## Apache Mesos

### Concept

*"[Apache Mesos](http://mesos.apache.org/) was created with this idea in mind, so that we can optimally use the resources available, even if we are running disparate applications on a pool of nodes. It helps us treat a cluster of nodes as one big computer, which manages CPU, memory, and other resources across a cluster. Mesos provides functionality that crosses between Infrastructure as a Service (IaaS) and Platform as a Service (PaaS)."*

### Features

- It can scale up to 10,000 nodes.
- It uses ZooKeeper for fault-tolerant replicated master and slaves.
- It provides support for Docker containers.
- It enables native isolation between tasks with Linux containers.
- It allows multi-resource scheduling (memory, CPU, disk, and ports).
- It uses Java, Python and C++ APIs to develop new parallel applications.
- It uses WebUI to view cluster statistics.
- It allows high resource utilization.
- It helps handling mixed workloads.
- It provides an easy-to-use container orchestration right out of the box.
- It ships binaries for different components (e.g. master, slaves, frameworks, etc.), which we can bind together to create our Mesos cluster.

### Components

- **Master**

  Master nodes are the brain of the cluster and provide a single source of truth for running tasks. There is one active master node at any point in time. The master node mediates between schedulers and slaves. Slaves advertise their resources to the master node, then the master node forwards them to the scheduler. Once the scheduler accepts the offer, it sends the task to run on the slave to the master, and the master forwards these tasks to the slave.

- **Slave**

  Slaves manage resources at each machine level and execute the tasks submitted via the scheduler.

- **Frameworks** 
  Frameworks are distributed applications that solve a specific use case. They consist of a scheduler and an executor. The scheduler gets a resource offer, which it can accept or decline. The executor runs the job on the slave, which the scheduler schedules. There are many existing frameworks and we can also create custom ones. Some of the existing frameworks are: Hadoop, Spark, Marathon, Chronos, Jenkins, Aurora, and many more.

- **Executor** Executors are used to run jobs on slaves. They can be shell scripts, Docker containers, and programs written in different languages (e.g. Java). 

![mesos_architecture](/images/mesos_architecture.jpg)

### Mesosphere DC/OS

#### Concept

*"Mesosphere Enterprise DC/OS, offers a one-click install and enterprise features like security, monitoring, user interface, etc. on top of Mesos. By default, DC/OS comes with the Marathon framework and others can be added as required".*

#### Features

The Marathon framework has the following features:

- It starts, stops, scales, and updates applications.
- It has a nice web interface, API.
- It is highly available, with no single point of failure.
- It uses native Docker support.
- It supports rolling deploy/restart.
- It allows application health checks.
- It provides artifact staging.

In addition to the Mesos features presented on the previous page, DC/OS provides the following ones:

- It provides an easy-to-use container orchestration right out of the box.
- It can configure multiple resource isolation zones.
- It can support applications with multiple persistent and ephemeral storage options.
- It allows you to install both public community and private community packaged applications.
- It allows you to manage your cluster and services using the web and command line interfaces.
- It allows you to easily scale up and scale down your services.
- It provides automation for updating services and the systems with zero downtime.
- Its *Enterprise* edition provides centralized management, control plane for service availability and performance monitoring.

#### Components

The DC/OS Master has the following default components:

- **Mesos Master Process** 
It is similar to the master component of Mesos.
- **Mesos DNS** 
It provides service discovery within the cluster, so applications and services running inside the cluster can reach to each other.
- **Marathon** 
It is a framework which comes by default with DC/OS and provides the *init* system*.*
- **ZooKeeper** 
It is a high-performance coordination service that manages the DC/OS services.
- **Admin Router** 
It is an open source Nginx configuration created by Mesosphere, providing central authentication and proxy to DC/OS services within the cluster.

The DC/OS Agent nodes have the following components:

- **Mesos Agent Process** 
It runs the **mesos-slave** process, which is similar to the slave component of Mesos.
- **Mesos Containerizer**
It provides lightweight containerization and resource isolation of executors, using Linux-specific functionality, such as cgroups and namespaces.
- **Docker Container** 
It provides support for launching tasks that contain Docker images.

DC/OS has its own [command line](https://docs.mesosphere.com/usage/cli/) and web [interfaces](https://docs.mesosphere.com/1.11/gui/), and comes with a simple packaging and installation.

![DC_OS_Architecture_Layers](/images/DC_OS_Architecture_Layers.png)

## Nomad

### Concept

*"[HashiCorp Nomad](https://www.nomadproject.io/) is a cluster manager and resource scheduler from [HashiCorp](https://www.hashicorp.com/), which is distributed, highly available, and scales to thousands of nodes. It is especially designed to run microservices and batch jobs, and it supports different workloads, like containers (Docker), VMs, and individual applications. It is also capable of scheduling applications and services on different platforms like Linux, Windows and Mac".*

### Features

- It handles both cluster management and resource scheduling.
- It supports multiple workloads, like containers (Docker), VMs, unikernels, and individual applications.
- It has multi-datacenter and multi-region support. We can have a Nomad *client*/*server* running in different clouds, which form the same logical Nomad *cluster*.
- It bin-packs applications onto servers to achieve high resource utilization.
- In Nomad, millions of containers can be deployed or upgraded by using the **job** file.
- It provides a built-in dry run execution facility, which shows the scheduling actions that are going to take place.
- It ensures that applications are running in *failure* scenarios.
- It supports long-running services, as well as batch jobs and cron jobs.
- It provides a built-in mechanism for rolling upgrades.
- Blue-green and canary deployments can be deployed using a declarative job file syntax.
- If nodes fail, Nomad automatically redeploys the application from unhealthy nodes to healthy nodes.

### NomadFile

It is distributed as a single binary, which has all of its dependency and runs in a *server* and *client* mode. To submit a job, the user has to define it using a declarative language called [HashiCorp Configuration Language](https://github.com/hashicorp/hcl) (HCL) with its resource requirements. Once submitted, Nomad will find available resources in the cluster and run it to maximize the resource utilization.

![NomadFile](/images/NomadFile.png)

## Amazon ECS

### Concept

*“[Amazon Elastic Container Service](https://aws.amazon.com/ecs/) (Amazon ECS) is part of the Amazon Web Services (AWS) offerings. It provides a fast and highly scalable container management service that makes it easy to run, stop, and manage Docker containers on a cluster.”*

It can be configured in the following two launch modes:

- **Fargate Launch Type**
  

AWS Fargate allows us to run containers without managing servers and clusters. In this mode, we just have to package our applications in containers along with CPU, memory, networking and IAM policies. We don't have to provision, configure, and scale clusters of virtual machines to run containers, as AWS will take care of it for us.

- **EC2 Launch Type**

  With the EC2 launch type, we can provision, patch, and scale the ECS cluster. This gives more control to our servers and provides a range of customization options.

![EC2_Launch_Type](/images/EC2_Launch_Type.png)

### Features

- It is compatible with Docker.
- It provides a managed cluster, so that users do not have to worry about managing and scaling the cluster.
- The task definition allows the user to define the applications through a .json file. Shared data volumes, as well as resource constraints for memory and CPU, can also be defined in the same file.
- It provides APIs to manage clusters, tasks, etc.
- It allows easy updates of containers to new versions.
- The monitoring feature is available through AWS CloudWatch.
- The logging facility is available through AWS CloudTrail.
- It supports third party Docker Registry or Docker Hub.
- AWS Fargate allows you to run and manage containers without having to provision or manage servers.
- AWS ECS allows you to build all types of containers. You can build a long-running service or a batch service in a container and run it on ECS.
- You can apply your Amazon Virtual Private Cloud (VPC), security groups and AWS Identity and Access Management (IAM) roles to the containers, which helps maintain a secure environment.
- You can run containers across multiple availability zones within regions to maintain High Availability.
- ECS can be integrated with AWS services like Elastic Load Balancing, Amazon VPC, AWS IAM, Amazon ECR, AWS Batch, Amazon CloudWatch, AWS CloudFormation, AWS CodeStar, AWS CloudTrail, and more.

### Components

- **Cluster** 
  It is a logical grouping of tasks or services. With the EC2 launch type, a cluster is also a grouping of container instances.
- **Container Instance** 
  It is only applicable if we use the EC2 launch type. We define the Amazon EC2 instance to become part of the ECS cluster and to run the container workload.
- **Container Agent**
  It is only applicable if we use the Fargate launch type. It allows container instances to connect to your cluster.
- **Task Definition** 
  It specifies the blueprint of an application, which consists of one or more containers. 
- **Scheduler** 
  It places tasks on the cluster.
- **Service** 
  It allows one or more instances of tasks to run, depending on the task definition.
- **Task** 
  It is a running container instance from the task definition.
- **Container** 
  It is a Docker container created from the task definition.

### AmazonEC2File

![AmazonEC2File1](/images/AmazonEC2File1.png)

![AmazonEC2File2](/images/AmazonEC2File2.png)

