```
Database Scaling
│
├── Vertical Scaling
├── Horizontal Scaling
│
├── Replication ⭐
│      ├── Master-Slave
│      ├── Master-Master
│      ├── Read Replicas
│      ├── Replication Lag
│      └── Sync vs Async Replication
│
├── Sharding
├── Partitioning
├── Caching
└── Distributed Databases
```


Today we'll cover the most important foundation:

- Replication
- Master-Slave Architecture
- Read Replicas
- Replication Lag

---

### Problem Statement

Imagine you built Instagram.

Initially,

```
           Users
              │
              ▼
         MySQL Database
```

100 users...

Everything is fine.

---

Now imagine

```
5 Million users

Everyone opens Instagram.

Everyone loads feed.
```

Database receives

```
Read:
SELECT * FROM posts

Read:
SELECT * FROM users

Read:
SELECT comments...

Read:
SELECT likes...
```

Millions of SELECT queries.

But writes are much fewer.

```
User uploads photo
User likes post
User comments

These are WRITE operations.
```

Notice something?

**Reads >> Writes**

Typically

```
Read Requests = 95%

Write Requests = 5%
```

Example

```
1000 Requests

950 Reads

50 Writes
```

So why should one database handle everything?

That's where Replication comes in.

---

### What is Replication?

Replication means

> Keeping multiple copies of the same database.

Instead of one database

```
Users  -->  MySQL
```

we create copies

```
		        Primary DB
	            (Master)

             /     |      \
            /      |       \
      Replica1 Replica2 Replica3
```

Each replica contains almost identical data.

---

### Why Replication?

Suppose

```
10 million users

Everyone opens Home Feed.
```

Without replication

```
All users  -->  Master Database
```

Master receives

```
10 million SELECT queries
```

CPU becomes overloaded.

Slow.

---

With Replication

```
                Master
               /  |  \
              /   |   \
             ▼    ▼    ▼
           R1    R2    R3
```

Now

```
Read queries

User1 → R1

User2 → R2

User3 → R3

User4 → R1

User5 → R2
```

Load is distributed.

---

### Master-Slave Architecture

This is the most common architecture.

```
         Master
        (Primary)

          INSERT
          UPDATE
          DELETE

                │

        Replicates Data

                ▼

             Replica 1

             Replica 2

             Replica 3
```

Master accepts

```
INSERT

UPDATE

DELETE
```

Replicas usually accept only

```
SELECT
```

---

Think of it like this

Teacher writes notes.

Students copy them.

```
Teacher

Writes notebook

↓

Student1 copies

↓

Student2 copies

↓

Student3 copies
```

Teacher = Master

Students = Replicas

Nobody writes in student's notebook.

Everyone copies teacher.

---

### Why not allow writing on replicas?

Suppose

Replica1 changes

```
User Name = Rahul
```

Replica2 changes

```
User Name = Rohit
```

Master has

```
Rahul Kumar
```

Now

Different databases contain different data.

Chaos.

Therefore

Only Master writes.

---

### Flow of Data

Imagine

```
User uploads profile picture.
```

Step 1

```
App

↓

Master

UPDATE users
SET profile_pic='abc.jpg'
```

Master updates.

---

Step 2

Master sends changes

```
Master

↓

Replica1

↓

Replica2

↓

Replica3
```

Now everyone has same data.

---

### Read Replicas

Read Replica simply means

A database that only serves read requests.

Example

```
SELECT

SELECT

SELECT

SELECT
```

No

```
INSERT

UPDATE

DELETE
```

---

Example

#### Instagram Feed

Millions of people opening feed.

```
SELECT posts
```

Should NOT go to master.

Instead

```
User

↓

Load Balancer

↓

Replica
```

Master stays free.

---

### But there is a problem...

[ Replication is NOT always instant. ]

This leads to

### Replication Lag

Suppose

Time

```
10:00:00
```

User updates profile.

Master

```
Name = Prashant
```

Replica still has

```
Name = Old Name
```

After

```
300 milliseconds
```

Replica finally updates.

---

Timeline

```
10:00:00

Write → Master

↓

10:00:00.100

Master updated

↓

10:00:00.300

Replica updated
```

Between these times

