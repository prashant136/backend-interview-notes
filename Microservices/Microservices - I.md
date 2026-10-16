
## Microservices in your own words ? How is it different from Monolith ? benefits of using MS and Downsides ?

### Imagine you're building an E-commerce website

Features:

- Login
- Product Catalog
- Search
- Cart
- Orders
- Payments
- Notifications

---

### Monolith

Everything is one big application.

```
                Ecommerce Application

     +------------------------------------+
     |                                    |
     | Login                              |
     | Products                           |
     | Search                             |
     | Cart                               |
     | Orders                             |
     | Payments                           |
     | Notifications                      |
     |                                    |
     +------------------------------------+

                One Database
```

One codebase.

One deployment.

One server (or multiple copies of the same app).

---

### Example

Suppose the application contains

```
src

 controllers
 services
 repositories

 LoginService

 ProductService

 CartService

 OrderService

 PaymentService

 NotificationService
```

Everything lives together.

If you deploy,

You deploy ALL services together.

---

### User places an order

```
Browser

     |

Monolith Application

     |

Login

     |

Cart

     |

Order

     |

Payment

     |

Notification

     |

Database
```

Everything happens inside one application.

Method calls are simply

```
orderService.placeOrder();

paymentService.pay();

notificationService.sendEmail();
```

No network call.

Only Java function calls.

Very fast.

---

### Problem starts growing...

Now imagine

10 developers become

100 developers.

Everyone changes the same project.

```
Developer A

Developer B

Developer C

Developer D

Developer E

Developer F

Same Repository
```

Now conflicts begin.

---

Suppose one small bug in Notification.

You changed

```
NotificationService.java
```

Still,

Entire application must be rebuilt.

Entire application must be tested.

Entire application must be deployed.

Even though nothing changed in Payment.

---

### Scaling Problem

Suppose

100 users

```
Login = 100 req/sec

Orders = 30 req/sec

Search = 8000 req/sec
```

Search becomes popular.

Need more servers.

With Monolith

You cannot scale only Search.

You must create

```
Monolith

Copy 1

Copy 2

Copy 3

Copy 4

Copy 5
```

Every copy contains

```
Login
Orders
Payments
Search
Notifications
```

Although only Search needed scaling.

Huge waste of memory and CPU.

---

### Now imagine Microservices

Instead of one big application,

Split it into independent applications.

```
               API Gateway

                     |

---------------------------------------------------------

|        |         |         |        |        |

Login  Product   Search   Order   Payment Notification

Service Service  Service   Service  Service    Service
```

Each is a completely separate application.

Each has

Own code

Own deployment

Own team

Sometimes even its own database.

---

### Difference

#### Monolith

```
Order -----> Payment

(Java Method Call)

Very Fast
```

---

#### Microservice

```
Order

     |

Network

     |

Payment

HTTP/gRPC/Kafka
```

There is network latency.

---

### Independent Deployment

Suppose Payment has a bug.

Monolith

```
Deploy Entire Application
```

Microservice

```
Deploy only Payment Service
```

Everything else keeps running.

---

### Independent Scaling

Traffic

```
Login = 200/sec

Orders = 100/sec

Search = 20,000/sec
```

Only Search is overloaded.

Microservice architecture:

```
Search Service

Pod1

Pod2

Pod3

Pod4

Pod5

Pod6

Pod7
```

Payment remains

```
Payment

Pod1
```

This saves money.

---

### Different Technologies

Suppose

Search requires speed.

Use

```
Java + Elasticsearch
```

Payments need reliability.

Use

```
Java + PostgreSQL
```

Recommendations need AI.

Use

```
Python
```

Notifications need concurrency.

Use

```
Node.js
```

Every service chooses the best technology.

Impossible in Monolith.

---

### Separate Databases

Instead of

```
One Database
```

We may have

```
User Service

↓

User DB

----------------------

Order Service

↓

Order DB

----------------------

Payment Service

↓

Payment DB
```

Each service owns its data.

No other service directly modifies another service's database.

---

### Real Example (Tyroo-like Architecture)

From what you've described at Tyroo:

```
                   API Gateway

                         |

----------------------------------------------------------

Campaign Service

Product Service

Creative Service

User Service

Analytics Service

Notification Service

----------------------------------------------------------

Redis

Kafka

Elasticsearch

PostgreSQL

Kubernetes
```

Each service runs in its own Docker container, and Kubernetes scales them independently. Kafka enables asynchronous communication between services, while Redis provides caching and Elasticsearch powers fast search.

**These services communicate over the network using HTTP/gRPC or asynchronously through Kafka or message queues.**

---

### Advantages of Microservices

#### 1. Independent deployment

Deploy one service without affecting others.

---

#### 2. Independent scaling

Scale only what's under load.

```
Search → 20 Pods

Payment → 2 Pods

Order → 5 Pods
```

---

#### 3. Smaller codebase

Developers work on smaller, focused projects.

---

#### 4. Fault isolation

If Notification crashes,

Orders can still work.

```
Order ✓

Payment ✓

Notification ✗
```

The whole application doesn't go down.

---

#### 5. Team ownership

Different teams can own different services.

```
Team A → Orders

Team B → Payments

Team C → Search
```

Less interference.

---

#### 6. Technology flexibility

Different services can use different programming languages or databases when appropriate.

---

### Downsides of Microservices

#### 1. Increased complexity

Instead of 1 application, you may have 20–100 services to manage.

---

#### 2. Network latency

Service-to-service communication is slower than in-process method calls because it happens over the network.

---

#### 3. Harder debugging

A single user request may pass through multiple services:

```
Gateway

↓

Order

↓

Inventory

↓

Payment

↓

Notification
```

Tracing failures requires distributed tracing and centralized logging.

