---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(11) DevOps and CI_CD
date: 2019-07-20 15:13:21
categories: Cloud Computing
tags:
- CI_CD
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Explain the concept of DevOps.
- Discuss about Continuous Integration and Continuous Deployment.
- Run automated tests using tools like Jenkins, Travis CI, Shippable, and Concourse.
- Understand Cloud Native CI/CD.



<!-- more -->



## Background

Every industry thrives for better quality and faster innovation. The IT industry is no exception and has to address numerous challenges, like the following:

- It must quickly go from business idea to market.
- It must lower the failure rate for new releases.
- It must have a shorter lead time between fixes.
- It must have a faster mean time to recovery.

Over the last decade or so, we gradually shifted from the [Waterfall model](https://en.wikipedia.org/wiki/Waterfall_model) to [Agile software development](https://en.wikipedia.org/wiki/Agile_software_development), in which teams deliver working software in smaller and more frequent increments. In this process, the IT operations (Ops) teams were unintentionally left behind, which put a lot of pressure on them, due to high end-to-end deployment rates. By putting the Ops teams in the loop from the very beginning of the development cycle, we can reduce this burn-out and can take advantage of their expertise in the continuous integration process.

## Concept

### DevOps

*“The collaborative work between Developers and Operations is referred to as DevOps. DevOps is more of a mindset, a way of thinking, versus a set of processes implemented in a specific way.”*

### CI/CD

*“Besides Continuous Integration (CI), DevOps also enables Continuous Deployment(CD), which can be seen as the next step of CI. In CD, we deploy the entire application/software automatically, provided that all the tests' results and conditions have met the expectations.”*

## Jenkins

### Concept

*“[Jenkins](https://jenkins-ci.org/2.0/) is one of the most popular and used tools for doing any kind of automation. It is an open source automation system which can provide Continuous Integration and Continuous Deployment. It is written in Java.”*

### Features

- Jenkins can build Freestyle, Apache Ant and Apache Maven-based projects. We can also extend the functionality of Jenkins, using [plugins](https://wiki.jenkins-ci.org/display/JENKINS/Plugins). Currently, Jenkins supports more than 1000 plugins in different categories, like *Source Code Management, Slave Launchers, Build tools, External tools/site integration*, etc.
- Jenkins also has the functionality to build a pipeline, which allows us to to define an entire application lifecycle. **Pipeline** is most useful for doing Continuous Deployment.
  - **Durable:** Pipelines can survive both planned and unplanned restarts of your Jenkins master.
  - **Pausable**: Pipelines can optionally stop and wait for human input or approval before completing the jobs for which they were built.
  - **Versatile**: Pipelines support complex real-world CD requirements, including the ability to fork or join, loop, and work in parallel with each other.
  - **Extensible**: The Pipeline plugin supports custom extensions to its DSL (domain scripting language) and multiple options for integration with other plugins.

![Jenkins_Pipeline](/images/Jenkins_Pipeline.png)

## Travis CI

### Concept

*“[Travis CI](https://travis-ci.com/) is a hosted, distributed CI solution for projects hosted only on GitHub. ”*

### Features

- Travis CI supports different databases, like MYSQL, RIAK, memcached, etc. We can also use Docker during the build.
- Travis CI supports most languages. To see a detailed list of the languages supported, please take a look at the "[*Language-specific* Guides](https://docs.travis-ci.com/user/language-specific/)" page.
- After running the test, we can deploy the application in many cloud providers, such as Heroku, AWS Codedeploy, Cloud Foundry, OpenShift, etc. A detailed list of providers is available in the "*Supported Providers*" page by Travis CI.

### Commands

To run the test with CI, first we have to link our GitHub account with Travis and select the project (repository) for which we want to run the test. In the project's repository, we have to create a **.travis.yml** file, which defines how our build should be executed step-by-step.

A typical build with Travis consists of two steps:

- **install**: to install any dependency or pre-requisite.
- **script**: to run the build script.

We can also add other optional steps, including the deployment steps. Following are all the build options one can put in a **.travis.yml** file.

```yaml
before_install
install
before_script
script
after_success or after_failure 
OPTIONAL before_deploy
OPTIONAL deploy
OPTIONAL after_deploy
after_script
```

## Shippable

### Concept

*“Shippable is a DevOps Assembly Lines Platform that helps developers and DevOps teams achieve CI/CD and make software releases frequent, predictable, and error-free. We do this by connecting all your DevOps tools and activities into a event-driven, stateful workflow”*

![codeToProdPipelines](/images/codeToProdPipelines.png)

### Features

- One can use Shippable as a SaaS service, install on-premise using Shippable Server or attach their own servers to Shippable Subscription.
- Shippable runs all the builds inside a Docker container, which are called **minions.** Shippable has minions for all the different combinations of programming languages and versions they support. But, if you are already doing development with Docker, you can either use your own image or build a new one, while doing the CI.
- Currently, Shippable supports the following programming languages for CI:
  - C/C++
  - Clojure
  - Go
  - Java
  - Node.js
  - PHP
  - Python
  - Ruby
  - Scala.
- Shippable integrates well with all popular CI/CD and DevOps tools like GitHub, Bitbucket, Docker Hub, JUnit, Kubernetes, Slack, Terraform, etc.

### Commands

To run CI tests with Shippable, we have to create a configuration file inside the project's source code repository which we want to test, called **shippable.yml**.

```yaml
resources:
  - name: docs_repo
    type: gitRepo
    integration: ric03uec-github
    pointer:
      sourceName: shippable/docs
      branch: master

  - name: aws_rc_cli
    type: cliConfig
    integration: aws_rc_access
    pointer:
      region: us-east-1

  - name: aws_prod_cli
    type: cliConfig
    integration: aws_prod_access
    pointer:
      region: us-west-2

  jobs:
  - name: publish_prod_docs
    type: runSh
    steps:
      - IN: docs_repo
        switch: off
      - IN: aws_prod_cli
      - TASK:
        - script: |
            pushd $(shipctl get_resource_state "docs_repo")
              ./deployDocs.sh s3://docs.shippable.com us-west-2 production
            popd

  - name: publish_rc_docs
    type: runSh
    steps:
      - IN: docs_repo
      - IN: aws_rc_cli
      - TASK:
        - script: |
            pushd $(shipctl get_resource_state "docs_repo")
              ./deployDocs.sh s3://rcdocs.shippable.com us-east-1 rc
            popd
```

## Concourse

### Concept

*“Concourse is an open source CI/CD system. With Concourse, we run series of tasks to perform desired operations. Each task runs inside a container. Using series of tasks along with resources, we can can build a job or pipeline.”*

### Features

- In Concourse, the necessary data to run the pipeline can be provided by resources. These resources never affect the performance of a worker.

### Commands

Following is a sample task file:

```yaml
platform: linux

image_resource:
  type: docker-image
  source: {repository: busybox}

run:
  path: echo
  args: [hello world]
```

Concourse is primarily driven via a CLI, which is referred to as *fly*. We can use *fly* to login to our Concourse setup, execute tasks, etc.

## Cloud Native CI/CD

###  Background

So far, we have seen that containers are now playing a major role in an application's lifecycle, from packaging to deployment. Containers have brought Dev, QA and Ops teams together, which led to a significant improvement in CI/CD. We can code our CI/CD pipeline and put it along the source code, like we've seen in the earlier sections.

We have also seen running containers at scale using container orchestrators. There are many options for orchestrators, but Kubernetes seems to be the preferred one.

### Concept

*“In the Cloud Native approach, we design a package and run applications on top of our infrastructure (on-premise or cloud) using operations tools like containers, container orchestration and services like continuous integration, logging, monitoring, etc. Kubernetes along with the tooling around it meets our requirements to run Cloud Native applications.”*

### Components

Let's now look at some of the tools which can help us in doing CI/CD with Kubernetes for Cloud Native applications.

- **Helm**

  It is the package manager for Kubernetes. Packages are referred as Charts. Using Helm, we can package, share, install or upgrade an application. It was recently incubated as a CNCF project.

- **Draft**

  It is being promoted as a developer tool cloud native application on Kubernetes. With Draft, we can containerize and deploy an application on Kubernetes.

- **Skaffold**

  It is a tool from Google that helps us build and deploy the code to the Kubernetes development environment each time the code changes locally. It also supports *Helm.*

- **Argo**

  It is a container-native workflow engine for Kubernetes. Its use cases include running Machine Learning, Data Processing and CI/CD tasks on Kubernetes.

- **Jenkins X**

  Jenkins has been a very popular tool for doing CI/CD and it can be used on Kubernetes as well. But the Jenkins team built a new cloud native CI/CD, Jenkins X from the ground up, to run directly on Kubernetes.

- **Spinnaker**

  It is an open source multi-cloud continuous delivery platform from Netflix for releasing software changes with high velocity. It supports all the major cloud providers like Amazon Web Services, Azure, Google Cloud Platform and OpenStack. It supports Kubernetes natively.

## GitOps and SecOps

### Concept

#### GitOps

“With GitOps, it is extended to the entire system. Git becomes the single point of truth for code, deployment configuration, monitoring rules, etc.. With just a Git pull request, we can do/update the complete application deployment. Some of the examples of GitOps are Weave Flux and GitKube.”

#### SecOps

“With SecOps, security would not be an afterthought when it comes tp application deployment. It would be included along with the application deployment, to avoid any security risks.”