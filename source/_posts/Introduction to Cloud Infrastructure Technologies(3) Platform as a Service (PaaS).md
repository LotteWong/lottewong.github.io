---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(3) Platform as a Service (PaaS)
categories: Cloud Computing
tags:
- PaaS
- Cloud Foudry
- OpenShift
- Heroku
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Explain the concept of Platform as a Service (PaaS).
- Distinguish between different PaaS providers.
- Deploy an application on top of different PaaS providers: Cloud Foundry, OpenShift, Heroku.



<!-- more -->



## Platform as a Service (PaaS)

### Concept

*"Platform as a Service (PaaS) is a class of cloud computing services which allows its users to develop, run, and manage applications without worrying about the underlying infrastructure. With PaaS, users can simply focus on building their applications, which is a great help to developers. We can either use PaaS services offered by different cloud computing providers like Amazon, Google, Azure, etc., or deploy it on-premise, using software like OpenShift Origin. PaaS can be deployed on top of IaaS, or, independently on VMs, bare metal, and containers”.*

### Cloud Foudry

#### Concept

*"[Cloud Foundry](https://www.cloudfoundry.org/) CF) is an open source Platform as a Service (PaaS) that provides a choice of clouds, developer frameworks, and application services. It can be deployed on-premise or on IaaS, like AWS, vSphere, or OpenStack. There are many commercial [CF cloud providers](https://www.cloudfoundry.org/learn/certified-providers/) as well, like IBM Cloud Foundry, SAP Cloud Platform, Pivotal Cloud Foundry, etc".*

#### Feature

Among the characteristics employed by Cloud Foundry are the following:

- Application portability

- Application auto-scaling
- Centralized platform management
- Centralized logging
- Dynamic routing
- Application health management
- Role-based application deployment
- Horizontal and vertical scaling
- Security
- Support for different IaaS technologies.

#### CF Application Runtime

##### Concept

*"[CF Application Runtime](https://www.cloudfoundry.org/application-runtime/), previously known as Elastic Runtime, is used by developers to run applications written in any language or framework on the cloud of their choice".*

##### Feature

- CF Application Runtime uses [buildpacks](https://docs.cloudfoundry.org/buildpacks/), which provide the framework and runtime support for the applications. They are programming language-specific and have information about how to download dependencies and configure specific applications. Developers can build new buildpacks or customize the existing ones.

- In addition, by using the Open Service Broker API, we can connect our application to external services like databases or third-party SaaS providers.

- As presented in the graphic below, the CF Application Runtime platform can manage the entire workflow and lifecycle of the application, from routing to logging. 

  ![CF_Application_Runtime](/images/CF_Application_Runtime.jpg)

  

#### CF Container Runtime

##### Concept

*"[CF Container Runtime](https://www.cloudfoundry.org/container-runtime/) gives Cloud Foundry the flexibility to deploy developer-built, pre-packaged applications using containers".*

##### Feature

- [BOSH](https://www.cloudfoundry.org/bosh/) is a cloud-agnostic open source tool for release engineering, deployment, and lifecycle management of complex distributed systems.

- Kubernetes is a container orchestrator which allows us to deploy containers.

- The CF Container Runtime platform does it by deploying containers on a [Kubernetes](https://kubernetes.io/) cluster, which is managed by CF BOSH. With CF BOSH, you can achieve high availability, scaling, VM healing and upgrades for the Kubernetes cluster.

  ![CF_Container_Runtime](/images/CF_Container_Runtime.jpg)

### OpenShift

#### Concept

*"[OpenShift](https://www.openshift.com/) is an open source PaaS solution provided by Red Hat. It is built on top of the container technology, which uses Kubernetes underneath. OpenShift can be deployed on top of a full-fledged Linux OS or on a Micro OS which is specifically designed to run containers and Kubernetes".*

#### Feature

- With OpenShift, we can deploy containerized applications. With application images and QuickStart application templates, applications can be deployed with one click.
- As OpenShift uses Kubernetes, we get all the features offered by Kubernetes, like adding or removing nodes at runtime, persistent storage, auto-scaling, etc.
- OpenShift has a framework called *Source to Image* (*S2I*), which enables us to create container images from the source code repository to deploy applications easily.
- OpenShift integrates well with Continuous Deployment tools to deploy applications as part of the CI/CD pipeline.
- With CLI, GUI and IDE integration applications can be managed easily.
- It enables application portability, meaning that any application created on OpenShift can run on any platform that supports Docker. OpenShift integrated Docker registry, automatic edge load balancing, cluster logging, and integrated metrics.

### Heroku

#### Concept

*“[Heroku](http://www.heroku.com/) is a fully-managed container-based cloud platform, with integrated data services and a strong ecosystem. Heroku is used to deploy and run modern applications”.*

- Applications should contain the source code, its dependency information and the list of named commands to be executed to deploy it, in a file called **Procfile.**
- For each supported language, it has a pre-built image which contains a compiler for that language. This pre-built image is referred to as a [buildpack](https://devcenter.heroku.com/articles/buildpacks). [Multiple buildpacks](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app) can be used together. We can also create a custom buildpack.
- While deploying, we need to send the application's content to Heroku, either via Git, GitHub, Dropbox or via an API. Once the application is received by Heroku, a buildpack is selected based on the language of preference.
- To create the runtime which is ready for execution, we compile the application after fetching its dependency and configuration variables on the selected buildpack. This runtime is often referred to as a slug.
- We can also use third party [add-ons](https://elements.heroku.com/addons) to get access to value-added services like logging, caching, monitoring, etc.
- A combination of slug, configuration variables, and add-ons is referred to as a release, on which we can perform upgrade or rollback.
- Depending on the process-type declaration in the **Procfile**, a virtualized UNIX container is created to serve the process in an isolated environment, which can be scaled up or down, based on the requirements. Each virtualized UNIX container is referred to as a dyno. Each dyno gets its own ephemeral storage.
- Dyno Manager manages dynos across all applications running on Heroku.

#### Feature

- Individual components of an application can be scaled up or down using dynos.
- It is a very rich ecosystem, and it allows us to extend the functionality of an application with [add-ons](https://elements.heroku.com/addons). Add-ons allow us to easily integrate our applications with other fully-managed cloud services like database, logging, email, etc.
- Applications can be easily integrated with Salesforce.
- It enables a development-friendly workflow.
- It provides the ability to do automated backups.