---

#### 4. Data consistency challenges

Each service owns its own database.

A workflow like:

```
Order

↓

Payment

↓

Inventory
```

can't rely on a single database transaction. Patterns like Saga and event-driven workflows are used to maintain consistency.

---

#### 5. More DevOps effort

You'll typically need infrastructure such as:

- Docker
- Kubernetes
- API Gateway
- Service Discovery
- Load Balancer
- Centralized Logging
- Monitoring
- Distributed Tracing
- CI/CD pipelines

Managing this ecosystem is significantly more complex than deploying a monolith.

---

### When to use what?

|Monolith|Microservices|
|---|---|
|Small team|Large teams|
|Small application|Large application|
|Simple deployment|Independent deployments|
|One database|Multiple service-owned databases|
|Easy to debug|More complex debugging|
|Scale entire app|Scale individual services|
|Fast internal method calls|Network-based communication|
|Easier to develop initially|Better for long-term growth|

#### A practical intuition

A **monolith is like a single large supermarket**. Everything is under one roof. It's simple to start and easy to manage when the business is small. But as it grows, expanding one department often affects the whole building.

A **microservices architecture is like a shopping mall**. Each store (service) is independently owned and operated. One store can renovate, expand, or close temporarily without shutting down the entire mall. The trade-off is that coordinating security, maintenance, and communication between stores is much more complex.

For interviews, a concise summary is:

> **A monolith is a single deployable application where all business modules run together and are usually backed by one database. A microservices architecture splits the application into independently deployable services, each responsible for a specific business capability. This enables independent scaling, deployment, and team ownership, but introduces challenges such as network communication, distributed data management, observability, and operational complexity.**


## 
-----
##

## How do microservices communicate with each other? Synchronous and Asynchronous modes ? REST/HTTPS, SOAP, gRPC, Message queues like RabbitMQ,KafKa etc.

### Synchronous & Asynchronous Communication

Synchronous and asynchronous communication are two core interaction patterns in microservices: synchronous uses direct, blocking request-responses (like HTTP/REST), while asynchronous uses non-blocking message brokers (like Kafka or RabbitMQ).

**Synchronous Communication**

- **How it works**: The sender sends a request and waits (blocks) for an immediate reply from the receiver before moving on.
- **Common protocols**: HTTP, REST, gRPC, and GraphQL.
- **Pros**: Simple to design, easy to debug.
- **Cons**: Tight coupling between services, vulnerable to cascading failures if one service goes down, and lower scaling capacity under heavy load.

**Asynchronous Communication**

- **How it works**: The sender publishes a message to a broker and continues its work instantly without waiting for a reply.
- **Common tools**: Apache Kafka, RabbitMQ, and AWS SQS.
- **Pros**: Loose coupling, high resilience and fault tolerance, and great scalability for background tasks.
- **Cons**: Harder to debug and trace, introduces eventual consistency, and adds infrastructure complexity.

#### Summary

|Synchronous|Asynchronous|
|---|---|
|Caller waits for response|Caller doesn't wait|
|Direct communication|Uses a message broker|
|Easy to understand|More complex|
|Low latency for immediate results|Better throughput and scalability|
|Tightly coupled|Loosely coupled|
|Failures can propagate|Better fault isolation|
|Best for request-response|Best for background processing and events|

-------

### Event Driven Architecture ?

**_event-driven microservice architecture_**, services communicate each-other via event messages. When business events occur, producers publish them with messages. At the same time, other services consume them through event listeners.
Thus, the main benefits of event-driven systems are asynchronous behavior and loosely coupled structures.

----------------

### Fault Tolerance



### Eventual consistency and why is it needed?

Eventual consistency means that while data may not be instantly consistent across services, it will become consistent after some time (eventually). This is especially useful in systems where availability and partition tolerance (from the CAP theorem) take precedence over immediate consistency. _More on CAP theorem in a minute._

#### Example Scenario:

- **OrderService** accepts an order and needs to notify **InventoryService** and **ShippingService**.
- Rather than coordinating a distributed transaction, OrderService publishes events.
- These events propagate to other services, which update their state independently.

#### Why eventual consistency is preferred over strong consistency in some scenario.

In a distributed microservice environment, maintaining **strong consistency** across services can be costly or infeasible due to network partitions, independent data stores, and service failures. Instead, many systems adopt **eventual consistency** — a model in which each service is independently updated and consistency is achieved over time through asynchronous communication.



---

### When should you use Event-Driven Architecture?

EDA is a great fit when:
- Multiple independent services need to react to the same business event.
- Some work can happen asynchronously (emails, notifications, analytics, logging, reporting).
- High throughput and scalability are important.
- Services should evolve independently without constantly changing each other's code.
- Temporary failures in downstream services shouldn't block the user experience.

Avoid EDA when:
- You need an immediate response from another service before continuing (e.g., checking payment authorization before confirming an order).
- A simple synchronous request-response flow is sufficient—introducing events would only add unnecessary complexity.

---

### How it relates to Kafka and Kubernetes

These technologies solve **different problems** and often work together:

```
User
  |
Order Service (Producer)
  |
Publish OrderCreated
  |
Kafka Topic
  |
+-------------------------+
| Inventory Consumer      |
| Email Consumer          |
| Analytics Consumer      |
| Warehouse Consumer      |
+-------------------------+
        |
  Kubernetes
(scales consumer pods)
```

- **Kafka** transports and stores events.
- **Microservices** contain the business logic.
- **Kubernetes** runs and scales the producer and consumer services.
- **Redis** may be used by individual services for caching but is not part of the event delivery mechanism.

### Interview definition

