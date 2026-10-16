
## What is Scaling?

Imagine you built an e-commerce website.

Initially you have

- 100 users/day
- 10 orders/day

One server is enough.

```
            Users
               |
        +--------------+
        | Web Server   |
        +--------------+
               |
         Database
```

Everything works perfectly.

---

Now your company becomes successful.

```
100 users/day
↓

10,000 users/day
↓

100,000 users/day
↓

10 million users/day
```

Now one server starts struggling.

Symptoms:

- Website becomes slow
- CPU reaches 100%
- Memory gets exhausted
- Requests timeout
- Database becomes overloaded

Now you need **Scaling**.

Scaling means:

> Increasing system capacity to handle more users, requests, traffic, or data.

There are two primary ways.

---

## Vertical Scaling (Scale Up)

Vertical Scaling means:

> Increase the power of the existing machine.

Instead of adding more machines,  
you upgrade the current one.

Example:

Current server

```
CPU : 2 cores
RAM : 4 GB
SSD : 100 GB

handles: 500 requests/sec
```

Upgrade to

```
CPU : 16 cores
RAM : 64 GB
SSD : 2 TB

handles: 5000 requests/sec
```

Same server.

Just more powerful.

---

## Advantages

### Simple

No architecture changes.

Just replace hardware.

---

### No Load Balancer

Only one machine exists.

```
Users

↓

Server
```

Easy.

---

### No Data Synchronization

Everything is stored on one machine.

No replication.

No consistency issues.

---

### Easier Deployment

Deploy application only once.

---

## Disadvantages

### Hardware Limit

Every server has a maximum.

You cannot keep increasing forever.

Example

```
256 GB RAM

↓

512 GB RAM

↓

1024 GB RAM

↓

Eventually impossible
```

---

### Single Point of Failure

If server crashes

Everything stops.

```
Users

↓

Server (Dead)

↓

Website Down
```

---

### Expensive

Large servers cost significantly more than multiple smaller ones, especially at the high end.

---

### Downtime

Many hardware upgrades require restarting or replacing the server.

---

#### Where Vertical Scaling is Used

Small applications

```
Personal Blog

Portfolio

Internal Company Tool

School Management System

Inventory Software
```

Usually enough.

---

## Horizontal Scaling (Scale Out)

Horizontal Scaling means:

> Add more machines instead of making one machine bigger.

Instead of

```
1 powerful server
```

Use

```
10 normal servers
```

---

### Horizontal Scaling Diagram

Without scaling

```
          Users

             |

       +------------+
       |  Server    |
       +------------+
```

With scaling

```
              Users

                 |

         +---------------+
         | Load Balancer |
         +---------------+

          /      |      \

     Server1  Server2  Server3
```

---

## Advantages

### Almost Unlimited Growth

Need more capacity?

Add more servers.

```
1 Server

↓

2 Servers

↓

10 Servers

↓

100 Servers

↓

1000 Servers
```

---

### High Availability

If one server dies

```
Server1 ❌

Server2 ✅

Server3 ✅
```

Users continue working.

---

### Better Fault Tolerance

Traffic automatically shifts to healthy servers.

---

### Cost Efficient

Many commodity servers are often cheaper and easier to replace than a single extremely powerful machine.

---

### Zero Downtime Scaling

Need more capacity?

Launch another server.

No need to shut everything down.

---

## Disadvantages

### More Complex

Need

- Load Balancer
- Health checks
- Monitoring
- Auto Scaling
- Service discovery
- Logging
- Distributed systems concepts

---

### Data Synchronization

If users upload files to Server A

```
Photo uploaded

↓

Server A
```

Next request goes to

```
Server B
```

Photo isn't there.

Need

- shared storage
- object storage
- replication
- distributed databases

---

### Session Problem

User logs in.

```
Login

↓

Server A
```

Next request

```
↓

Server B
```

Server B doesn't know the login state if sessions are stored locally.

Solutions:

- Store sessions in a shared cache like Redis.
- Use stateless authentication such as JWT.

