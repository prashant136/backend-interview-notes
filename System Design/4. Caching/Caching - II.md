
## Caching Strategy -
	1. Cache-Aside (Lazy Loading)
	2. Read-Through Cache
	3. Write-Through Cache
	4. Write-Behind (Write-Back)
	5. Cache Refresh-Ahead Cache


Many developers memorize them as definitions, but interviewers expect you to answer:

> **Why was this pattern invented?**
> 
> **When should I use it?**
> 
> **What problem does it solve?**
> 
> **What are its drawbacks?**

We'll go through them exactly like a real production system.

---

### Before Learning Cache Patterns

Suppose we have

```
Spring Boot
      │
      ▼
    Redis
      │
      ▼
 PostgreSQL
```

Let's assume our application is an E-commerce website.

Database contains

```
Product
--------
ID
Name
Price
Stock
Description
```

Now users call

```
GET /products/1001
```

Question is

**Who should read/write Redis?**

Depending on the answer, we get different caching strategies.

---

## 1. Cache-Aside (Lazy Loading)

This is the **most commonly used caching pattern**.

Almost every company (Amazon, Flipkart, Uber, Swiggy, Netflix) uses this for many read-heavy APIs.

---

#### Concept

The application is responsible for managing the cache.

Redis does **nothing automatically**.

Application decides

- Read from cache
- Read from database
- Write to cache
- Delete cache

Everything is controlled by your code.

---

#### Architecture

```
Client
   │
   ▼
Spring Boot
   │
   ▼
Redis
   │
 Cache Hit?
   │
 ┌──────┴───────┐
 │              │
Yes             No
 │              │
 ▼              ▼
Return       PostgreSQL
                 │
                 ▼
         Save in Redis
                 │
                 ▼
              Return
```

---

### Why is it called Lazy Loading?

Because data is loaded into Redis **only when someone asks for it**.

Nothing is cached beforehand.

Imagine Redis is empty.

User requests

```
GET /products/1001
```

Application

```
Redis↓Not Found
```

Then

```
Database↓Product Found↓Save in Redis↓Return
```

Second request

```
Redis↓Found↓Return
```

Database isn't touched.

---

### Real World Example

Amazon

Suppose product

```
iPhone 17 Pro
```

gets 2 million views today.

Initially

Redis

```
Empty
```

First customer

```
query to database -> store in redis -> response return
```

Every remaining customer

```
response return from redis (no db query)
```

Database gets only **one read** instead of millions.

---

### What happens if product price changes?

Suppose

```
₹80,000

↓

₹75,000
```

Database updated.

Redis still has

```
₹80,000
```

Wrong.

Application removes cache

```
DEL product:1001
```

Next request

```
Redis Miss

↓

Database

↓

₹75,000

↓

Redis Updated
```

---

#### Advantages

✔ Easy to implement

✔ Database protected

✔ Redis stores only frequently used data

✔ Memory efficient

---

#### Disadvantages

First request is slow.

```
Redis Miss

↓

Database
```

---

#### Spring Boot Flow

```
Product getProduct(Long id){

    Product product = redis.get(id);

    if(product != null){
        return product;
    }

    product = repository.findById(id);

    redis.set(id, product);

    return product;
}
```

Very simple.

This is called **Cache-Aside**.

---

## 2. Read-Through Cache

Now let's change one thing.

Instead of application talking to database,

Redis talks to database.

---

## Concept

Application only asks Redis.

Redis itself knows

- If data exists
- If data doesn't exist
- How to load from database

Application never queries database directly.

---

Architecture

```
Client

↓

Spring Boot

↓

Redis

↓

Hit?

↓

Return

↓

Miss

↓

Redis loads Database

↓

Stores Cache

↓

Returns Data
```

Notice

Application doesn't know database exists.

---

#### Why was this invented?

Large organizations don't want every service implementing cache logic.

Instead

Redis layer automatically handles it.

So every service simply does

```
redis.get(key)
```

No cache code.

---

#### Real Example

Imagine

100 microservices.

Without Read Through

Every service writes

```
if(cache miss)
```

logic.

Huge duplication.

Instead

```
Redis SDK

↓

Automatically

↓

Database

↓

Redis

↓

Return
```

---

#### Example

Spotify

Song metadata

Every service simply asks cache.

Cache handles misses automatically.

---

