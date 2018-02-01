---
layout: post
title: Microservices and Docker with .Net Core and Azure Service Fabric - Part One
comments: true
excerpt: The first time I listened about Microservices I was impressed about the concept and even more when I saw the microservices in action, it was a kind of love at first sight, but a complicated one, because it was a pretty complex topic (even now). By that time, I had spent some time studying DDD (Domain Driven Design), and for me, it was incredible that a book wrote in 2003 (more than the book the topic itself because Eric Evans created a new architectural style. -A lot of people consider DDD an architectural pattern, but for me is more than that, because DDD touches a lot of edges that simply a specific problem-) would have so importance, similarities and would fit so well (from domain side) with a “modern” architecture such as Microservices. I know that the Microservices concept (or at least the core ideas) comes from many years ago when Carl Hewitt in the early 70’s started to talk about his Actors Model and even later when SOA architecture had solved a lot of problems in the distributed systems, even when a lot of people say “Microservices is SOA well done”. Maybe is right (I don't think so), but the truth is that concepts such as redundant implementation (scale out), service registry, discoverability, orchestration and much more which are the building block of Microservices, comes from SOA.
keywords: "asp.net core, docker, docker compose, linux, C#, c-sharp, DDD, .net core, dot net core, .net core 2.0, dot net core 2.0, .netcore2.0, asp.net, entity framework, domain driven design, CQRS, command and query responsibility segregation, azure, microsoft azure, azure service fabric, service fabric, cosmos db, mongodb, sql server, rabbitmq, rabbit mq, amqp, asp.net web api, azure service bus, service bus"
---

The first time I listened about Microservices I was impressed about the concept and even more when I saw the microservices in action, it was a kind of love at first sight, but a complicated one, because it was a pretty complex topic (even now). By that time, I had spent some time studying DDD (Domain Driven Design), and for me, it was incredible that a [book](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) wrote in 2003 (more than the book the topic itself because Eric Evans created a new architectural style. ~~A lot of people think DDD is an architectural pattern, but for me is more than that, because DDD touches a lot of edges that simply a specific problem~~) would have so importance, similarities and would fit so well (from the domain side) with a “modern” architecture such as Microservices. I know that the Microservices concept (or at least the core ideas) comes from many years ago when [Carl Hewitt](https://en.wikipedia.org/wiki/Carl_Hewitt) in the early 70’s started to talk about his [Actors Model](https://en.wikipedia.org/wiki/Actor_model) and even later when [SOA](https://en.wikipedia.org/wiki/Service-oriented_architecture) architecture had solved a lot of problems in the distributed systems; even when a lot of people say *“Microservices is SOA well done”*. Maybe is right (I don't think so), but the truth is that concepts such as *redundant implementation (scale out)*, *service registry*, *discoverability*, *orchestration* and much more which are the building block of Microservices, comes from SOA.

So, after that, I decided to study the Microservices basis in order to understand its origin and then I got a SOA Architecture certification (that’s not the important thing, it was the journey) and I managed to learn deeper and understand how SOA architecture has helped along from these last years to "evolve" what today we know like Microservices (and finally understand why a bunch of people say *“Microservices is SOA well done”*). Later, after an SOA conscientious study, I learned a lot of things related with Microservices, but I put my eye especially on [CQRS](https://martinfowler.com/bliki/CQRS.html) (I strongly recommend you read this [book](https://msdn.microsoft.com/en-us/library/jj554200.aspx)), which is an architectural pattern that combined with [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html) are a very useful and powerful tool when we're gonna work with Microservices.

So this time, I would like to show you in several posts how to build microservices using [.Net Core](https://dotnet.github.io/) and [Docker](https://www.docker.com/) applying DDD, CQRS and other architectural/design patterns, and finally how to use [Azure Service Fabric](https://azure.microsoft.com/en-us/services/service-fabric/) to deploy our microservices. At the end, I just wanna tell you what was my focused in the Microservices journey and how I started to study it and how I put that knowledge in practice, I just wanna encourage you to jump in in the microservices world and learn a lot of cool things related with this challenging but awesome world.

> The scope of these series of posts won't explain how DDD and CQRS work, I'm just gonna explain how they both can help into a Microservices architecture and how to implement them in this kind of architecture. On the other hand, I highly recommend you to read the ***[Eric Evans](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)*** and ***[Vaughn Vernon](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577/)***'s books if you wanna learn more about DDD and, the ***[CQRS Journey](https://msdn.microsoft.com/en-us/library/jj554200.aspx)*** book if you wanna learn more about CQRS.

I'm gonna start highlighting the most important benefits to work with microservices and on the other hand, the great challenges that bring this approach in order to we stay aware of when and why we can use it. Also, I'm gonna explain how DDD and CQRS can help when we're working with microservices and finally how Docker containers is a great option to isolate our microservices and how its isolation can help us a lot in a development environment and when we need to deploy in our production environments, in this case, with help of Azure Service Fabric as Orchestrator to manage our microservices. So, at the end of the day, I'll walk you through in an introduction to microservices with a practical example that we're gonna develop and deploy in these series of posts, Let's get start it!

## What are Microservices?

In a nutshell, Microservices architecture is an approach to build small, autonomous, independent and resilient services running in its own process. Each service must implement a specific responsibility in the domain, it means a microservice can't mix domain/business responsibilities, because it is autonomous and independent, so in the end, each microservice has its own database.

### Benefits

#### Resiliency:
When a single microservice fails for whatever reason (service is down, the node was restarted/shut down or another temporal error), it won't break the whole application, instead, another microservice could respond to that fail request and "do the work" for the instance with error. (It's like if would have a friend that helps you when you're in troubles) So, is important to implement techniques in order to enable resiliency and manage the unexpected failures, such as circuit-breaking, latency-aware, load balancing, service discovery, retries, etc. (Much of these techniques are already implemented by the orchestrators)

#### Scalability:
Each microservice can scale out independently, so, you don't need to scale the whole system (unlike the monolithic applications), instead, you can scale out only the microservices that you need when you need. In the end, it allows you to saving in costs because you're gonna need less hardware.

#### Data isolation:
Because every microservice has its own database is much easier scale out the database or data layer, and changes related with a data structure or even data, have less impact because the changes only affect one part of the system (one microservice), making the database more maintainable and helping with the governability. Also, it allows you have a polyglot persistence system and choose a more suitable database depending on the needs of your microservice.

#### Small and interdisciplinary teams:
Due to each microservice is small and has a single responsibility in terms of domain and business, every microservice could have a small team, since it doesn't share the code nor database, so is easier to make a change or add a new feature because it hasn't dependencies whit other microservices or another system component. Besides thanks to the small team, it promotes the agility. Also, the team is interdisciplinary since every microservice has a single responsibility it allows every microservice has a domain expert or more depending on the bounded context complexity.

#### Mix of technologies:
Thanks to every single team is enough small and independent, it allows to have a rich microservices ecosystem because you could have a team working with .Net Core for one microservice and another one working with NodeJS for a different microservice and it doesn't matter because none of the microservice depends on each other.

#### Long-term agility:
Since microservices are autonomous, they are deployed independently, so that makes easier to manage the releases or bug fixes, unlike monolithic applications where any bug could block the whole release process while the team have to wait for the bug is fixed, integrated, tested and published, even though when the bug isn't related to the new feature. So, you can update a service without redeploying the whole application or roll back an update if something goes wrong.

### Challenges

#### Choosing right size:
When you design a microservice you need to think carefully about its purpose and responsibility in order to build a consistent and autonomous microservice, so it not should be too big nor too small. DDD is a great approach to design your microservices (it’s not mandatory nor a golden hammer, but in this case we gonna use it to design our system) because DDD helps you to keep your domain decouple and consistent, so if you already know something about DDD, you probably know that a [Bounded Context](https://martinfowler.com/bliki/BoundedContext.html) is a great candidate to be a microservice. At the end, the key point is choosing the right service boundaries for your microservices, independently if you use DDD or not.

#### Complexity:
Unlike monolithic applications where you deal only with one big piece, in a microservices architecture you have to deal with a bunch of pieces (services), so, while in a monolithic application one business operation (or business capability) could interact with one service (or even none) in a microservices architecture one business operation could interact with a lot of services, so you need to manage a lot of things, such as: communication between client and microservices, communication between microservices, coordination, handling errors, compensating transactions and so on. Also, microservices requires more effort in governability stuff, like continuous integration and delivery (CI/CD).

#### Queries:
Since every microservice has its own database you couldn't simply make a query joining tables, because, for instance, you can´t access a customer information from the invoice microservice or even from the client, or even something more complicated, you could have different kinds of databases (SqlServer, MongoDB, ElasticSearch, etc) for every microservice. So, in this case, we gonna use [CQRS](http://microservices.io/patterns/data/cqrs.html) to figure it out.

#### Data consistency and integrity:
One of the biggest challenges in microservices is to keep the data consistent, because as you already know every microservice manage its own data. So, if you need to keep a transaction along multiples microservices you couldn't use an ACID transaction because your data is distributed in several databases. So, one of the common and good solutions is to implement the [Compensating Transaction pattern](http://soapatterns.org/design_patterns/compensating_service_transaction). On the other hand, other common approaches like distributed transactions are not a good idea in a microservices architecture because many modern (NoSQL) databases don’t support it, also it is a blocking protocol and commonly relies on third-party product vendors like Oracle, IBM, etc. Lastly one of the biggest considerations about distributed transactions is the [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem) that states that in a distributed data store is impossible to guarantee consistency and availability at the same time, so you need to choose one of them and pay off. In other words, the CAP theorem means if you're using a blocking strategy like ACID or 2PC transactions you're not being available (for the time the resources are blocking) even if you're using compensating transactions you´re not being consistent because of the delay of the undo operations along the involved microservices, so in the end, as I said, you need to choose and pay off.

#### Communication:
As I said earlier since you have a lot of small services, the communication between the client and different microservices could a headache and pretty complex task, so there are several and common solutions such as an [API Getaway](http://microservices.io/patterns/apigateway.html), [service mesh](https://buoyant.io/2017/04/25/whats-a-service-mesh-and-why-do-i-need-one/) or a [reverse proxy](https://www.nginx.com/resources/glossary/reverse-proxy-server/).

Finally that now we know what are microservices, its advantages and challenges, I'm gonna propose a handy problem and we gonna see how a microservice architecture can help us, and we gonna develop a solution based on the earlier concepts, and at the end of these series of posts we should able to see a microservices solution working and solved the problem proposed.

## The problem

DUber is a public transport company that join drivers with users that required a taxi service in order to move them from one place to another through an App that allows them to request a service using their current location and picking up the destination on a map. The main problems that DUber is facing at this time are:

* DUber became a pretty popular application and it's used by millions of people, but currently, it has scaling problems due to its monolithic architecture.
* In the rush hours the DUber's services collapse because of the system can't support the big ammount of requests.
* DUber is facing problems tracking all about the trip, since it starts until it ends. So user and driver aren't aware, for instance when a service is canceled or the driver is on the way, etc.
* Since the current architecture is a monolithic one and the team is very big, the release process in DUber takes a long time, especially for bugs because until to release the fix, is necessary to test and redeploy the whole application.
* Sometimes the development team lose a lot of time setting the development environment up due to the dependencies and even in the QA and production environments there are errors like: "I don't know why, but in my local machine works like a charm"

As you can see DUber is facing problems related to scalability, disponibility, agility and tracking business objects/workflows. So, we gonna tackling those problems with a Microservice architecture helped by DDD, CQRS, Docker and Azure Service Fabric mainly, but first, we gonna start analyzing the problem making a business domain model helped by DDD.

### Business domain model

Here is when DDD comes into play to help us into a architecture based on Microservices. Before to understand the problem the first thing is to understand the business, the domain, so, after that, you will be able to make a domain model, which is a high-level overview to the system. It helps to organize the domain knowledge and provides a common language for developers and domain experts which Eric Evans called [ubiquitous language](https://martinfowler.com/bliki/UbiquitousLanguage.html). The main idea is mapping all of the business functions and their connections which is a task that involves domain experts, software architects and other stakeholders.

<figure>
  <img src="{{ '/images/Duber_Business_Domain.png' | prepend: site.baseurl }}" alt=""> 
  <figcaption>Fig1. - Business Domain Model</figcaption>
</figure>

After that analysis you can notice that there are five main components and how is the relation between them:

* **Trip**:  is the heart of the system, it’s because is placed in the center of the diagram.
* **Driver**: It's part of the system core because enables the Trip functionality.
* **User**: It's part of the system core as well and manage all information related with the user.
* **Invoicing**: takes care of pricing and coordinates the payment. 
* **Payment**: it's an external system which makes the payment itself.

### Bounded Contexts

This diagram represents the boundaries within the domain, how they are related to each other and identifies easily the subsystems into the whole domain, which ones could be a microservices in our system since a [bounded context](https://martinfowler.com/bliki/BoundedContext.html) marks the boundary of a particular domain model and as we already know a microservice only has one particular responsibility, so the functionality in a microservice should not span more than one bounded context. If you find that a microservice mixes different domain models together, that's a sign that is something wrong with your domain analysis and you may need to go back and refine it.

<figure>
  <img src="{{ '/images/Duber_Bounded_Contexts.png' | prepend: site.baseurl }}" alt=""> 
  <figcaption>Fig2. - Bounded Contexts</figcaption>
</figure>

As you can see there are five bounded contexts there (one external system between them), so, they are candidates to be microservices, but not necessarily every bounded context has to be it, it depends on the problem and your needs, so in this case and based on the problem proposed earlier, we're gonna pick *Trip* and *Invoicing* bounded contexts up in order to they will be our microservices for this problem, since as you already know, the problem here is related with the scalability and disponibility around the trips.

### Classes model

This is a very simple abstraction just to model this problem in a very basic but useful way, in order to applies DDD in our solution, is because of that you will see things like ***[aggregates](https://martinfowler.com/bliki/DDD_Aggregate.html)***, ***[entities and value objects](https://martinfowler.com/bliki/EvansClassification.html)*** in the next diagram. Notice that there isn't nothing about the external system, but it doesn't mean that you not should worries about to model it, in this case, is just for the example propose, but to deal with that, we're gonna use a pattern that Eric Evans called[ ***Anti-corruption layer***](https://docs.microsoft.com/en-us/azure/architecture/patterns/anti-corruption-layer).

<figure>
  <img src="{{ '/images/Duber_Business_Classes_Model.png' | prepend: site.baseurl }}" alt=""> 
  <figcaption>Fig3. - Classes model</figcaption>
</figure>

At this point we have spent a lot of time understanding the problem and designing the solution, that’s good and we always need to spend the enough time in this phase. Usually at this point we haven’t made any decisions about implementation or technologies (beyond what I have tell you about Docker and Azure Service Fabric), so in the next post we’re gonna propose the architecture and we’re gonna make some decisions about technologies and implementation, so stay tune because the next posts gonna be really interesting!