---

# Vertical vs Horizontal Scaling

|Feature|Vertical Scaling|Horizontal Scaling|
|---|---|---|
|Meaning|Upgrade one server|Add more servers|
|Cost Pattern|Higher-end hardware becomes very expensive|Scale incrementally by adding servers|
|Complexity|Low|High|
|Load Balancer|Not required|Required|
|Downtime|Often needed for upgrades|Usually not|
|Fault Tolerance|Poor (single machine)|Good|
|Maximum Limit|Limited by hardware|Much higher practical limit|
|High Availability|No|Yes|
|Performance|Strong single-machine performance|Better aggregate throughput|
|Used By|Small apps, databases that scale vertically|Large-scale web services|

---
-----
----

# What is a Load Balancer ?

A Load Balancer sits between clients and your servers. It distributes incoming network traffic across multiple servers to ensure no single server becomes overwhelmed, maximizing application reliability, performance, and uptime

Every request first reaches the load balancer.

It chooses which server should handle it.

---

### Why not let users choose?

Suppose users randomly connect.

```
Server A

900 Users

Server B

20 Users

Server C

30 Users
```

One server crashes while others sit mostly idle.

A load balancer prevents this by distributing traffic intelligently.

---

## Load Balancing Algorithms

There are many algorithms. The three most common are:

1. Round Robin
2. Least Connections
3. Weighted (Weighted Round Robin)

---

### 1. Round Robin

This is the simplest algorithm.

It sends each new request to the next server in order.

Servers:

```
A

B

C
```

Requests:

```
Request 1 → A

Request 2 → B

Request 3 → C

Request 4 → A

Request 5 → B

Request 6 → C
```

Pattern:

```
A

B

C

A

B

C

A

B

C
```

Every server gets an equal number of requests.

---

#### When is Round Robin good?

When:

- All servers have the same hardware.
- Requests take roughly the same amount of time.

Example:

A static website serving HTML, CSS, and images.

Each request is lightweight and similar.

---

#### When does it fail?

Suppose:

```
Server A
```

is processing a report that takes **30 seconds**.

Server B and C are idle.

Round Robin still sends the next request to Server A when its turn comes.

Now Server A becomes overloaded while others remain underutilized.

---

### 2. Least Connections

Instead of rotating blindly, the load balancer checks:

> Which server currently has the fewest active connections?

Example:

```
Server A

80 Active Connections
```

```
Server B

10 Active Connections
```

```
Server C

5 Active Connections
```

New request:

```
↓

Server C
```

because it has the least work.

---

Another request:

```
A = 80

B = 10

C = 6
```

Again:

```
↓

Server C
```

Eventually the load becomes balanced.

---

#### Best use cases

Applications where request durations vary widely, such as:

- Video uploads
- File processing
- Long-running API calls
- Chat/WebSocket connections
- Streaming services

Because some requests stay connected much longer than others.

---

#### Drawback

The load balancer must continuously track active connections on every server, making it more complex than Round Robin.

---

### 3. Weighted Round Robin

Sometimes servers are not identical.

Example:

```
Server A

32 CPU

128 GB RAM
```

```
Server B

8 CPU

16 GB RAM
```

```
Server C

4 CPU

8 GB RAM
```

Giving each server exactly one-third of the traffic would overload the smaller machines.

Instead, assign weights.

Example:

```
Server A = Weight 4

Server B = Weight 2

Server C = Weight 1
```

Traffic is distributed proportionally:

```
A

A

A

A

B

B

C
```

Then the cycle repeats.

Out of every 7 requests:

- Server A gets 4
- Server B gets 2
- Server C gets 1

---

### Best use cases

When your infrastructure contains machines with different capabilities, such as:

- Gradually upgrading servers
- Mixed cloud instance sizes
- Hybrid environments where some nodes are more powerful than others

---

### Comparison