#### Advantages

Application becomes simpler.

All caching centralized.

---

#### Disadvantages

Harder to implement.

Redis itself doesn't directly support this pattern out of the box. You usually need a caching layer or managed cache service that integrates with your data source.

---

Spring Boot

Usually not implemented directly.

Because Spring Boot generally uses Cache Aside.

---

### Difference

Cache Aside

```
Application

↓

Redis

↓

Database
```

Read Through

```
Application

↓

Redis

↓

Database
```

Looks identical.

Difference is

Who decides cache miss?

Cache Aside

```
Application
```

Read Through

```
Redis / Cache Provider
```

---

## 3. Write-Through Cache

Now let's discuss writes.

Suppose

User updates product price.

Question

Should we update database first?

Or Redis?

---

Concept

Application writes to Redis.

Redis immediately writes to database.

Both become consistent.

---

Architecture

```
Application

↓

Redis

↓

Database
```

Every write

updates

Redis

AND

Database.

---

Example

Update Product

```
PUT /products/1001
```

Price

```
80000
↓
75000
```

Application

```
Redis Updated

↓

Database Updated

↓

Success
```

Now

Read

always gets latest value.

---

#### Real Example

User Profile

Facebook

```
Name
Profile Picture
Bio
```

Whenever profile changes

Update cache

Update database

Immediately.

---

#### Advantages

Always consistent.

No stale cache.

---

#### Disadvantages

Slower write.

Every write

hits database.

Even if nobody reads.

---

Spring Boot Flow

```
redis.set(key, product);

repository.save(product);
```

---

## 4. Write-Behind (Write-Back)

This is opposite.

Application updates Redis only.

Database update happens later.

---

Architecture

```
Application

↓

Redis

↓

Success

↓

Background Worker

↓

Database
```

User doesn't wait.

---

#### Example

Gaming

Player score

changes

1000 times.

Without Write Behind

```
1000 Database Writes
```

Waste.

Instead

```
Redis

↓

Score Updated

↓

Every 30 sec

↓

Database
```

---

#### Real Example

Leaderboard

Player

```
100

↓

105

↓

108

↓

120
```

Redis

keeps updating.

Database

updated once.

---

#### Example

IoT Sensors

Temperature

Every second.

Millions of writes.

Database cannot handle.

Redis stores

Latest values.

Background worker

flushes every minute.

---

#### Advantages

Extremely fast.

Less database writes.

Handles write-heavy systems well.

---

#### Disadvantages

Suppose Redis crashes.

Before writing to database.

All updates lost.

Need

- Redis Persistence (RDB/AOF)
- Replication
- Reliable background workers

---

Spring Boot

```
Save Redis

↓

Kafka Event

↓

Consumer

↓

Database
```

or

```
Redis Queue

↓

Worker

↓

Database
```

---

## 5. Refresh-Ahead Cache

Suppose cache TTL

```
5 minutes
```

Exactly after 5 minutes

First user experiences

```
Cache Miss

↓

Database

↓

Slow Response
```

Can we avoid this?

Yes.

---

#### Concept

Refresh cache

before

it expires.

---

#### Example

TTL

```
5 minutes
```

At

```
4 min 50 sec
```

Background job

refreshes cache.

So users never experience cache miss.

---

#### Architecture

```
Redis

↓

Background Scheduler

↓

Database

↓

Refresh Redis

↓

TTL Reset
```

---

#### Real Example

Stock Market

Stock price

must remain fresh.

Can't wait for cache miss.

Background refresh

every few seconds.

---

#### Another Example

Cricket Live Score

Millions watching.

If cache expires

Everyone hits database.

Disaster.

Instead

Background refresh.

Users always get

fresh cache.

---

#### Example

Weather App

Every

5 minutes

Weather API called.

Redis updated.

Users always read Redis.

---

#### Advantages

No cache miss latency.

Better user experience.

---

#### Disadvantages

May refresh data that nobody reads anymore, wasting CPU, network, and database resources.

---

Spring Boot

```
@Scheduled(fixedRate = 240000)
public void refreshProduct(){

    Product product = repository.findById(1001L);

    redis.set("product:1001", product);
}
```

In production, instead of refreshing a single hardcoded key, you typically refresh only **hot keys** (frequently accessed data) or refresh on demand based on access statistics.

---

### Comparison Table

