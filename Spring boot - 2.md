
**Question 1 :- How would you handle inter-service communication in a microservices architecture using Spring Boot?**

Reactive systems - 
	asynchronous ho, funcitnal style coding mostly, data flow as event driven, backpressure of data stream

Traditional blocking system me request kisi service me pass gyi aur uska wait krega jab tak response nhi mil jata. jisse request timeout bhi ho skta h. 
Reactive system me request block nhi hoti. woh diffrent services ko call asynchrously krti hai.

For inter-service communication, I choose approach based on requirement. 
i. For synchronous calls, I prefer Feign Client with service discovery and proper timeout/retry handling. 
ii. For reactive systems, I use WebClient instead of RestTemplate. 
iii. For asynchronous communication, I use Kafka or RabbitMQ to decouple services. 
iv. Additionally, I implement circuit breakers using Resilience4j to handle failures gracefully.

---------

**Question 2 :- Can you explain the caching mechanisms available in Spring Boot?**

`@Cacheable` is  cache abstraction layer of spring boot to store results of expensive operations and reuse them. This abstraction allows us to use the same annotations like `@Cacheable, @CachePut, @CacheEvict` regardless of the underlying cache provider (Caffeine, Redis).

Q: You are using Redis. Why would you use @Cacheable instead of RedisTemplate ?

Redis ko use karne ke two method hai :- 
1. RedisTemplate => Programmatic Caching (Low-Level Control) for redis. execute redis command by calling their explicit method. Advanced operations ke liye best hai (e.g., Transactions, Pub/Sub, Hash/Set/Sorted Set manipulation). Manual cache checking, serialization, and error handling code likhna padta hai. (jayada control rhta hai). gives us programmatic and fine-grained access to Redis.
 
2. @Cacheable => High-Level Abstraction for redis (any cache). loose coupling so we can change Cache provider (e.g., Caffeine, Ehcache), then we need not to change code. basic Read/Write/Delete operations ke liye best h. less boilerplate code.

we need more control the use RedisTemplate and for for basic operartion use @Cacheable for redis.

Spring Boot's default cache mechanism is a simple in-memory store that uses `ConcurrentHashMap` through the `ConcurrentMapCacheManager`.

**Redis and Caffeine -**

```
Caffeine
→ In-Memory Cache /local cache - Best for Single Microservice
→ Very fast
→ Each instance has its own cache
→ Good for frequently-read local data
→ Caffeine technically "Spring app ke andar" nahi, application/JVM memory ke andar run karta hai.


Redis
→ External/shared cache - for Multiple Microservice
→ Multiple instances can access same data
→ Useful for distributed caching/session/rate limiting/etc.
→ Diffrent server port pr run hota hai.
```

----------

**@Cacheable (fetch & cache) -**

```java
** Cache Hit: Method execute nahi hota, direct cache se result mil jata hai.
** Cache Miss: Method execute hota hai, return Value cache me save hoti hai aur user ko return ki jaati hai.
  
@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")
    public User getUserById(Long id) {
        System.out.println("Fetching user from Database...");
        return userRepository.findById(id).orElse(null);
    }
}

- Jab aap pehli baar `getUserById(101)` call karenge, tab DB query chalegi. Dusri baar same `101` ke liye DB query nahi chalegi, data direct Redis/Caffeine se aayega.
```

**@CacheEvict (Update Cache Without Skipping Method) -** 

```java
- Jab database me koi data update kar rahe hote hain aur hum chahte hain ki cache me bhi latest updated data turant save ho jaye.

@Service
public class UserService {

    @CachePut(value = "users", key = "#user.id")
    public User updateUser(User user) {
        System.out.println("Updating user in Database...");
        return userRepository.save(user); // Updated user return kar raha hai
    }
}

- Es method ke chalne par DB me user update hoga aur same `user.id` ke key par Redis/Caffeine me bhi nayi value store ho jayegi. Agli baar jab @Cacheable wala getUserById chalega, toh usko fresh updated data milega.

```

**@CachePut (Remove/Delete from Cache) -**

