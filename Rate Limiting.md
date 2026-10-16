
## What is rate limiting ? Client-Side Vs Server-Side Rate Limiting ?

### Part 1: Why do we even need Rate Limiting?

Imagine you own a restaurant.

- 5 waiters
- 20 tables
- Kitchen can cook 50 meals/minute.

Now imagine one customer suddenly orders **1000 pizzas**.

What happens?

- Kitchen gets blocked.
- Other customers wait.
- Everyone gets poor service.

Should you allow one customer to order unlimited food?

**No.**

Instead you tell them

> "Maximum 5 orders every minute."

That is **Rate Limiting**.

---

Now replace

Restaurant → Server

Customer → Client

Orders → API Requests

Exactly same idea.

---

### Definition

Rate limiting is a mechanism that controls **how many requests a client is allowed to make during a given time period.**

Example

```
Maximum:
100 requests/minute
```

or

```
10 requests/second
```

If client exceeds the limit

Server returns

```
HTTP 429 Too Many Requests
```

---

### Intuition

Without rate limiting

```
Client

Request
Request
Request
Request
Request
Request
Request
Request
Request
....

↓

Server
```

Server accepts everything.

---

With rate limiting

```
Allowed

Request ✓
Request ✓
Request ✓
Request ✓
Request ✓

Blocked

Request ✗
Request ✗
Request ✗

↓

HTTP 429
```

---

### Why do companies use Rate Limiting?

Because servers have limited resources.

Every request consumes

- CPU
- Memory
- Database connections
- Network bandwidth
- Thread
- Cache
- Disk I/O

Unlimited requests can crash the system.

---

### Real-world Example 1 (Instagram)

Suppose one user refreshes feed

```
1000 times/second
```

Server has to

- Fetch posts
- Fetch likes
- Fetch comments
- Fetch stories

Millions of database queries.

One user could affect millions.

Instagram limits requests.

---

### Real-world Example 2 (Google Maps API)

Google Maps APIs are paid.

Suppose someone writes

```
while(true){
	callMapAPI();
}
```

Millions of requests.

Huge cloud bill.

So Google says

```
1000 requests/day
```

After that

```
429 Too Many Requests
```

---

### Real-world Example 3 (OTP)

Suppose login page

```
Send OTP
```

Without rate limiting

Attacker sends

```
Send OTP

100000 times
```

Problems

- SMS cost
- User receives thousands of OTPs
- Twilio/AWS SMS charges money

Therefore

```
Only 3 OTP/minute
```

---

### Real-world Example 4 (Login API)

Attacker tries

```
password1

password2

password3

...

password100000
```

This is

**Brute Force Attack**

Rate limiting

```
5 login attempts/minute
```

After that

```
429
```

---

### Real-world Example 5 (Public APIs)

GitHub

Twitter

Stripe

OpenAI

Weather APIs

Everyone rate limits APIs.

Otherwise one customer can consume all server resources.

---

### What exactly can we limit?

Many people think rate limiting is only per user.

No.

We can limit based on many things.

---

#### Per User

```
User A

100 requests/minute
```

---

#### Per IP

```
192.168.x.x

50 requests/minute
```

Useful before login.

---

#### Per API

```
Login API

10 requests/minute

Search API

100 requests/minute

Payment API

5 requests/minute
```

---

#### Per API Key

Used by public APIs.

```
API KEY

5000 requests/day
```

---

#### Per Organization

Example

Slack Workspace

```
100000 requests/hour
```

shared by organization.

---

#### Per Device

Example

Mobile device

```
200 requests/hour
```

---

#### What happens when limit exceeds?

Server usually returns

```
HTTP 429
Too Many Requests
```

Often with headers

```
Retry-After: 60
```

Meaning

Wait

```
60 seconds
```

before retrying.

---

### Client-side vs Server-side Rate Limiting

Many beginners confuse these.

---

#### Client-side Rate Limiting

Client itself decides

> "I won't send too many requests."

Example

Search Box

You type

```
H

He

Hel

Hell

Hello
```

Without client-side limiting

Browser sends

```
/search?q=H

/search?q=He

/search?q=Hel

/search?q=Hell

/search?q=Hello
```

Five requests.

Instead browser waits

```
300 ms
```

Then sends only

```
Hello
```

This is often achieved with **debouncing** or **throttling** in the frontend.

Benefits

- Saves bandwidth
- Better UX
- Reduces server load

But...

Can attacker ignore it?

Yes.

Because attacker writes

```
curl

or

Postman

or

Python script
```

Client-side protection disappears.

Therefore

**Never rely only on client-side rate limiting for security.**

---

### Server-side Rate Limiting

Server tracks requests.

Regardless of

- Browser
- Curl
- Postman
- Python
- Hacker

Everyone goes through server.

Server decides

```
Allowed?

Yes → Process

No → 429
```

This is the real protection.

---

### Comparison

|Client-side|Server-side|
|---|---|
|Browser controls|Server controls|
|Easy to bypass|Cannot be bypassed easily|
|Reduces unnecessary traffic|Protects backend resources|
|Improves UX|Provides security and fairness|
|Optional|Essential|

---

### Types of Rate Limiting

Generally two broad categories.

#### 1. Fixed Rate

```
100 requests/minute
```

Simple.

---

#### 2. Dynamic Rate

Different limits.

Example

Free User