Replica contains old data.

---

Example

You change your Instagram bio.

Immediately refresh.

You still see

Old bio.

Refresh again.

Now new bio appears.

Why?

Replication Lag.

---

### Why does Replication Lag happen?

Because

Master must

```
Receive write

↓

Commit transaction

↓

Generate replication log

↓

Send to replicas

↓

Network transfer

↓

Replica processes log

↓

Replica updates data
```

Every step takes time.

Usually

```
10 ms

50 ms

100 ms

500 ms

Sometimes seconds
```

---

### Real World Example

Suppose

WhatsApp Status.

You upload a status.

Friend immediately opens.

Sometimes

Status doesn't appear.

Refresh after 2 seconds.

Now it appears.

This delay can occur due to replication or caching.

---

### Another Example

Amazon

You buy the last iPhone.

Master

```
Stock = 0
```

Replica still says

```
Stock = 1
```

Another user purchases.

Oops!

Overselling.

> Therefore, critical operations (like inventory or payments) should read from the master or use stronger consistency guarantees instead of relying on replicas.

---

### When should we read from Master?

Always read from Master when the latest data is required:

- After login (if you immediately need updated session information)
- After changing a password
- Bank account balance after a transfer
- Inventory after a purchase
- Payment status
- Order confirmation immediately after placing an order

---

### When can we read from Replicas?

Read replicas are ideal when slightly stale data is acceptable:

- Instagram feed
- News articles
- Product catalog browsing
- Blog posts
- Public user profiles
- Comments
- Search results (often acceptable if a few seconds behind)

---

### Advantages of Replication

✅ Scales read traffic

```
100M Reads

↓

10 Replicas

↓

10M reads each
```

✅ Faster queries

Users hit the nearest or least-loaded replica.

✅ Better availability

If one replica fails

```
Replica1 ❌

Replica2 ✅

Replica3 ✅
```

Application continues serving reads.

---

### Disadvantages

❌ Replication lag

❌ More infrastructure cost

❌ Operational complexity (monitoring, failover, backups)

❌ Writes can still become a bottleneck because they all go through the master in a Master-Slave setup

---

### Complete Flow

```
                  Users
                     │
                     ▼
               Application
               /          \
              /            \
          WRITE             READ
            │                │
            ▼                ▼
      +-------------+   +-------------+
      |   Master    |   |  Load Bal.  |
      | INSERT/UPDATE|   +------+------+
      +------+------+
             │
     Replication
             │
     +--------+--------+
     |                 |
+-----------+    +-----------+
| Replica 1 |    | Replica 2 |
|  SELECT   |    |  SELECT   |
+-----------+    +-----------+
```

### Interview Questions

1. **Why use replication instead of a bigger database server?**
    - Replication scales **reads horizontally**, while a bigger server (vertical scaling) has hardware limits and is more expensive.
2. **Can a replica accept writes?**
    - In a typical Master-Slave setup, no. It serves reads only to avoid data conflicts.
3. **What is replication lag?**
    - The delay between a write being committed on the master and that write becoming visible on replicas.
4. **When should you avoid reading from a replica?**
    - When your application requires the most up-to-date data immediately after a write, such as banking, payments, inventory, or order confirmation.


----
------
-----

## Synchronous vs Asynchronous Replication (how replicas receive updates and how it affects consistency and latency)

Before understanding these two, let's first understand **how replication actually works.**

Suppose we have:

```
                Master
                  │
        -------------------
        │                 │
        ▼                 ▼
    Replica 1         Replica 2
```

A user changes his password.

```
UPDATE users
SET password='newPassword'
WHERE id=10;
```

Question:

**When should the Master tell the application "Your data is saved successfully"?**

There are two answers.

---

### Option 1: Synchronous Replication

In synchronous replication,

**Master waits until Replica confirms the update.**

#### Step-by-step

```
User

↓

Master receives UPDATE

↓

Master writes data

↓

Master sends data to Replica

↓

Replica writes data

↓

Replica says

"I have updated."

↓

Master says

"Success"

↓

User gets response
```

Flow

```
User
 │
 ▼
Master
 │
 ▼
Replica
 │
 ▼
ACK (Confirmation)
 │
 ▼
Master
 │
 ▼
User
```