|Algorithm|How it works|Best for|Limitation|
|---|---|---|---|
|Round Robin|Cycles through servers in order|Similar hardware and short requests|Ignores current server load|
|Least Connections|Chooses the server with the fewest active connections|Long-lived or variable-duration requests|Requires tracking connection counts|
|Weighted Round Robin|Distributes traffic according to server capacity|Servers with different hardware|Doesn't account for current load, only configured weights|

---

### Which algorithms do big companies use?

Large companies rarely rely on a single algorithm everywhere.

- **Static content (CDNs):** Often simple algorithms like Round Robin work well because requests are short and uniform.
- **REST APIs:** Weighted Round Robin is common when servers have different capacities.
- **WebSocket services (chat, gaming):** Least Connections is preferred because connections remain open for a long time.
- **Modern cloud load balancers:** They often combine multiple signals—health checks, latency, CPU utilization, active connections, and geographic proximity—to make smarter routing decisions.


----
----
----

# Forward Proxy & Reverse Proxy

### In Client → Server Architecture What if someone stands in the middle ?

Suppose someone stands here.

```
You

↓

Someone

↓

Amazon
```

Now every request passes through that person.

```
Client

↓

Middle Machine

↓

Server
```

That middle machine is called a **Proxy**.

---

### What does Proxy mean?

The English word "Proxy" means:

> Someone acting on behalf of someone else.

Example:

Suppose your boss says

> "I'm busy. Go attend the meeting on my behalf."

You become his proxy.

You represent him.

Exactly the same idea exists in networking.

A proxy represents someone.

---

### Basic Diagram

Without proxy

```
Client

↓

Server
```

With proxy

```
Client

↓

Proxy

↓

Server
```

The proxy receives the request first.

Then decides what to do.

---

### But WHY do we need it?

Excellent question.

Because the proxy can perform useful work before forwarding the request.

For example it can

- Hide identity
- Filter requests
- Cache responses
- Block websites
- Balance traffic
- Compress data
- Encrypt
- Authenticate
- Monitor

Everything passes through it.

## There are TWO types of Proxy

## Forward Proxy

Let's first understand the client.

Suppose your office has 500 employees.

Everyone wants to access Google.

```
Employee A

Employee B

Employee C

Employee D

↓

Google
```

The company doesn't want employees to directly access the internet.

Instead

```
Employee

↓

Company Proxy

↓

Google
```

---

Diagram

```
          Employees

             |

     +----------------+
     | Forward Proxy  |
     +----------------+

             |

         Internet

             |

      Google / Amazon
```

Notice carefully.

The proxy is **near the client**.

Not near Google.

This is why it is called **Forward Proxy**.

---

### Real World Example

Schools

```
Students

↓

School Proxy

↓

Internet
```

```
The proxy checks 

if Allowed -> request forword

else -> block
```
Blocked websites

- Games
- Adult websites
- Social media

The students never directly access the internet.

---

### Another example

Corporate Office

```
Employees

↓

Proxy

↓

Internet
```


Company policy may allow

```
github.com
```

Allow

```
stackoverflow.com
```

Block

```
netflix.com
```

---

### Why use Forward Proxy?

#### 1. Hide Client IP

Without proxy

```
Google sees

Your IP
```

With proxy

```
Google sees

Proxy IP
```

Your identity is hidden.

This is how many web proxies and some privacy tools work.

---

#### 2. Block websites

School

```
Student

↓

Proxy

↓

TikTok

↓

Blocked
```

---

#### 3. Cache websites

Suppose 100 employees download the same Linux ISO.

Without proxy

```
v
```

Huge bandwidth usage.

With proxy

First employee

```
Download

↓

Store in Proxy Cache
```

Next 99 employees

```
↓

Serve from Cache
```

Internet bandwidth saved.

---

#### 4. Logging

Company knows

```
Who visited

Which website

At what time
```

Useful for security and compliance.

---

#### Summary

Forward Proxy protects the **Client**.

It represents the client.

```
Client

↓

Forward Proxy

↓

Internet
```

---

## Reverse Proxy

Now let's move to the server side.

Suppose you own Amazon.