```
100/day
```

Premium

```
10000/day
```

Admin

Unlimited

Very common.

---

### Where is Rate Limiting usually implemented?

Many places.

```
Browser

↓

CDN

↓

API Gateway  ← very common

↓

Load Balancer

↓

Application

↓

Database
```

Usually companies implement at

- API Gateway
- Reverse Proxy
- Application

before requests reach expensive backend processing.

---

### Rate Limiting vs Throttling

Many people use them interchangeably, but there's a subtle difference.

#### Rate Limiting

Controls **how many** requests are allowed over a period.

Example:

```
100 requests/minute
```

The 101st request is rejected with `429 Too Many Requests`.

#### Throttling

Controls the **speed** at which requests are processed.

Example:

```
1000 requests arrive at once
```

Instead of rejecting them immediately, the system may process only:

```
10 requests/second
```

The remaining requests are delayed, queued, or slowed down.

A simple analogy:

- **Rate limiting**: A movie theater allows only the first 100 people inside; everyone else must wait or leave.
- **Throttling**: The theater lets everyone in eventually but only allows 10 people through the entrance every second to avoid crowding.

---

### Common Interview Questions

#### Q1. Why not simply scale servers instead of rate limiting?

Scaling helps, but resources are still finite and expensive. Without rate limiting, a malicious or buggy client can keep consuming the additional capacity as well. Rate limiting protects availability, controls costs, and ensures fair usage.

---

#### Q2. Can rate limiting stop DDoS attacks?

It helps against some abuse patterns, especially from individual clients or IPs, but by itself it is **not sufficient** against large distributed attacks. DDoS protection typically combines rate limiting with CDNs, Web Application Firewalls (WAFs), traffic filtering, and specialized mitigation services.

---

#### Q3. Is rate limiting only for public APIs?

No. Internal microservices often rate-limit requests too, preventing one overloaded service from cascading failures to others.

---

#### Big Picture

```
                Client
                   │
                   ▼
            API Gateway
                   │
          ┌────────────────┐
          │ Rate Limiter   │
          │ Count Requests │
          └────────────────┘
             │         │
       Under Limit   Exceeded
             │         │
             ▼         ▼
      Forward to API   HTTP 429
             │
             ▼
         Backend Service
```

#### Key Takeaways

- **Rate limiting** controls how many requests are allowed within a time window.
- It protects servers from abuse, accidental overload, brute-force attacks, and excessive costs.
- **Client-side** rate limiting improves user experience but is not a security measure because it can be bypassed.
- **Server-side** rate limiting is the authoritative enforcement point.
- Limits can be applied per user, IP, API, API key, device, or organization.
- When the limit is exceeded, servers typically return **HTTP 429 Too Many Requests**.
- Different algorithms (such as Fixed Window, Sliding Window, Token Bucket, and Leaky Bucket) implement rate limiting with different trade-offs.

##
------
##

## Evolution of Rate Limiting Algorithms

Think of it as solving problems one by one.

```
No Rate Limiting
        │
        ▼
Fixed Window
        │
        │ Problem: Burst at window boundary
        ▼
Sliding Window Log
        │
        │ Problem: Uses too much memory
        ▼
Sliding Window Counter
        │
        │ Better memory usage
        ▼
Token Bucket
        │
        │ Allows controlled bursts
        ▼
Leaky Bucket
        │
        │ Smooths traffic completely
        ▼
Modern Systems
```

Notice that **every new algorithm was invented because the previous one had a drawback.**

---

## [ Algorithm 1 : Fixed Window Counter ]

This is the simplest algorithm.

### Intuition

Imagine a cinema.

The manager says

> "Only 100 people can enter every hour."

He resets the count exactly every hour.

```
2 PM ---------------------- 3 PM

Allowed = 100
```

At exactly 3 PM

```
Count becomes 0
```

Everyone can enter again.

---

### Server Perspective

Suppose

```
Limit

5 requests/minute
```

Window

```
12:00:00 → 12:00:59
```

Maintain only one counter.

```
user123

Count = 0
```

Whenever request comes

```
Count++
```

If

```
Count <= 5

Allow
```

Else

```
Reject
```

At

```
12:01:00
```

Reset

```
Count = 0
```

That's it.

Very simple.

---

### Example

Limit

```
5 requests/minute
```

Timeline

```
12:00:05  ✓

Count=1

12:00:10 ✓

Count=2

12:00:20 ✓

Count=3

12:00:40 ✓

Count=4

12:00:55 ✓

Count=5
```

Now another request

```
12:00:58
```

Count

```
6

Rejected
```

At

```
12:01:00
```

Counter resets

Now

```
Count = 0
```

Next request

```
Allowed
```

---

### Visualization

```
Window 1

0--------------------------------60 sec

✓ ✓ ✓ ✓ ✓ ✗ ✗ ✗

Reset

Window 2

0--------------------------------60 sec

✓ ✓ ✓ ✓ ✓
```

---

#### Data Structure

Very little memory.

```
HashMap

User

↓

Counter

↓

Reset Time
```

Example

```
{
	UserA
	
	Count = 4
	
	Expires = 12:01
}
```

Only one integer per user.

Very cheap.

---

### Complexity

```
Memory

O(number of users)
```

Request

```
O(1)
```

Very fast.

---

### But there is a huge problem...

Imagine

Limit

