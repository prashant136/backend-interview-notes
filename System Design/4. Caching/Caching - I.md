
### What is Caching?

Caching is storing frequently accessed data in a faster storage layer so future requests can be served much faster.

Instead of

```
Client
   │
   ▼
Server
   │
   ▼
Database
```

we do

```
Client
   │
   ▼
Server
   │
   ▼
Cache (Redis)
   │
   ▼
Database
```

Now instead of

```
300 ms
```

it becomes

```
5 ms
```

---

#### Benefits

- Faster response
- Lower database load
- Lower latency
- Better scalability
- Lower infrastructure cost
- Better user experience

---



---

### Client Cache

This is the first cache.

```
Browser
Mobile App
Desktop App
```

stores data locally.

```
Profile picture
CSS
JS
Logo
Fonts
```

are already stored.

Browser doesn't download them again.

---

#### Example

```
www.instagram.com
```

Browser requests

```
logo.png
```

Response

```
Cache-Control: max-age=86400
```

Browser stores it for

```
24 hours
```

Next request

```
No network request.
Loaded from Browser Cache.
```

---

#### Example in Mobile App

WhatsApp

Chat list

is stored locally.

When app opens

```
Show local cache immediately

↓

Background API refresh

↓

Update UI
```

User never waits.

#### Problems

Stale data.

If profile picture changes. Browser may still show Old picture.

Need proper cache expiration.

---

### CDN Cache

One of the most important interview topics.

---

What is CDN?

Content Delivery Network.

Many servers worldwide.

```
India

USA

Japan

Germany
```

All store copies.

Instead of requesting

```
USA Server
```

Indian users hit

```
Mumbai CDN
```

---

Example

```
Image
Video
CSS
JS
Fonts
```

---

Suppose

Instagram image

```
cat.jpg
```

stored in

```
AWS S3
```

Without CDN

```
India

↓

USA

↓

Return Image
```

Latency

```
250 ms
```

With CDN

```
India

↓

Mumbai CDN

↓

20 ms
```

Huge improvement.

---

#### Flow

```
Browser

↓

CloudFront

↓

S3
```

First request

```
CloudFront

↓

Miss

↓

S3

↓

Store in CDN

↓

Return
```

Second request

```
CloudFront

↓

Hit

↓

Return directly
```

No S3 call.

---

Real Example

Netflix

Movies are cached at ISP/CDN edge servers.

Thousands of users stream

Same movie

without contacting origin server.

---

Another Example

YouTube

Trending videos

cached globally.

---

Can Redis replace CDN?

No.

Redis

```
inside backend
```

CDN

```
edge network
```

Different purpose.

---

### Web Server Cache

Now request reached

```
Nginx

Apache
```

before application.

Web server itself can cache responses.

---

Example

Homepage

```
GET /
```

Changes every

```
30 seconds
```

Nginx can cache it.

Instead of

```
Nginx

↓

Spring Boot

↓

Redis

↓

Database
```

Request becomes

```
Nginx

↓

Cache

↓

Return
```

---

Useful for

```
GET requests
Static HTML
Static JSON
Images
```

---

Application isn't called.

---

### Application Cache (Redis)

we will discuss later in detail

### Database Cache

Database itself caches data.

PostgreSQL
MySQL
Oracle

all have internal cache.

---

When SQL executes

```
SELECT * FROM users WHERE id=5
```

Database loads page into memory.

Second query

No disk access.

Memory only.

Example -

```
10 Million Rows
```

Frequently used pages stay in RAM.

---

PostgreSQL

```
Shared Buffer
```

is internal cache.

---

MySQL

```
InnoDB Buffer Pool
```

stores pages.

---

Without database cache

Every query

```
Disk
↓
Read
↓
Return
```

Very slow.

---

With cache

```
RAM
↓
Return
```


---

### Complete Request Flow

Suppose

```
GET /product/100
```

```
Browser Cache
        │
        ▼
Found?
        │
        ├── Yes → Return
        │
        ▼
CDN
        │
Found?
        │
        ├── Yes → Return
        │
        ▼
Nginx Cache
        │
Found?
        │
        ├── Yes → Return
        │
        ▼
Spring Boot
        │
        ▼
Redis
        │
Found?
        │
        ├── Yes → Return
        │
        ▼
Database Buffer Cache
        │
Found?
        │
        ├── Yes → Return
        │
        ▼
Disk
```

Notice that **every layer tries to avoid going to the next, slower layer**. The closer the cache is to the user, the lower the latency and the less load reaches your backend.

---

### Real-World Example: Instagram Feed

Let's walk through what happens when a user opens their feed:

1. **Client Cache:** The app immediately displays previously cached posts and profile pictures so the screen appears instantly.
2. **CDN Cache:** Images and videos are fetched from the nearest CDN edge instead of the origin server.
3. **Web Server Cache:** If the feed metadata or a public endpoint is cacheable, a reverse proxy like Nginx may serve it directly.
4. **Application Cache (Redis):** The feed service checks Redis for the user's precomputed feed (`feed:user:12345`). If present, it returns the post IDs without querying the database.
5. **Database Cache:** If Redis misses, the application queries PostgreSQL. Frequently accessed index and table pages are likely already in PostgreSQL's shared buffer, avoiding disk I/O.
6. **Disk:** Only if the required pages are not in the database's memory cache does PostgreSQL read from disk, which is the slowest step.

-----
----
----

 
 This is one of the most important parts of **System Design interviews**. FAANG interviews don't just ask **"What is Redis?"** They ask:

- Which caching strategy would you use?
- Why?
- What happens if Redis goes down?
- How do you keep cache and DB consistent?
- What happens when millions of requests hit an expired cache?

The key to understanding caching is **thinking in terms of data flow**, not Redis commands.


### Cache Hit vs Cache Miss

This is the foundation. Every caching strategy is built around these two concepts.

### Cache Hit

The requested data already exists in the cache.

```
Client
   │
   ▼
Spring Boot
   │
   ▼
Redis
   │
Found
   │
   ▼
Return Response
```

Database is **never touched**.

### Example

Instagram profile API

```
GET /users/101
```

Redis contains

```
user:101

{
   id:101,
   name:"John",
   followers:25000
}
```

Flow

```
User

↓

Spring Boot

↓

Redis

↓

Found

↓

Return JSON
```

Time

```
Redis = 1 ms
```

instead of

```
Database = 30 ms
```

This is called a **Cache Hit**.

---

### Cache Miss

Requested data is NOT in Redis.

```
Client

↓

Spring Boot

↓

Redis

↓

Not Found

↓

Database

↓

Redis

↓

Client
```

Flow

```
Redis Miss

↓

Read Database

↓

Save in Redis

↓

Return
```

### Example

Amazon Product

```
GET /product/500
```

Redis

```
No key exists
```

Spring Boot

```
SELECT * FROM products WHERE id=500
```

Gets product

Stores

```
product:500
```

Next request

```
Redis Hit
```

---

### Real World Scenario

Suppose a product becomes viral.

First user

```
Redis Miss
```

Remaining 2 million users

```
Redis Hit
```

Only **one database query** was needed.