Millions of users arrive.

```
Users

↓

Amazon Server
```

Problem

Users now know

```
Real Server IP
```

Attackers can directly target it.

Also

One server can't handle all users.

So Amazon places another server in front.

```
Users

↓

Reverse Proxy

↓

Amazon Server
```

Notice carefully.

The proxy is now near the **server**.

This is Reverse Proxy.

---

Diagram

```
Users

↓

Reverse Proxy

↓

Server 1

Server 2

Server 3
```

The users never talk directly to the application servers.

---

### Why use Reverse Proxy?

#### 1. Hide Servers

Users think

```
Reverse Proxy

↓

Amazon
```

They don't know how many servers exist or their IP addresses.

---

#### 2. Load Balancing

Suppose

```
1000 Requests
```

Reverse Proxy distributes them.

```
Server A

Server B

Server C
```

This is why many load balancers (like NGINX or HAProxy) are also reverse proxies.

---

#### 3. SSL/TLS Termination

Suppose HTTPS is used.

Without reverse proxy

Every server performs encryption and decryption.

```
Server A

HTTPS

Server B

HTTPS

Server C

HTTPS
```

Instead

```
Users

↓

HTTPS

↓

Reverse Proxy

↓

HTTP (inside trusted network)

↓

Servers
```

The reverse proxy handles the expensive TLS work once, reducing load on application servers. (Some organizations keep HTTPS end-to-end depending on security requirements.)

---

#### 4. Caching

Suppose

```
Home Page
```

is requested by one million users.

Instead of

```
Every request

↓

Server
```

The reverse proxy caches the response.

```
Users

↓

Reverse Proxy

↓

Cache

↓

Server only when needed
```

The server handles far fewer requests.

---

#### 5. Security

Reverse proxies can

- Block malicious IPs
- Rate-limit requests
- Filter common attacks
- Help absorb traffic before it reaches application servers

---

### Real World Example

Suppose Netflix has

```
50 Million Users
```

Without Reverse Proxy

```
Users

↓

Application Servers
```

Every server is exposed.

Instead

```
Users

↓

NGINX

↓

Application Servers
```

NGINX (or similar software) acts as a reverse proxy.

It

- Accepts incoming requests
- Balances traffic
- Caches content
- Handles HTTPS
- Protects backend servers

---

### Forward vs Reverse Proxy

|Feature|Forward Proxy|Reverse Proxy|
|---|---|---|
|Represents|Client|Server|
|Location|Near the client|Near the server|
|Client knows proxy?|Yes|Usually no|
|Server knows proxy?|Often no|Yes|
|Purpose|Privacy, filtering, access control, caching|Load balancing, security, SSL termination, caching|
|Common users|Schools, companies, privacy services|Large websites and cloud services|

---

# Easy Way to Remember

Think about **who asked the proxy to help**.

### Forward Proxy

The **client** says:

> "Please go to the internet for me."

```
You
↓
Forward Proxy
↓
Google
```

The proxy acts on **your behalf**.

---

### Reverse Proxy

The **server** says:

> "Please receive visitors for me and decide where they should go."

```
Users
↓
Reverse Proxy
↓
Amazon Servers
```

The proxy acts on the **server's behalf**.

---

### Complete Internet Flow in a Modern Website

A request to a large website like Amazon might look like this:

```
                 Client (Browser)
                        |
                (Optional Forward Proxy)
                        |
                    Internet
                        |
              Reverse Proxy / Load Balancer
                        |
          +-------------+-------------+
          |             |             |
     App Server 1   App Server 2   App Server 3
          |             |             |
          +-------------+-------------+
                        |
                  Database Cluster
               (Primary + Read Replicas)
```

This diagram ties together the concepts you've learned:

- **Forward Proxy** (optional): protects or manages the client.
- **Reverse Proxy**: protects and manages access to servers.
- **Load Balancer**: often integrated into the reverse proxy to distribute requests.
- **Database Cluster**: scales and provides high availability for the data layer.

