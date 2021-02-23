---
title: \#Cloud Computing\# Introduction to Cloud Infrastructure Technologies(8) Microservices
date: 2019-07-17 16:13:21
categories: Cloud Computing
tags:
- Microservices
thumbnail: /images/cloud_computing.png
---



**Welcome to LFS151x: Introduction to Cloud Infrastructure Technologies**

By the end of this chapter, you should be able to:

- Explain the concept of microservices.
- Discuss the benefits and challenges of using microservices.



<!-- more -->

## Background

Over the years, with different experiments, we evolved towards a new approach, in which a single application is deployed and managed via a small set of services. Each service runs its own process and communicates with other services via lightweight mechanisms like REST APIs. Each of these services is independently deployed and managed. Technologies like containers and unikernels are becoming default choices for creating such services.

![Monoliths-Microservices](/images/Monoliths-Microservices.png)

## Concept

*“Microservices are small, independent processes that communicate with each other to form complex applications which utilize language-agnostic APIs. These services are small building blocks, highly decoupled and focused on doing a small task, facilitating a modular approach tosystem-building. The microservices architectural style is becoming the standard for building continuously deployed systems.”*

## Approaches

- If you have a complex monolith application, then it is not advisable to rewrite the entire application from scratch. Instead, you should start carving out services from the monolith, which implement the desired functionalities for the code we take out from the monolith. Over time, all or most functionalities will be implemented in the microservices architecture.
- We can split the monoliths based on the business logic, front-end (presentation), and data access. In the microservices architecture it is recommended to have a local database for individual services. And, if the services need to access the database from other services, then we can implement an event-driven communication between these services. 
- As mentioned earlier, we can split the monolith based on the modules of the monolith application and each time we do it, our monolith shrinks. 