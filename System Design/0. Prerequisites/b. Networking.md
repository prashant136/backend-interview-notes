

### What is Internet?

Imagine there are only **two computers**.

```
Computer A        -----------        Computer B
```

Now suppose A wants to send

```
Hello
```

to B.

Question:

**How?**

The answer is

> Through a network.

Now imagine millions of computers.

```
Laptop

Mobile

Server

Cloud VM

Database Server

Printer

TV

Smart Watch
```

All connected together.

That giant network is called

> **Internet**

So,

**Internet = Network of Networks**

---

### But how does Computer A know where Computer B is?

Every computer has an address.

Just like your house has

```
Flat 101

Building ABC

Delhi
```

A computer has

```
142.250.183.206
```

called

> **IP Address**

Example

```
My Laptop

IP = 192.168.1.10

Server

IP = 35.190.10.4
```

Every communication on Internet happens using IP addresses.

---

### Problem

Can humans remember

```
142.250.183.206
```

?

No.

Instead we remember

```
google.com
```

Later we'll learn

DNS converts

```
google.com

↓

142.250.183.206
```

---

### Suppose you type

```
www.amazon.com
```

What happens?

A LOT of things happen.

```
You type

↓

DNS finds IP

↓

TCP connection established

↓

TLS handshake (HTTPS)

↓

HTTP Request

↓

Server processes request

↓

Database query

↓

HTTP Response

↓

Browser renders page
```

This entire pipeline is the foundation of System Design.

We'll learn every box individually.

---

### First Concept

### HTTP

HyperText Transfer Protocol

People memorize the full form.

Don't.

Understand WHY it exists.

---

Imagine your friend lives in another city.

You want to ask

```
What's your age?
```

You write a letter.

```
Dear Rahul,
What is your age?
Thanks
```

Your friend replies

```
I'm 24.
```

HTTP is exactly this.

It is simply a **language (protocol)** that client and server agree to speak.

---

Example

Browser

```
GET /products
```

Server understands

"Oh, client wants products."

Server replies

```
200 OK[ Product1, Product2]
```

HTTP defines

- How request should look
- How response should look

Nothing more.

---

### Client and Server

Suppose

```
You open Instagram
```

Who sends request?

You.

```
Browser
```

is the

Client.

Who answers?

Instagram server.

```
Browser ------------> Server
           Request

Browser <------------ Server
          Response
```

---

#### HTTP Request

Suppose

```
https://amazon.com/products
```

Browser sends

```
GET /products HTTP/1.1

Host: amazon.com

User-Agent: Chrome

Accept: application/json
```

Think of it as an envelope.

```
Request

----------------

Method

URL

Headers

Body
```

---

#### Method

Method tells

"What do you want?"

Example

```
GET
```

means

Give me data.

```
POST
```

means

Create data.

```
PUT
```

means

Replace.

```
PATCH
```

means

Update partially.

```
DELETE
```

means

Delete.

---

#### Headers

Headers are metadata.

Example

```
Authorization

Language

Content-Type

Accept

Cookie
```


Headers don't contain actual message.

They contain information about message.


---

### Body

Contains actual data.

Example

```
POST /users

Body

{
   "name":"Prashant",
   "age":30
}
```

---

### Response

Server replies

```
Status Code

Headers

Body
```

Example

```
200 OK

{
  "name":"Prashant"
}
```

---

### HTTP Status Codes

Most important interview topic.

#### 2xx

Success

```
200 OK

201 Created

204 No Content
```

---

#### 3xx

Redirect

```
301 Permanent

302 Temporary
```

---

#### 4xx

Client mistake

```
400 Bad Request

401 Unauthorized

403 Forbidden

404 Not Found
```

---

#### 5xx

Server mistake

```
500 Internal Server Error

502 Bad Gateway

503 Service Unavailable
```

---

#### Is HTTP secure?

No.

Imagine sending a postcard.

Anyone handling it can read it.

```
Browser

-------------

Name

Password

-------------

Server
```

Everyone between them can read it.

This is plain HTTP.

---

### HTTPS

HTTPS = HTTP + Encryption

Think of sending the same message in a locked box instead of a postcard.

Only the intended receiver has the key.

```
Browser

Encrypted Data

====================

Server
```

Even if someone intercepts the traffic, they see only unreadable ciphertext.

---

#### Why HTTPS?

Without HTTPS

```
Username

Password

Credit Card

OTP

Cookies
```

could potentially be read or modified by someone on the network.

With HTTPS

```
ajk83hJH#8@!...
```

is all an attacker sees.

---

#### How does HTTPS achieve this?

HTTPS relies on **TLS (Transport Layer Security)** (formerly SSL) to:

- Encrypt data (confidentiality)
- Detect tampering (integrity)
- Verify the server's identity using certificates (authentication)

We'll study the TLS handshake in detail in the dedicated TLS/SSL lecture.

---

### Summary

|Concept|Meaning|
|---|---|
|Internet|Network of networks|
|IP Address|Unique address of a device|
|HTTP|Protocol for communication between client and server|
|Client|Sends request|
|Server|Processes request and sends response|
|Request|Method + URL + Headers + Body|
|Response|Status + Headers + Body|
|HTTPS|HTTP protected by TLS encryption|

---