Notice something.

The user gets a response **only after the replica confirms**.

---

#### Advantages

##### 1. Strong Consistency

All databases have the same data.

No mismatch.

---

##### 2. No Replication Lag

Everything updates together.


---

#### Disadvantages

Suppose Replica becomes slow.

```
Master → 2ms

Replica → 400ms
```

Master has to wait.

User also waits.

Everything becomes slower.

---

Imagine Replica crashes.

```
Master

↓

Waiting...

Waiting...

Waiting...
```

No confirmation.

Write operation may fail or be blocked.

---


---

### Option 2: Asynchronous Replication

This is the opposite.

Master **does NOT wait**.

---

#### Step-by-step

```
User

↓

Master updates

↓

Master immediately replies

"Success"

↓

Later...

↓

Replica receives update
```

Flow

```
User

↓

Master

↓

User gets response

↓

Master sends data

↓

Replica updates later
```

Notice

User doesn't wait.

---

#### Real World Example

Suppose you send WhatsApp photos to your friends.

You press Send.

WhatsApp instantly shows

```
✓ Sent
```

But your friend's phone may receive it

after 2 seconds.

You didn't wait.

This is asynchronous.

---

#### Advantages

##### 1. Very Fast

Master doesn't wait.

User gets response quickly.

---

##### 2. High Performance

Can handle millions of writes.

---

##### 3. Better User Experience

Instagram

Facebook

Twitter

You don't want users waiting after every Like.

---

#### Disadvantage

Replication Lag.

Example

Master

```
Name = Prashant
```

Replica

```
Name = Old Name
```

for a few milliseconds.

---

### Comparison

|Feature|Synchronous|Asynchronous|
|---|---|---|
|Master waits?|✅ Yes|❌ No|
|Response Speed|Slow|Fast|
|Replication Lag|None|Possible|
|Consistency|Strong|Eventual|
|Used In|Banking, Payments|Social Media, E-commerce, Blogs|

---

### Which one do big companies use?

Usually **Asynchronous Replication**.

Why?

Imagine Instagram.

1 billion users.

If every Like waited for replicas...

Instagram would become much slower.

Instead

```
User Likes

↓

Master updates

↓

Instant Success

↓

Replica updates later
```


----
-----


## Leader-Follower Failover (what happens if the master crashes) ?

Now suppose we have

```
              Master

             /      \

        Replica1   Replica2
```

Master suddenly crashes.

```
              ❌            
               
            Master
```

Question:

Who handles writes now?

Nobody.

Users can't

- Register
- Like posts
- Upload photos

Everything stops.

This is where **Failover** comes in.

---

### What is Failover?

Failover means

> Automatically promoting a healthy replica to become the new leader (master) when the current master fails.

---

Suppose

Before crash

```
           Master

          /      \

        R1       R2
```

Master crashes.

```
        ❌ Master
```

System promotes

```
Replica1
```

Now

```
          New Master

             │

        Replica2
```

Application changes

Instead of writing

```
Master
```

It now writes to

```
Replica1
```

Users barely notice if failover is fast.

---

### Failover Steps

#### Step 1

Master crashes.

```
Master ❌
```

---

#### Step 2

Monitoring detects failure.

```
Heartbeat missing
```

---

#### Step 3

Choose healthiest replica.

```
Replica1
```

---

#### Step 4

Promote Replica1.

```
Replica1 → Master
```

---

#### Step 5

Application reconnects.

```
Writes

↓

New Master
```

Everything resumes.

---

### Real World Example

Amazon

Order database.

Master machine dies.

Within seconds

Replica becomes Master.

Customers keep ordering.

Without failover

Amazon loses millions of dollars.

---

### Interview Question

**What happens if the master crashes?**

Good answer:

> The application cannot perform writes until a new leader is elected. A failover process promotes a replica to leader, redirects writes to it, and the remaining replicas start replicating from the new leader.


-----
----


# What is database sharding ?

## Replication vs Sharding

Before learning sharding, ask yourself this question.

Suppose you have this database.

```
                Master
                   │
        ---------------------
        │         │         │
        ▼         ▼         ▼
     Replica1  Replica2  Replica3
```

