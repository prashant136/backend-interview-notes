
### Why do we even need Pub/Sub?

> The **pub/sub (publish/subscribe) model** broadcasts real-time messages to active subscribers and discards them once delivered. 
   The **event streaming model** stores messages in a durable, append-only log, letting consumers read, replay, and reprocess data at their own pace.

Imagine you have a simple e-commerce application.

A customer places an order.

Without Pub/Sub:

```
Order Service
    │
    ├── Call Inventory Service
    ├── Call Payment Service
    ├── Call Notification Service
    ├── Call Analytics Service
    ├── Call Recommendation Service
    └── Call Loyalty Points Service
```

Problems:

- Order Service knows every other service.
- Adding a new service means modifying Order Service.
- If Notification Service is down, order placement may fail.
- Tight coupling.
- Difficult to scale.

This is not how large distributed systems are designed.

---

### Think differently

Instead of saying:

> "Inventory, do this."

the Order Service simply says:

> **"An order has been placed."**

It doesn't care who is listening.

```
Order Service

        │
        ▼

"Order Placed"

        │

??????????

        │

Inventory
Notification
Analytics
Recommendation
Loyalty
```

That "??????????" is the Pub/Sub system.

---

### What does Publish mean?

Publishing means

> "I am announcing that something happened."

Example:

```
Order Placed
```

or

```
Product Uploaded
```

or

```
User Registered
```

or

```
Payment Completed
```

Notice something.

The publisher isn't asking anyone to perform work.

It simply announces an event.

---

### What does Subscribe mean?

A subscriber says

> "Whenever this event happens, tell me."

Example

Inventory Service

```
I care about Order Placed
```

Notification Service

```
I also care about Order Placed
```

Analytics

```
Me too.
```

Recommendation

```
Me too.
```

Now one event reaches everyone.

---

### Real-world analogy: YouTube

Think about subscribing to a YouTube channel.

You subscribe to a creator.

When the creator uploads a video:

```
Creator  -->  Publishes Video
```

Subscribers receive notifications.

```
You
Your Friend
Another Subscriber
Millions of Others
```

The creator doesn't know who each subscriber is.

The creator uploads once.

The platform delivers it to everyone.

That's Publish/Subscribe.

---

### Compare Request/Response vs Pub/Sub

#### Request/Response

```
Order Service

↓

Inventory

↓

Response

↓

Notification

↓

Response

↓

Analytics

↓

Response
```

Everything is synchronous.

---

#### Publish/Subscribe

```
Order Service

↓

Publish

↓

Broker

↓

Inventory

↓

Notification

↓

Analytics

↓

Recommendation
```

Much more scalable.

---

### Where does the Broker come in?

The broker is the middleman.

```
Publisher

↓

Broker

↓

Subscribers
```

Without a broker:

```
Order

↓

Inventory

↓

Notification

↓

Analytics

↓

Recommendation
```

Every connection must be maintained manually.

With a broker:

```
Publisher

↓

Broker

↓

Everyone else
```

Publisher doesn't know who receives the message.

Subscribers don't know who produced it.

Everyone is independent.

This is called **loose coupling**.

---

### Pub/Sub Example in an E-commerce Site

Customer buys an iPhone.

Publisher sends:

```
OrderPlaced
{
  orderId: 123
  userId: 55
}
```

Broker receives it.

Subscribers:

Inventory

```
Reduce stock
```

Notification

```
Send SMS
```

Analytics

```
Increase sales count
```

Recommendation

```
Update user preferences
```

Loyalty

```
Give reward points
```

Search

```
Update trending products
```

The publisher doesn't know any of these services exist.

---

### What is a Topic?

The broker organizes messages by category.

Example:

```
order-created

payment-success

product-uploaded

user-created

inventory-updated
```

Publisher publishes to a topic.

Subscribers subscribe to that topic.

Example:

```
Topic

order-created
```

Subscribers:

```
Inventory

Notification

Analytics

Shipping
```

Everyone listening receives the event.

---

### Why is this powerful?