```java
- Jab DB se data delete hota hai ya koi, usse related purana data cache me invalid (cache invalidated) ho jata hai, tab hum @CacheEvict se cache entry ko delete karte hain.
  
  @Service
public class UserService {

    // 1. Single Key Eviction
    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        System.out.println("Deleting user from Database...");
        userRepository.deleteById(id);
    }

    // 2. Clear All Entries in "users" cache space
    @CacheEvict(value = "users", allEntries = true)
    public void clearAllUserCache() {
        System.out.println("Clearing all users from cache...");
    }
}
```

Question: Spring @Cacheable me condition and unless attributes ?

 **condition** expression method call hone se pehle evaluate hota hai. if condition is false -> it bypass he cache mechanism.

condition is true -> cache mechanism (cache hit/miss)

```java
@Service
public class UserService {

    // Sirf tabhi cache hoga jab id > 100 ho
    @Cacheable(value = "users", key = "#id", condition = "#id > 100")
    public User getUserById(Long id) {
        System.out.println("Fetching user from Database for ID: " + id);
        return userRepository.findById(id).orElse(null);
    }
}
```

- **ID = 105:** Condition `true` → Cache me check hoga, data store/fetch hoga.
- **ID = 50:** Condition `false` → Caching completely ignore hogi, direct DB call chalegi.


**unless** expression method ke finish hone ke baad uske return result ko inspect karta hai.

Aap return object / output ke basis par decision lena chahte hain. Example: _Agar DB ne `null` return kiya, toh us `null` ko cache me save mat karo._

result is false -> store in cache

```java
@Service
public class UserService {

    // Cache karo, UNLESS result null ho (yaani null object ko cache mat karo)
    @Cacheable(value = "users", key = "#id", unless = "#result == null")
    public User getUserById(Long id) {
        System.out.println("Fetching user from Database...");
        return userRepository.findById(id).orElse(null);
    }
}
```

- Result = User Object: unless condition (`#result == null`) `false` aayegi → Result Cache **ho jayega**.
- Result = `null`: unless condition (`#result == null`) `true` aayegi → Caching reject ho jayegi (null Redis me store nahi hoga).

----------

1.) Cache Aside (Lazy Loading) - app sabse pehle Cache me check karti hai. (read heavy)
	If Cache Hit -> data return.
	if Cache Miss --> querying Database  --> store in cache -> data return.

2.) Write Through - app data ko cache me write karti hai and Cache layer synchronously DB update karti hai. (cache + DB  dono ek sath write hogi).
Adv. - No Stale Data; Cache hamesha updated rehta hai.
Disadv. - Write Latency high hoti hai (cache + DB  dono ek sath write hogi).

3.) Write Behind - app data ko Cache me write karke immediately success return kar deti hai. Internal background worker (Asynchronously) kuch interval ke baad (Batch processing) data ko Database me sync karta hai.
Adv = **Ultra-fast Writes**; DB I/O par load nahi aata. 
disadv = Risk of Data Loss (Cache crash hone par DB me save nahi hoga).

-----------

Cache Eviction policy (TTL â€“ Time to Live)
Cache Consistency
Cache Miss / Cache Hit
Serialization (for Redis)
Stale data handling

--------

`@RequiredArgsConstructor`

```java
import lombok.RequiredArgsConstructor;
import lombok.NonNull;

@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository; // Included (final)
    @NonNull private NotificationService notificationService; // Included (@NonNull)
    private String currentStatus; // Ignored (normal field)
}

// without lombok -
public class UserService {
    private final UserRepository userRepository;
    private NotificationService notificationService;
    private String currentStatus;

    // Generated Constructor
    public UserService(UserRepository userRepository, NotificationService notificationService) {
        if (notificationService == null) {
            throw new NullPointerException("notificationService is marked non-null but is null");
        }
        this.userRepository = userRepository;
        this.notificationService = notificationService;
    }
}
```

`@AllArgsConstructor`

```java
import lombok.AllArgsConstructor;

@AllArgsConstructor
public class UserDto {
    private final Long id;          // Included
    private String username;       // Included
    private String email;          // Included
}


// without lombok -
public class UserDto {
    private final Long id;
    private String username;
    private String email;

    // Generated Constructor
    public UserDto(Long id, String username, String email) {
        this.id = id;
        this.username = username;
        this.email = email;
    }
}
```