This is **Replication**.

Question:

**Can replication increase write capacity?**

**No.**

Why?

Because

Every write still goes to

```
Master
```

Imagine

```
1 Million Reads

100,000 Writes/sec
```

Reads are distributed.

But

All writes go here

```
                Master
                   ▲
                   │
            100,000 writes/sec
```

Eventually

CPU becomes overloaded.

Disk becomes overloaded.

Memory becomes overloaded.

Master cannot handle more writes.

Adding 100 replicas doesn't help.

Why?

Because replicas **don't accept writes**.

So what do we do?

---

### The Problem

Suppose Instagram now has

```
2 Billion Users
```

Table

```
Users
--------------------------------------

ID

Name

Email

Country

CreatedAt
```

Rows

```
2,000,000,000 users
```

Problems:

- Huge indexes
- Slow inserts
- Slow backups
- Huge storage
- Expensive hardware

Eventually

One database server becomes too small.

This is called the **Single Database Bottleneck**.


## Solution → [ Sharding ]

Instead of making **copies** of the database,

we **split** the database.

Imagine you have

```
10 Crore users
```

Instead of

```
One database

10 Crore users
```

we create

```
DB1

2.5 Crore users

DB2

2.5 Crore users

DB3

2.5 Crore users

DB4

2.5 Crore users
```

Each database stores only a portion of the data.

This is called **Sharding**.

---

### Definition

> **Database Sharding is the process of horizontally partitioning data across multiple database servers so that each server stores only a subset of the total data.**

Notice

Replication

```
Copies Data
```

Sharding

```
Splits Data
```

Huge difference.

---

### Visual Difference

### Replication

```
Master

Users A-Z

↓

Replica

Users A-Z

↓

Replica

Users A-Z
```

Everyone has the same data.

---

### Sharding

```
Shard1

Users A-F

Shard2

Users G-L

Shard3

Users M-R

Shard4

Users S-Z
```

No duplicate data.

Each shard owns different users.


Sharding Strategies -

- Range Based
- Hash Based
- Directory Based

will do later




-----
----
----


## Master-Master Replication 

Many people think Master-Master means "two databases with the same data." That's only partially true.

The important point is:

> **Both databases can accept writes, and both continuously replicate changes to each other.**

### Single Master (Leader-Follower)

```
              Master
                 │
         ----------------
         │              │
         ▼              ▼
     Replica1       Replica2

Writes → Master only
Reads  → Replicas
```

Only one database is writable.

---

### Master-Master

```
          Master A  ⇄  Master B

Writes → A
Writes → B

Both synchronize with each other.
```

Both are writable.

---

### Real-world scenario: Global E-commerce

Imagine an e-commerce company with users in India and the US.

#### Without Master-Master

```
Users in India ──► India App ──► US Database

Users in US ─────► US App ─────► US Database
```

Indian users must wait for every write to travel to the US, increasing latency.

---

#### With Master-Master

```
India Region                    US Region

Master A  ⇄──────────────⇄  Master B

Indian users write here.     US users write here.
```

Each region writes locally, then the changes are replicated across regions.

---

### How replication works

Suppose a user in India updates their address.

```
User

↓

Master A

Address = Mumbai

↓

Replicate

↓

Master B

Address = Mumbai
```

Now a US customer service representative sees the same updated address.

---

### The biggest challenge: Write conflicts

Imagine the same customer updates their profile in two regions almost simultaneously.

At 10:00:01

India:

```
Phone = 9999999999
```

At 10:00:01

US:

```
Phone = 8888888888
```

When the databases synchronize, both changes conflict.

```
Master A           Master B

9999999999   ⇄   8888888888
```

The system must decide which value wins or how to merge them.

Common approaches include:

- **Last Write Wins (LWW):** the update with the newest timestamp wins.
- **Application-specific rules:** for example, merge shopping carts instead of overwriting them.
- **Version vectors / conflict detection:** detect concurrent updates and let the application resolve them.

---

### Why isn't Master-Master common?

Because conflict resolution, replication ordering, and operational complexity are difficult.

For most applications, it's simpler and safer to use:

```
One Leader
      │
      ▼
Many Read Replicas
```

