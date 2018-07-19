---
title: microservices
date: 2018-07-18 21:22:54
categories:
- Microservices
tags:
- Microservices
---
# What is Microservices

A microservice is an isolated, loosely-coupled unit of development that works on a single concern.

* Decomposable
* Autonomous
* Scalable
* Communicable

As a possible macro strategy, split your application using these three guidelines:

* Split services by capabilities
* Try to keep subdomains on a single service
* Prepare for scale, but don't scale while there's no need to

# Benefits

* Freedom to pick the right tool: Is that new library or development platform something you always wanted to use? You can (if it's the right tool for the job).
* Quick iteration: Was the first version suboptimal? No problem, version 2 can be out the door in no time. Because microservices tend to be small, changes can be implemented relatively quickly.
* Rewrites are a possibility: In contrast with monolithic solutions, since microservices are small, rewrites are a possibility. Was the technology stack the wrong pick? No problem, switch to the right alternative.
* Code quality and readability: Isolated development units tend to be of higher quality and new developers can get up to speed with the existing code fairly easily.

# Prerequisites for Microservices

* Domain Maturity: Know enough about your domain to decompose it cleanly
* Organizational Maturity: Small teams with well-defined areas of responsibility
* Process Maturity: Automated Testing and Continuous Delivery
* Operational Maturity: Monitoring, alerting, oncall

# Production-quality microservices

* Cross-cutting concerns must be implemented in a way such that microservices need not deal with details regarding problems outside their specific scope. For instance, authentication can be implemented as part of any API gateway or proxy.
* Data sharing is hard. Microservices tend to favor per-service or per-group databases that can be updated directly. When doing data modeling for your application, notice whether this way of doing things fits your application. For sharing data between databases, it may be necessary to implement an internal process that handles internal updates and transactions between databases. It is possible to share a single database between many microservices; just keep in mind that this may limit your options when and if you need to scale in the future.
* Availability: Microservices, by virtue of being isolated and independent, need to be monitored to detect failures as early as possible. In a big software stack, one service that goes down may go unnoticed for some time. Account for this when picking your software stack for managing services.
* Evolution: Microservices tend to evolve fast. When dedicated teams deal with specific concerns, new and better solutions are found quickly. Therefore, it is necessary to account for versioning of services. Old versions are usually available as long as there are clients who need to consume data from them. Newer versions are exposed in an application-specific way. For instance, with an HTTP/REST API, the version of the microservice may be part of a custom header, or be embedded in the returned data. Account for this.
* Automated deployment: The whole reason that microservices are so convenient nowadays is that it is so easy to deploy a new service from a completely clean environment. See Heroku, Amazon Web Services, Webtask.io or other PaaS providers. If you are going for your own in-house approach, keep in mind that the complexity of deploying new services or versions of preexisting services is critical to the success of your solution. If deployment is not handled in a convenient, automated way, you risk eventually reaching a level of complexity that outweighs the benefits originally brought about by the approach.
* Interdependencies: Keep them to a minimum. There are different ways of dealing with dependencies between services. We will explore them further later in this blog post series. For now, just keep in mind that dependencies are one of the biggest problems with this approach, so seek ways to keep them to a minimum.
* Transport and data format: Microservices are fit for any transport and data format; however, they are usually exposed publicly through a RESTful API over HTTP. Any data format fit for your information works. HTTP + JSON is very popular these days, but there is nothing stopping you from using protocol-buffers over AMQP, for instance.

# Techniques and Patterns

## API Gateway

Microservices are developed almost in isolation. Cross-cutting concerns are dealt with by upper layers in the software stack. The API gateway is one of those layers.

### Authentication

Most gateways perform some sort of authentication for each request (or series of requests). According to rules that are specific to each service, the gateway either routes the request to the requested microservice(s) or returns an error code (or less information). Most gateways add authentication information to the request when passing it to the microservice behind them. This allows microservices to implement user specific logic whenever required.

### Transport security

Many gateways function as a single entry point for a public API. In such cases, the gateways handle transport security and then dispatch the requests either by using a different secure channel or by removing security constraints that are not necessary inside the internal network. For instance, for a RESTful HTTP API, a gateway may perform "SSL termination": a secure SSL connection is established between the clients and the gateway, and proxied requests are then sent over non-SSL connections to internal services.

### Load-balancing

Under high-load scenarios, gateways can distribute requests among microservice-instances according to custom logic. Each service may have specific scaling limitations. Gateways are designed to balance the load by taking these limitations into account.

### Request-dispatching

Even under normal-load scenarios, gateways can provide custom logic for dispatching requests. In big architectures, internal endpoints are added and removed as teams work or new microservice instances are spawned (due to topology changes, for instance). Gateways may work in tandem with service registration/discovery processes or databases that describe how to dispatch each request. This provides exceptional flexibility to development teams. Additionally, faulty services can be routed to backup or generic services that allow the request to complete rather than fail completely.

### Dependency resolution

As microservices deal with very specific concerns, some microservice-based architectures tend to become "chatty": to perform useful work, many requests need to be sent to many different services. For convenience and performance reasons, gateways may provide facades ("virtual" endpoints) that internally are routed to many different microservices.

### Transport transformations

Microservices are usually developed in isolation and development teams have great flexibility in choosing the development platform. This may result in microservices that return data and use transports that are not convenient for clients on the other side of the gateway. The gateway must perform the necessary transformations so that clients can still communicate with the microservices behind it.

## Service registry

The service registry is a database populated with information on how to dispatch requests to microservice instances. Interactions between the registry and other components can be divided into two groups, each with two subgroups:

1. Interactions between microservices and the registry (registration)
* Self-registration
* Third-party registration

2. Interactions between clients and the registry (discovery)
* Client-side discovery
* Server-side discovery

### Registration

Most microservice-based architectures are in constant evolution. Services go up and down as development teams split, improve, deprecate and do their work. Whenever a service endpoint changes, the registry needs to know about the change. This is what registration is all about: who publishes or updates the information on how to reach each service.

Self-registration forces microservices to interact with the registry by themselves. When a service goes up, it notifies the registry. The same thing happens when the service goes down.

Third-party registration is normally used in the industry. In this case, there is a process or service that manages all other services. This process polls or checks in some way which microservice instances are running and it automatically updates the service registry.

### Discovery

When a client wants to access a service, it must find out where the service is located (and other relevant information to perform the request).

Client-side discovery forces clients to query a discovery service before performing the actual requests.

Server-side discovery makes the API gateway handle the discovery of the right endpoint (or endpoints) for a request. This is normally used in bigger architectures.

## Dependencies and Data Sharing

### Separating concerns

* Loose coupling: which means microservices should be able to be modified without requiring changes in other microservices.
* Problem locality: which means related problems should be grouped together (in other words, if a change requires an update in another part of the system, those parts should be close to each other).

In concrete, loose coupling means microservices should provide clear interfaces that model the data and access patterns related to the data sitting behind them, and they should stick to those interfaces (when changes are necessary, versioning, which we will discuss later, comes into play). Problem locality means concerns and microservices should be grouped according to their problem domain. If an important change is required in a microservice related to billing, it is much more likely other microservices in the same problem domain (billing) will require changes, rather than microservices related to, say, product information. In a sense, developing microservices means drawing clear boundaries between different problem domains, then splitting those problem domains into independent units of work that can be easily managed. It makes much more sense to share data inside a domain boundary if required than share data between unrelated domains.

microservices inside a problem domain is: are these services talking too much with eachother? If so, consider the impact of making them a single service. Microservices should be small, but no smaller than necessary to be convenient. Bam! Data sharing and dependency problems are gone. Of course, the opposite applies: if you find your services getting bigger and bigger to reduce chattiness, then perhaps you should rethink how your data is modeled, or how your problem domains are split. Trying to keep balance is the key.

### Data sharing

#### Static data

Static data is data that is usually read but rarely (if ever) modified. 

* Keep it in a database
* Embed it in the code or share it as a file
* Make it into a service

#### Mutable data

As we have mentioned before, the biggest problem with shared data is what to do when it changes.

##### Shared database

When dealing with shared data across databases (or tables within a database) there are essentially two approaches: transactions and eventual consistency.

Transactions are mechanisms that allow database clients to make sure a series of changes either happen or not. In other words, transactions allow us to guarantee consistency. In the world of distributed systems, there are distributed transactions. There are different ways of implementing distributed transactions, but in general, there is a transaction manager that must be notified when a client wants to start a transaction. Only if the transaction manager (that usually communicates this intention to other clients) allows us to move forward the transaction can be performed. The downside to this approach is that scaling is usually harder. Transactions are useful in the context of small or quick changes.

Eventual consistency deals with the problem of distributed data by allowing inconsistencies for a time. In other words, systems that rely on eventual consistency assume the data will be in an incosistent state at some point and handle the situation by postponing the operation, using the data as-is, or ignoring certain pieces of data. Eventual consistency systems are easier to reason about but not all data models or operations fit its semantics. Eventual consistency is useful in the context of big volumes of data.

##### Another microservice

In this approach rather than allowing microservices to access the database directly, a new microservice is developed. This microservice manages all access to the shared data by the two services.

##### Event/subscription model

In this approach, rather than relying on each service fetching the data, services that make changes to data or that generate data allow other services to subscribe to events. When these events take place, subscribed services receive the notification and make use of the information contained in the event.

##### Data pump model

This is related to the eventual consistency case and the additional microservice case: a microservice handles changes in one part of the system (either by reading from a database, handling events or polling a service) and updates another part of the system with those changes atomically. In esence, data is pumped from one part of the system to the other. A thing to keep in mind: consider the implications of duplicating data across microservices. Remember that duplicated data means changes in one copy of the data create inconsistencies unless updates are performed to each copy. This is useful for cases where big volumes of data need to be analyzed by slow processes (consider the case of data analytics, you need recent copies of the data, but not necessarily the latests changes). For long running pumps, remember that consistency requirements are still important. One way to do this is to read the data from a read-only copy of the database (such as a backup).

### Versioning and failures

An important part of managing dependencies has to do with what happens when a service is updated to fit new requirements or solve a design issue. Other microservices may depend on the semantics of the old version or worse: depend on the way data is modeled in the database. As microservices are developed in isolation, this means a team usually cannot wait for another team to make the necessary changes to a dependent service before going live. The way to solve this is through versioning. All microservices should make it clear what version of a different microservice they require and what version they are. A good way of versioning is through semantic versioning, that is, keeping versions as a set of numbers that make it clear when a breaking change happens (for instance, one number can mean that the API has been modified).

The problem of dependency and changes (versions) rises an interesting question: what if things break when a dependency is modified (in spite of our efforts to use versioning)? Failure. We have discussed this briefly in previous posts in this series and now is good time to remember it: graceful failure is key in a distributed architecture. Things will fail. Services should do whatever is possible to run even when dependencies fail. It is perfectly acceptable to have a fallback service, a local cache or even to return less data than requested. Crashes should be avoided, and all dependencies should be treated as things prone to failure.