| Pattern                        | Reads                             | Writes                                       | Best Use Case                              | Pros                                  | Cons                                             |
| ------------------------------ | --------------------------------- | -------------------------------------------- | ------------------------------------------ | ------------------------------------- | ------------------------------------------------ |
| **Cache-Aside (Lazy Loading)** | Cache → DB on miss                | DB first, then invalidate/update cache       | Product catalog, user profiles, blog posts | Simple, memory efficient, most common | First request after a miss is slow               |
| **Read-Through**               | Cache automatically loads from DB | Usually combined with another write strategy | Centralized caching infrastructure         | Application code stays clean          | Requires cache provider/supporting layer         |
| **Write-Through**              | Read from cache                   | Cache and DB updated synchronously           | User profiles, account settings, inventory | Cache and DB stay consistent          | Higher write latency                             |
| **Write-Behind (Write-Back)**  | Read from cache                   | Cache updated immediately, DB later          | Gaming, analytics, IoT, click streams      | Very fast writes, fewer DB operations | Risk of data loss if cache fails before flushing |
| **Refresh-Ahead**              | Read from cache                   | Background refresh before expiry             | Live scores, stock prices, dashboards      | Avoids cache-miss latency             | Can refresh unused data                          |

---

### Which Pattern Is Used Most?

In large-scale production systems, it's common to combine patterns rather than choose only one.

|Company|Common Strategy|
|---|---|
|Amazon|Cache-Aside + Refresh-Ahead for hot products|
|Netflix|Cache-Aside + Refresh-Ahead for popular content|
|Instagram|Cache-Aside for feeds and profiles, Write-Through for some profile updates|
|Uber|Cache-Aside for driver/rider lookups, Write-Behind for telemetry and analytics|
|Swiggy|Cache-Aside for restaurant menus, Refresh-Ahead for trending restaurants|

**For Spring Boot applications, Cache-Aside is by far the most common and the one you should master first.** Frameworks like Spring Cache (`@Cacheable`, `@CacheEvict`, `@CachePut`) are designed around this pattern, and it is the strategy you'll most often implement in interviews and real-world backend services.

-----
-----
-----


## TTL and eviction policies ? Cache invalidation strategies ?

These two topics are considered **the hardest part of caching** because **most caching bugs are not about Redis—they're about stale data and memory management**.

A famous saying in software engineering is:

> **"There are only two hard things in Computer Science: Cache invalidation and naming things."**

Let's understand why.

---

## Part 1 : TTL (Time To Live)

TTL means

> **How long should data stay in cache before Redis automatically removes it?**

Without TTL

```
Redis

product:1001

↓

Forever
```

With TTL

```
Redis

product:1001

↓

Expire after 10 minutes
```

After 10 minutes

```
Redis

Deletes automatically
```

---

### Why do we need TTL?

Imagine an e-commerce website.

Database

```
iPhone

₹80,000
```

Redis

```
iPhone

₹80,000
```

Now admin changes price

```
₹80,000
↓
₹75,000
```

Database updated.

Redis still contains

```
₹80,000
```

Wrong data.

If TTL is

```
10 minutes
```

Worst case

Users see old price for only 10 minutes.

After that

Redis automatically expires.

Next request

```
Redis Miss

↓

Database

↓

₹75,000

↓

Redis Updated
```

---

### TTL Flow

```
Client

↓

Redis

↓

Exists?

↓

Yes

↓

Return

↓

TTL Ends

↓

Redis Deletes Key

↓

Next Request

↓

Database

↓

Redis
```

---

### Real World Example 1

#### Product Catalog

Amazon

Product description

doesn't change often.

TTL

```
1 hour
```

Millions of users

↓

Redis

↓

Database protected.

---

### Real World Example 2

#### News Website

Breaking News

Changes every minute.

TTL

```
30 seconds
```

Fresh data.

---

### Real World Example 3

#### Weather App

Weather updates

Every

10 minutes.

TTL

```
10 minutes
```

Perfect.

---

### Real World Example 4

#### OTP

```
OTP

123456
```

Must expire.

TTL

```
5 minutes
```

Redis automatically deletes.

No cleanup code.

---

### Real World Example 5

#### Login Session

User logs in.

Store

```
session:abc123
```

TTL

```
30 minutes
```

Inactive user

↓

Redis removes session.

---