> **Event-Driven Architecture (EDA)** is a software architecture where services communicate by publishing and consuming events instead of calling each other directly. An event represents something that has already happened (such as `OrderCreated` or `PaymentCompleted`). This approach creates loosely coupled systems that are more scalable, resilient, and easier to extend, especially in distributed microservice environments.

-----
------
-----

## What is the role of an Event Bus or Message Broker in EDA ?

An event bus is middleware that enables publish/subscribe communication between microservices.  An Event Bus is software that accepts events from producers and delivers them to interested consumers.

### Responsibilities of an Event Bus

Let's see them one by one.

---

### 1. Receive Events

Producer publishes

```
OrderCreated
```

Event Bus receives it.

It acts like an inbox.

---

### 2. Store Events (sometimes)

Some brokers store events.

Kafka does.

```
Event Bus

OrderCreated
PaymentCompleted
RefundCreated
InvoiceGenerated
```

Kafka keeps them on disk for a configurable retention period.

Why?

Because consumers may be offline.

Later they come back.

Still can read.

---

#### Example

10:00 AM

```
OrderCreated
```

Email service crashes.

11:00 AM

Email comes back.

Kafka still has the event.

Email processes it.

No data lost.

---

### 3. Deliver Events

The broker sends the event to interested consumers.

Producer doesn't know who receives it.

---

### 4. Route Events

Suppose multiple event types exist.

```
OrderCreated

PaymentCompleted

UserRegistered

ProductUpdated
```

The broker routes each event to the right consumers.

Example:

```
OrderCreated

↓

Inventory

Warehouse

Email

Analytics
```

```
PaymentCompleted

↓

Invoice

Accounting
```

Different consumers receive different events.

---

### 5. Buffer Traffic

Imagine **Black Friday**.

Normally:

```
100 orders/min
```

Suddenly:

```
100,000 orders/min
```

Email service can't process everything instantly.

Without a broker:

Requests fail.

With a broker:

```
100,000 Events

↓

Event Bus Queue

↓

Consumers process gradually
```

The broker acts as a buffer.

Nothing is lost.

---

### 6. Retry Failed Messages

Email service crashes.

Broker notices.

Keeps the message.

Email restarts.

Broker delivers again.

```
Email ❌

↓

Broker keeps event

↓

Email restarts

↓

Broker retries
```

Very useful.

---

### 7. Enable Loose Coupling

Producer only knows

```
Publish Event
```

That's all.

Consumers decide

```
I want this event.
```

Neither side depends on the other's implementation.

### Event Bus vs Message Broker

People often use these terms interchangeably, but there's a subtle distinction.

|Event Bus|Message Broker|
|---|---|
|Architectural concept|Software implementation|
|Defines how events flow between services|Implements event delivery|
|Can be built using Kafka, RabbitMQ, SQS, Pub/Sub|Kafka, RabbitMQ, SQS, etc. are actual brokers|

So:

```
Event-Driven Architecture
        │
        ▼
   Event Bus (concept)
        │
        ▼
Message Broker (implementation)
        │
        ▼
Kafka / RabbitMQ / SQS / Pub/Sub
```

---

### Complete Flow

```
             Customer
                 │
                 ▼
          Order Service
                 │
        Publish OrderCreated
                 │
                 ▼
        ┌─────────────────┐
        │   Event Bus     │
        │ (Kafka Broker)  │
        └─────────────────┘
          │    │    │    │
          ▼    ▼    ▼    ▼
     Inventory Email Analytics Warehouse
```

Notice:

- The **Order Service only talks to the Event Bus**.
- Consumers only talk to the Event Bus.
- Producers and consumers never communicate directly.

This separation is what makes Event-Driven Architecture scalable, resilient, and easy to extend.

---

### Interview answer (30 seconds)

> **An Event Bus (implemented by a Message Broker like Kafka or RabbitMQ) is the communication backbone of an Event-Driven Architecture. Producers publish events to the broker instead of calling services directly. The broker receives, stores (if supported), routes, buffers, retries, and delivers events to interested consumers. This decouples producers from consumers, allowing services to evolve, scale, and recover independently.**


-----
------
-----

##  Explain the concept of Event Sourcing and how it differs from traditional CRUD.

Event Sourcing records every change as an event instead of just storing the current state, creating a complete history of data changes. The current state can be rebuilt anytime by replaying these events, helping track how data evolved over time.

will do later....




##
-------
##  

## What are the challenges of implementing distributed transactions in EDA and how do you solve them? Using Saga pattern how can we solve these problems ?



---
### [ Why not use Two Phase Commit (2PC) ? ]

###  ☯️ Two-Phase Commit (2PC) in Distributed Transactions


Before learning the Saga pattern, you must understand **why Two-Phase Commit (2PC) was invented**.

### Interview Line -
> The Two-Phase Commit (2PC) protocol is an atomic commitment algorithm used in distributed systems. It ensures that all participating nodes either successfully commit a transaction or completely roll it back, maintaining strict ACID consistency across multiple databases or services.

Instead of directly committing,

A **Coordinator** controls all participants.

Example:

```
                 Coordinator

             /        |       \

         Payment   Inventory  Shipping
```

Coordinator asks:

> "Can everyone commit?"

Only if **every participant says YES** does it allow the commit.

Otherwise everyone rolls back.

There are two phase to this algorithm: 
- **prepare phase**: asking other nodes whether they can commit the proposed transaction.
- **commit phase**: commanding other nodes to either commit or abort the proposed transaction.

```
Phase 1
Prepare

Phase 2
Commit / Rollback
```

---

### Two-Phase Commit (2PC) Pattern

> The 2PC Pattern is all about updating resources on multiple nodes in a single atomic operation.

In 2PC, it carries out an update in two phases (Figure 02).