```
5 requests/minute
```

Timeline

```
12:00:55 ✓

12:00:56 ✓

12:00:57 ✓

12:00:58 ✓

12:00:59 ✓
```

Limit reached.

Now

Counter resets.

```
12:01:00
```

Immediately user sends

```
12:01:00 ✓

12:01:01 ✓

12:01:02 ✓

12:01:03 ✓

12:01:04 ✓
```

How many requests?

```
10 requests
```

How much time?

```
10 seconds
```

Although limit was

```
5/minute
```

The server actually accepted

```
10 requests in ~10 seconds
```

This is called the **Boundary Problem** or **Burst at Window Boundary**.

---

### Why is this bad?

Imagine payment service.

```
Limit

100 requests/minute
```

Attacker sends

```
100 requests at 12:00:59
```

Then immediately

```
100 requests at 12:01:00
```

Server suddenly receives

```
200 requests
```

Although your limit says

```
100/minute
```

The rate limiter technically followed its rules, but it allowed an undesirable burst.

---

### Advantages

✅ Extremely simple.

✅ Very fast.

✅ Minimal memory.

✅ Easy to implement using Redis.

---

### Disadvantages

❌ Burst problem.

❌ Unfair near window boundaries.

❌ Doesn't represent a true "last 60 seconds" rate.

---

### Interview Question

**Q: Why is Fixed Window not ideal for production?**

Answer:

Because it allows bursts at window boundaries. A client can send requests just before the window resets and immediately after it resets, effectively doubling the intended rate in a very short period.

---

## [ Algorithm 2 : Sliding Window Log ]

Engineers saw the boundary problem and asked:

> "Instead of resetting every minute, why don't we always look at the last 60 seconds?"

That's how Sliding Window Log was born.

---

### Intuition

Imagine a security guard.

Instead of saying

```
Current minute
```

he says

```
Show me everyone who entered in the LAST 60 seconds.
```

Notice the difference.

Not

```
12:00–12:01
```

Instead

```
Current Time - 60 sec
```

The window moves continuously.

Hence the name

**Sliding Window**.

---

### Example 

Limit

```
5 requests / 60 sec
```

Suppose current time

```
12:01:10
```

Now look backward

```
12:00:10 ----> 12:01:10
```

Only requests inside this moving window count.

Anything older is ignored.

---

### Data Structure

Instead of storing a counter...

Store every request timestamp.

Example

```
UserA

[
	12:00:15
	
	12:00:25
	
	12:00:40
	
	12:00:58
]
```

Each incoming request:

1. Remove timestamps older than 60 seconds.
2. Count remaining timestamps.
3. If count < limit, allow and append the new timestamp.
4. Otherwise reject.

---

### Visualization

Limit

```
5 requests
```

Timeline

```
12:00:20

12:00:30

12:00:40

12:00:50

12:01:00
```

Current time

```
12:01:10
```

Window

```
12:00:10 → 12:01:10
```

All five timestamps are still inside.

Next request

```
Rejected
```

Five seconds later...

Current time

```
12:01:15
```

Window becomes

```
12:00:15 → 12:01:15
```

Now the oldest request falls out of the window.

Count becomes

```
4
```

The next request is allowed.

Notice there is **no sudden reset**.

The limit changes smoothly over time.

---

### Why is it better?

Remember the Fixed Window burst problem?

Now imagine:

```
12:00:59
```

Five requests arrive.

At

```
12:01:00
```

Another five arrive.

Sliding Window still looks back over the last 60 seconds, so those earlier five requests are still counted.

The second batch is rejected.

Problem solved.

---

### Complexity

Memory:

```
O(number of requests within the window)
```

If a user can send 10,000 requests in a minute, you may need to store 10,000 timestamps for that user.

Processing:

- Remove expired timestamps.
- Add new timestamp.

Typically efficient with a queue or deque, but still more expensive than Fixed Window.

---

### Drawback

Memory.

Imagine:

```
10 million users
```

Each stores hundreds of timestamps.

Memory usage becomes huge.

This is why pure Sliding Window Log is rarely used at very large scale.

---

### Real-world Use Cases

Sliding Window Log is useful when:

- Accuracy is critical.
- Request volume is moderate.
- Memory usage is acceptable.

Examples:

- Login attempts
- Password reset endpoints
- Financial APIs
- Sensitive administrative operations

---

### Advantages

✅ Very accurate.

✅ No boundary burst problem.

✅ Fair to all users.

---

### Disadvantages

❌ High memory usage.

❌ More CPU work than Fixed Window.

❌ Doesn't scale well when request volume is extremely high.

---

### Comparison So Far

|Feature|Fixed Window|Sliding Window Log|
|---|---|---|
|Memory|Very low|High|
|Speed|Very fast|Slower|
|Accuracy|Low|Very high|
|Boundary burst|Yes|No|
|Stores|Counter|Individual timestamps|
|Best for|Simple, high-speed limits|Accurate rate limiting|

---

### Why did engineers invent the next algorithm?

Sliding Window Log fixed the **accuracy** problem, but introduced a **memory** problem.

The next algorithm, **Sliding Window Counter**, keeps almost the same fairness while using far less memory by storing **counts instead of every timestamp**.

In the next lesson, we'll cover **Sliding Window Counter** in depth, including:

- Why it doesn't need to store every request.
- How it estimates the current request rate using two windows.
- The weighted calculation (the part many candidates struggle to explain in interviews).
- Real-world examples, Redis-friendly implementation intuition, and interview questions.

------------------------

## [ Algorithm 3 : Sliding Window Counter ]

Many companies don't use **Sliding Window Log** because storing every request timestamp is expensive.

Engineers thought:

> **"Can we get almost the same accuracy without storing every request?"**

The answer is **Sliding Window Counter** (also called **Sliding Window Approximation**).

-----------------
### Why was it invented?

Let's recap.
#### Fixed Window

Stores

```
UserA → Count = 45
```

Pros

- Very little memory
- Very fast

Problem

- Boundary burst

---

#### Sliding Window Log

Stores

```
UserA →

10:00:01
10:00:05
10:00:09
10:00:11
10:00:15
...
```

Pros

- Perfectly accurate

Problem

Imagine

```
1000 requests/user/minute
```

For

```
1 million users
```

You may need to store **billions of timestamps**.

Memory becomes huge.

---

So engineers asked

> "Do we really need every timestamp?"

Answer

**No.**

We only need a **good estimate** of how many requests occurred in the last minute.

---

### Core Idea

Instead of storing

```
10:00:01

10:00:03

10:00:05

10:00:08

10:00:10
```

Store only

```
Current Window Count

Previous Window Count
```

That's it.

Only **two integers**.

Huge memory saving.

---

### Intuition

Imagine you're standing at 10:00:40.

Window size

```
60 seconds
```

Instead of checking every request,

you ask

> "How many requests happened this minute?"

and

> "How many happened in the previous minute?"

Then estimate how much of the previous minute still overlaps.

---

### Visual Timeline

Suppose each window is **60 seconds**.

```
Previous Window            Current Window

9:59 ----------------10:00----------------10:01

        Previous             Current
```

Current time

```
10:00:30
```

Notice something.

The last 60 seconds are

```
9:59:30 -----------10:00:30
```

That period includes

- half of previous window
- half of current window

So we should count

- ALL current requests
- ONLY HALF of previous requests

This is the entire algorithm.

---

#### Example

Limit

```
100 requests/minute
```

Current time

```
10:00:30
```

Suppose

Previous window

```
9:59-10:00

Count = 80
```

Current window

```
10:00-10:01

Count = 30
```

Should we simply add

```
80 + 30 = 110 ?
```

No.

Why?

Because half of those 80 requests happened before 9:59:30.

Those are already outside our last 60 seconds.

We should count only the overlapping portion.

---

### Calculate Overlap

Current time

```
10:00:30
```

Window started

```
9:59:30
```

Previous window

```
9:59 --------10:00
```

How much overlaps?

```
30 seconds
```

Out of

```
60 seconds
```

Overlap percentage

```
30 / 60 = 0.5
```

Meaning

Use only

```
50%
```

of previous count.

---

### Formula

Estimated Count

```
Current Count

+

Previous Count × Overlap Percentage
```

---

Our numbers

Current

```
30
```

Previous

```
80
```

Overlap

```
0.5
```

Estimated count

```
30 + (80 × 0.5) = 70
```

Estimated requests

```
70
```

Since

```
70 < 100
```

Allow request.

---

### Another Example

Current time

```
10:00:50
```

Window

```
9:59:50 ----10:00:50
```

Now only

```
10 seconds
```

of previous window overlap.

Overlap

```
10 / 60 = 0.166
```

Previous count

```
60
```

Contribution

```
60 × 0.166 ≈ 10
```

Suppose current count

```
40
```

Estimated

```
40 + 10 = 50
```

See?

As time moves forward,

the previous window contributes less and less.

Eventually

```
10:01:00
```

Previous contribution

```
0
```

Very smooth.

---

### Visualization

```
Current Time

10:00:10

Previous Window Contribution

███████████████████████

83%

-------------------------

10:00:20

████████████████

66%

-------------------------

10:00:30

██████████

50%

-------------------------

10:00:40

████

33%

-------------------------

10:00:50

█

16%

-------------------------

10:01:00

0%
```

Notice there is **no sudden reset** like Fixed Window.

The previous window fades away gradually.

---

### Why doxes this work?

Think about rainfall.

Suppose someone asks

> "How much rain fell in the last hour?"

You don't know every raindrop.

But you know

```
9–10 AM

20 mm
```

and

```
10–11 AM

10 mm
```

At

```
10:30
```

you estimate

```
Half of 9–10 + Half of 10–11
```

Not exact.

But very close.

Exactly the same idea.

---

### Data Structure

Instead of storing

```
[

10:00:01

10:00:03

10:00:08

10:00:10

]
```

Store

```
UserA

Current Count

Current Window Start

Previous Count
```

Only

```
3 numbers
```

Memory becomes tiny.

---

### Complexity

Memory

```
O(number of users)
```

Same as Fixed Window.

Request processing

```
O(1)
```

Very fast.

---

### Is it perfectly accurate?

No.

It assumes requests are **uniformly distributed** inside each window.

Reality isn't always like that.

Example:

Previous window

```
100 requests
```

What if

all 100 arrived in the **last second**?

Our algorithm still estimates using overlap percentage.

So the estimate can be slightly wrong.

This is why it's called

> **Sliding Window Approximation**.

---

### Real Example

Imagine

```
GitHub API

5000/hour
```