Suppose tomorrow you create a new service.

```
Fraud Detection
```

Just subscribe to

```
order-created
```

Done.

No change in Order Service.

That's the biggest benefit of Pub/Sub.

---

### Where does Redis Pub/Sub fit?

Redis Pub/Sub is the simplest implementation.

```
Publisher

↓

Redis Channel

↓

Subscribers
```

If subscribers are online:

They receive the message.

If they're offline:

The message is gone.

There is **no persistence**.

Think of it like a live radio broadcast.

If you weren't listening when the announcement happened, you miss it forever.

---

### Where does Kafka fit?

Kafka also uses the Pub/Sub pattern, but with a crucial difference.

```
Publisher

↓

Kafka Topic

↓

Stored on Disk

↓

Subscribers
```

If a subscriber is offline for an hour:

No problem.

It comes back and reads all missed messages.

Kafka keeps messages for a configurable retention period (for example, several days), so consumers can process them later.

---

### Redis Pub/Sub vs Kafka

|Feature|Redis Pub/Sub|Kafka|
|---|---|---|
|Pub/Sub|✅|✅|
|Stores messages|❌ No|✅ Yes|
|Replay old messages|❌|✅|
|Multiple consumers|✅|✅|
|Consumer groups|❌|✅|
|Very high throughput|Limited|Excellent|
|Suitable for distributed event streaming|❌|✅|

---

### One important realization

**Pub/Sub is a design pattern, not a product.**

Many technologies implement it differently:

- Redis → lightweight, in-memory Pub/Sub
- Kafka → durable, distributed event streaming
- RabbitMQ → Pub/Sub plus task queues and routing
- Cloud messaging systems (like managed cloud queues and event buses) also support Pub/Sub concepts

-----
----
----
--------
------


# Kafka

### What problem does Kafka solve?

Imagine you are building **Instagram**.

Users perform millions of actions every day.

- Upload Post
- Like Post
- Comment
- Follow User
- Send Message
- Watch Reel
- Search
- Receive Notifications

Suppose one user uploads a photo.

Without Kafka:

```
User
   |
   v
Upload API
   |
   +------> Save Post
   |
   +------> Notify Followers
   |
   +------> Update Feed
   |
   +------> Generate Thumbnail
   |
   +------> Send Analytics
   |
   +------> Detect Spam
```

One request is trying to do **6 different jobs**.

Problems:

- Slow response
- One service failing can fail everything
- Difficult to scale
- Heavy load

Instead, companies use Kafka.

---

#### With Kafka

```
          Upload API
               |
               |
        Publish Event
               |
               v

            Kafka

        "Post Uploaded"

               |
   -----------------------------
   |      |      |      |      |
 Feed  Notification Analytics Spam Thumbnail
```

The Upload API only says

> "Hey everyone, a post was uploaded."

Whoever is interested listens.

Nobody waits.

Everything becomes asynchronous.

---

#### Real Example

You upload a photo.

Instead of doing everything immediately,

Upload API sends

```
POST_UPLOADED
{
   postId : 100,
   userId : 25
}
```

Kafka stores this.

Now different services consume it.

```
Notification Service

"Oh...
Post Uploaded."

Send notification.


Feed Service

"Oh...
Need to add post to followers."


Analytics Service

"Oh...
Increase upload count."


Recommendation Service

"Oh...
New content available."
```

Everyone works independently.

This is why almost every large company uses Kafka.

---

#### Kafka Architecture

Kafka has only a few important concepts.

```
Producer
Broker
Topic
Partition
Consumer
Consumer Group
```

## 1. Producer

A Producer is simply an application that **sends messages to Kafka**.

That's it.

Think of Producer as the **sender**.

---

#### Real Instagram Example

You upload a photo.

```
You -> Upload API -> Producer -> Kafka
```

The Upload API creates an event.

```
POST_UPLOADED

{
   postId : 101,
   userId : 25,
   image : xyz.jpg
}
```

Then it sends this event to Kafka.

The Upload API becomes the Producer.

