---
layout: post
title: Microservices and Doker with .Net Core and Azure Service Fabric - Part two
comments: true
excerpt: In the previous post, we talked about what Microservices are, its basis, its advantages, and its challenges, also we talked about how Domain Driven Design (DDD) and Command and Query Responsibility Segregation (CQRS) come into play in a microservices architecture, and finally we proposed a handy problem to develop and deploy across these series of posts, where we analyzed the domain problem, we identified the bounded contexts and finally we made a pretty simple abstraction in a classes model. Now it’s time to talk about things even more exciting, today we’re going to propose the architecture to solve the problem, exploring and choosing some technologies, patterns and more, to implement our architecture using .Net Core, Doker and Azure Service Fabric mainly.
keywords: "asp.net core, Doker, Doker compose, linux, C#, c-sharp, DDD, .net core, dot net core, .net core 2.0, dot net core 2.0, .netcore2.0, asp.net, entity framework, domain driven design, CQRS, command and query responsibility segregation, azure, microsoft azure, azure service fabric, service fabric, cosmos db, mongodb, sql server, rabbitmq, rabbit mq, amqp, asp.net web api, azure service bus, service bus"
published: false
---

In the [previous post](http://elvanydev.com/Microservices-part1/), we talked about what Microservices are, its basis, its advantages, and its challenges, also we talked about how [Domain Driven Design (DDD)](https://en.wikipedia.org/wiki/Domain-driven_design) and [Command and Query Responsibility Segregation (CQRS)](https://martinfowler.com/bliki/CQRS.html) come into play in a microservices architecture, and finally we proposed a handy problem to develop and deploy across these series of posts, where we analyzed the domain problem, we identified the bounded contexts and finally we made a pretty simple abstraction in a classes model. Now it’s time to talk about things even more exciting, today we’re going to propose the architecture to solve the problem, exploring and choosing some technologies, patterns and more, to implement our architecture using [.Net Core](https://dotnet.github.io/), [Doker](https://www.Doker.com/) and [Azure Service Fabric](https://azure.microsoft.com/en-us/services/service-fabric/) mainly.

I would like starting to explain the architecture focused on development environment first, so I'm going to explain why it could be a good idea having different approaches to different environments (development and production mainly), at least in the way services and dependencies are deployed and how the resources are consumed, because, in the end, the architecture is the same both to development and to production, but you will notice a few slight but too important differences.

## Development Environment Architecture

<figure>
  <img src="{{ '/images/Duber_Development_Environment_Architecture.png' | prepend: site.baseurl }}" alt=""> 
  <figcaption>Fig1. - Development Environment Architecture</figcaption>
</figure>

After you see the above image, you can notice at least one important and interesting thing: all of the components of the system (except the external service, obviously) are contained into one single host (later we're going to explain why), in this case, the developer's one (which is also a Linux host, by the way).

We're going to start describing in a basic way the system components (later we'll detail each of them) and how interacts every component to each other.

* **Duber website:** it's an Asp.Net Core Mvc application and implements the User and Driver bounded context, it means, users and drivers management, service request, user and driver's trips, etc.
* **Duber website database:** it's a SQL Server database and is going to manage user, driver, trip and invoice data (last two tables are going to be a denormalized views to implement the query side of CQRS pattern).
* **Trip API:** it’s an Asp.Net Core Web API application, receives all services request from Duber Website and implements everything related with the trip (Trip bounded context), such as trip creation, trip tracking, etc.
* **Trip API database:** it’s a MongoDB database and will be the ***[Event Store](https://en.wikipedia.org/wiki/Event_store)*** of our Trip Microservice in order to implement the ***[Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)*** pattern.
* **Invoice API:** it's an Asp.Net Core Web API application and takes care of creating the invoice and call the external system to make the payment (Invoicing bounded context).
* **Invoice API database:** it's a SQL Server database and is going to manage the invoice data.
* **Payment service:** it's just a fake service in order to simulate a payment service.

### Why Doker?

I would like starting to talk about [Doker](https://www.Doker.com/), in order to understand why is a key piece of this architecture. First of all, in order to understand how Doker works we need to understand a couple of terms first, such as *Container image* and *Container*.

* **Container image:** A package with all the dependencies and information needed to create a container. An image includes all the dependencies (such as frameworks) plus deployment and execution configuration to be used by a container runtime. Usually, an image derives from multiple base images that are layers stacked on top of each other to form the container’s filesystem. ***An image is immutable once it has been created.***
* **Container:** An instance of a Container image. A container represents the execution of a single application, process, or service. It consists of the contents of a Doker image, an execution environment, and a standard set of instructions. When scaling a service, you create multiple instances of a container from the same image. Or a batch job can create multiple containers from the same image, passing different parameters to each instance.

Having said that, we can understand why one of the biggest benefits to use Doker is ***isolation***, because an image makes the environment (dependencies) the same across different deployments (Dev, QA, staging, production, etc.). This means that you can debug it on your machine and then deploy it to another machine with the same environment ***guaranteed***. So, when using Doker, you will not hear developers say, “It works on my machine, why not in production?” because the packaged Doker application can be executed on any supported Doker environment, and it will run the way it was intended to on all deployment targets, and in the end Docker ***simplifies the deployment process*** eliminating deployment issues caused by missing dependencies when you move between environments.

Another benefit of Doker is ***scalability***. You can scale out quickly by creating new containers, due to a container image instance represents a single process. Doker helps for ***reliability*** as well, for example with the help of an orchestrator (you can do it manually if you don't have any orchestrator) if you have five instances and one fails, the orchestrator will create another container instance to replicate the failed process.

Another benefit that I want to note is that ***Doker Containers are faster*** compared with Virtual Machines due to they share the OS kernel with other containers, so they require far fewer resources because they do not need a full OS, thus they are easy to deploy and they start fast.

Now that we understand a little bit about Doker (or at least the key benefits that it gives us to solve our problem), we can understand our Development Environment Architecture, so, we have six Doker images, (the one for SQL Server is the same for both Invoice Microservice and Duber Website), one image for Duber Website, one for SQL Server, one for Trip Microservice, one for MongoDB, one for Invoice Microservice and one image for RabbitMQ, all of them running inside the developer host (in the next post we're going to see how [Docker Compose](https://docs.docker.com/compose/) and Visual Studio 2017 help us doing that). So, why those amount of Doker images, what is the advantage to use them in a development environment? well, think about this: have you ever have struggled  trying to set up your development environment, have you lost hours or even days doing that? (I did! it's awful), well, for me, there are at least two great advantages with this approach (apart to the isolation), the first one is it helps to ***avoid developers waste time setting up the local environment***, thus it speeds up the onboarding time for a new developer in the team, so, this way you only need cloning the repository and press F5, and that's it! you don't have to install anything on your machine or configure connections or something like that (the only thing you need to install is [Doker CE for Windows](https://store.Doker.com/editions/community/Doker-ce-desktop-windows?tab=description)), that's awesome, I love it!

Another big advantage of this approach is ***saving resources***. This way you don’t need to consume resources for development environment due to all of them are in developer’s machine (in a green-field scenario). So, in the end, you’re saving worth resources, for instance, in Azure or in your own servers. Of course you’re going to need a good machine for developers in order to they can have a good experience working locally, but in the end, we always need a good one!

As I said earlier, all of these images are Linux based on, so, how's this magic happening in a Windows host? well, Doker image containers run natively on Linux and Windows, Windows images run only on Windows hosts and Linux images run only on Linux hosts. So, Doker for Windows uses Hyper-V to run a Linux VM which is the by default Doker host. I'm assuming you're working on a Windows machine, but if don't, you can develop on Linux or macOS as well, for Mac, you must install [Docker for Mac](https://docs.docker.com/docker-for-mac/install/), for Linux you don't need to install anything, so, in the end, the development computer runs a Docker host where Docker images are deployed, including the app and its dependencies. On Linux or macOS, you use a Docker host that is Linux based and can create images only for Linux containers.

> Docker is not mandatory to implement microservices, it's just an approach, actually, microservices does not require the use of any specific technology!

### Why .Net Core?

Is well known that .Net Core is cross-platform and also it has a modular and lightweight architecture that makes it perfect for containers and fits better with microservices philosophy, so I think you should consider .Net Core as the default choice when you're going to create a ***new*** application based on microservices.

So, thanks to .Net Core's modularity, when you create a Docker image, is far ***smaller*** than a one created with .Net Framework, so, when you deploy and start it, is significative ***faster*** due to .Net Framework image is based on Windows Server Core image, which is a lot heavier that Windows Nano Server or Linux images that you use for .Net Core. So, that's a great benefit because in microservices we need to start containers fast and want a small footprint per container to achieve better density or more containers per hardware unit in order to lower our costs.

Additionally, .NET Core is ***cross-platform***, so you can deploy server apps with Linux or Windows container images. However, if you are using the traditional .NET Framework, you can only deploy images based on Windows Server Core.

Also, Visual Studio 2017 has a great support to work with Docker, you can take a look at [this](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/visual-studio-tools-for-docker).

## Production Environment Architecture

<figure>
  <img src="{{ '/images/Duber_Production_Environment_Architecture.png' | prepend: site.baseurl }}" alt=""> 
  <figcaption>Fig2. - Production Environment Architecture</figcaption>
</figure>

Before to talk about why Azure Service Fabric I would like 