GitHub doesn't necessarily care whether the estimate is off by 2 or 3 requests.

What matters is

- fairness
- speed
- scalability

Sliding Window Counter is often a great fit for this kind of workload.

---

### Advantages

✅ Almost no burst problem.

✅ Tiny memory.

✅ Very fast.

✅ Much more scalable than Sliding Window Log.

✅ Smooth rate limiting.

---

### Disadvantages

❌ Slightly approximate.

❌ Can be inaccurate if requests are heavily clustered within a window.

❌ More complex than Fixed Window.

---

### Fixed Window vs Sliding Log vs Sliding Counter

|Feature|Fixed Window|Sliding Window Log|Sliding Window Counter|
|---|---|---|---|
|Memory|⭐ Very Low|❌ High|⭐ Very Low|
|Accuracy|❌ Low|⭐ Perfect|✅ Very Good|
|Boundary Burst|❌ Yes|✅ No|✅ Almost None|
|Speed|⭐ Fast|❌ Slower|⭐ Fast|
|Stores|One Counter|Every Timestamp|Two Counters|
|Complexity|Easy|Medium|Medium|

---

### Which one do companies use?

It depends on the use case:

#### Fixed Window

Use when:

- Simplicity is most important.
- Small bursts are acceptable.
- Internal tools or admin dashboards.

---

#### Sliding Window Log

Use when:

- Every request matters.
- Moderate request volume.
- Security-sensitive APIs like login or password reset.

---

#### Sliding Window Counter

Use when:

- High traffic.
- Millions of users.
- You need a balance of fairness, speed, and memory efficiency.

---

### Interview Questions

#### Q1. Why is Sliding Window Counter better than Sliding Window Log?

**Answer:** It reduces memory usage dramatically by storing only the current and previous window counts instead of every request timestamp. It achieves nearly the same fairness with much better scalability.

---

#### Q2. Why isn't it perfectly accurate?

**Answer:** It estimates the contribution from the previous window based on the overlap percentage, assuming requests are evenly distributed within that window. If requests are clustered, the estimate can differ slightly from the actual count.

---

#### Q3. If Sliding Window Counter is so good, why do we need Token Bucket?

Because **Sliding Window Counter still focuses on enforcing a maximum request rate**. It doesn't handle **short, legitimate bursts** gracefully.

Imagine a user who hasn't made any requests for a minute and suddenly needs to send five requests at once (for example, loading a dashboard that triggers multiple API calls). A strict sliding window may reject some of them even though the user was idle before.

**Token Bucket** introduces the idea of **earning credits (tokens) while idle**, allowing controlled bursts without exceeding the long-term average rate.

That's why Token Bucket is one of the most widely used algorithms in API gateways, cloud services, and networking.

---

### Where we're headed next

The next algorithm, **Token Bucket**, is arguably the most important one to understand for interviews. We'll cover:

- The bucket-and-token intuition.
- Why idle users can "save up" capacity.
- How bursts are allowed while maintaining an average rate.
- Real-world examples from APIs, Kubernetes, networking, and cloud platforms.
- Why many production systems (such as API gateways) prefer Token Bucket over window-based algorithms.

------

## [ Algorithm 4 :  Token Bucket ]

**Token Bucket** is probably the **most important rate limiting algorithm** for interviews.

If an interviewer asks:

> **"Which rate limiting algorithm is most commonly used in production?"**

A very strong answer is:

> **"Token Bucket is one of the most widely used because it allows controlled bursts while still enforcing a long-term average request rate."**

Many API Gateways (NGINX, Envoy), cloud services, networking equipment, Kubernetes components, and distributed systems use Token Bucket or a variation of it.

---

### Why do we need another algorithm?

Let's first understand the limitation of all previous algorithms.

Suppose the limit is

```
60 requests/minute
```

Meaning

```
1 request/second
```

Imagine a user does **nothing** for one minute.

```
10:00 ---------------- 10:01

No Requests
```

Now at **10:01**

the user opens your dashboard.

The browser immediately sends

```
/users

/profile

/orders

/notifications

/messages
```

Five API calls almost simultaneously.

This is completely normal.

But a strict window-based algorithm might reject some of them because it only looks at the current request rate.

---

### Think about real life

Imagine a parking lot.

Every minute

the parking lot receives permission for

```
10 cars
```

Suppose

for the last hour

nobody came.

Should the first 10 cars arriving together be rejected?

No.

Since the parking lot was empty,

it makes sense to allow them.

That is exactly the idea behind Token Bucket.

---

### Core Idea

Instead of counting requests,

we store **tokens**.

Think of tokens as **permission slips**.

One request needs

```
1 Token
```

No token?

No request.

---

### The Bucket

Imagine a bucket.

```
          _________
         |         |
         |         |
         | Tokens  |
         |         |
         |_________|
```

Tokens continuously fall into the bucket.

Like water dripping from a tap.

---

Example

Bucket capacity

```
10 Tokens
```

Token generation rate

```
1 Token/second
```

Initially

```
Bucket10 Tokens
```

(full bucket)

---

#### Rule 1

Every request consumes

```
1 Token
```

---

Suppose

Bucket

```
10 Tokens
```

Request arrives

```
Consume1 Token
```

Remaining

```
9 Tokens
```

Easy.

---

#### Rule 2

Tokens regenerate automatically.