---

#### Real Life Analogy

Suppose you're sending a courier.

You package it.

```
Courier Office

↓

Parcel

↓

Warehouse
```

Who sends the parcel?

You.

You are the Producer.

The warehouse is Kafka.

---

#### Producer does NOT process data

Many beginners think Producer does work.

No.

Producer only says

```
Hey Kafka,

Please store this message.
```

Kafka replies

```
Stored.
```

Producer finishes.

---

#### Producer Flow

```
User Uploads Photo

      |

Upload Service

      |

Create Event

      |

POST_UPLOADED

      |

Kafka Topic
```

That's all.

---

#### Where does Producer send?

Producer never sends directly to a Broker.

Instead it says

```
Topic = POST_UPLOADED
```

Kafka decides

- Broker
- Partition

automatically.

Producer doesn't care.


## 2. What is a Broker?

A **Broker** is simply a **Kafka server**.

It stores messages.

Nothing more.

Example -

You send

```
User Uploaded Photo
```

The broker stores it.

```
Broker

--------------------

Message 1

Message 2

Message 3

Message 4
```

Think of it as a very fast storage server.

---

#### Example

Instagram Upload Service

```
Producer
```

sends

```
POST_UPLOADED
```

to

```
Broker
```

Broker stores it.

Later

```
Notification Service
```

reads it.

---

#### Why Multiple Brokers?

One machine cannot handle billions of events.

Companies have many Kafka servers.

```
Broker 1

Broker 2

Broker 3

Broker 4

Broker 5
```

Together,

they form a Kafka cluster.

Example -

How does Netflix handle Millions of events/sec.

One server cannot handle this.

They use many brokers.

```
Producer

      |

----------------------------------

Broker1

Broker2

Broker3

Broker4

----------------------------------
```

Now load is shared.

---

## 3. What is a Topic?

Now imagine inside the warehouse,

there are different rooms.

```
Sports
Electronics
Clothes
Furniture
```

Packages go to different rooms.

Kafka calls these rooms **Topics**.

---

Example

Instagram

Each event type gets its own topic.

```
Broker

----------------------------

Topic

POST_UPLOADED

Topic

LIKED_POST

Topic

COMMENT_ADDED

Topic

MESSAGE_SENT

----------------------------
```

The broker stores many topics.

---

Real Example

User uploads photo.

Producer writes

```
POST_UPLOADED
```

into

```
Topic

POST_UPLOADED
```

User likes a post.

Producer writes

```
LIKE_CREATED
```

into

```
Topic

LIKES
```

Different events.

Different topics.

---

### Why Topics?

Imagine everything goes into one bucket.

```
Upload

Payment

Login

Search

Comments

Orders

Refund

Likes

Messages
```

Impossible to manage.

Instead

```
Orders Topic

Payments Topic

Comments Topic

Likes Topic
```

Much cleaner.

---

## 4. What is a Partition?

This is the most important concept.

Suppose one topic becomes huge.

```
Topic

POST_UPLOADED

100 Billion Messages
```

Can one file hold everything?

Very difficult.

Instead Kafka splits a topic.

---

One Topic

```
POST_UPLOADED
```

is divided into

```
Partition 0

Partition 1

Partition 2
```

Visual

```
Topic

POST_UPLOADED

----------------------------

Partition 0

Message

Message

Message

----------------------------

Partition 1

Message

Message

----------------------------

Partition 2

Message

Message

----------------------------
```

Each partition is an ordered sequence of messages.

---

### Why Partition?

Suppose

Instagram gets

```
10 Million uploads/sec
```

One machine cannot write that fast.

Instead

```
Partition 0

stores some uploads

Partition 1

stores some uploads

Partition 2

stores some uploads

Partition 3

stores some uploads
```

Now many machines work together.

Huge performance improvement.

---
#### Important Property

Inside **one partition**, order is guaranteed.

Example

Partition 0

```
Post Created
Like
Comment
Delete
```

Kafka preserves this order.

But across different partitions,