This architecture meets the needs of the vast majority of web applications while avoiding the complexity of multi-leader conflict resolution.

---

### Complete Scaling Journey

```
Small Application
        │
        ▼
Single Database
        │
        ▼
Vertical Scaling (bigger machine)
        │
        ▼
Replication (more read capacity)
        │
        ▼
Leader-Follower Failover (high availability)
        │
        ▼
Sharding (more write capacity and storage)
        │
        ▼
Multi-region / Master-Master (global low-latency writes, if needed)
```


-----
-----
----

## diffrence b/w Partitioning and Sharding ?

### Imagine You Have One Users Table

```
Users

------------------------------------

ID

Name

Email

Country

CreatedAt
```

Suppose it contains

```
100 Million users
```

The database becomes slow.

Now there are **two ways** to split the data.

---

### Option 1: Partitioning

The data is split into multiple pieces **inside the same database server**.

```
                    Database Server

                  +------------------+

                  | Users Table      |

                  |------------------|

                  | Partition 1      |

                  | Partition 2      |

                  | Partition 3      |

                  | Partition 4      |

                  +------------------+
```

Notice

Everything is still on **one machine**.

Only the table is divided.

---

### Example

Partition by Year

```
Orders

↓

2022

↓

Partition1

----------------

2023

↓

Partition2

----------------

2024

↓

Partition3
```

Each partition stores different rows.

---

### Query

```
SELECT *
FROM orders
WHERE order_year = 2024;
```

Database directly checks

```
Partition3
```

instead of scanning the entire table.

Much faster.

---

### Option 2: Sharding

Now imagine

One database server itself becomes too small.

Instead of one server

```
Server1
```

we create

```
Server1

Server2

Server3

Server4
```

Each server stores different data.

```
Users

-------------------

Shard1

User 1-25M

-------------------

Shard2

26M-50M

-------------------

Shard3

51M-75M

-------------------

Shard4

76M-100M
```

Each shard is a separate database.

---

### Biggest Difference

#### Partitioning

```
One Database

↓

Multiple Partitions
```

Still one database.

---

#### Sharding

```
Multiple Databases

↓

Multiple Servers
```

Completely different servers.

---

### Visual Comparison

### Partitioning

```
               Database Server

      +--------------------------+

      | Partition1               |

      | Partition2               |

      | Partition3               |

      | Partition4               |

      +--------------------------+
```

---

### Sharding

```
          Database 1

          Partition A

----------------------------

          Database 2

          Partition B

----------------------------

          Database 3

          Partition C
```

Notice

Different machines.

---

### Why Partition?

Suppose

```
Orders

500 Million rows
```

Query

```
SELECT *
FROM orders
WHERE year = 2024;
```

Without partitioning

Database scans

```
500 Million rows
```

With partitioning

```
2024 Partition
↓
20 Million rows
```

Much faster.

---

### Why Sharding?

Suppose

Database size

```
12 TB
```

Server can store only

```
4 TB
```

Impossible.

Need

```
Server1

4 TB

Server2

4 TB

Server3

4 TB
```

That's sharding.

---

### Another Way to Remember

Partitioning solves

```
Performance
```

inside one database.

---

Sharding solves

```
Scalability
```

across multiple databases.

---

### [ Can Partitioning and Sharding be used together? ]

Yes.

Large companies do this.

Example

```
Shard1

↓

Partition1

Partition2

Partition3

--------------------

Shard2

↓

Partition1

Partition2

Partition3
```

Every shard can itself contain partitions.

---

### Instagram Example

Suppose Instagram has

```
2 Billion users
```

First

They shard users.

```
Shard1

US Users

----------------

Shard2

Europe Users

----------------

Shard3

Asia Users
```

Now inside

Asia shard

```
Posts

2023

2024

2025
```

They partition posts by year.

So

```
Sharding

↓

Partitioning
```

Both together.

---

### MySQL Example

#### Partitioning

One database

```
instagram_db
```

One table

```
posts
```

Internally

```
posts_2023

posts_2024

posts_2025
```

The application still queries:

```
SELECT * FROM posts;
```

MySQL decides which partition to access.

The application usually doesn't know partitions exist.