Example

Every second

```
+1 Token
```

If bucket isn't full,

a new token is added.

---

Suppose

Current

```
7 Tokens
```

After

```
1 second
```

Bucket becomes

```
8 Tokens
```

---

#### Rule 3

Bucket has a maximum size.

Suppose

Capacity

```
10 Tokens
```

Current

```
10 Tokens
```

After one second

Can we have

```
11 Tokens?
```

No.

Maximum stays

```
10
```

Extra tokens are discarded.

Think of an overflowing water bucket.

---

### Complete Example

Capacity

```
5 Tokens
```

Generation

```
1 Token/second
```

Initially

```
Bucket

★★★★★

5 Tokens
```

---

User sends

3 requests

```
Req1 ✓

Req2 ✓

Req3 ✓
```

Bucket

```
★★

2 Tokens
```

---

Now

User waits

```
3 seconds
```

During waiting

Bucket refills

```
★★

↓

★★★★★

5 Tokens
```

Now user again sends

```
5 Requests
```

All succeed.

Why?

Because while idle,

the bucket refilled.

---

### Timeline Example

Let's go step by step.

Capacity

```
5
```

Refill

```
1 token/sec
```

### Time = 0

```
Tokens

★★★★★

5
```

---

##### User sends 4 requests

```
★★★★

Consumed
```

Remaining

```
★

1 Token
```

---

##### Wait 2 seconds

Bucket

```
★

↓

★★★

3 Tokens
```

---

User sends

```
2 Requests
```

Remaining

```
★

1 Token
```

---

Wait

```
4 Seconds
```

Bucket

```
★★★★★

5
```

Back to full.

---

### [ Burst Handling ]

This is why Token Bucket is so powerful.

Suppose

Capacity

```
20 Tokens
```

User remains idle

```
20 seconds
```

Bucket becomes full.

Now

User suddenly sends

```
20 Requests
```

All succeed immediately.

Because

they already earned those tokens.

---

Without Token Bucket,

many of these requests might have been rejected.

---

### Visualization

```
Time

0

Bucket

★★★★★

↓

User sends 5 requests

Bucket

_____

↓

Idle

↓

Tokens refill

★

↓↓↓

★★★

↓↓↓

★★★★★
```

Notice

Idle time is rewarded.

---

### Mathematical View

Bucket Capacity

```
B
```

Token Generation Rate

```
R Tokens/sec
```

Current Tokens

```
T
```

Whenever a request comes

First

Calculate

```
Elapsed Time
```

New Tokens

```
Elapsed Time × R
```

Update

```
Tokens = Min(Capacity, Old Tokens + New Tokens)
```


---

### Real World Example

Imagine

Google Drive.

You haven't uploaded anything

for

```
30 minutes
```

Now you select

```
20 files
```

Should the server reject them?

No.

Token Bucket allows that burst because you accumulated tokens while idle.

---

### Another Example

GitHub API

Suppose

```
5000 requests/hour
```

You make

no requests

for

30 minutes.

Your token bucket fills.

Now you run a script.

Hundreds of API calls happen.

Allowed.

Why?

Because you saved your tokens.

---

### OTP Example

Capacity

```
3 Tokens
```

Generation

```
1 token/minute
```

Initially

```
★★★

3 Tokens
```

User requests

3 OTPs.

Allowed.

Bucket

```
Empty
```

Fourth OTP?

Rejected.

Wait

```
1 minute
```

Bucket

```
★

1 Token
```

Now one more OTP is allowed.

---

### Data Structure

Per user we only store:

```
User

↓

Current Tokens

↓

Last Refill Timestamp
```

Example

```
UserA

Tokens = 7

Last Updated = 10:01:35
```

That's all.

Very memory efficient.

---

### Complexity

Memory

```
O(number of users)
```

Processing

```
O(1)
```

Excellent for millions of users.

---

### Why is it better than Sliding Window?

Suppose

Limit

```
60 requests/minute
```

Sliding Window

```
1
1
1
1
...
```

Very strict.

Token Bucket

```
Idle

↓

Tokens saved

↓

Burst allowed

↓

Refill again
```

Much more natural for real applications.

---

### Advantages

✅ Allows short bursts.

✅ Prevents long-term abuse.

✅ Very little memory.

✅ Constant-time processing.

✅ Excellent scalability.

✅ Works beautifully for APIs.

---

### Disadvantages

❌ Slightly harder to implement than Fixed Window.

❌ Choosing the right bucket size and refill rate requires tuning.

❌ If the bucket is too large, users can create larger bursts than you intended.

---

### Token Bucket vs Sliding Window Counter

|Feature|Sliding Window Counter|Token Bucket|
|---|---|---|
|Idle user gets extra capacity?|❌ No|✅ Yes|
|Allows bursts?|Limited|✅ Yes|
|Memory|Low|Low|
|Speed|Fast|Fast|
|Fairness|High|High|
|Best for|Strict API rate limits|User-facing APIs, API gateways, networking|

---

### Where is Token Bucket used?

Token Bucket (or close variants) is commonly used in:

- **API Gateways** (e.g., NGINX, Envoy)
- **Cloud APIs** (AWS, Google Cloud, Azure)
- **Networking** (routers and switches for traffic shaping)
- **Kubernetes** (client request throttling)
- **Microservices** (protecting downstream services)
- **Public APIs** (GitHub, Stripe, many SaaS platforms)