global ordering is **not** guaranteed.

---

### Putting Everything Together

```
                    Kafka Cluster

      ------------------------------------

           Broker 1

          Topic

       POST_UPLOADED

      Partition 0

      Partition 1

      ---------------------------

           Broker 2

          Topic

      COMMENT_CREATED

      Partition 0

      Partition 1

      ---------------------------

           Broker 3

          Topic

       LIKE_CREATED

      Partition 0

      Partition 1

      ------------------------------------
```

---

### Instagram Flow

```
User uploads photo

        |

Upload API

        |

Producer

        |

Topic

POST_UPLOADED

        |

Partition 2

        |

Broker

        |

Notification Service

Feed Service

Analytics Service

Thumbnail Service
```

The Upload API returns quickly after publishing the event, while downstream services process it independently.


### How Kafka chooses a Partition

Suppose Topic has 3 partitions.

```
Orders

Partition 0

Partition 1

Partition 2
```

Producer sends

```
ORDER_CREATED
```

Kafka chooses one partition.

How?

There are three methods.

---

#### Method 1 - Round Robin

```
Message1 -> P0
Message2 -> P1
Message3 -> P2
Message4 -> P0
Message5 -> P1
```

Load is balanced.

---

#### Method 2 - Key-Based Partitioning (Most Common)

Instagram

User uploads.

```
userId = 100
```

Kafka calculates

```
formula - partion = hash(key) mod (number of partition)

hash(userId)

↓

Partition 2
```

Every upload by User 100 always goes to Partition 2.

Why?

To maintain order.

---

## 5. Consumer

Consumer reads messages from Kafka. 
A Kafka consumer is a client application that reads data from Apache Kafka topics.

---

Instagram

```
POST_UPLOADED
```

Notification Service wants it.

```
							Kafka
							/ | \
						   /  |  \ 
			   Notification  Feed  Analytics
```

Notification Service, Feed Service and Analytics Service are Consumer.

---

#### Multiple Consumers

One message

```
POST_UPLOADED
```

can be read by

```
Notification

Analytics

Recommendation

Feed

Search Index
```

One event.

Many readers.

Producer doesn't know who reads.

Producer doesn't care.

---

### [ Important ]

Consumer never deletes messages.

Kafka keeps them.

Consumers only read.

**This is different from traditional message queues where a message disappears immediately after being consumed.**

---

## 6. Consumer Groups

This is where Kafka becomes powerful.

Imagine Flipkart during a sale.

```
50000 Orders/sec
```

One Order Service cannot process everything.

Need multiple instances.

```
Consumer1

Consumer2

Consumer3

Consumer4
```

How should they work?

---

Wrong Approach

All Consumers read everything.

```
Consumer1

Order1

Order2

Order3

Consumer2

Order1

Order2

Order3
```

Duplicate processing.

Bad.

---

#### Kafka Solution

Consumer Group.

---

Suppose

Topic

```
Orders
```

has

```
Partition0
Partition1
Partition2
```

Now, Consumer Group

```
Order Service

Instance1
Instance2
Instance3
```

Kafka assigns

```
P0 → Instance1

P1 → Instance2

P2 → Instance3
```

No overlap.

---

Visual

```
Partition0 ------ Consumer1
Partition1 ------ Consumer2
Partition2 ------ Consumer3
```

Each partition has exactly one consumer within the same consumer group.

---

#### Real Instagram Example

Notification Topic

```
Partition0
Partition1
Partition2
```

Consumers

```
Notification Server1
Notification Server2
Notification Server3
```

Each server processes different partitions.

More servers = more speed.

---

#### What if there are more Consumers than Partitions?

Example

```
Partitions = 3

Consumers = 5
```

Kafka assigns

```
Consumer1 → P0
Consumer2 → P1
Consumer3 → P2
Consumer4 → Idle
Consumer5 → Idle
```

Two consumers do nothing.

Reason

One partition cannot be read by two consumers in the same group.

### Different Consumer Groups ??

will discuss later


------
-------
-------