1. **Prepare**: Each node, participating in the transaction, whether it is able to carry out an update in the second phase. Once each node is able to ensure it, the coordinator will be notified. If any of the nodes are unable to make it, the coordinator is notified to roll back releasing any locks they have with nodes.
2. **Commit**: Carrying out the update and completing the transaction.

![](https://miro.medium.com/v2/resize:fit:1262/1*kap06STHbYfxhxe1rfrmQg.png)


2PC represents a synchronous strong consistency approach within a distributed transaction.
However, 2PC is not fully recommended for microservices-based applications due to its **synchronous blocking**.


## Saga Pattern -

#### Definition

> A Saga is a sequence of local transactions where each service commits its own database transaction independently.
> 
> If one service fails later, previously completed services execute **compensating transactions** to undo their work.

Notice one important thing. Unlike 2PC, 
**there is NO global transaction.**

---------------

## Compensating Transaction

The **Compensating Transaction pattern** is a mechanism used in distributed systems (like microservices) to reverse operations that have already been committed.

Instead of

```
Do everything
then Commit
```

Saga does

```
Do Step 1
Commit

↓

Do Step 2
Commit

↓

Do Step 3
Commit

↓

Failure?

↓

Undo Step 2

↓

Undo Step 1
```

This undo is called **Compensation Transaction**.

-----

## Saga Types -

### Before Learning Saga Types

Let's first understand **WHY Saga needs two different approaches.**

Imagine you're building **Amazon**.

One order touches multiple services.

```
                 Customer
                     |
                     |
              Place Order
                     |
                     |
    -------------------------------------
    |        |        |        |        |
 Order   Payment  Inventory Shipping Notification
```

Question:

> Who should tell everyone what to do?

There are only **two possibilities**.

#### Option 1

Nobody controls anyone.

Each service reacts when something happens.

Example

```
Order finishes

↓

Payment starts

↓

Inventory starts

↓

Shipping starts
```

This is **Choreography**.

---

#### Option 2

One manager controls everything.

```
Manager

↓

Order

↓

Payment

↓

Inventory

↓

Shipping
```

This is **Orchestration**.

That's it.

These are literally the only two possible coordination models.

---

### Let's use one Real World Scenario throughout

Suppose customer buys

```
MacBook
```

Order amount

```
₹1,50,000
```

Services involved

```
Order Service
Payment Service
Inventory Service
Shipping Service
Notification Service
```

Each service owns its own database.

```
Order DB

Payment DB

Inventory DB

Shipping DB
```

No shared database.

---

### First Type : Choreography Saga

Saga choreography is a decentralized microservices design pattern where distributed transactions are managed via event-driven communication. Instead of a central controller, individual services listen for specific domain events, execute their local transaction, and publish new events to a message broker (e.g., Kafka) to trigger the next step.

#### Intuition

Imagine there is **no manager** in a company.

Employees simply know

> "Whenever Event X happens,  
> I have to perform Task Y."

Nobody gives commands.

Everyone reacts automatically.

---

### Flow

Customer clicks

```
Buy Now
```

Order Service executes

```
Create Order
```

It commits.

Database

```
Order

Status = CREATED
```

Immediately after commit

Order Service publishes

```
OrderCreated Event
```

Notice

It DOES NOT call Payment Service.

It simply says

```
"I created an order."
```

That's all.

---

#### Step-by-Step Flow

```
Customer

     |

Order Service

     |

Create Order

     |

Commit Order DB

     |

Publish Event

OrderCreated
```

---

Payment Service is listening.

```
Kafka

↓

Payment Service
```

Payment receives

```
OrderCreated
```

Now Payment says

```
Okay

I'll charge customer.
```

Payment DB

```
Balance deducted

Status = SUCCESS
```

Commit.

Publish

```
PaymentCompleted
```

---

Inventory Service listens

```
PaymentCompleted
```

Inventory

```
Reserve Stock

↓

Commit

↓

Publish

InventoryReserved
```

---

Shipping listens

```
InventoryReserved
```

Creates shipment.

Publishes

```
ShipmentCreated
```

---

Notification listens

```
ShipmentCreated
```

Sends Email.

Done.

---

### Complete Flow Diagram

```
Customer
    |
    v

+------------------+
| Order Service    |
+------------------+
       |
       | Publish OrderCreated
       v
==========================
      Kafka/Event Bus
==========================
       |
       v

+------------------+
| Payment Service  |
+------------------+
       |
       | Publish PaymentCompleted
       v
==========================
      Kafka/Event Bus
==========================
       |
       v

+--------------------+
| Inventory Service  |
+--------------------+
       |
       | Publish InventoryReserved
       v
==========================
      Kafka/Event Bus
==========================
       |
       v

+------------------+
| Shipping Service |
+------------------+
       |
       | Publish ShipmentCreated
       v
==========================
      Kafka/Event Bus
==========================
       |
       v

+-----------------------+
| Notification Service  |
+-----------------------+
```

Notice

Nobody knows the entire workflow.

Every service only knows

```
Receive Event

↓

Do Work

↓

Publish Event
```


---

### Problem starts...

Imagine after 2 years.

You add

```
Fraud Service

Coupon Service

Tax Service

Invoice Service

Recommendation Service

Analytics Service

Reward Points
```

Now events become

```
OrderCreated

↓

PaymentDone

↓

FraudChecked

↓

RewardAdded

↓

InventoryReserved

↓

InvoiceCreated

↓

CouponApplied

↓

NotificationSent

↓

AnalyticsStored
```

Now interviewer asks

> Who refunds payment?

Answer?

You don't know.

Need to inspect many services.

This is called

> **Event Chain Problem**

or

**Distributed Spaghetti**

> The "Distributed Spaghetti" problem refers to the chaotic, unmaintainable mess of intertwined event flows that occurs when using the **Choreography approach** to the **Saga pattern**. As microservices grow, direct service-to-service event dependencies become incredibly hard to trace, debug, and safely modify

---

### Failure Scenario in Choreography

Suppose

```
Order Created

Payment Success

Inventory Failed
```

Flow

```
Customer

↓

Order Service

↓

OrderCreated

↓

Payment Service

↓

PaymentCompleted

↓

Inventory Service

↓

Stock NOT Available
```

Inventory publishes

```
InventoryFailed
```

Now

Payment Service listens

```
InventoryFailed

↓

Refund Customer

↓

Publish

PaymentRefunded
```

Order Service listens

```
PaymentRefunded

↓

Cancel Order

↓

Publish

OrderCancelled
```

Final Flow

```
OrderCreated

↓

PaymentCompleted

↓

InventoryFailed

↓

PaymentRefunded

↓

OrderCancelled
```

Nobody instructed anyone.

Everyone reacted to events.

---

# How does Choreography ensure Eventual Consistency?

Initially

```
Order = CREATED

Payment = SUCCESS

Inventory = FAILED
```

This is inconsistent.

But events continue.

```
InventoryFailed

↓

Refund Payment

↓

Cancel Order
```

Finally

```
Order = CANCELLED

Payment = REFUNDED

Inventory = AVAILABLE
```

Every service eventually reaches a consistent business state.

Notice

There was a short period where data was inconsistent.

That's why it's called

**Eventually Consistent**

NOT

Immediately Consistent.

```
Eventual consistency prioritizes speed and availability, allowing temporary data mismatches across nodes that sync up later. Immediate consistency prioritizes accuracy, guaranteeing every user sees the exact same, up-to-date data instantly.
```

---

### Now let's learn Orchestration

Imagine a school.

Without principal.

Teachers coordinate themselves.

Messy.

Now imagine

```
Principal

↓

Teacher A

↓

Teacher B

↓

Teacher C
```

Everything becomes organized.

Principal knows entire workflow.

That's Orchestration.

---

### Architecture

```
                 Customer
                     |
                     |
             Saga Orchestrator
                     |
      --------------------------------
      |         |        |          |
   Order     Payment  Inventory Shipping
```

Instead of services talking to each other,

Everything goes through

```
Saga Orchestrator
```

---

### Flow

Customer

```
Place Order
```

Orchestrator receives request.

Step 1

```
Create Order
```

Order replies

```
SUCCESS
```

Orchestrator

```
Charge Payment
```

Payment replies

```
SUCCESS
```

Orchestrator

```
Reserve Inventory
```

Inventory replies

```
FAILED
```

Now Orchestrator immediately knows

```
Inventory failed.
```

It executes compensation.

```
Refund Payment

↓

Cancel Order
```

Done.

---

### Complete Flow Diagram

```
                Customer
                    |
                    |
                    v

         +----------------------+
         | Saga Orchestrator    |
         +----------------------+

                    |
                    | Create Order
                    v

          +------------------+
          | Order Service    |
          +------------------+

             SUCCESS
                |
                v

         +----------------------+
         | Saga Orchestrator    |
         +----------------------+

                |
                | Charge Payment
                v

          +------------------+
          | Payment Service  |
          +------------------+

             SUCCESS
                |
                v

         +----------------------+
         | Saga Orchestrator    |
         +----------------------+

                |
                | Reserve Stock
                v

         +----------------------+
         | Inventory Service    |
         +----------------------+

             FAILED
                |
                v

         +----------------------+
         | Saga Orchestrator    |
         +----------------------+

          Refund Payment

                |

          Cancel Order
```

Notice

The orchestrator knows:

- completed steps,
- current step,
- failed step,
- compensation sequence.

---

### How does Orchestration ensure Eventual Consistency?

Suppose

```
Order Created

Payment Success

Inventory Failed
```

Temporary state

```
Order DB

CREATED

Payment DB

SUCCESS

Inventory DB

FAILED
```

The orchestrator detects the failure and issues compensation commands in reverse order:

```
Inventory Failed

↓

Refund Payment

↓

Cancel Order
```

Eventually the databases converge to a valid business state:

```
Order

CANCELLED

Payment

REFUNDED

Inventory

UNCHANGED
```

Again, the system is temporarily inconsistent, but it automatically repairs itself.

---

### Failure Scenario 1 — Payment Fails

```
Create Order

↓

Charge Payment

↓

FAILED
```

The orchestrator knows only the order was committed, so it compensates only that step:

```
Cancel Order
```

Final state:

```
Order = CANCELLED

Payment = FAILED

Inventory = NOT TOUCHED
```

---

### Failure Scenario 2 — Inventory Fails

```
Create Order ✓

↓

Payment ✓

↓

Inventory ✗
```

Compensation:

```
Refund Payment

↓

Cancel Order
```

Final state:

```
Order = CANCELLED

Payment = REFUNDED

Inventory = AVAILABLE
```

---

### Failure Scenario 3 — Shipping Fails

```
Order ✓

↓

Payment ✓

↓

Inventory ✓

↓

Shipping ✗
```

Reverse compensation order:

```
Release Inventory

↓

Refund Payment

↓

Cancel Order
```

Why reverse order?

Think of stacking plates:

```
1. Create Order
2. Take Money
3. Reserve Stock
```

Undo them from the top:

```
Release Stock

↓

Refund Money

↓

Cancel Order
```

Exactly like unwinding a stack.

---

### [ Failure Scenario 4 — Compensation Itself Fails ]

Remember one golden rule:

> **A Saga is itself a long-running process.**

Unlike a database transaction (which lasts milliseconds),

a Saga may run for

- 5 seconds
- 30 seconds
- even several minutes.

During that time,  
anything can fail.

---

Most people think:

> Inventory failed → Refund Payment

Done.

But what if...

**Refund itself fails?**

This is the real question.

---

#### Example

Customer buys MacBook.

#### Step 1

Order Service

```
Order = CREATED
```

Success.

---

#### Step 2

Payment Service

```
Money Deducted = ₹1,50,000
```

Success.

---

#### Step 3

Inventory

```
Reserve Stock
```

Oops...

```
Stock Not Available
```

Inventory fails.

Now Saga starts compensation.

---

#### Compensation begins

```
Refund Customer
```

But...

Payment Gateway is down.

```
Refund API

↓

Timeout
```

or

```
Refund API

↓

500 Internal Server Error
```

Now what?

Customer money is still deducted.

Order is cancelled.

This is inconsistent.

---

### [ What does Saga do? ]

It retries.

Imagine you're calling someone.

```
Call

↓

Busy

↓

Call Again

↓

Busy

↓

Call Again

↓

Connected
```

Exactly the same.

---

#### Retry Flow

```
Refund

↓

Failed

↓

Wait 5 seconds

↓

Retry

↓

Failed

↓

Wait 10 seconds

↓

Retry

↓

Success
```

This is called

**Retry with Backoff**

because we increase waiting time.

---

#### But what if retry also fails?

Suppose

```
Retry 1 ❌

Retry 2 ❌

Retry 3 ❌

Retry 4 ❌

Retry 5 ❌
```

Still failed.

Now continuously retrying forever is bad.

Why?

Because

```
CPU Waste

Logs Filled

Millions of Retry Requests
```

Not good.

So the message is moved somewhere else.

This place is called

###  ☯️ Dead Letter Queue (DLQ)

---

### What is DLQ?

Think about a courier company.

Most packages deliver successfully.

Some packages have problems.

```
Wrong Address

Phone Switched Off

Customer Not Available
```

Courier company doesn't throw away package.

Instead

they move it to

```
Problem Shelf
```

Later,

an employee checks it.

DLQ is exactly that shelf.

---

### Flow Diagram

```
Refund Request

      |
      v

Payment Service

      |

Refund Failed

      |

Retry

      |

Retry Failed

      |

Retry Failed

      |

Retry Failed

      |

-----------------------
Move Message to DLQ
-----------------------
```

DLQ stores

```
Refund Order #101

Reason:
Payment Gateway Timeout
```

Nothing is lost.

---

#### Then what happens?

Suppose after one hour

Payment Gateway becomes healthy.

Operations team (or another worker) reads DLQ.

```
DLQ

↓

Refund Order #101

↓

Execute Again

↓

Success
```

Problem solved.

---

### Why not simply discard the message?

Imagine Amazon.

If refund message is discarded,

Customer loses ₹1,50,000.

Impossible.

Therefore

Messages are **never silently lost**.

---

### Where does DLQ come from?

Suppose you're using Kafka.

```
Topic

↓

Consumer

↓

Processing Failed
```

Consumer retries.

Still fails.

Kafka (or the application framework) sends that message to another topic.

```
payment-topic

↓

refund failed

↓

payment-DLQ-topic
```

That's DLQ.

Same in RabbitMQ.

Same in SQS.

Every message broker supports some version of DLQ.

---

### Simple Architecture

```
		        Kafka

        payment-topic
              |
              |
              v

      Payment Consumer

              |

         Refund Failed

              |

      Retry Retry Retry

              |

      Still Failed

              |

              v

        payment-DLQ
```

---

### Real Production Example

Imagine Razorpay.

Customer refund request.

Gateway crashed.

```
Refund

↓

Retry

↓

Retry

↓

Still Failed

↓

DLQ
```

After payment gateway recovers,

background worker processes DLQ.

Customer finally receives refund.

This is why large companies rarely lose requests.

---

### Failure Scenario 5 — Orchestrator Crashes

This confuses almost everyone.

Let's simplify.

---

Imagine you're cooking.

Steps:

```
Boil Rice

↓

Cook Curry

↓

Make Roti
```

Suddenly

Electricity goes.

You stop.

Later electricity comes.

Question:

Do you start cooking from beginning?

No.

You remember

```
Rice already cooked.
```

You continue.

Exactly the same idea.

---

### Saga Example

Customer places order.

```
Create Order ✓

↓

Payment ✓

↓

Inventory

(Not Started Yet)
```

Suddenly

Orchestrator server crashes.

```
Power Off
```

---

Question

How does it remember

where it stopped?

---

Answer

It saves progress.

Think of a game.

After every level

Game automatically saves.

```
Level 1

SAVE

↓

Level 2

SAVE

↓

Level 3

SAVE
```

If game crashes,

you restart from

last save.

Not from beginning.

Saga works exactly like this.

---

### Saga State Table

Every Saga has an ID.

Example

```
Saga ID = 101
```

Database stores

|Saga ID|Current Step|Status|
|---|---|---|
|101|Payment Completed|Running|

That's all.

---

Before starting Inventory,

Orchestrator crashes.

Database still contains

```
Saga 101

Payment Completed

Next Step = Inventory
```

---

Server restarts.

First thing orchestrator does

```
Read Saga Table
```

It finds

```
Saga 101

Not Finished
```

Continue from

```
Inventory
```

---

Flow

```
Order

↓

Payment

↓

Save State

↓

CRASH

↓

Restart

↓

Read Database

↓

Continue Inventory
```

No work is repeated.

---

### Why save after every step?

Imagine

```
Order

↓

Payment

↓

Inventory

↓

Shipping
```

Suppose crash occurs after Payment.

Without saving

Orchestrator thinks

```
Nothing happened.
```

Then

Payment may execute again.

Customer charged twice.

Bad.

So after every successful step

it writes

```
Current Step = Payment Done
```

to database.


# Failure Scenario 6 — Duplicate Messages

will discuss in next question.

---

### Choreography vs Orchestration (Intuition)

|Choreography|Orchestration|
|---|---|
|Like dancers following each other's moves|Like a manager directing employees|
|No central controller|Central workflow controller|
|Services react to events|Services execute explicit commands|
|Easy to extend with new event consumers|Easy to understand complex workflows|
|Can become difficult to trace in large systems|Single place to view and modify business flow|

---

### The most important interview takeaway

A Saga **does not maintain consistency by preventing failures**. It maintains consistency by **detecting failures and executing compensating transactions until all services converge to a valid business state**.

That is exactly what **eventual consistency** means:

```
That is exactly what eventual consistency means:

Temporary State

Order = CREATED
Payment = SUCCESS
Inventory = FAILED

↓

Compensation Executes

↓

Final State

Order = CANCELLED
Payment = REFUNDED
Inventory = AVAILABLE
```

The system may be inconsistent for a short time, but **it will eventually become consistent without using distributed locks or a global database transaction**


The natural next topic is **Compensation Transactions**, where we'll dive into:

- Why compensation is **not** the same as a database rollback.
- How to design compensation APIs in Spring Boot.
- Idempotency, retries, Outbox Pattern, Kafka integration, DLQs, and exactly-once behavior in production-grade Saga implementations. These are the topics that are most frequently discussed in senior backend and system design interviews.

-------------------

## How do you implement compensating transactions in a Saga? How do you handle retries and idempotency in Saga steps ?

interviewer wants to know:
- How compensation is implemented in code.
- How retries work.
- What if compensation executes twice?
- What if a service crashes?
- How duplicate Kafka messages are handled.
- How eventual consistency is guaranteed.

Let's build it from scratch.

---

### Part 1 - What is a Compensating Transaction?

Many beginners think:

> Compensation = Rollback

This is **incorrect**.

Let's understand why.

Suppose we have three services.

```
Order Service
Payment Service
Inventory Service
```

Each service has its own database.

```
Order DB

Payment DB

Inventory DB
```

Customer buys a laptop.

Saga Flow

```
Create Order

↓

Charge Payment

↓

Reserve Inventory
```

Inventory fails.

Question:

Can Payment Service execute

```
ROLLBACK;
```

No.

Why?

Because Payment already committed.

```
Payment DB

COMMIT
```

Database transaction is over.

Rollback is impossible.

---

### Then how do we undo?

Instead of database rollback,

we perform another business transaction.

Example

Instead of

```
ROLLBACK Payment
```

we do

```
Refund Payment
```

Notice

These are completely different.

---

### Rollback vs Compensation

|Database Rollback|Compensation|
|---|---|
|Database operation|Business operation|
|Happens before COMMIT|Happens after COMMIT|
|Automatic|Explicitly coded|
|Removes DB changes|Creates new reverse transaction|
|SQL rollback|Refund, Cancel, Release Stock|


---

### Implementing Compensation

Let's use Order System.

Steps

```
1 Create Order

2 Charge Payment

3 Reserve Inventory

4 Create Shipment
```

Each step has an opposite step.

|Forward Transaction|Compensation|
|---|---|
|Create Order|Cancel Order|
|Charge Payment|Refund Payment|
|Reserve Inventory|Release Inventory|
|Create Shipment|Cancel Shipment|

This is the first design principle.

> Every Saga step should have a compensating action.

---

### Flow

```
Create Order      ✓

↓

Charge Payment   ✓

↓

Reserve Stock    ✗

↓

Refund Payment

↓

Cancel Order
```

Notice

Compensation executes

**in reverse order.**

Exactly like stack unwinding.

---

### How is it implemented?

Imagine an Orchestrator.

Instead of only remembering completed steps,

it also remembers

```
Step Name

Compensation API
```

Example

```
Step 1

Create Order

Compensation

Cancel Order
```

Step 2

```
Charge Payment

Compensation

Refund Payment
```

Step 3

```
Reserve Inventory

Compensation

Release Inventory
```

When a step fails,

Orchestrator walks backward.

```
Completed Steps

Order

Payment

Inventory Failed

↓

Compensate Payment

↓

Compensate Order
```

---

### Spring Boot Example (Pseudo Code)

Imagine the orchestrator maintains a list:

```
Step 1
forward()      -> createOrder()
compensate()   -> cancelOrder()

Step 2
forward()      -> chargePayment()
compensate()   -> refundPayment()

Step 3
forward()      -> reserveInventory()
compensate()   -> releaseInventory()
```

If inventory fails

```
releaseInventory()

↓

refundPayment()

↓

cancelOrder()
```

Simple.

---

### Part 2 - Why do we need Retries?

Now comes the interesting part.

Suppose

Inventory failed.

Compensation starts.

```
Refund Payment
```

But Payment Service is temporarily down.

```
Refund API

↓

500 Error
```

Should Saga fail forever?

No.

It retries.

---

#### Retry Intuition

Imagine you're calling your friend.

```
Call

↓

Busy

↓

Call Again

↓

Busy

↓

Call Again
```

Exactly same.

Temporary failures usually recover.

---

#### Retry Flow

```
Refund

↓

Failed

↓

Retry

↓

Failed

↓

Retry

↓

Success
```

This solves

- temporary network issue
- service restart
- database lock
- timeout
- payment gateway unavailable

---

### But why not retry forever?

Suppose payment gateway is down for 3 hours.

Infinite retry means

```
Retry

Retry

Retry

Retry

Retry

Millions of Requests
```

Bad.

So production systems usually do

```
Retry 1

↓

Retry 2

↓

Retry 3

↓

Move to DLQ
```

---

### Retry with Exponential Backoff

Instead of

```
1 sec

1 sec

1 sec

1 sec
```

Production systems increase waiting time.

```
Retry 1

after 2 sec

↓

Retry 2

after 4 sec

↓

Retry 3

after 8 sec

↓

Retry 4

after 16 sec
```

Why?

Otherwise thousands of services hit the recovering server simultaneously (a **retry storm**).

---

### Part 3 - Why do we need Idempotency?

This is probably the most important interview topic.

---

Suppose Kafka sends

```
Charge Payment
```

Payment succeeds.

Before Kafka receives ACK

Payment Service crashes.

Kafka thinks

```
Maybe consumer never processed it.
```

Kafka sends the message again.

Now Payment Service receives

```
Charge Payment

Again
```

Without protection

```
First charge

₹1,50,000

↓

Second charge

₹1,50,000

↓

Customer charged ₹3,00,000 😱
```


---

### [ Solution ]

Idempotency.

Meaning

> Executing the same request multiple times produces the same final result.

> **idempotency** ensures that executing the exact same local transaction, command, or compensation step multiple times produces the same system state.

---

#### Everyday Example

Power Button

Press once

```
Light ON
```

Press again

```
Still ON
```

Not

```
ON

↓

OFF

↓

ON
```

Power button is idempotent.

### [ How do we prevent this? ]

With **Idempotency**.

The payment service stores an **idempotency key** or checks the Saga/Order ID.

First request:

```
Saga ID = 101

Charge Payment

↓

Success

↓

Record:
Saga 101 = Already Charged
```

Duplicate request arrives:

```
Saga ID = 101

↓

Already Processed?

↓

YES

↓

Ignore
```

No second charge.

---

#### Duplicate Refund Example

Exactly the same applies to compensation.

Suppose Kafka delivers:

```
Refund Order #101
```

twice.

Without idempotency:

```
Refund ₹1,50,000

↓

Refund ₹1,50,000

↓

Customer gets ₹3,00,000
```

With idempotency:

```
Refund Order #101

↓

Already Refunded?

↓

YES

↓

Ignore
```

### How is Idempotency Implemented?

Usually every request contains a unique identifier.

Example

```
Saga ID = 501

Order ID = ORD123

Transaction ID = TXN789
```

Payment Service stores

|Transaction ID|Status|
|---|---|
|TXN789|SUCCESS|

Duplicate arrives.

Look up table.

Already exists.

Return previous result.

---

### Complete Production Flow

Let's put everything together.

```
Customer

↓

Create Order

✓

↓

Charge Payment 

✓

↓

Reserve Inventory ❌

✗

↓

Start Compensation 

↓

Refund Payment

↓

Timeout

↓

Retry after 2 sec

↓

Retry after 4 sec

↓

Retry after 8 sec

↓

Still Failed

↓

Move to DLQ

↓

Payment Gateway Recovers

↓

DLQ Worker Picks Message

↓

Refund Again

↓

SUCCESS

↓

Cancel Order

↓

Saga Completed
```

Notice what happened:

- Compensation **did not stop** after the first failure.
- Retries handled temporary problems.
- DLQ preserved the failed request.
- When the gateway recovered, compensation continued.
- If the refund command was delivered twice, idempotency prevented a double refund.

The system eventually reached a consistent state.

---

### How retries, idempotency, and compensation work together

```
                 Inventory Fails
                        |
                        v
               Start Compensation
                        |
                        v
                Refund Payment
                        |
            +-----------+-----------+
            |                       |
        Success                 Failure
            |                       |
            |                  Retry (2s)
            |                       |
            |                  Retry (4s)
            |                       |
            |                  Retry (8s)
            |                       |
            |              +--------+--------+
            |              |                 |
            |          Success          Still Fails
            |              |                 |
            |              |             Move to DLQ
            |              |                 |
            +--------------+-----------------+
                           |
                           v
                  Cancel Order
                           |
                           v
                    Saga Completed

Duplicate refund command?
        |
        v
Already refunded?
        |
     +--+--+
     |     |
    Yes    No
     |      |
 Ignore   Process Refund
```

---

### Interview Answer (2–3 minutes)

If an interviewer asks:

> **How do you implement compensating transactions and handle retries and idempotency in Saga?**

A strong answer would be:

> "In a Saga, each forward transaction has a corresponding compensating transaction. For example, `CreateOrder` is compensated by `CancelOrder`, `ChargePayment` by `RefundPayment`, and `ReserveInventory` by `ReleaseInventory`. If a later step fails, the Saga executes compensations in reverse order of the completed steps. Since compensation calls can fail due to temporary issues, they are retried using exponential backoff. If retries are exhausted, the failed command is moved to a Dead Letter Queue (DLQ) for later processing instead of being lost. Because message brokers like Kafka typically provide at-least-once delivery, duplicate commands are possible, so every Saga step and every compensation must be idempotent. Services use a unique Saga ID, Order ID, or Transaction ID to detect duplicate requests and safely return the previous result instead of executing the business action again. Together, compensation, retries, DLQs, and idempotency ensure the system reaches eventual consistency without relying on distributed transactions or database rollbacks."


**Interviewer:**

> If compensation fails, is Saga failed forever?

**Answer:**

No.

Saga enters a **Compensating** or **Compensation Failed** state. The system retries the failed compensation. If retries are exhausted, the command is moved to a **Dead Letter Queue (DLQ)** for later processing or manual intervention. The goal is to **eventually complete all compensation steps** so the system reaches a consistent business state.

##
----------------
##


## What are the challenges of implementing Saga in a system with high throughput ? How about when need to design a Saga for a globally distributed system with strict latency requirements ?

will do later ....