The exact implementation varies, but the underlying concept is the same.

---

### Interview Questions

#### Q1. Why is Token Bucket preferred over Fixed Window?

Because it allows legitimate short bursts while still enforcing a long-term average request rate. Fixed Window may reject bursty but valid traffic or allow unfair boundary bursts.

---

#### Q2. What do the two configuration parameters mean?

- **Bucket Capacity**: The maximum number of tokens that can be stored. It determines the largest burst that can be handled immediately.
- **Refill Rate**: How quickly new tokens are generated. It determines the sustained average request rate over time.

For example:

```
Capacity = 20 tokens
Refill Rate = 5 tokens/second
```

This means:

- A user can make **up to 20 immediate requests** if the bucket is full.
- After that, the user can continue at an average rate of **5 requests per second**.

---

#### Q3. Why not just refill the bucket every second?

In production systems, we **don't run a timer for every user**. Imagine millions of users—it would waste CPU.

Instead, we use **lazy refill**:

When a request arrives:

1. Check the current time.
2. Calculate how much time has elapsed since the last refill.
3. Add the appropriate number of tokens (up to the bucket capacity).
4. Process the request.

This gives the same behavior without constantly updating every bucket.

-----

## [ Algorithm 5 :  Leaky Bucket  ]

Many people confuse **Token Bucket** and **Leaky Bucket** because both use the word _bucket_.

In reality, they solve **different problems**.

- **Token Bucket** → Controls **how fast requests are allowed to enter** the system while allowing bursts.
- **Leaky Bucket** → Controls **how fast requests leave** the system, producing a smooth and constant output.

This distinction is one of the most common interview questions.

---

### Before Learning Leaky Bucket

Let's compare the mindset.

#### Token Bucket says

> "You can come in quickly if you've saved tokens."

Example

```
Idle for 20 seconds

↓

Saved 20 tokens

↓

20 requests arrive

↓

All 20 accepted immediately
```

Burst is allowed.

---

#### Leaky Bucket says

> "I don't care how many requests arrive. I'll process them at a fixed speed."

Example

```
20 requests arrive

↓

Process

1 request/second
```

Burst is **not** allowed.

Everything becomes smooth.

---

### Why was Leaky Bucket invented?

Imagine Netflix.

Millions of users click **Play** simultaneously.

If every request immediately reaches the backend,

the database suddenly receives

```
100000 requests
```

Even if the database can eventually handle them,

that sudden spike may overload CPU, connections, or caches.

Instead, we would like to smooth the traffic.

Instead of

```
100000 requests NOW
```

We prefer

```
100 requests/sec

100 requests/sec

100 requests/sec
```

Same total work.

Much smoother.

This is the purpose of Leaky Bucket.

---

### Real-Life Analogy

Imagine a bucket with a **small hole** at the bottom.

Water is poured into the top.

```
          Water

       ↓↓↓↓↓↓↓↓↓

      ┌──────────┐
      │          │
      │          │
      │          │
      └────┬─────┘
           │
           │
          Drop
```

Water can enter very quickly.

But it always leaves

```
Drop...

Drop...

Drop...
```

at a constant speed.

Exactly the same happens with requests.

---

### Bucket becomes Queue

Unlike Token Bucket,

this bucket stores

**requests**

instead of

**tokens**.

```
Incoming Requests

↓

Bucket (Queue)

↓

Constant Processing Rate

↓

Backend
```

---

### Core Idea

Requests arrive

```
R1

R2

R3

R4

R5
```

They are stored in a queue.

The server processes

```
1 request/sec
```

regardless of arrival speed.

---

### Example

Queue Capacity

```
5 Requests
```

Leak Rate

```
1 request/second
```

Initially

```
Queue

Empty
```

---

Suddenly

Five requests arrive

```
R1

R2

R3

R4

R5
```

Queue

```
R1

R2

R3

R4

R5
```

Full.

---

Now

Every second

one request leaves.

#### Second 1

```
Processed

R1
```

Queue

```
R2

R3

R4

R5
```

---

#### Second 2

```
Processed

R2
```

Queue

```
R3

R4

R5
```

---

Eventually

Queue becomes empty.

Notice

Backend never saw

```
5 requests together.
```

Instead it saw

```
1/sec

1/sec

1/sec
```

Very smooth.

---

### What if Queue is Full?

Suppose

Queue size

```
5
```

Already full.

Another request arrives.

```
Queue

R1

R2

R3

R4

R5

+

R6
```

Cannot fit.

Two possible behaviors:

#### Option 1 (Most common)

Reject request.

```
HTTP 429
```

---

#### Option 2

Drop oldest request.

Used in some networking scenarios where fresh data is more important than old data.

---

### Visualization

```
Incoming

↓↓↓↓↓↓↓↓↓↓

R1 R2 R3 R4 R5

↓

Queue

[R1][R2][R3][R4][R5]

↓

Leak

↓

Every second

↓

Backend
```

---

### Timeline

Capacity

```
5
```

Leak

```
1/sec
```

#### Time = 0

```
Queue

Empty
```

---

Five requests

```
Queue

12345
```

---

One second later

```
Processed

1

Queue

2345
```

---

One second later

```
Processed

2

Queue

345
```

---

Eventually

```
Queue

Empty
```

Very predictable.