### Choosing TTL

This is a design decision.

|Data|Suggested TTL|
|---|---|
|OTP|5 min|
|Login Session|30 min|
|Product|1 hour|
|User Profile|15 min|
|News|30 sec|
|Weather|10 min|
|Live Score|2 sec|
|Stock Price|1 sec|
|Country List|24 hours|

Notice

**Frequently changing data → Small TTL**

**Rarely changing data → Large TTL**

---

### Can TTL be too long?

Yes.

Imagine

```
Product Price

TTL

24 hours
```

Price changes after

5 minutes.

Users see wrong price

for

23 hours 55 minutes.

Bad.

---

### Can TTL be too short?

Also bad.

TTL

```
2 seconds
```

Every request

↓

Cache Miss

↓

Database

Redis becomes useless.

---

### TTL is a tradeoff

```
Large TTL

↓

Less Database Load

↓

More stale data
```

vs

```
Small TTL

↓

Fresh Data

↓

More Database Reads
```

System Design is all about balancing these tradeoffs.

---

## Part 2 : Eviction Policies

TTL removes data based on **time**.

Eviction removes data based on **memory**.

A Redis eviction policy dictates how the server deletes existing data to free up space when it reaches its `maxmemory` limit and new data is written.

---

Imagine

Redis Memory

```
1 GB
```

Now

Application stores

```
1.2 GB
```

Impossible.

Redis must remove something.

Question

**Which keys should Redis remove?**

Answer

Depends on eviction policy.

---

### Example

Redis

```
Memory

1 GB
```

Stored

```
Product

User

Order

Cart

Feed

Sessions
```

Memory full.

Need space.

Redis chooses a victim.

---

### Common Eviction Policies

#### 1. LRU (Least Recently Used)

Most common.

Rule

> Remove the data that hasn't been used for the longest time.

Example

```
Product A

Last Access

1 hour ago

Product B

2 minutes ago

Product C

1 second ago
```

Memory full.

Redis removes

```
Product A
```

because nobody uses it.

---

#### Real World

Amazon

Popular products

keep getting accessed.

Old products

disappear automatically.

Very good.

---

#### Example

```
iPhone

10000 requests

TV

5 requests

Old Nokia

0 requests
```

Redis removes

Old Nokia.

Makes sense.

---

#### 2. LFU (Least Frequently Used)

Rule

Remove

Least frequently accessed.

---

Example

```
Product

Access Count

Laptop

5000

TV

300

Printer

2
```

Memory full.

Redis removes

Printer.

---

Difference

LRU

looks at

Last Access Time.

LFU

looks at

Total Usage Count.

---

#### Real Example

Netflix

Trending movies

stay.

Rarely watched movies

removed.

---

#### LRU vs LFU

Imagine

Movie

```
Avatar
```

Yesterday

1 million users.

Today

No users.

LRU

May keep it.

Because recently accessed.

LFU

Definitely keeps it.

Because historically popular.

---

Now

Movie

```
Random Documentary
```

Viewed

Once.

LFU removes quickly.

---

### Redis Policies

Real Redis has policies like:

```
allkeys-lru
```

Remove any least recently used key.

---

```
volatile-lru
```

Only remove keys that have TTL.

---

```
allkeys-lfu
```

Least frequently used.

---

```
volatile-ttl
```

Remove keys with the shortest remaining TTL first.

---

LRU is the most common because it's simple and works well for many workloads. LFU is preferred when access patterns are stable and "hot" data should remain cached even if it hasn't been accessed very recently.

---

## Part 3 : Cache Invalidation

This is the most important topic.

Cache invalidation is the process of invalidating cache by removing data from a system’s cache when that data is no longer valid or useful.

---

### Strategy 1 : Delete Cache (Most Common)

Whenever database changes

Delete Redis.

Example

```
Update Product

↓

Database Updated

↓

DEL product:1001
```

Next request

```
Redis Miss

↓

Database

↓

Redis Updated
```

---

#### Why is this the most popular strategy?

Because deleting is simple and safe.

You don't have to worry about whether the cached object is partially updated or whether multiple cache keys reference the same data.

---

#### Example: Product Update

Database

```
Name : iPhone

Price : ₹80,000

Stock : 20
```

Redis

```
product:1001
```

Admin changes only stock

```
Stock

20

↓

18
```

Instead of updating Redis manually