### Notes for your interview notebook

> **HTTP (HyperText Transfer Protocol)** is an application-layer protocol that defines how clients and servers exchange requests and responses over the web. It is stateless by design and commonly runs over TCP. A request contains a method, URL, headers, and optionally a body. A response contains a status code, headers, and a body.

> **HTTPS** is HTTP running over TLS. It provides confidentiality (encryption), integrity (tamper detection), and authentication (server identity verification through digital certificates), making it suitable for sensitive communication such as logins, payments, and APIs.

-----
-----

## TCP vs UDP (what transport protocols do and why HTTP traditionally uses TCP) 3-way handshake Reliability and acknowledgments Packet loss and retransmission Why video calls often use UDP while banking apps use TCP ?


Almost every interview question about networking eventually comes down to:

> **Why did they choose TCP instead of UDP?**

After this lecture, you'll understand why **HTTP uses TCP**, **Zoom uses UDP**, and **banking apps never use UDP**.

---

## Lecture 2 - TCP vs UDP

Before learning TCP and UDP, let's understand **where they fit**.

---

|Layer|Responsibility|
|---|---|
|HTTP|Defines how requests and responses look|
|TCP / UDP|Moves data between applications|
|IP|Finds the destination device|
|Network|Physically transmits bits|

Today we'll focus on the **Transport Layer**.

---

### What is a Transport Protocol?

Imagine you're sending a parcel.

The postal service needs to decide:

- Should delivery be guaranteed?
- Should speed be prioritized?
- What if the parcel is lost?
- What if parcels arrive out of order?

TCP and UDP answer these questions for computers.

> **A transport protocol defines how data is delivered from one application to another.**

---

#### Imagine sending a 100 MB file

Suppose you upload

```
Vacation.mp4

100 MB
```

Can the Internet send all 100 MB at once?

No.

The file is broken into many small pieces called **packets**.

```
Vacation.mp4

↓

Packet 1

Packet 2

Packet 3

Packet 4

Packet 5

...
```

Each packet travels independently.

---

#### Problem

Suppose packet 3 disappears.

```
1 ✅

2 ✅

3 ❌

4 ✅

5 ✅
```

Now what?

Should we

- Ignore it?
- Wait?
- Send again?

This is where TCP and UDP differ.

---

### TCP (Transmission Control Protocol)

Think of TCP as a trusted courier service.

It promises:

- I'll deliver everything.
- I'll deliver in order.
- I'll tell you if something is lost.
- I'll resend lost packets.

Because of this, TCP is called a **reliable transport protocol**.

---

#### TCP Flow

```
Packet 1

↓

Delivered

↓

ACK

↓

Packet 2

↓

Delivered

↓

ACK

↓

Packet 3

↓

Lost

↓

Resend Packet 3

↓

Delivered

↓

ACK
```

TCP constantly verifies delivery.

---

### UDP (User Datagram Protocol)

UDP is completely different.

Imagine throwing letters into a mailbox.

```
Letter 1

Letter 2

Letter 3

Letter 4

Letter 5
```

You never ask

> "Did you receive Letter 3?"

You just keep sending.

That's UDP.

---

#### UDP Flow

```
1

2

3

4

5

↓

Receiver gets

1

2

4

5
```

Packet 3 is gone forever.

UDP doesn't care.

---

#### Why?

Because waiting is expensive.

Imagine a live football match.

If packet 100 is lost...

Should the video stop?

No.

It's better to skip one frame than freeze the entire stream.

That's why UDP is used.

---

#### TCP vs UDP Comparison

|Feature|TCP|UDP|
|---|---|---|
|Connection|Yes|No|
|Reliable|✅|❌|
|Ordered Delivery|✅|❌|
|Acknowledgments|✅|❌|
|Retransmission|✅|❌|
|Speed|Slower|Faster|
|Overhead|Higher|Lower|

---

#### Why is TCP slower?

Because it does extra work.

Imagine sending documents.

```
Sender

↓

Did you receive?

↓

Receiver

↓

Yes

↓

Next packet

↓

Did you receive?

↓

Yes
```

There are extra messages going back and forth.

UDP simply sends packets continuously without waiting.

---

### Connection-Oriented vs Connectionless

This is a common interview question.

#### TCP is Connection-Oriented

Before sending data, both sides establish a connection.

Like making a phone call.

```
You

↓

Can you hear me?

↓

Yes

↓

Let's talk.
```

Only then does communication begin.

---

### UDP is Connectionless

It's like sending SMS messages.

You simply send them.

No setup.

No confirmation.

---

### TCP Three-Way Handshake

One of the most frequently asked interview topics.

Question:

How do two computers know they are ready to communicate?

TCP uses a **3-way handshake**.

Imagine calling someone.

---

#### Step 1

Client:

```
Hi!

Can we talk?
```

This is called

```
SYN
```

---

#### Step 2

Server:

```
Yes.

I heard you.

Can you hear me?
```

This is

```
SYN + ACK
```

---

#### Step 3

Client:

```
Yes.

Let's start.
```

This is

```
ACK
```

Now the connection is established.

```
Client

SYN -------->

      <-------- SYN + ACK

ACK -------->

Connection Established
```

After this, HTTP requests can be sent.

---

### Why not 2 steps?

Suppose we only had:

```
Client

↓

SYN

↓

Server

↓

ACK
```

How does the server know the client's acknowledgment reached back?

The third step confirms:

> "Both sides can send and receive."

---

### What does SYN mean?

SYN stands for

**Synchronize**

Specifically, synchronize **sequence numbers**.

Sequence numbers let TCP know:

- which packet is first,
- which packet comes next,
- whether any packet is missing.

Without them, reliable delivery would not be possible.

---

### Reliability

Suppose we send five packets.

```
1

2

3

4

5
```

The receiver gets

```
1

2

4

5
```

Packet 3 is missing.

The receiver notices the gap and acknowledges only up to packet 2 (or repeatedly indicates it's expecting packet 3).

The sender retransmits packet 3.

```
Send

1

2

3

4

5

↓

Receive

1

2

4

5

↓

Missing 3

↓

Resend 3

↓

Receive 3

↓

Complete
```

This is **retransmission**.

---

### Acknowledgments (ACK)

Every time the receiver gets data, it responds with an acknowledgment.

Example:

```
Client

Packet 1

↓

Server

ACK

↓

Client

Packet 2

↓

Server

ACK
```

If the sender doesn't receive an ACK within a timeout, it assumes the packet was lost and sends it again.

---

#### Packet Loss

Packets can disappear because of:

- Network congestion
- Router failures
- Wireless interference
- Buffer overflows
- Physical link problems

TCP hides these problems by retransmitting missing data.

UDP leaves handling (if any) to the application.

---

### Real-World Example 1 – Banking

Suppose you transfer ₹50,000.

Would you accept this?

```
Packet

Amount = ₹50,000

↓

Lost

↓

Ignore it
```

Absolutely not.

Banking systems require:

- Every byte delivered
- Correct order
- No duplicates
- Guaranteed delivery

That's why they use TCP.

---

### Real-World Example 2 – Video Call

Imagine you're on a video call.

Frame 101 is lost.

Should the application wait?

```
Frame 100

Frame 101 ❌

Frame 102
```

If it waits for frame 101 to be resent, the video freezes.

Instead, the application skips the missing frame and continues with frame 102.

Users usually prefer a tiny visual glitch over noticeable lag.

That's why many real-time audio/video applications are built on UDP (often with additional protocols on top).

---

### Real-World Example 3 – Online Games

Position updates might look like this:

```
Player

x = 10

↓

x = 11

↓

x = 12

↓

x = 13
```

If the update for `x = 11` is lost, the game can still continue with `x = 12`.

Waiting for the missing update would make gameplay feel sluggish.

Low latency is more important than perfect delivery.

---

### Then why does HTTP traditionally use TCP?

HTTP is a request-response protocol.

Suppose you request:

```
GET /profile
```

If the response arrives as:

```
{ "name":"profile"
```

because packets were lost or reordered, the response is corrupted.

HTTP needs:

- Complete data
- Correct order
- No missing bytes

TCP provides exactly these guarantees.

Historically, **HTTP/1.1 and HTTP/2 run over TCP**.

---

### What about HTTP/3?

This is a favorite modern interview question.

Older versions:

```
HTTP

↓

TCP
```

Modern HTTP/3:

```
HTTP/3

↓

QUIC

↓

UDP
```

QUIC is a protocol built on top of UDP that adds reliability, encryption, and other features while reducing connection setup latency. So although HTTP/3 uses UDP underneath, it still provides reliable delivery to applications.

---

## Interview Questions

### Q1. Which is faster?

**UDP**, because it has less protocol overhead.

---

### Q2. Which is reliable?

**TCP**, because it uses acknowledgments, retransmissions, and ordered delivery.

---

### Q3. Why is TCP slower?

Because it spends extra time establishing connections, confirming receipt, retransmitting lost packets, and maintaining order.

---

### Q4. Which protocol does HTTP traditionally use?

**TCP** (HTTP/1.1 and HTTP/2). HTTP/3 uses QUIC over UDP.

---

### Q5. Which protocol would you choose for:

|Application|Protocol|Reason|
|---|---|---|
|Banking|TCP|Every byte must arrive correctly|
|File Download|TCP|Data integrity is essential|
|Email|TCP|Messages cannot be incomplete|
|Web Browsing|TCP|Complete HTML/CSS/JS must arrive|
|Video Streaming|Mostly UDP-based transport for real-time scenarios|Lower latency is preferred|
|Voice Calls|UDP|Small losses are acceptable|
|Live Gaming|UDP|Fast updates matter more than perfect delivery|
|DNS Lookup|Usually UDP|Small, fast request-response messages|

---

### Notes for your notebook

> **TCP (Transmission Control Protocol)** is a connection-oriented transport protocol that provides reliable, ordered, and error-checked delivery of data. It establishes a connection using the three-way handshake, uses sequence numbers and acknowledgments, and retransmits lost packets to ensure complete delivery.

> **UDP (User Datagram Protocol)** is a connectionless transport protocol that sends packets without establishing a connection or guaranteeing delivery, order, or retransmission. Its low overhead and low latency make it ideal for real-time applications such as voice calls, live streaming, and online gaming.

---
---
---

# When I type `https://google.com`, how does my computer know where Google is ?

> **When I type `https://google.com`, how does my computer know where Google is?**

And after that:

> **How does HTTPS keep my data secure?**

These two topics (DNS and TLS) are among the most frequently asked networking concepts in System Design interviews.

---

### Lecture 3 — DNS (Domain Name System)

---

### The Biggest Question

Imagine your friend asks you:

> "Open Google."

You type

```
https://google.com
```

But computers don't understand

```
google.com
```

Computers communicate using IP addresses.

Example:

```
142.250.183.206
```

Question:

**How does your computer know Google's IP?**

Answer:

**DNS**

---

### [ What is DNS ? ]

DNS stands for

**Domain Name System**

Think of DNS as the **Internet's Phone Book**.

Suppose you know

```
Rahul
```

But not his phone number.

You open your contacts.

```
Rahul

↓

+91 XXXXX XXXXX
```

DNS does exactly this.

```
google.com

↓

142.250.xxx.xxx
```

---

### Real World Analogy

Imagine ordering food.

You tell Zomato:

```
Restaurant Name

↓

Burger King
```

Zomato doesn't need you to know the GPS coordinates.

Internally it finds

```
Burger King

↓

Latitude

↓

Longitude
```

DNS works the same way.

Humans remember names.

Computers use IP addresses.

---

### Why Not Use IP Directly?

Imagine remembering

```
172.217.163.110

31.13.71.36

142.250.183.206

104.244.42.129
```

instead of

```
Google

Facebook

Twitter

Instagram
```

Impossible.

So DNS exists.

---

### Complete Journey

You type

```
www.google.com
```

Browser needs

```
IP Address
```

It asks DNS.

```
Browser

↓

DNS

↓

142.250.xxx.xxx

↓

Connect
```

---

### [ DNS Resolution ]

Let's see what actually happens.

```
User

↓

Browser

↓

Operating System

↓

DNS Resolver

↓

Root Server

↓

TLD Server

↓

Authoritative Server

↓

IP Address

↓

Browser connects
```

Looks scary.

We'll understand every step.

---

#### Step 1

You type

```
www.google.com
```

Browser asks

```
Operating System

Do you already know Google's IP?
```

OS checks cache.

```
DNS Cache
```

If found

Done.

No Internet request.

---

#### Why Cache?

Suppose you open Google 100 times today.

Without cache

```
DNS

DNS

DNS

DNS

DNS
```

Millions of unnecessary requests.

Instead

```
Cache

↓

IP found

↓

Done
```

Much faster.

---

#### If Cache Misses

OS asks

```
Recursive DNS Resolver
```

Usually provided by

- Your ISP
- Public DNS like `8.8.8.8`
- Public DNS like `1.1.1.1`

The resolver's job is:

> "I'll find the IP for you."

---

### What is a Recursive Resolver?

Imagine asking your assistant:

```
Find Rahul's phone number.
```

You don't care how.

They do all the work.

Same here.

Browser asks

```
Resolver

↓

Give me Google IP
```

Resolver starts searching.

---

### Step 1 — Root Server

DNS root servers are the foundational infrastructure of the Internet's Domain Name System (DNS). They serve as the master directory, answering queries for the root zone and directing traffic to the appropriate Top-Level Domain (TLD) servers (like `.com`, `.org`, or `.in`) to translate human-readable names into machine-readable IP addresses.

Resolver asks

```
Root Server

Where is google.com?
```

Root server replies

```
I don't know Google's IP.

But I know who manages ".com".
```

Notice:

Root servers **don't store every website's IP**.

They simply point to the correct next step.

```
Root

↓

.com server
```

---

### What is ".com"?

```
google.com

amazon.com

microsoft.com
```

Everything after the last dot is the

```
Top-Level Domain (TLD)
```

Examples:

```
.com

.org

.net

.in

.ai
```

---

### Step 2 — TLD Server

A Top-Level Domain (TLD) nameserver is the second step in the DNS resolution process, sitting between root servers and authoritative nameservers It manages all domains ending with a specific extension (e.g., `.com`, `.net`, `.org`) and directs queries to the specific authoritative server holding the website's IP address.

Resolver now asks

```
.com server

Where is google.com?
```

TLD replies

```
I don't know Google's IP.

But Google's authoritative server is here.
```

Again

No IP.

Just direction.

---

### Step 3 — Authoritative Name Server

Authoritative DNS server is the final source of truth that holds the official IP addresses and DNS records for a specific domain. It provides definitive answers to recursive DNS resolvers so user devices can successfully connect to websites and services.

Now resolver asks Google's authoritative server.

```
Where is google.com?
```

This server finally replies

```
142.250.xxx.xxx
```

This is the **source of truth** for the domain.

---

#### Resolver Returns the Answer

```
Browser

↓

Resolver

↓

Google IP

↓

Browser connects
```

The IP is also cached so future lookups are much faster.

---

### Complete Flow

```
Browser

↓

OS Cache

↓

Resolver

↓

Root

↓

TLD

↓

Authoritative

↓

IP

↓

Browser

↓

TCP Connection

↓

TLS Handshake

↓

HTTP Request
```

This is exactly what happens before the browser can even send the first HTTP request.

---

### DNS Caching

Caching happens at several levels:

```
Browser Cache

↓

Operating System Cache

↓

Recursive Resolver Cache = (
	A Recursive Resolver Cache temporarily stores the final IP addresses             of domain names discovered during previous DNS lookups. When a client requests a domain, the resolver first checks this cache. If the record exists and has not expired, the resolver instantly returns the IP address, drastically reducing lookup time.
)
```

Each cache stores results for a limited time.

---

### What is TTL?

Every DNS record has

```
TTL(Time To Live)
```

Example

```
TTL = 300 seconds
```

Meaning

```
Keep this IP 

for

5 minutes
```

After that

Ask again.

---

#### Why TTL Exists

Suppose Google changes its IP.

If everyone cached forever

Nobody could reach the new server.

TTL prevents stale information.

---

### DNS Records

DNS stores different kinds of information.

### A Record

Maps

```
Domain
↓
IPv4 Address
```

Example

```
google.com
↓
142.250.xxx.xxx
```

---

### AAAA Record

Maps

```
Domain
↓
IPv6 Address
```

---

### CNAME

Alias.

```
www.example.com
↓
example.com
```

---

### MX Record

Mail server.

```
gmail.com
↓
Mail Server
```

---

### TXT Record

Stores text information.

Commonly used for:

- Domain verification
- Email security (SPF, DKIM)
- Other metadata

---

## Interview Questions

### Why is DNS needed?

Because humans remember names, while computers communicate using IP addresses.

---

### Why cache DNS?

To reduce latency and avoid repeated lookups.

---

### Why not store every domain in the root server?

There are hundreds of millions of domains.

The DNS hierarchy keeps the system scalable.

---

### Why is DNS distributed?

No single server could handle all Internet traffic or be a single point of failure.

---

# Notes for your notebook

> **DNS (Domain Name System)** is a hierarchical and distributed naming system that translates human-readable domain names into IP addresses. A typical lookup involves checking local caches, querying a recursive resolver, contacting the root server, the appropriate top-level domain (TLD) server, and finally the authoritative name server. Results are cached according to their TTL (Time To Live) to improve performance and reduce network traffic.

## Lecture 4 — TLS / SSL

Now that DNS gave us Google's IP address, the browser is ready to connect.

Without encryption, the communication would look like this:

```
Laptop
        -------- Internet --------
Server

Username = Prashant

Password = abc123
```

Anyone intercepting the traffic could read it.

That's unacceptable for banking, shopping, or logging in.

TLS solves this problem.

---

### What is TLS?

**Transport Layer Security (TLS)** is the standard cryptographic protocol used to securely transmit data over a network.

TLS stands for

**Transport Layer Security**

Older systems used **SSL (Secure Sockets Layer)**.

SSL has known security weaknesses and has been replaced by TLS, but people still casually say "SSL certificate."

Today:

```
HTTPS = HTTP + TLS
```


-----
----
-----


# What is polling ? long and short polling ? What is WebSocket ?

### The Fundamental Problem

Imagine you're using different applications:

- Gmail
- WhatsApp
- Instagram
- Stock Market App
- Cricket Live Score
- Uber
- Google Maps

Ask yourself:

> **How does the server send new information to my phone?**

For example:

```
WhatsApp

Friend sends:
"Hello"

↓

How does your phone receive it immediately?
```

Or

```
Uber

Driver moved.

↓

How does your map update every second?
```

Or

```
IPL Match

Score changed.

↓

How does Cricbuzz update automatically?
```

This is where **Polling**, **WebSockets**, and **gRPC** come into the picture.

----
## Evolution of Communication

#  [ Polling ]

**Polling** in web development is a data-fetching technique where a client application repeatedly sends HTTP requests to a server at regular intervals to check for new data or state updates.

Think of it like checking your physical mailbox every minute.

```
Did I get a letter?

No.

1 minute later...

Did I get a letter?

No.

1 minute later...

Did I get a letter?

Yes.
```

Exactly the same idea.

---

#### Diagram

```
Client              Server

Request ----------->

Anything new?

<---------- No

(wait)

Request ----------->

Anything new?

<---------- No

(wait)

Request ----------->

Anything new?

<---------- Yes
```

Every request is a brand new HTTP request.

---

#### Real Example

Imagine Instagram notifications.

Every 30 seconds

```
GET /notifications
```

Server replies

```
[]
```

30 seconds later

```
GET /notifications
```

Again

```
[]
```

Finally

```
GET /notifications
```

Server

```
[  
	"John liked your photo"
]
```

---

#### Problem

Suppose

Server gets a new notification

after

```
2 seconds
```

Client checks only after

```
30 seconds
```

User waits

```
28 seconds
```

before seeing the notification.

Poor user experience.

---

#### Another Problem

Imagine

1 million users.

Each user checks every 5 seconds.

```
1,000,000 users

↓

Every 5 sec

↓

200,000 requests/sec
```

Most requests return

```
Nothing changed.
```

Huge waste.

---

### [ Short Polling ]

This is the polling we just discussed.

```
Client

↓

Every X seconds

↓

Send request

↓

Server responds immediately
```

Even if nothing changed.

---

#### Timeline

```
0 sec

GET

↓

Nothing

5 sec

GET

↓

Nothing

10 sec

GET

↓

Nothing

15 sec

GET

↓

Message arrived
```

Most requests are useless.

---

### [ Long Polling ]

Engineers thought,

"Instead of replying immediately..."

What if server waits?

---

#### Example

Client sends

```
GET /messages
```

Instead of saying

```
No message
```

Server keeps the request open.

```
Client ------------------>

Waiting...

Waiting...

Waiting...
```

Now Rahul sends message.

Immediately server replies.

```
<-----------------

Hello
```

Client instantly sends another request.

---

#### Diagram

```
GET

↓

Server waits...

↓

New message comes

↓

Server replies

↓

Client sends another GET
```

Notice:

Only one request is active.

---

#### Timeline

```
Client

GET ----------->

Server

(wait)

(wait)

(wait)

New message

↓

Reply

↓

Client again

GET ---------->
```

---

#### Advantages

Much fewer useless requests.

Lower latency.

Better than short polling.

---

#### Problems

Still HTTP.

Still opening many connections over time.

Still overhead.

Still not truly real-time.

---

### Then came WebSocket

Imagine telephone.

Instead of

```
Call

Hang up

Call

Hang up

Call

Hang up
```

You call once.

Keep talking.

```
Hello

How are you?

Fine

What's up?

Nothing.
```

Single connection.

Continuous communication.

That is WebSocket.

---

# [ WebSocket ]

> WebSocket is a protocol that provides continuous, two-way (full-duplex) communication between a client and a server over a single, persistent TCP connection

> WebSocket creates a persistent full-duplex communication channel.

Two keywords:

Persistent

Full Duplex

---

#### Persistent

Connection stays alive.

```
Client ---------------- ServerConnectedConnectedConnectedConnected
```

No reconnect after every message.

---

### Full Duplex

Both can send anytime.

```
Client ---------------- Server

Connected

Connected

Connected

Connected
```

No waiting.

---

### Real Chat

```
You

↓

Hello
```

Immediately

Server

↓

Stores message

↓

Pushes message

↓

Friend receives instantly.

Nobody asks

"Any message?"

Server pushes it.

---

### [ WebSocket Flow ]

```
Browser

↓

HTTP Request

Upgrade: websocket

↓

Server

↓

101 Switching Protocols

↓

Connection upgraded

↓

Now

Client <-------------> Server

Forever
```

Only first request is HTTP.

After that

No HTTP.

Pure WebSocket frames.

---

### [ Why WebSocket for Chat? ]

#### for WhatsApp

Millions of users.

Suppose every user polls every second.

```
50 million users

↓

50 million HTTP requests/sec
```

Most return

```
No new message.
```

Impossible to scale efficiently.

---

#### With WebSocket

```
50 million users

↓

50 million connections

↓

No requests

↓

Only actual messages travel.
```

Much more efficient for real-time communication.

---

### Real-world Uses of WebSocket

|Application|Why WebSocket?|
|---|---|
|WhatsApp|Instant messaging|
|Slack|Team chat|
|Discord|Voice/chat signaling|
|Online multiplayer games|Real-time player positions|
|Stock market dashboards|Live price updates|
|Crypto exchanges|Live order book changes|
|Google Docs|Collaborative editing|
|Uber|Driver location updates|
|Food delivery|Delivery tracking|
|Live sports|Ball-by-ball score updates|

---

### Polling vs Long Polling vs WebSocket

|Feature|Short Polling|Long Polling|WebSocket|
|---|---|---|---|
|HTTP|✅|✅|Only for initial handshake|
|Connection|New every request|Held until update|Persistent|
|Real-time|❌|Better|✅|
|Server Push|❌|Limited|✅|
|Latency|High|Medium|Very low|
|Network Overhead|High|Medium|Low|
|Chat Apps|Poor|Acceptable|Excellent|


## Q: Simplex, Half duplex and Full Duplex Transmission Modes ?

### Simplex (One-Way Communication)

Data flows in **only one direction**.

```
Sender  --------->  Receiver
```

- Sender can only send.
- Receiver can only receive.
- No reply is possible.

### Real-world examples

- 📺 Television broadcast
- 📻 Radio broadcast
- 🖥️ Computer → Monitor
- ⌨️ Keyboard → Computer (the keyboard sends key presses; it doesn't receive data back for typing)


---

### Half Duplex (Two-Way, But One at a Time)

**Definition:**  
Both devices can send and receive, **but not simultaneously**.

```
A  <-------->  B

(Only one side speaks at a time)
```

### Real-world example

- 🚓 Walkie-talkie
- 📡 CB Radio

Example:

```
Person A:
"Hello"

(releases button)

Person B:
"Hi!"
```

If both press the button together, communication becomes difficult.

---

### Full Duplex (Two-Way Simultaneously)

**Definition:**  
Both devices can send and receive **at the same time**.

```
A  <========>  B
```

### Real-world examples

- 📱 Phone call
- 💻 Video call
- 🌐 WebSocket communication
- 💬 WhatsApp chat (through persistent connections)

During a phone call:

```
You: "Hello"

Friend: "Hi"

(Both can speak and hear simultaneously.)
```


---

### Easy Interview Trick (Remember in 10 Seconds)

```
Simplex
A -------> B
(One-way)

Half Duplex
A <-------> B
(One at a time)

Full Duplex
A <=======> B
(Both at the same time)
```

### One-line interview definitions

- **Simplex:** Communication is only in one direction.
- **Half Duplex:** Communication is bidirectional, but only one device can transmit at a time.
- **Full Duplex:** Communication is bidirectional, and both devices can transmit and receive simultaneously.

-----------------------------------
# [ gRPC ]

This is one of the most frequently asked backend interview topics in companies like Amazon, Google, Uber, Netflix, and Microsoft.

Most people memorize:

> "gRPC is faster than REST."

This is **not enough**.

You should understand **why RPC was invented**, **why REST wasn't enough**, and **why almost every large company uses gRPC internally**.

---

#### Today's Roadmap

```
1. Why RPC was invented?
2. What is RPC?
3. Local Function Call vs Remote Function Call
4. Problem with REST
5. What is gRPC?
6. How gRPC works internally?
7. Protocol Buffers
8. HTTP/2
9. Streaming
10. Real-world Architecture
11. REST vs gRPC
12. When to use what?
13. FAANG Interview Questions
```

---

### Chapter 1 - Imagine There is No Internet

Suppose you wrote a Java program.

```
public class Calculator {

    public int add(int a, int b){
        return a + b;
    }
}
```

Now in another class,

```
Calculator calculator = new Calculator();

int result = calculator.add(10,20);

System.out.println(result);
```

Output

```
30
```

Question

What happened?

```
Main()

↓

Calls

↓

Calculator.add()

↓

Returns 30
```

Everything happens inside **one JVM**.

Everything is local.

This is called a **Local Function Call**.

No network.

No HTTP.

No JSON.

No TCP.

Just RAM.

Very fast.

---

### Now Imagine Google

Google doesn't have one Java class.

It has thousands of servers.

```
User Service

Payment Service

Notification Service

Inventory Service

Search Service

Recommendation Service
```

Each service runs on **different machines**.

Example

```
Machine A

Order Service
```

```
Machine B

Inventory Service
```

Now Order Service wants inventory.

Can it do

```
inventory.checkProduct();
```

No.

Because the Inventory Service is running on another computer.

This introduces a new challenge.

How do you call a function on another computer?

This is where **RPC** comes in.

---

### RPC

RPC stands for

> **Remote Procedure Call**

Meaning:

> Call a function that exists on another machine as if it were a local function.

---

Imagine

```
Server A

↓

calculator.add(10,20)

↓

Actually executes on

↓

Server B
```

The developer should feel like calling

```
calculator.add(10,20)
```

even though the function is running on another server.

That's the idea of RPC.

### Without RPC

Imagine writing networking manually.

```
Open TCP Socket

↓

Convert Java Object into JSON

↓

Send Bytes

↓

Receive Bytes

↓

Parse JSON

↓

Handle Errors

↓

Close Socket
```

For every request!

Very painful.

RPC hides all of this.

---

### What does RPC do?

Instead of

```
Networking

Serialization

TCP

HTTP

Socket

Parsing

Retry
```

Developer simply writes

```
inventoryService.checkStock(1001);
```

Looks local.

Actually remote.

---

#### But RPC is an Idea, Not a Product

RPC is a concept.

Many implementations exist.

Examples:

- Java RMI
- XML-RPC
- SOAP
- Apache Thrift
- gRPC

Among these,

Google created **gRPC**.

---

### [ Why did Google create gRPC ? ]

Imagine Google Search.

One search request may involve many services.

```
Search Service

↓

User Service

↓

Ranking Service

↓

Ads Service

↓

Spell Check Service

↓

Image Service

↓

Recommendation Service
```

One search could trigger dozens of internal service calls.

If every call uses verbose JSON, the overhead becomes significant.

Google wanted:

- Faster communication
- Smaller payloads
- Strong contracts
- Streaming
- High throughput

So they built **gRPC**.

---

#### REST API

Rest api is Human-readable. req/res is JSON based. JSON is text but large in size.

Suppose

```
{
   "productId":1001,
   "name":"iPhone",
   "price":80000,
   "category":"Mobile"
}
```

Everything is stored as text.

Lots of bytes.

Lots of parsing.

---

### [ gRPC Uses Protocol Buffers ]

Instead of JSON,

Google invented **Protocol Buffers (Protobuf)**.

You first define the API in a `.proto` file.

```
syntax = "proto3";

service InventoryService {

    rpc CheckStock(ProductRequest)
        returns (StockResponse);

}

message ProductRequest {

    int32 productId = 1;

}

message StockResponse {

    bool available = 1;

    int32 quantity = 2;

}
```

This file is the **contract** between client and server.

---

#### Code Generation

From that `.proto` file, tools generate Java classes automatically.

You don't manually write JSON parsing or HTTP clients.

Client code becomes:

```
StockResponse response =
inventoryStub.checkStock(
      ProductRequest.newBuilder()
                    .setProductId(1001)
                    .build());
```

It looks almost like calling a local method.

The generated client (called a **stub**) handles:

- Serialization to Protobuf
- Sending over HTTP/2
- Receiving the response
- Deserialization back into Java objects

---

### What Actually Travels?

You wrote

```
inventoryStub.checkStock(...)
```

Internally

```
Java Object

↓

Protocol Buffer

↓

Binary

↓

HTTP/2

↓

TCP

↓

Server

↓

Binary

↓

Java Object
```

You never see the networking details.

---

### Why Binary is Faster

JSON

```
{
   "name":"Prashant"
}
```

Binary

```
011010101010101...
```

Binary has several advantages:

- Smaller payload size
- Faster serialization/deserialization
- Lower CPU usage
- Lower bandwidth

Machines naturally process binary data.

Humans prefer text.

---

### HTTP/1.1 vs HTTP/2

REST commonly uses HTTP/1.1.

Imagine one road.

```
Car -> Truck -> Bike -> Bus
```

Traffic builds up.

HTTP/2 introduces multiplexing.

Think of it as multiple lanes on the same highway.

```
Lane 1

Lane 2

Lane 3

Lane 4
```

Many requests can travel simultaneously over one TCP connection.

Benefits:

- Fewer connections
- Less latency
- Better utilization

`gRPC builds on HTTP/2`

---

### Streaming

REST

```
Request -> Response -> Done
```

If you want another update:

```
New Request -> New Response
```

---

gRPC supports **streaming**.

There are four communication patterns.

#### 1. Unary RPC (Request → Response)

Exactly like REST.

```
Client -> One Request -> Server -> One Response
```

Example:

```
GetUser(101)
```

---

### 2. Server Streaming

Client sends one request.

Server sends many responses.

```
Client

↓

Give me logs

↓

Server

Log1

Log2

Log3

Log4
```

Real-world use:

- Live log viewer
- Downloading large datasets
- Continuous stock prices

---

#### 3. Client Streaming

Client sends many messages.

Server sends one final response.

```
Sensor

↓

Temperature

↓

Temperature

↓

Temperature

↓

Temperature

↓

Server

↓

Average Temperature
```

Real-world use:

- IoT sensors
- Uploading chunks of a file
- Telemetry

---

#### 4. Bidirectional Streaming

Both sides can send messages independently.

```
Client  <------------->  Server
```

Neither side has to wait for the other.

Real-world use:

- Chat
- Multiplayer games
- Real-time collaboration
- Voice/video signaling

---

### Real Example - Netflix

Suppose user opens Netflix Home Page.

```
Mobile App

↓

API Gateway

↓

User Service

↓

Recommendation Service

↓

Watch History Service

↓

Movie Metadata Service

↓

Subtitle Service

↓

Billing Service
```

One page load can involve dozens of service-to-service calls.

Internally, many of these are excellent candidates for gRPC because:

- The services are controlled by the same organization.
- Performance matters.
- Teams can share `.proto` contracts.

The public mobile app, however, often communicates with the API Gateway using REST or GraphQL because those are easier for external clients.

---

### REST vs gRPC

|Feature|REST|gRPC|
|---|---|---|
|Style|Resource-oriented (`/users/123`)|Procedure/function-oriented (`GetUser`)|
|Transport|HTTP/1.1 (commonly), HTTP/2 possible|HTTP/2|
|Data Format|JSON (usually)|Protocol Buffers (binary)|
|Payload Size|Larger|Smaller|
|Speed|Good|Faster|
|Human Readable|✅ Yes|❌ No|
|Browser Friendly|✅ Excellent|Limited without additional support|
|Strong Contract|Optional (e.g., OpenAPI)|Built-in via `.proto`|
|Code Generation|Optional|Automatic|
|Streaming|Limited|Native support|
|Best For|Public APIs|Internal microservices|

---

### When should you use REST?

Use REST when:

- Building public APIs
- Supporting browsers and third-party developers
- Human-readable payloads are useful
- Simplicity is more important than raw performance

Example:

```
Amazon Public API

GitHub API

Stripe API
```

---

### When should you use gRPC?

Use gRPC when:

- Microservices communicate frequently
- Low latency is important
- Payload size matters
- You control both client and server
- You need streaming

Example:

```
Google Search backend

Netflix internal services

Uber dispatch system - (real-time matchmaking engine. It instantly connects rider demand with available driver supply, aiming to minimize rider wait times, reduce driver pickup distances, and ensure an optimal travel experience)

Large banking microservices

Cloud infrastructure
```

---

### Interview Questions

#### Q1: If gRPC is faster, why don't we use it everywhere?

Because public APIs prioritize compatibility and ease of use. Browsers, mobile apps, curl, Postman, and countless third-party tools work naturally with REST and JSON. gRPC's binary protocol and HTTP/2 ecosystem make it ideal for controlled environments, but less convenient for general web clients.

---

### Q2: Does gRPC replace REST?

No.

They solve different problems.

- REST is excellent for exposing APIs to external consumers.
- gRPC excels at efficient communication between internal services.

Many companies use **both** in the same architecture.

---

### Q3: How does gRPC make a remote call look like a local function call?

The `.proto` file is compiled into client and server code. The generated **client stub** exposes methods such as `checkStock()`. When you call that method, the stub automatically serializes the request into Protobuf, sends it over HTTP/2, waits for the response, deserializes it, and returns a typed object. From the developer's perspective, it feels like a normal method call, even though the work happens on another machine.

----------------
-------------------------
------------------



# MQTT 

Before launching Messenger, Facebook Chat operated using a combination of **HTTP Long-Polling** on desktop browsers and the lightweight **MQTT** (Message Queuing Telemetry Transport) protocol for mobile devices.