---

#### Sharding

Now

```
DB Server 1

instagram_us

------------------

DB Server2

instagram_asia

------------------

DB Server3

instagram_europe
```

Application decides

```
Indian User

↓

Asia Database
```

The application (or a routing layer) must know **which shard** contains the data.

---

### SQL Example

Partitioning

```
SELECT *
FROM orders
WHERE order_date >= '2025-01-01';
```

Database checks only the relevant partition.

---

Sharding

Application first routes the request:

```
User ID = 500001

↓

Shard3
```

Then executes:

```
SELECT *
FROM users
WHERE id = 500001;
```

Only on **Shard3**.

---

### Interview Comparison Table

|Feature|Partitioning|Sharding|
|---|---|---|
|Data split|✅ Yes|✅ Yes|
|Multiple servers|❌ No|✅ Yes|
|Multiple databases|❌ Usually No|✅ Yes|
|Increases storage capacity|❌ Limited by one server|✅ Yes|
|Improves read performance|✅ Yes|✅ Yes|
|Improves write scalability|⚠️ Limited|✅ Yes|
|Application routing required|❌ No|✅ Yes|
|Typical use case|Very large tables on one server|Massive datasets that exceed one server's capacity|

---

## Simple Interview Answer (One Line)

> **Partitioning splits a table into smaller pieces within the same database server to improve query performance and manageability, whereas sharding splits data across multiple database servers to improve scalability, storage capacity, and write throughput.**


-----
----
----


## CAP Theorem -

### Why Do We Need CAP Theorem?

Let's start with a simple system.

```
    Database
```

Only one database.

No replication.

No sharding.

Life is easy.

---

Now we scale.

```
           Master

          /      \

      Replica1   Replica2
```

Now data exists on multiple servers.

Question:

Suppose

Master updates

```
Balance = ₹500
```

Replica still has

```
Balance = ₹1000
```

Which one is correct?

How do we handle this?

This is where distributed systems become difficult.

---

### CAP Theorem Definition

> **A distributed system can guarantee at most two of the following three properties simultaneously:**

- **C – Consistency**
- **A – Availability**
- **P – Partition Tolerance**

You **cannot guarantee all three at the same time** when a network partition occurs.

Think of it as a triangle.

```
           Consistency
             /      \
            /        \
           /          \
Availability--------Partition
```

You can choose only **two** during a network partition.

#### What is Consistency?

> Every user sees the latest data.

#### What is Availability?

> Every request receives a response. Even if something goes wrong, the server replies.

#### What is Partition Tolerance?

> The system continues working even if network communication between servers is interrupted.

### What is a Network Partition?


```
       Server A

         ❌

Network Broken

         ❌

       Server B
```

The network cable is cut. Servers cannot communicate. This is called a Network Partition.

The servers themselves are running.

Only the communication between them has failed.


### CP Systems

Choose

```
Consistency
+
Partition Tolerance
```

Reject requests

if data isn't guaranteed to be current.

Example use cases:

- Banking
- Stock trading
- Payment systems
- Inventory management

---

Example

Amazon

Only one iPhone left.

Two customers order simultaneously.

Better reject one request

than sell the same phone twice.

Consistency matters more.

---

### AP Systems

Choose

```
Availability
+
Partition Tolerance
```

Always respond.

Eventually synchronize.

Example:

- Instagram
- Facebook
- Twitter/X
- YouTube likes
- News Feed

If someone sees a like count of

```
100
```

instead of

```
101
```

for a few seconds,

it's acceptable.

---

### CA Systems

Choose

```
Consistency
+
Availability
```

No network partitions.

This works only when everything is on a single machine or when you assume perfect networking.

Example

```
Single MySQL Server
```

No distributed system.

No partition.

### Interview Answer (30 Seconds)

> **CAP Theorem states that when a network partition occurs in a distributed system, it is impossible to simultaneously guarantee both Consistency and Availability. Since network partitions are inevitable, distributed systems typically choose either CP (Consistency + Partition Tolerance) or AP (Availability + Partition Tolerance), depending on the application's requirements. Banking systems usually prefer CP, while social media platforms often prefer AP because temporary stale data is acceptable in exchange for high availability.**