```
Delete

↓

Next Read

↓

Fresh Data
```

Much simpler.

---

### Strategy 2 : Update Cache Immediately

Instead of deleting

Update Redis too.

```
Database Updated

↓

Redis Updated
```

---

#### Example

Facebook

User changes profile picture.

You don't want millions of users seeing the old picture until the cache expires.

Update both immediately.

---

### Advantage

No cache miss.

Latest data immediately.

---

### Problem

Suppose

```
Profile

↓

User Feed

↓

Friends Feed

↓

Followers Feed
```

The same profile information might exist in multiple cached objects.

Now you must update

- profile cache
- feed cache
- search cache
- recommendation cache

Deleting is often easier than updating every copy.

---

### Strategy 3 : TTL Based Invalidation

Don't delete anything.

Wait.

TTL expires.

Redis removes automatically.

---

Example

Weather

TTL

10 minutes.

Weather changes.

Redis still serves slightly old data.

After

10 minutes

Fresh weather arrives.

Perfectly acceptable for many applications.

---

Works well when

- Small delays are acceptable
- Data changes predictably
- Absolute real-time accuracy isn't required

---

### Strategy 4 : Event-Based Invalidation

Used in microservices.

Example

Product Service

updates product.

Publishes event.

```
Product Updated

↓

Kafka

↓

Inventory Service

↓

Search Service

↓

Recommendation Service

↓

Redis Cleared
```

Every service deletes its own cache.

---

#### Real World

Amazon

Price updated.

Event

```
ProductUpdated
```

sent through

Kafka.

Every microservice clears its own cache.

This avoids tight coupling between services.

---

### Strategy 5 : Version-Based Cache

Suppose

```
Product

Version

5
```

Redis Key

```
product:1001:v5
```

Admin updates.

Version

```
6
```

Now

Redis Key

```
product:1001:v6
```

Old key

never used again.

Eventually

TTL removes it.

---

#### Real Example

CDNs often use versioned file names:

```
style.css?v=1

↓

style.css?v=2
```

Browsers fetch the new version immediately instead of using the old cached file.

---

### Comparison

|Strategy|How it Works|Best For|Pros|Cons|
|---|---|---|---|---|
|Delete Cache|Remove cache after DB update|Most CRUD applications|Simple, safe|First read after update is a cache miss|
|Update Cache|Update DB and cache together|User profiles, inventory|Always fresh|Hard when the same data exists in many caches|
|TTL|Wait until cache expires|Weather, news, catalogs|Very simple|Data can be stale until expiry|
|Event-Based|Publish update events to invalidate caches|Microservices|Scales across services|More infrastructure and complexity|
|Version-Based|Create a new cache key version|Static assets, APIs|No stale-key conflicts|Old versions consume memory until evicted|

---

### How Large Companies Combine These

Large systems almost never rely on just one strategy.

**Amazon Product Catalog**

- Product details: Cache-Aside
- On product update: Delete the Redis key
- TTL: 1 hour as a safety net
- Kafka event invalidates caches in search and recommendation services

**Instagram**

- User profile: Update database and invalidate profile cache
- Feed: Rebuilt asynchronously because updating every cached feed would be expensive
- Images: Versioned URLs on the CDN (`profile_123_v42.jpg`) so browsers fetch the latest image automatically

**Banking**

- Account balance: Update cache immediately (stale balances are unacceptable)
- Exchange rates: TTL of a few minutes plus scheduled refresh
- Customer profile: Cache-Aside with explicit invalidation on updates

---

### Interview Tip

When asked:

> **"How would you cache product data in an e-commerce system?"**

A strong answer is:

1. Use **Cache-Aside** with Redis.
2. Set a **TTL** (for example, 1 hour) as a fallback so abandoned entries eventually expire.
3. On product updates, **invalidate (delete)** the Redis key immediately after the database transaction succeeds.
4. In a microservices architecture, publish a **ProductUpdated** event so other services (search, recommendations, etc.) can invalidate their own caches.
5. Configure **LRU or LFU eviction** so Redis automatically frees memory by removing the least valuable cached entries when memory is full.

This combination provides high read performance, bounded memory usage, and minimizes the risk of serving stale data.

-----
-----
----

## Common caching pitfalls (cache stampede, cache penetration, cache avalanche, hot keys ?

will do later ?

