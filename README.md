# Distributed systems topics

**This repo is work in progress**

This is a list of topics and resources related to distributed systems, system design, microservices, scalability and performance, etc

- [Distributed systems topics](#distributed-systems-topics)
  - [Distributed systems integration](#distributed-systems-integration)
    - [Synchronous communication](#synchronous-communication)
    - [Asynchronous communication](#asynchronous-communication)
    - [Data serialization](#data-serialization)
    - [Api Gateway](#api-gateway)
  - [Scalability](#scalability)
    - [Performance and availability](#performance-and-availability)
      - [Creating redundancy](#creating-redundancy)
      - [Autoscaling](#autoscaling)
      - [Load balancing](#load-balancing)
        - [Layer 4 load balancing](#layer-4-load-balancing)
        - [Layer 7 load balancing](#layer-7-load-balancing)
    - [Databases](#databases)
      - [Data replication](#data-replication)
      - [Data partitioning](#data-partitioning)
      - [Data Denormalization](#data-denormalization)
      - [Materialized views](#materialized-views)
      - [CQRS](#cqrs)
    - [Caching](#caching)
      - [Caching strategies](#caching-strategies)
      - [Distributed caches](#distributed-caches)
      - [CDNs](#cdns)
    - [Coupling](#coupling)
      - [Location coupling](#location-coupling)
      - [Temporal coupling](#temporal-coupling)
        - [Message queues](#message-queues)
      - [Business data and logic coupling](#business-data-and-logic-coupling)
      - [Shared-nothing architecture](#shared-nothing-architecture)
      - [Selective data replication](#selective-data-replication)
      - [Event-driven architecture](#event-driven-architecture)
        - [Event-driven end-to-end](#event-driven-end-to-end)
        - [Designing around dataflow](#designing-around-dataflow)
  - [Consistency](#consistency)
    - [Different views on data](#different-views-on-data)
    - [Distributed transactions](#distributed-transactions)
      - [Two-phase commit (2PC)](#two-phase-commit-2pc)
      - [Sagas](#sagas)
        - [Orchestration](#orchestration)
        - [Choreography](#choreography)
        - [Process manager](#process-manager)
    - [Dual write problems](#dual-write-problems)
      - [The Outbox Pattern](#the-outbox-pattern)
      - [Retrying failed steps](#retrying-failed-steps)
    - [Change Data Capture](#change-data-capture)
    - [Idempotent consumer](#idempotent-consumer)
    - [Lamport timestamps](#lamport-timestamps)
    - [Conflict-free replicated data type (CRDT)](#conflict-free-replicated-data-type-crdt)
    - [Consensus algorithms](#consensus-algorithms)
      - [Byzantine generals problem](#byzantine-generals-problem)
      - [Raft](#raft)
      - [Paxos](#paxos)
  - [Other topics/patterns](#other-topicspatterns)
    - [Event Sourcing](#event-sourcing)
      - [Projections](#projections)
    - [Local-first software](#local-first-software)
    - [Consistent Hashing](#consistent-hashing)
      - [Hash ring](#hash-ring)
    - [Peer-to-peer](#peer-to-peer)
      - [Gossip protocol](#gossip-protocol)
    - [OSI Model](#osi-model)
  - [More resources](#more-resources)
    - [Books](#books)
    - [Articles](#articles)
    - [Blogs](#blogs)
    - [Websites](#websites)
    - [Github Repositories](#github-repositories)
    - [Videos](#videos)

## Distributed systems integration

### Synchronous communication

In synchronous communication a client depends on an answer. In this form of communication the client sends a request and waits for a response before continuing.

Examples of a synchronous communication are [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call) principles such as [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol), [gRPC](https://en.wikipedia.org/wiki/GRPC), [Apache Thrift](https://en.wikipedia.org/wiki/Apache_Thrift) etc. Also [RabbitMQ supports RPC pattern](https://www.rabbitmq.com/tutorials/tutorial-six-javascript.html).

Synchronous calls can be a good choice for:

- Communication between your client (like web UI / mobile app) and your backend server (HTTP).
- Communication with internal shared/common services, like email service or a service to process some data (RPC or HTTP calls).
- Communication with external third-party services.

Synchronous calls between distributed systems is not always a good idea. It creates coupling, system becomes fragile and slow. For example, in microservices world synchronous communication between microservices can transform them to a distributed monolith. Try to avoid that as much as possible.

### Asynchronous communication

In async communication, client or message sender doesn't wait for a response. It just sends a message which is stored until the receiver takes it (fire and forget).

Examples: message brokers developer on top of [AMQP protocol](https://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol) (like [RabbitMQ](https://www.rabbitmq.com/)) or event streams like [Kafka](https://en.wikipedia.org/wiki/Apache_Kafka).

When changes occur, you need some way to reconcile changes across the different systems. One solution is eventual consistency and event-driven communication based on asynchronous messaging.

Ideally, you should try to minimize the communication between the distributed systems. But it is not always possible.
In microservices world, prefer asynchronous communication since it is better suited for communication between microservices than synchronous. The goal of each microservice is to be autonomous. If one of the microservices is down that shouldn't affect other microservices. Asynchronous protocols help with that. This topic is discussed more in details below in [coupling](#coupling) section.

Read more:

- [Asynchronous message-based communication](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/asynchronous-message-based-communication)
- [RabbitMQ publish/subscribe pattern](https://www.rabbitmq.com/tutorials/tutorial-three-javascript.html)
- [Microservice architecture with RabbitMQ, why?](https://larionov.pro/en/articles/2019/msa-rabbitmq-why/)

### Data serialization

We need to address the issue of multi-language microservices communication. A possible solution is to describe our data models in some language-agnostic way so we can generate the same models for each language. This requires the use of an IDL, or [interface definition language](https://en.wikipedia.org/wiki/Interface_description_language).

One of the most common formats for that lately is [JSON](https://en.wikipedia.org/wiki/JSON). It is simple and human readable. For most systems out there this format is a great choice.

But when your system is growing, there are better choices.

[Protocol buffers](https://en.wikipedia.org/wiki/Protocol_Buffers), [Apache Avro](https://en.wikipedia.org/wiki/Apache_Avro), [Apache Thrift](https://en.wikipedia.org/wiki/Apache_Thrift) provide tools for cross platform and language neutral data formats to serialize data. Those formats are useful for services that communicate over a network or for storing data.

Some pros for using a schema and serializing data include:

- Messages are serialized to binary format which is compact (binary messages occupy about twice less storage than JSON)
- Provides interoperability with other languages (language agnostic).
- Provides forward and backward-compatible schema for your messages/events

Read more:

- [Why Use gRPC and Thrift for Remote Procedure Calls](https://dzone.com/articles/why-use-grpc-and-thrift-for-remote-procedure-calls?fromrel=true)
- [The best serialization strategy for Event Sourcing](https://blog.softwaremill.com/the-best-serialization-strategy-for-event-sourcing-9321c299632b)

### Api Gateway

Your clients need to communicate to your system somehow. Here is where Api Gateway comes in.

> An API gateway is an API management tool that sits between a client and a collection of backend services. An API gateway acts as a reverse proxy to accept all application programming interface (API) calls, aggregate the various services required to fulfill them, and return the appropriate result.
> <small>Quote from [What does an API gateway do?](https://www.redhat.com/en/topics/api/what-does-an-api-gateway-do#:~:text=An%20API%20gateway%20is%20an,and%20return%20the%20appropriate%20result.)</small>

You can implement your own API Gateway if needed, or you could use existing solutions like [Kong](https://konghq.com/kong/), [nginx](https://www.nginx.com/) can be used as API Gateway etc.

Api Gateway can also handle:

- [Load balancing](#load-balancing) to evenly distribute load across your nodes
- [Rate limiting](https://github.com/Sairyss/backend-best-practices#rate-limiting) to protect from [DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack) and [Brute Force](https://en.wikipedia.org/wiki/Brute-force_attack) attacks
- [Authentication](https://en.wikipedia.org/wiki/Authentication) and [Authorization](https://en.wikipedia.org/wiki/Authorization) to allow only trusted clients to access your APIs
- [Circuit Breaker](https://microservices.io/patterns/reliability/circuit-breaker.html) to prevent an application from repeatedly trying to execute an operation that's likely to fail
- Compose data from multiple microservices for your client
- Analytics and monitoring for your API calls
- and much more

Read more:

- [Pattern: API Gateway / Backends for Frontends](https://microservices.io/patterns/apigateway.html)
- [Building Microservices: Using an API Gateway](https://www.nginx.com/blog/building-microservices-using-an-api-gateway/)
- [My experiences with API gateways](https://mahesh-mahadevan.medium.com/my-experiences-with-api-gateways-8a93ad17c4c4)
- [The API gateway pattern versus the Direct client-to-microservice communication](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/direct-client-to-microservice-communication-versus-the-api-gateway-pattern)

## Scalability

Scalability is the capability to handle the increased workload by repeatedly applying a cost-effective strategy for extending a system’s capacity.

**Vertical scaling** means scaling by adding more power (CPU, RAM, Storage, etc.) to your existing servers.

**Horizontal scaling** means adding more servers into your pool of resources.

Below are discussed some of the problems with scalability and patterns to solve those problems.

### Performance and availability

**[Performance](https://en.wikipedia.org/wiki/Computer_performance)** is the amount of useful work accomplished by a computer system at reasonable time.

**[Availability](https://www.techtarget.com/searchdatacenter/definition/high-availability)** is often quantified by uptime (or downtime) as a percentage of time the service is available. For example, around 8 hours of downtime per year may be considered as a 99.9% available system.

Below we will discuss some techniques that can help with system performance and availability.

#### Creating redundancy

[Redundancy](<https://en.wikipedia.org/wiki/Redundancy_(engineering)>) means duplication of critical components like data, nodes and running processes, etc.
Redundancy can improve flexibility, reliability, availability, scalability and performance of your system, failure resistance and recovery. Below we discuss techniques and patterns for creating redundancy.

#### Autoscaling

[Autoscaling](https://en.wikipedia.org/wiki/Autoscaling) is a method used in cloud computing that dynamically adjusts the amount of computational resources in a server based on the load.

Read more:

- [What is Autoscaling? How Does It Work In the Cloud – Simply Explained](https://www.scaleyourapp.com/what-is-autoscaling-how-does-it-work-in-the-cloud-simply-explained/)
- [Kubernetes Autoscaling: 3 Methods and How to Make Them Great](https://spot.io/resources/kubernetes-autoscaling-3-methods-and-how-to-make-them-great/)

#### Load balancing

To distribute traffic between multiple instances/nodes, you will need load balancing tools.
[Load balancing](<https://en.wikipedia.org/wiki/Load_balancing_(computing)>) is a technique to distribute network or application traffic across a number of servers.

##### Layer 4 load balancing

Layer 4 load balancers operate at the transport level and make their routing decisions based on address information extracted from the first few packets in the TCP stream and do not inspect packet content.

##### Layer 7 load balancing

Level 7 load balancing deals with the actual content of each message enabling the load balancer to make smarter load‑balancing decisions, can apply optimizations and changes to the content (such as compression and encryption) and make decisions based on the message content. Layer 7 gives a lot of interesting benefits, but is less performant.

**Note**: Layers 4 and 7 are a part of [OSI Model](#osi-model).

[Kong](https://konghq.com/kong/), [nginx](https://www.nginx.com/), [kubernetes](https://kubernetes.io/) include load balancing capabilities.

Read more:

- [System Design — Load Balancing](https://medium.com/must-know-computer-science/system-design-load-balancing-1c2e7675fc27)
- [What Is Layer 4 Load Balancing?](https://www.nginx.com/resources/glossary/layer-4-load-balancing/)
- [What Is Layer 7 Load Balancing?](https://www.nginx.com/resources/glossary/layer-7-load-balancing/)

### Databases

#### Data replication

[Data replication](<https://en.wikipedia.org/wiki/Replication_(computing)>) is when the same data is intentionally stored in more than one node.

Having multiple copies of data can improve read performance, availability, prevent disasters in cases when one of the databases loses it's data.

Read more:

- [Read Consistency with Database Replicas](https://shopify.engineering/read-consistency-database-replicas)

#### Data partitioning

[Partitioning](<https://en.wikipedia.org/wiki/Partition_(database)>) is a division of a database into distinct independent parts. When you have big amounts of data that don't fit into one database partitioning is a logical next step to improve database performance.

Read more:

- [Data partitioning guidance](https://docs.microsoft.com/en-us/azure/architecture/best-practices/data-partitioning)

#### Data Denormalization

[Denormalization](https://en.wikipedia.org/wiki/Denormalization) is the process of trying to improve the read performance of a database by sacrificing some write performance.

Read more:

- [Denormalization in Databases](https://www.geeksforgeeks.org/denormalization-in-databases/)
- [Building robust distributed systems](https://kislayverma.com/software-architecture/building-robust-distributed-systems/)

#### Materialized views

[Materialized view](https://en.wikipedia.org/wiki/Materialized_view) is a pre-computed data set derived from a query specification and stored for later use.

Materialized view can be just a result of a `SELECT` statement:

```sql
CREATE MATERIALIZED VIEW my_view AS SELECT * FROM some_table;
```

Result of this `SELECT` statement will be stored as a view so every time you query for it there is no need to compute the result again.

Materialized views can improve performance, but data may be outdated since refreshing a view can be an expensive operation that is only done periodically (for example once a day). This is great of some things like daily statistics, but not good enough for real time systems.

Read more:

- [Working with Materialized Views](https://docs.snowflake.com/en/user-guide/views-materialized.html#:~:text=A%20materialized%20view%20is%20a,base%20table%20of%20the%20view.)

#### CQRS

CQRS stands for Command and Query Responsibility Segregation.

When using CQRS we write data to a database that provides fast writing speeds, and create a copy of that data in a database that is optimized for reads. Commands are forwarded to a write database, queries are forwarded to a read database. This can improve read and write performance of your application.

CQRS is a great fit when used together with [Event Sourcing](#event-sourcing).

Read more:

- [CQRS pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/cqrs)

### Caching

A [cache](<https://en.wikipedia.org/wiki/Cache_(computing)>) is a high-speed data storage layer which stores a subset of data, typically temporary, so that future requests for that data are served up faster than by accessing the data’s primary storage.

For caching you can use tools like [Redis](https://redis.io/), [Memcached](https://memcached.org/), or other key-value stores.

#### Caching strategies

There are several caching strategies worth knowing. Most common are:

- Cache-aside (lazy loading)
- Write-through
- Write-behind
- Refresh-ahead

Read more:

- [Database Caching Strategies Using Redis](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/database-challenges.html)
- [Architecture Patterns: Caching (Part-1)](https://kislayverma.com/software-architecture/architecture-patterns-caching-part-1/)

#### Distributed caches

A [distributed cache](https://en.wikipedia.org/wiki/Distributed_cache#:~:text=In%20computing%2C%20a%20distributed%20cache,database%20and%20web%20session%20data.) is an extension of the cache from a single server to multiple servers. A distributed cache can improve the performance and scalability of the system.

Read more:

- [Distributed cache system design](https://medium.com/system-design-concepts/distributed-cache-system-design-9560f7dd07f2)
- [Architecture Patterns: Caching (Part-2)](https://kislayverma.com/software-architecture/architecture-patterns-caching-part-2/)

#### CDNs

[Content delivery network](https://en.wikipedia.org/wiki/Content_delivery_network) is geographically distributed group of servers which work together to provide fast delivery of Internet content.

A CDN allows for the quick transfer of assets needed for loading Internet content including HTML pages, javascript files, stylesheets, images, and videos.

CDN can increase content availability, redundancy, improve website load times.

Read more:

- [What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/)

### Coupling

One reason why distributed systems are so hard to design is coupling. One of the ways you can scale your system is by reducing it.
Below are some topics and techniques that can help prevent coupling.

#### Location coupling

When a program assumes that something is available at a known location, we can call that a Location coupling.
For example, when one service knows the location of another service (lets say knows its URL).

It is difficult to horizontally scale location coupled systems. Location can change, or there may be multiple locations if there are multiple nodes running. Also systems can assume that accessing those locations may be fast, but usually in distributed systems a remote location can be on a entirely different server in a different country, thus a network trip to may be slow.

Location coupling is a problem when considering horizontal scalability because it directly prevents resources from being added dynamically at different locations.

To break location coupling you can abstract the specifics of accessing another part of the system. Your application should be agnostic to the called system.

This can be implemented differently in different scenarios:

- At the network layer, we can use DNS to mask the specific IP addresses of remote servers
- Load balancers can hide that there are multiple instances of some particular system are running to service high workloads
- Clients can hide the details of sharded database clusters
- In application that consists of microservices, your services shouldn't know the location of each other. Instead, you can publish your commands/messages/events to a common place (like a message broker) and let other services pick it up. Now all of your services will know about a common location (message broker), but not about each other. In other words, prefer [Asynchronous communication](#asynchronous-communication).

Abstracting the specifics of accessing another part of the system makes application depend only on that abstraction instead of keeping bunch of locations it needs. This lets application scale better. For example, this lets you dynamically add more services and instances to the network when you need it (during peak hours) without changing anything.

#### Temporal coupling

Temporal coupling usually happens in synchronously communicated systems (a part of a system expects all other parts on which it depends to serve its needs instantaneously). If one part fails, all its dependent systems will also fail, creating a chain of failures.

##### Message queues

The most common approach to breaking temporal coupling is the use of [message queues](https://en.wikipedia.org/wiki/Message_queue). Instead of invoking other parts of a system synchronously (calling and waiting for the response), the caller instead puts a request on a queue and the other system consumes this request when it is available.

A message queue is a form of [asynchronous communication](#asynchronous-communication) used in some distributed systems (like microservices). Messages are stored on the queue until they are processed. Each message should be processed only once, by a single consumer.

Examples of a queue/brokers include [RabbitMQ](https://www.rabbitmq.com/) and [Kafka](https://en.wikipedia.org/wiki/Apache_Kafka).

Read more:

- [System Design — Message Queues](https://medium.com/must-know-computer-science/system-design-message-queues-245612428a22)

#### Business data and logic coupling

To be autonomous, each component of a system must own its domain data and logic (this is called data ownership).

Try not to share business data between components. Otherwise you may end up with a distributed monolith where every change can impact and break the whole system. This is especially relevant in microservices world.

> Regardless of the method we use to share data across service boundaries we’ll end up with services that are not autonomous. Autonomous services can evolve independently from each other, non-autonomous ones cannot. They are coupled, they won’t be able to evolve without breaking each other, causing evolution to stop or causing friction at development time, deployment time, and run time.
>
> As soon as we allow cross-service data sharing, for example by allowing services to request data from another service, we end up with the worst of two worlds: a monolith suddenly turns into a distributed monolith with all distributed system problems on top of a monolithic one.

Exception to this rule:

- Sharing IDs. For example an "order" microservice may need to know what user made an order. In this case you can save a user ID in both services, but not the entire user object with all its properties.
- Some data, lets say email address, can be needed in multiple services, for example authentication service (to login your user) and a marketing service (to send marketing emails). In those cases you may need to duplicate emails between those services. Duplication is better than calling one service from another, but keep in mind that when updating email you'll need to guarantee tis update in both services. We discuss this further in a section below.

Your components must be independently deployable. In a microservices world, if deploying one microservice you have to change other microservices this means you have a tightly coupled architecture.

#### Shared-nothing architecture

[Shared-Nothing Architecture](https://en.wikipedia.org/wiki/Shared-nothing_architecture) is a distributed computing architecture that consists of multiple separated nodes that don’t share resources.

This architecture reduces coupling, scales better, simplifies upgrades preventing downtime, eliminates single points of failure, allowing the overall system to continue operating despite failures in individual nodes, etc.

Read more:

- [Shared Nothing Architecture Explained](https://phoenixnap.com/kb/shared-nothing-architecture)

#### Selective data replication

Selective data replication is creating a copy of the data needed from other remote system (external api, microservice etc) into the database of our system.

This approach reduces coupling, runtime dependency, improve latency and makes system more scalable and reliable. Even if external source of data is unaccessible, you still have this data replicated in your own database.

Data that is frequently accessed but changes regularly can be cached temporarily with periodic cache refreshes. Data that changes less frequently or never can be stored in our system's database directly.

Read more:

- [Querying data across microservices](https://medium.com/@john_freeman/querying-data-across-microservices-8d7a4667668a)

#### Event-driven architecture

An event is a broadcast by a software system about something which has happened within its boundary. For example, when the system successfully create an order, it can publish an `ORDER_CREATED` event.

Using asynchronous communication and event-driven architecture ensures loose-coupled components in a system.

TODO

Read more:

- [Why Microservices Should Be Event Driven: Autonomy vs Authority](https://www.google.com/search?client=firefox-b-d&q=consistent+hashing)
- [Using Events to build evolutionary architectures](https://kislayverma.com/software-architecture/using-events-to-build-evolutionary-architectures/)

##### Event-driven end-to-end

In typical web pages, when you load a page and data on the server changes afterwards, the browser does not know about this change until you reload the page. The state displayed on a page is stale cache that is not updated until you explicitly poll or query for changes.

More recent protocols have moved beyond simple request/response pattern. [WebSockets](https://en.wikipedia.org/wiki/WebSocket) provide communication channels which allow your browser to establish a TCP connection to your server. Your server can push messages to your browser as long as it remains connected. This provides an ability for the server to actively inform end-users about any state changes on the server.

Recent tools for developing client applications, like React, Redux, Flux etc. already manage client-side state by subscribing to a stream of changes that represent responses from a server. It would be very natural to extend this model to also allow server to push state-changing events to a client. This way changes and events could flow end-to-end, from the interaction of one device that triggers an event, to the server and event bus, to processing this event, storing it in a database, and back to the UI client that is observing the state changes on another device.

The idea of subscribing for changes instead of querying opens a bunch of new possibilities for more responsive user interfaces. It is already used in some places, like online games or messaging apps.

For typical data centric or CRUD services there may be not a lot of benefit designing an entire application using this approach, but some parts of your application can definitely benefit from it.

Read more:

- [6 Event-Driven Architecture Patterns — Part 1](https://medium.com/wix-engineering/6-event-driven-architecture-patterns-part-1-93758b253f47)

##### Designing around dataflow

Some microservices example:

- In regular approach, when purchasing a product, your microservice can query an exchange rate service to get currency exchange rate.
- In a dataflow approach, the code that processes purchases would subscribe to a stream of exchange rate updates ahead of time and record the current rate in a local database whenever it changes. When processing a purchase it only needs to query its local database ([Selective data replication](#selective-data-replication)).

Same as changing a single number in a spreadsheet column will change the output of a formula that depends on this value.

> Subscribing to a stream of changes, rather than querying the current state when needed, brings up closer to a spredsheet-like model of computation: when some piece of data changes, any derived data that depends on it can swiftly be updated. - <small>Martin Kleppman</small>

By combining techniques like [Event-driven Architecture](#event-driven-architecture) and [Selective Data Replication](#selective-data-replication) we can achieve interesting results.

Read more:

- [Designing Data-Intensive Applications](https://dataintensive.net/) chapter: "Designing applications around dataflow"

## Consistency

Large applications typically use data that is spread across multiple data stores. Managing and maintaining data consistency in this environment is an important aspect of the system.

- **Weak consistency** - After a write, there is no guarantee that reads will see it. Works well in real time use cases such as voice or video chat, and realtime multiplayer games, etc.
- **Strong consistency** - After a write, there is a guarantee that reads will se it. Data is replicated synchronously, usually using some kind of [transaction](https://en.wikipedia.org/wiki/Database_transaction).
- **Eventual consistency** - After a write, readers will eventually see written data (usually within seconds) and data is replicated asynchronously. Eventual consistency works best for distributed systems at a large scale.

Below we will discuss where and why we need consistency, how to achieve it and some associated problems.

### Different views on data

Sometimes there is no single piece of software that satisfies all your requirements. One database may not be enough for all your needs. Application wants to view data as [OLTP](https://en.wikipedia.org/wiki/Online_transaction_processing), analytics team want to view it as [OLAP](https://en.wikipedia.org/wiki/Online_transaction_processing). With high loads, you may also need to give users an ability to access data very fast using [cache](<https://en.wikipedia.org/wiki/Cache_(computing)>) or [full-text search](https://en.wikipedia.org/wiki/Full-text_search). There also may be benefit in creating connections between different data using a [graph database](https://en.wikipedia.org/wiki/Graph_database).

Typically you may need to use:

- Relational (SQL) databases like [MySQL](https://www.mysql.com/), [PostgreSQL](https://www.postgresql.org/) etc
- Non-relational (NoSQL) databases like [MongoDB](https://www.mongodb.com/), [Couchbase](https://www.couchbase.com/) etc
- Graph databases for data with complex relations like [Neo4j](https://neo4j.com/developer/graph-database/), [dgraph](https://dgraph.io/), [ArangoDB](https://www.arangodb.com/) etc.
- For caching you can use [Redis](https://redis.io/) or [Memcached](https://memcached.org/)
- Search engines like [Elasticsearch](https://www.elastic.co/), [Solr](https://solr.apache.org/) etc.

When there is no single storage solution that satisfies all your needs, you may end up composing your services out of multiple software solutions that work together tied by your code. Same data may end up in multiple different storages to satisfy everyone's needs while querying, thus creating a lot of redundancy and data duplication, but providing performance benefits and ability to analyze and query data from different perspectives.

Keeping writes to several storage systems in sync is necessary. Just writing to multiple databases without any additional layer that ensures consistency is a naive approach that can lead to dual-write problems and inconsistencies. We will discuss techniques and patterns that can help reliably replicate data to multiple storages in the next sections.

Read more:

- [Online analytical processing (OLAP)](https://docs.microsoft.com/en-us/azure/architecture/data-guide/relational-data/online-analytical-processing)
- [Data Consistency Primer](<https://docs.microsoft.com/en-us/previous-versions/msp-n-p/dn589800(v=pandp.10)>)

### Distributed transactions

Another reason why distributed systems are hard is consistency problems.

In a typical non-distributed application, you can use database transactions to achieve strong consistency pretty easily. But in a distributed world, when databases can be on a different servers, simple transactions won't work anymore so we need to find workarounds. And with those workarounds we usually can only achieve weak or eventual consistency, because strong consistency usually means low performance and coupling, which is unacceptable when you distribute.

#### Two-phase commit (2PC)

[Two-phase commit (2PC)](https://en.wikipedia.org/wiki/Two-phase_commit_protocol) is one of the ways how to achieve strong consistency in a distributed world. Using 2PC, each server participating in a transaction starts a local transaction, and they all commit only when other participants finish executing the transaction.

2PC can guarantee strong consistency, but it is rarely used because:

- It is blocking. If the coordinator fails permanently, some participants will never resolve their transactions
- It is slow and affects performance of participating systems
- It creates coupling between systems

Read more:

- [Two Phase Commit](https://martinfowler.com/articles/patterns-of-distributed-systems/two-phase-commit.html)

#### Sagas

In modern distributed systems, eventual consistency is preferred because it scales better. But how do we achieve it?

One of the patterns for this is Sagas. A saga is a sequence of local transactions, but unlike 2PC, each transaction commits immediately, without waiting for other participants to finish. This way, all transactions will be committed asynchronously(eventually). Each participant must have a compensating mechanism to revert a saga, so if one of the participants fails, saga should start a revert process to undo all the changes.

Read more:

- [Pattern: Saga](https://microservices.io/patterns/data/saga.html)
- [Saga distributed transactions](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)

##### Orchestration

Orchestration is a centralized approach. Parts of the systems that are participating in a workflow are controlled by a single entity called Orchestrator.

Orchestrator is responsible for executing each step in a flow and ensuring consistency.

##### Choreography

Choreography is a decentralized approach. Unlike orchestration, when using choreography there is no central place that coordinates workflows. Each part of the system is responsible for subscribing to events it needs. This way data just flows through the system from one part of the system to another.

Choreography is flexible and doesn't create much coupling (unlike Orchestration), but it can be hard to follow. When you have complex workflows with a lot of steps, it will be hard to track everything that is happening across the system. One event may trigger another one, then another one, and so on. To track the entire workflow you'll have to go multiple places and search for an event handler for each step, which is hard to maintain. In this case, using a orchestration might be a preferred approach compared to choreography since you will have an entire workflow in one place. This might create some coupling, but is easier to maintain. A lot of systems use both orchestration and choreography depending on the use case, you don't necessarily need to choose just one. Pick the right tools for the job.

Read more:

- [Microservice Orchestration Vs Choreography](https://softobiz.com/microservice-orchestration-vs-choreography/)

##### Process manager

[Process manager](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html) is used to maintain the state of the sequence and determine the next processing step based on intermediate results.

Process manager is very similar to Orchestrator. In fact, a lot of people use the terms like "Saga" and "Orchestration" when in reality they are talking about the process manager.

Process managers are state machines that store some state, while sagas usually don't have a state and operate only on data that they receive in events.

- [Saga vs. Process Manager](https://blog.devarchive.net/2015/11/saga-vs-process-manager.html?m=1)

### Dual write problems

A dual write describes the situation when you change data in 2 systems, e.g., a database and Apache Kafka, without an additional layer that ensures data consistency over both services.

For example:

```javascript
await database.save(user);
await eventBus.publish(userCreatedEvent);
```

Something can happen in between those operations (a network lag, process restart, event broker crash etc.) so the second operation will fail resulting in a loss of data. This problem is called "dual writes problem". Dual writes are unsafe and can lead to data inconsistencies - data in one microservice is saved, but not indexed (1st example) or other microservices are not notified since the event was lost (2nd example). This scenario can happen if you have multiple asynchronous operations executing one by one in a single place, so you should design your system to execute only one such operation at a time, or have some process that guarantees consistency.

Some of the techniques for to avoid dual writes are described below.

Read more:

- [Dual Writes – The Unknown Cause of Data Inconsistencies](https://thorben-janssen.com/dual-writes/)

#### The Outbox Pattern

Subscribing to an event from a different part of the system and creating your own view of that event (saving it to the database in a way that your system needs) is one of the ways we introduce redundancy. But saving some data and publishing an event that notifies other parts of the system about what happened can face dual write problems.

_The Outbox Pattern_ can help with consistency when writing data to a database and publishing an event to a message broker.

What you have to do is save your message/event as a part of database transaction, together with a result of an operation that is being executed. For example, if you are creating a user and sending an event that notifies other services that a user was created, you should save a user object and a message that you want to publish in a single database transaction. User goes to a "user" table and a message goes to an "outbox" or "messages" table. After that a separate Message Relay process publishes the events inserted into database to a message broker. You can use tools like [Debezium](https://debezium.io/) that publish changes from your "outbox" table to kafka, or create a custom process that checks database every X seconds and sends pending events.

This way we can be sure that no message/event is lost and will be published eventually, even if a message broker is down or a network is lagging. This will guarantee consistency of your data across multiple services and reliability of your application overall.

**Note**: when using [Event Sourcing](#event-sourcing) you may not need an Outbox Pattern since all the changes are stored as events anyway.

Read more:

- [Pattern: Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html)
- [The Outbox Pattern](http://www.kamilgrzybek.com/design/the-outbox-pattern/)

#### Retrying failed steps

Another way to prevent dual write problems is by retrying. For example, create a job that makes sure data is eventually consistent in a case of failure.

Consider this example: you need to save some data in your local database and index it in an external API

```typescript
await database.save(movie);
await externalAPI.index(movie);
```

What if your application crashes in the middle of this operation, or if external API is unavailable? A movie will be saved to a local database, but not indexed in an external API.

One of the ways to solve this is to have a periodic job that queries the database every X minutes and retries indexing movies that were not indexed.

You can see this implemented in code in this repo: [Fullstack application example](https://github.com/Sairyss/full-stack-application-example/tree/master/apps/api/src/app/modules) - check out a movie and indexer services. Movie is saved as `indexed: false` by default and indexer service has a `@Interval()` decorator meaning that it is executed periodically to make sure that if something failed during indexing it will be retried later (meaning it is eventually consistent).

One important detail: when retrying make sure your consumers are [idempotent](#idempotent-consumer).

### Change Data Capture

[Change data capture (CDC)](https://en.wikipedia.org/wiki/Change_data_capture) is a pattern that tracks changes to data in a database and provides real-time movement of data allowing to process it continuously as new database events occur.

CDC is one of the techniques that can help with data consistency when you deal with multiple storages.

Databases usually append changes to its tables/documents to a [log data structure](https://scaling.dev/storage/log) to keep track of what's happening (usually called change logs, commit logs or [transaction logs](https://docs.microsoft.com/en-us/sql/relational-databases/logs/the-transaction-log-sql-server?view=sql-server-ver15)). We can utilize that log to duplicate changes from a main database to other databases in a consistent manner (because simply saving data to multiple databases at once is not safe and leads to inconsistency).

When data crosses the boundary between different technologies, asynchronous event log with [idempotent writes](#idempotent-consumer) is reliable and robust solution.

Read more:

- [Change Data Capture (CDC): What it is and How it Works](https://www.striim.com/change-data-capture-cdc-what-it-is-and-how-it-works/)
- [Domain Events versus Change Data Capture](https://kislayverma.com/software-architecture/domain-events-versus-change-data-capture/)

### Idempotent consumer

Messages, Commands and Events can be delivered twice (or multiple times), for example, when consuming an event from a message broker, an acknowledgement can fail for some reason or event is retried due to a timeout. It is important to process those events only **once**, otherwise your data may end up corrupted or some unwanted actions can occur in the system (for example you could change your user's credit card twice, or send the same email multiple times etc). We need some mechanism for event de-duplication.

To prevent processing events twice you should make your event consumers [idempotent](https://en.wikipedia.org/wiki/Idempotence).

Some actions are by definition idempotent. For example, updating a username twice with the same name will end up in the same result. But some actions are not idempotent, like charging money from a wallet.

One of the most used ways to make consumers idempotent is saving some kind of an idempotency token (uuid generated by a user, or an ID of the event) in the database, in the same atomic transaction used to save changes caused by this event and set a unique constraint on that id. This way, even if an event/message is received multiple times, it will be processed only once.

Read more:

- [Pattern: Idempotent Consumer](https://microservices.io/patterns/communication-style/idempotent-consumer.html)
- [Idempotent receiver](https://martinfowler.com/articles/patterns-of-distributed-systems/idempotent-receiver.html)

### Lamport timestamps

[Lamport timestamp](https://en.wikipedia.org/wiki/Lamport_timestamp) algorithm is a simple logical clock algorithm used to determine the order of events in a distributed computer system.

TODO

Read more:

- [Lamport Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/lamport-clock.html)

### Conflict-free replicated data type (CRDT)

[Conflict-free replicated data type (CRDT)](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) is a data structure which can be replicated across multiple computers in a network, where the replicas can be updated independently and concurrently without coordination between the replicas, and where it is always mathematically possible to resolve inconsistencies that might come up.

Read more:

- [crdt.tech](https://crdt.tech/)
- [Automerge](https://github.com/automerge/automerge) - JavaScript library of data structures based on CRDTs for building collaborative applications

### Consensus algorithms

#### Byzantine generals problem

[Byzantine generals problem](https://en.wikipedia.org/wiki/Byzantine_fault)

TODO

#### Raft

[Raft](<https://en.wikipedia.org/wiki/Raft_(algorithm)>)

TODO

Read more:

- [The Raft Consensus Algorithm](https://raft.github.io/)

#### Paxos

[Paxos](<https://en.wikipedia.org/wiki/Paxos_(computer_science)>)

TODO

## Other topics/patterns

### Event Sourcing

When using Event sourcing your data is saved as a sequence of immutable events.
This is valuable when auditing is needed, for example when dealing with money transactions.

TODO

Read more:

- [Event sourcing pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)
- [Event immutability and dealing with change](https://www.eventstore.com/blog/event-immutability-and-dealing-with-change)

#### Projections

Projections are usually used in Event Sourced systems to create a "view" from a stream of events by reducing those events to a single value (imagine JS reduce function `[event1, event2, event3].reduce(...)`). This is essentially a [left-fold](<https://en.wikipedia.org/wiki/Fold_(higher-order_function)>) operation in the functional world.

With projections you can create as many views on your data as you want. Projection can be a simple SQL table, a Document in a NoSQL database, a node in a Graph Database, etc. This can satisfy query needs for everyone, since you can't really query much from a stream of events.

Read more:

- [Projections 1: Theory](https://www.eventstore.com/blog/projections-1-theory)

### Local-first software

TODO

Read more:

- [Local-first software](https://www.inkandswitch.com/local-first/)

### Consistent Hashing

[Consistent hashing](https://en.wikipedia.org/wiki/Consistent_hashing)

TODO

Read more:

- [Consistent Hashing](https://medium.com/system-design-blog/consistent-hashing-b9134c8a9062)

#### Hash ring

TODO

Read more:

- [Consistent Hash Rings Explained Simply](https://akshatm.svbtle.com/consistent-hash-rings-theory-and-implementation)

### Peer-to-peer

TODO

#### Gossip protocol

[Gossip protocol](https://en.wikipedia.org/wiki/Gossip_protocol) is a peer-to-peer communication mechanism that allows designing highly efficient distributed communication systems (P2P).

Read more:

- [[YouTube] Parallel & Distributed Computing - Gossip Protocol](https://www.youtube.com/watch?v=qJpPjzg44R8)

### OSI Model

OSI stands for [Open Systems Interconnection](https://en.wikipedia.org/wiki/OSI_model).

You need to know basics of OSI model in order to have a better understanding of some problems in distributed computing (for example Layer 4 and 7 load balancing).

It is a 7 layer architecture with each layer having specific functionality to perform:

1. Physical layer - responsible for the physical connection between the devices
2. Data-link - responsible for the node-to-node delivery of the message
3. Network - works for the transmission of data from one host to the other located in different networks
4. Transport - provides services to the application layer and takes services from the network layer
5. Session - responsible for the establishment of connection, maintenance of sessions, authentication, ensures security
6. Presentation - data from the application layer is extracted here and manipulated as per the required format to transmit over the network
7. Application - this layer is implemented by the network applications

Read more:

- [OSI Model Layers and Protocols in Computer Network](https://www.guru99.com/layers-of-osi-model.html#:~:text=OSI%20model%20is%20a%20layered,mostly%20implemented%20only%20in%20software.)

## More resources

### Books

- [Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321) by Martin Kleppmann (must-read)

### Articles

- [On building scalable systems](https://kislayverma.com/software-architecture/on-scalable-software/)

### Blogs

- [Kislay’s Blog](https://kislayverma.com/)

### Websites

- [Microsoft - Cloud Design Patterns](https://docs.microsoft.com/en-us/azure/architecture/patterns/index-patterns)
- [Patterns of Distributed Systems](https://martinfowler.com/articles/patterns-of-distributed-systems/)
- [microservices.io](https://microservices.io)

### Github Repositories

- [The System Design Primer](https://github.com/donnemartin/system-design-primer)
- [awesome-distributed-systems](https://github.com/theanalyst/awesome-distributed-systems)

### Videos

- [Scaling up to your first 10 million users (YouTube)](https://www.youtube.com/watch?v=kKjm4ehYiMs)
- [Developing microservices with aggregates - Chris Richardson (YouTube)](https://youtu.be/7kX3fs0pWwc)
- [InfoQ Channel (YouTube)](https://www.youtube.com/nctv)