---

### Another Example

Email sending service.

Suppose

marketing team uploads

```
100000 emails
```

Should SMTP server receive all

100000

immediately?

No.

Instead

Queue

↓

100 emails/sec

↓

SMTP

Much safer.

---

### Another Example

Payment Gateway

Suppose

Black Friday Sale.

Suddenly

```
50000 payment requests
```

Instead of sending

everything immediately,

Leaky Bucket smooths requests

before payment processor.

This protects downstream systems.

---

### Data Structure

Very simple.

Store

```
Queue + Leak Rate
```

Example

```
Queue

[R1,R2,R3,R4]

Leak

1/sec
```

---

### Complexity

Memory

```
O(queue size)
```

Request

```
O(1)
```

Processing

```
O(1)
```

---

### Advantages

✅ Produces smooth traffic.

✅ Protects downstream systems.

✅ Prevents sudden spikes.

✅ Predictable request flow.

✅ Easy to reason about.

---

### Disadvantages

❌ Doesn't allow bursts.

❌ Good requests may wait in queue.

❌ Queue can overflow.

❌ Adds latency because requests wait.

---

### Token Bucket vs Leaky Bucket

This is one of the most asked interview comparisons.

|Feature|Token Bucket|Leaky Bucket|
|---|---|---|
|Stores|Tokens|Requests|
|Allows bursts|✅ Yes|❌ No|
|Output rate|Variable|Constant|
|Idle time gives benefit|✅ Yes|❌ No|
|Extra requests|Rejected only when tokens run out|Queued until full, then rejected|
|Main goal|Control average rate while allowing bursts|Smooth traffic and protect downstream systems|

---

### Visual Comparison

#### Token Bucket

```
Idle

↓

★★★★★ Tokens

↓

20 Requests

↓

✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓

Immediately
```

Burst allowed.

---

#### Leaky Bucket

```
20 Requests

↓

Queue

↓

1/sec

↓

Backend
```

Burst removed.

---

### Which One Should You Choose?

#### Use **Token Bucket** when:

- Users naturally make short bursts of requests (e.g., opening a dashboard that triggers several API calls).
- You want good user experience while still enforcing an average rate.
- You're building API gateways or public APIs.

Examples:

- REST APIs
- Mobile applications
- Public SaaS APIs
- Kubernetes client throttling

---

#### Use **Leaky Bucket** when:

- Downstream systems cannot handle traffic spikes.
- You need a stable, predictable processing rate.
- A small amount of latency is acceptable.

Examples:

- Email sending
- Log processing
- Payment processing
- Message queues
- Network traffic shaping

---

### Real Production Example

Imagine an e-commerce application.

A customer opens the homepage.

The frontend makes these calls almost simultaneously:

```
/profile
/cart
/orders
/recommendations
/notifications
```

A **Token Bucket** at the API Gateway allows this legitimate burst because the user has accumulated tokens while idle.

Behind the gateway, all accepted requests are placed onto a message queue for inventory updates.

A **Leaky Bucket** (or an equivalent queue with controlled worker throughput) ensures inventory updates are processed at a steady rate, preventing the inventory database from being overwhelmed.

So a single architecture can benefit from **both** concepts:

- **Token Bucket** at the edge to provide a responsive user experience.
- **Leaky Bucket** deeper in the system to smooth work for downstream services.

---

### Summary of All Four Algorithms

|Algorithm|Memory|Burst Allowed|Accuracy|Best Use Case|
|---|---|---|---|---|
|**Fixed Window**|⭐ Very Low|❌ Boundary bursts|Low|Simple internal APIs, admin tools|
|**Sliding Window Log**|❌ High|❌ No|⭐ Exact|Security-sensitive endpoints (login, password reset)|
|**Sliding Window Counter**|⭐ Low|⚠️ Limited|⭐ Very Good (Approximate)|High-traffic APIs needing fairness and efficiency|
|**Token Bucket**|⭐ Low|✅ Yes|⭐ Very Good|API gateways, public APIs, interactive user traffic|
|**Leaky Bucket**|Medium (queue)|❌ No|⭐ Predictable output|Traffic shaping, queues, email systems, payment pipelines|

---

### Interview Questions

#### Q1. Why can't Leaky Bucket replace Token Bucket?

Because they solve different problems.

- **Token Bucket** controls how many requests may be accepted while allowing short bursts.
- **Leaky Bucket** controls how quickly accepted requests are processed, smoothing traffic for downstream systems.

---

#### Q2. If a user is idle for 10 minutes, does Leaky Bucket give any benefit?

**No.** Unlike Token Bucket, Leaky Bucket doesn't accumulate credits or capacity during idle time. It simply processes requests at a fixed rate whenever they arrive.

---

#### Q3. Which algorithm is best for an API Gateway?

In most cases, **Token Bucket** is preferred because it provides a better user experience by allowing legitimate bursts while still enforcing a long-term average request rate.

---

### Final Mental Model

Remember these two pictures:

#### 🪙 Token Bucket

> **"Do you have enough permission tokens?"**

```
Tokens → Request → Allowed
```

It decides **whether a request may enter**.

---

#### 🪣 Leaky Bucket

> **"Requests can enter the bucket, but they'll leave only at a fixed speed."**

```
Requests → Queue → Constant Output
```

It decides **how fast accepted requests are processed**.

------

