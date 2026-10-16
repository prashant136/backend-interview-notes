
**Question: What is Solid Principles ?**

Solid principle is set of 5 design principles how these principles helps engineers to write clean, maintainable code, flexible architecture, scalable products.

when solid principles are powerful -
1. App (spring boot, django, express)
2. Microservices
3. Large enterprise system
4. SaaS

```java

1️⃣ S - Single Responsibility Principle (SRP) - har class ki ek resposibility hogi.

- Each class has one job.
- changes are isolated.
- code becomes testable.

### 3 classes -> 3 diffrent isolated responsibilities (inko combine nhi karenge)
### reduces production bug and team conflicts.
Ex :-
public class InvoiceService {    
    // calculating the total amount of invoice.  
    void calculateTotal() {}  
}

public class InvoiceRepository {  
    // saving invoice date to DB  
    void save() {}  
}

public class EmailService {  
    // send email notification to user about their invoices  
    void sendEmailNotification() {}  
}


2️⃣ O - Open/Closed principle - software should open for extension but closed for modification. (naye features add ho sakte with changing the code)

good practice - use Polymorphism
ex - hume multiple discount k liye services method likhni. one way is ek class banake usne if-else ki help se cheeze bnate jaye but wo scalabe nhi hoga.
rather than ek interface bnake uske multiple method implements kiye jaye.

public interface Discount {    
    double calculate() {}  
}
// iske multiple classes based on condition
public class VipDiscount implements Discount {  
    @Override
    public double calculate() {
	    throw new UnsupportedOperationException("unimplement method calculate");
    }
}

public class NewuserDiscount implements Discount {  
    @Override
    public double calculate() {
	    print("unimplement method calculating new use discount");
    }
}


3️⃣ L - Liskov Substitution Priciple (LSP) - subclass must be replaceble for their parent class without breaking behavoir.

public interface Bird {}
public interface FlyingBird extends Bird {
	void fly();
}

public class Penguin extends Bird {
	void fly() {
		print("Penguins can't fly");
	}
}
public class Sparrow extends FlyingBird {
	void fly() {
		print("Sparrow can fly");
	}
}
// yaha Bird ko sirf Penguin class implement kr sakti h. FlyingBird ko Penguin practically nhi kr sakti. but sparrow class FlyingBird ko implement kr sakti h.
     Bird -------------> FlyingBird ----------> Sparrow
       |
       |------------> Penguin



4️⃣ I - Interface Segregation Principle (ISP) - Clients should not ne forced to depend upon methods they do not use.

// maan lo humne ek interface banayi Workable jisme do method hai work() aur eat(). Workable interface ko Robot class implement kiya toh dono method implement krne honge but Rotot class ke liye eat() method unnessacy hai.

// ### Lage interface cause tight coupling, confusion and hard testing. isiliye chhote interface banane best hota hai (specially in micorservices).
public interface Workable {
	void work();
}
public interface Eatable {
	void eat();
}

public class Robot implements Workable {
	@Override
	public void work() {
		print("robot is working");
	}
}
public class Human implements Workable, Eatable {
	@Override
	public void work() {
		print("human is working");
	}
	
	@Override
	public void eat() {
		print("human is eating");
	}
}


5️⃣ D - Dependency Inversion Principle - (spring, angular ka backbone)
Two module should not depend on each tightly. Both should depend on abstraction. (agar do module hai, toh dono ki dependency tightly nhi honi chahiye ek durse pr)

loose couple hai - DIP === Dependency Injection
```





#

#


#### Question1: Prepare Production ready GET APIS ?

A GET API retires data from server without modifying it.

1.) Properties (keep in mind while designing GET apis) -
	Read only operation
	Idempotent  = same request -> same result (kitni baar bhi request hit karo)
	Cacheble
	used heavily in frontend apps
	
	
2.) Why GET APIS need proper Design -
Most developers think GET APIs are  but in real systems they are the most complex because:
- Large datasets - (fetching 10M data at once)
- Performance issues
- Scalability challenges
- User experience requirements

Example problems:
- Returning 10M records → server crash
- Slow queries → bad UX
- No filtering → useless API



3.) How Industry design GET APIs -
	production level get apis me ye points hone chahiye jo must h.

|Concept|Simple Definition|Example|
|---|---|---|
|**Pagination**|Bahut saare data ko chhote-chhote pages mein divide karke dena.|10,000 products hain → API ek baar mein 20 products degi: `page=1, size=20`, phir `page=2`.|
|**Filtering**|Data mein se sirf wahi records nikalna jo kisi condition ko satisfy karein.|Products mein `price < ₹1000` wale products hi dikhana.|
|**Searching**|Kisi specific data ko keyword/query ke basis par find karna.|Amazon par `"iPhone 15"` search karna.|
|**Sorting**|Data ko kisi field ke according ascending/descending order mein arrange karna.|Products ko `price` ke according low → high sort karna.|
|**Field Selection**|Response mein sirf required fields bhejna, unnecessary fields nahi.|User ka `id, name` chahiye → poora address, phone, DOB nahi bhejna.|
|**Caching**|Frequently requested data ko temporary fast storage mein rakhna taaki baar-baar DB se na lana pade.|Popular product ka data Redis mein rakhna → next request Redis se quickly mil jayega.

User request karta hai:

```
GET /products?page=2&size=20
    &category=shoes
    &search=nike
    &sort=price_asc
    &fields=id,name,price
```

without pagiantion -
	Huge memory usage, slow db queries, high network cost

Yahan:
- **Pagination** → sirf 20 products
- **Filtering** → category = shoes
- **Searching** → `"nike"`
- **Sorting** → price low → high
- **Field Selection** → sirf `id, name, price`
- **Caching** → agar same popular query baar-baar aa rahi hai, result Redis se mil sakta hai



isko bbad me thik krte hai. 



Bilkul. **Offset pagination vs Cursor pagination** ko ek real e-commerce example se samjho. Sabse important difference hai:

> **Offset pagination bolta hai: "kitne records skip karne hain?"**  
> **Cursor pagination bolta hai: "last record ke baad se records do."**

---

# 1. Pehle basic example

Maan lo DB mein products hain:

|id|name|price|
|---|---|---|
|1|Nike Shoe|1000|
|2|Adidas Shoe|1200|
|3|Puma Shoe|1500|
|4|Reebok Shoe|1800|
|5|Nike T-Shirt|2000|
|6|Adidas T-Shirt|2200|
|7|Puma T-Shirt|2500|
|8|Nike Watch|3000|
|9|Casio Watch|3500|
|10|Titan Watch|4000|

Suppose API ko **3 products per page** dene hain.

---

# 2. Offset Pagination

Offset ka simple meaning:

> **Pehle kitne records skip karne hain?**

Usually API:

```
GET /products?page=2&limit=3
```

ya:

```
GET /products?offset=3&limit=3
```

### SQL

```
SELECT *
FROM products
ORDER BY id
LIMIT 3 OFFSET 3;
```

Meaning:

```
OFFSET 3
   ↓
skip 1, 2, 3

LIMIT 3
   ↓
give 4, 5, 6
```

Result:

|id|name|
|---|---|
|4|Reebok Shoe|
|5|Nike T-Shirt|
|6|Adidas T-Shirt|

---

# 3. Page number se OFFSET kaise calculate hota hai?

Agar:

```
page = 1
size = 3
```

then:

```
OFFSET = (page - 1) × size
       = (1 - 1) × 3
       = 0
```

SQL:

```
LIMIT 3 OFFSET 0
```

Result:

```
1 2 3
```

### Page 2

```
OFFSET = (2 - 1) × 3
       = 3
```

```
LIMIT 3 OFFSET 3
```

Result:

```
4 5 6
```

### Page 3

```
OFFSET = (3 - 1) × 3
       = 6
```

```
LIMIT 3 OFFSET 6
```

Result:

```
7 8 9
```

---

# 4. Offset pagination ka problem

Ab maan lo user **page 1** dekh raha hai.

```
Page 1:

1 Nike
2 Adidas
3 Puma
```

Ab database mein ek **new product** insert ho gaya:

```
id = 11
```

But suppose sorting `created_at DESC` hai, so new product sabse upar aa gaya.

Pehle:

```
1
2
3
4
5
6
```

New product ke baad:

```
11  ← new
1
2
3
4
5
6
```

User ab page 2 request karta hai:

```
GET /products?page=2&limit=3
```

Offset:

```
OFFSET = 3
```

DB skip karega:

```
11
1
2
```

aur return karega:

```
3
4
5
```

Notice karo:

**Page 1 mein user ko `1,2,3` already mile the.**

Page 2 mein:

```
3,4,5
```

`3` duplicate ho gaya.

Isi tarah records insert/delete hone par offset pagination mein **duplicate ya missing records** aa sakte hain.

---

# 5. Cursor Pagination

Cursor pagination ka concept thoda different hai.

Instead of:

> "3 records skip karo"

we say:

> **"Is particular record ke baad ke 3 records do."**

Example:

Page 1:

```
1
2
3
```

Ab last item:

```
3
```

Toh server client ko ek cursor de sakta hai:

```
cursor = 3
```

Next API:

```
GET /products?limit=3&cursor=3
```

SQL:

```
SELECT *
FROM products
WHERE id > 3
ORDER BY id
LIMIT 3;
```

Result:

```
4
5
6
```

---

# 6. Cursor ka actual meaning

Is query ko dekho:

```
WHERE id > 3
ORDER BY id
LIMIT 3;
```

Meaning:

```
last seen ID = 3

       ↓

3 ke BAAD wale records do
```

So:

```
1 2 3 | 4 5 6 | 7 8 9 | 10
        ↑
      cursor
```

Next request:

```
cursor = 6
```

SQL:

```
SELECT *
FROM products
WHERE id > 6
ORDER BY id
LIMIT 3;
```

Result:

```
7
8
9
```

Next:

```
cursor = 9
```

SQL:

```
SELECT *
FROM products
WHERE id > 9
ORDER BY id
LIMIT 3;
```

Result:

```
10
```

---

# 7. GET API ka complete example

### First request

```
GET /products?limit=3
```

Response:

```
{
  "data": [
    {
      "id": 1,
      "name": "Nike Shoe"
    },
    {
      "id": 2,
      "name": "Adidas Shoe"
    },
    {
      "id": 3,
      "name": "Puma Shoe"
    }
  ],
  "nextCursor": "3"
}
```

Client ko mila:

```
1, 2, 3
```

Aur server bol raha hai:

```
nextCursor = 3
```

---

### Second request

Client:

```
GET /products?limit=3&cursor=3
```

Server:

```
SELECT *
FROM products
WHERE id > 3
ORDER BY id
LIMIT 3;
```

Response:

```
{
  "data": [
    {
      "id": 4,
      "name": "Reebok Shoe"
    },
    {
      "id": 5,
      "name": "Nike T-Shirt"
    },
    {
      "id": 6,
      "name": "Adidas T-Shirt"
    }
  ],
  "nextCursor": "6"
}
```

Then:

```
GET /products?limit=3&cursor=6
```

SQL:

```
SELECT *
FROM products
WHERE id > 6
ORDER BY id
LIMIT 3;
```

---

# 8. Real-world example: Twitter/Instagram feed

Imagine:

```
10 million posts
```

User scroll kar raha hai.

Offset:

```
SELECT *
FROM posts
ORDER BY created_at DESC
LIMIT 20 OFFSET 1000000;
```

Database ko bahut saare rows skip karne pad sakte hain.

Cursor approach:

```
SELECT *
FROM posts
WHERE created_at < '2026-09-24 10:30:00'
ORDER BY created_at DESC
LIMIT 20;
```

Meaning:

> "Last post ke timestamp se purane 20 posts do."

Ye large, continuously changing datasets ke liye commonly useful pattern hai.

---

# 9. Cursor mein ID use karna better hai ya timestamp?

Dono possible hain.

### ID

```
WHERE id > :cursor
ORDER BY id
LIMIT 20;
```

Simple hai.

### Timestamp

For feed:

```
WHERE created_at < :cursor
ORDER BY created_at DESC
LIMIT 20;
```

Lekin timestamp alone problematic ho sakta hai.

Suppose:

```
id | created_at
---|-------------------
101| 10:00:00
102| 10:00:00
103| 10:00:00
```

Same timestamp multiple records ka ho sakta hai.

Isliye production systems mein **compound cursor** use karna common hai:

```
(created_at, id)
```

Query:

```
SELECT *
FROM posts
WHERE
    created_at < :createdAt
    OR (
        created_at = :createdAt
        AND id < :id
    )
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Isse ordering stable ho jaati hai.

---

# 10. Offset vs Cursor — ekdum simple comparison

||Offset|Cursor|
|---|---|---|
|Concept|Skip N records|Last record ke baad se|
|API|`page=5&limit=20`|`cursor=abc&limit=20`|
|SQL|`LIMIT 20 OFFSET 80`|`WHERE id > 80 LIMIT 20`|
|Jump to page 50|Easy|Usually not natural|
|Large dataset|Less efficient at large offsets|Generally better|
|Data changes frequently|Duplicate/missing records possible|More stable when cursor/order is designed correctly|
|Infinite scroll|Less suitable|Very suitable|
|Simple admin tables|Very suitable|Often unnecessary|

---

# 11. Interview mein kaise explain karna hai?

Agar interviewer pooche:

**"What is the difference between offset and cursor pagination?"**

Tum bolo:

> **Offset pagination** uses `LIMIT` and `OFFSET` to skip a certain number of records before returning the next page. It is simple and supports jumping directly to a page, but large offsets can become expensive and changes in the underlying data can cause duplicates or missing records.
> 
> **Cursor pagination** uses a value from the last item of the current page as a cursor. The next query fetches records after/before that cursor. It is generally more suitable for large, frequently changing datasets and infinite-scroll APIs.
> 

advantage of offset pagination -
	very fast
	stable results
	perfect for infinte scroll
	scales to millions of rows



#################

```
### curser based
GET /products?limit=3&cursor=3

SELECT *
FROM products
WHERE id > 3
ORDER BY id
LIMIT 3;

### offset based

GET /products?page=2&limit=3

SELECT *
FROM products
ORDER BY id
LIMIT 3 OFFSET 3;

offset = (page - 1) * limit

// cursor pagination for high scale system like instagram, facebook, twitter, youtube, linkedin.
```

indexing - with index direct lookup

select * from posts
where id < 1000
order by id desc
limit 20;

yaha id feild pe indexing lahga doge toh jyada better rahega.

composite index - ek sath do index
select * from posts
where id < 1000
order by created_at desc, id
limit 20;

INDEX(created_at, id) -> order By must match index order.

pagiantion performace = index quality.

#### filters - query paramas ka use karo.

##### GET /products?category=laptop&brand=dell&rating=4

##### search - /articles?serach=springboot

lld - LIKE%keyword% ka use krte hai search me but ye slow hota hai large dataset me.

large dataset - progression SQL like full text serach(index)


performace design -

db indexing, projection (select only needed fields), large vs eager loading

scalability - caching get apis, CDN caching, read replicas

security - rate limiting, access control, feild levle visibility


------------------------
-------------------------
------------------
# Redis cache -

```
suppose hamare databse me 1 sec me 100000 request aa gaye toh usko kaise handle karege. 
infact cursor pagination se, indexing se performace improve hoti hai. but phir bhi utni scale nhi hoti. uske liye hum caching use krte hai jaise redis.

1 req -> cache (empty) -> db hit -> respose store in cache and return to user
2,3......n req -> cache (data) -> retrun form cache (DB hit nhi hoga)

-----------------------------------------------------------------------------

					

mysql, postgres ke case me data hard disk me store (redis se slow hota hai)

					--------- Redis ---------
1. redis in-memory database (RAM me store hoga), frequently data access k liye
2. message broker jaisa kaam bhi krta hai
3. session store (user session)
4. single-threaded hota hai but extra threads for auxiliary tasks
5. ultra fast -- (RAM me store)
6. key value pair me data store hoga
7. scalable
8. persistance
9. atomic operation (transaction ko alag se handle krne ki need nhi h)


#### Data types -
string, hash, list, sorted-set, json ...etc 


#### If we want to scale apis (architecture)

diagram



#### common opertions -

SET
GET
INCR
DECR
APPEND
DEL


### List Operations -
	ordered collection of elements: 
	LPUSH
	RPUSH
	LPOP
	RPOP
	LRANGE
	
### Set Operations -
	unordered collecton of unique elements
	SADD
	SMEMBERS
	SREM
	
### Hash Operations - (like hashmap)
	key --> value
	HSET
	HGET
	HGETALL
	
### Expiration operations - key automatically expire
	EXPIRE
	TTL
	SETEX
```

```
Steps to step up redis in spring boot -

1️⃣ add dependencies :-
   
	<dependency>  
	    <groupId>org.springframework.boot</groupId>  
	    <artifactId>spring-boot-starter-cache</artifactId>  
	</dependency>  
	  
	<dependency>  
	    <groupId>redis.clients</groupId>  
	    <artifactId>jedis</artifactId>  
	</dependency>
	

2️⃣ application.properties :-

	spring.data.redis.host=localhost  
	spring.data.redis.port=6379  
	#spring.data.redis.password=  
	spring.data.redis.timeout=5000  
	# Maximum number of active connection  
	spring.data.redis.jedis.pool.max-active=10  
	# min and max number of idle (inactive), ready to use resources (database connection, threads or server instances)  
	spring.data.redis.jedis.pool.max-idle=5  
	spring.data.redis.jedis.pool.min-idle=2  
	# max wait time for connection  
	spring.data.redis.jedis.pool.max-wait=2000
   
   
3️⃣ Make RedisConfig class for redis bean configration :-






4️⃣ RedisTamplate - RedisTemplate is a core utility class in the Spring Data Redis framework that simplifies interaction with Redis by abstracting away low-level connection management and serialization. It handles opening and closing connections and provides a high-level, thread-safe API to perform data operations, similar to how JdbcTemplate works for relational databases.

Instead of writing raw Redis commands, `RedisTemplate` provides dedicated operation interfaces grouped by Redis data structures:

- opsForValue(): handles String operations (Key-Value pairs).
  
- opsForList(): handles Redis Lists (ordered collections for queues/stacks).
	  leftPush(key, value) (LPUSH): add element to the front of the list.
	  rightPush(key, value) (RPUSH): add element to the back of the list.
	  leftPop(key) (LPOP): removes and return the first element.
	  rightPop(key) (RPOP): removes and return the last element.
  
- opsForSet(): handles Sets (unordered collections of unique elements).
  
- opsForZSet(): handles Sorted Sets (elements ordered).
  
- opsForHash(): handles Redis Hashes (objects with fields and values).

JdbcTemplate -> designed to simplify database operations by removing standard JDBC boilerplate code. It automatically handles opening/closing database connections, managing statements, processing result sets, and resource cleanup.

5️⃣ adfsf              
```

--------------
----------
-------

# N + 1 Query

```

why ORM - if we don't use ORM, codebase is databse dependent, future me agr DB change kia toh code change krna hoga. ORM (take care of DB operation using their methods), so that dev don't need to write raw sql. loose coupling
dev (write oops) ------- ORM --- covert into sql --- DB

Hibernate (ORM)
👉 N+1 query issue is generally seen when we use ORM. JPA joins lagake data fetch nhi krta.


✅ N + 1 -> corresponding to 1 parent query, it will fire up additional N queries to featch related information.

20 Authors (author table) ---> 1000 books (book table): 

for every author query + N book query (ORM does)

means 
	select * from authors;
	|_select * from book where author_id=1;
	|_select * from book where author_id=1;
	|_select * from book where author_id=1;

it will kill app performance, slows api

FetchType.LAZY aud FetchType.EAGER decide when you load parent entity (Author), how its child realted entity loads.

FetchType.EAGER = once parent entity load, then child will load. ek bar me unnessary data bhi la dega.

FetchType.LAZY = when parent gets demand for child.

❌ fetch.eagar koi solution nhi -
Kyunki findAll() pehle sirf Parent Query chalata hai, then Hibernate eagerly mapped every parent item to its child, its will fire N SQL queries.

-----------------------------------------------------------------

🔋🔋 Solutions -

1️⃣ JOIN FETCH - data 1 query me join hoker load ho.

@Query("select a from Author a join fetch a.books")  
List<Author> findAuthorWithBooks();

SELECT 
    a.id AS author_id,
    a.name AS author_name,
    b.id AS book_id,
    b.author_id AS book_authorId,
    b.name AS book_name 
FROM author a 
JOIN book b 
ON a.id = b.author_id;



2️⃣ @EntityGraph -



3️⃣ DTO Projection - hume jitna data chahiye usko ek DTO bnake fetch krte h. custom fields fetch karte hai. (fetch only required data -> faster)

@Getter  
@Setter  
@NoArgsConstructor  
@AllArgsConstructor  
public class AuthorWithBookDto {  
    private Long id;  
    private String authorName;  
    private String bookName;  
}

@Query("select new com.mybasket.app.youtube.queryproblems.AuthorWithBookDto(a.id, a.name, b.name) from Author a join a.books b")  
List<AuthorWithBookDto> findAuthorWithBooksDto();

SELECT 
    a.id AS author_id,
    a.name AS author_name,
    b.name AS book_name
FROM author a 
JOIN book b 
ON a.id = b.author_id;



4️⃣ BAtch fetching (hibernate optimzation)

```


---------
--------
------
# File handling in backend

we have image (products, profiles), invoices (pdf, documents) videos.
Traditionally we have server (jaha hum apni file store krte hai, file uploading, serving, process at server). simple use case ke case ke liye thik hai minimal user ka sath.
but agar tum scalable apps ki baat karo ko local server pe file store is not good idea.

There are some limitation storing files in local storage :- 

1️⃣  ❌  Not Scalable (Biggest problem) - lets say we have three diffrent services (user, product and order) aur ye api gateway se connected hai. ye tino services apas me communicate kr skti hai.  suppose maine file store user service me kia. product service ko product ki image chahiye jo user service ke pass hai. services toh distribute ho gyi but file storage distribute nhi huyi.
product service ne user service ko call kari toh problems aa sakti hai jaise filenotfoundException,
broken image etc.

```
						                    ┌─────────────────┐
						                    │      Users      │
						                    │    Service      │
						                 |->└────────┬────────┘
				┌──────────────┐         |           |
				│ API Gateway  ──────────|           |
				└───────┬──────┘				   	 │
						                             ▼
						                    ┌─────────────────┐
						                    │     Product     │
						                    │    Service      │
						                    └────────┬────────┘
						                             │
						                             ▼
						                    ┌─────────────────┐
						                    │      Order      │
						                    │    Service      │
						                    └─────────────────┘

```

2️⃣  ❌ Tight Coupling - server me file rkhi huyi hai. server crash ho gya toh files ki delete ho gyi. ya server restart ho gyi by any reason files delete ho jayegi. isilye files ki kisi server ke local storage me rkhna safe nhi hai. server aur file dono loosely coupled hona chahiye tha. server me kuch changes kiye aur redeploy kia toh files may deleted.
docker container ko restart kiya aur volumes sahi se manage nhi ho paye (volumes ka state persistant nhi ho paya),  files delete ho gya.

```
                 ┌─────────────────────────┐
                 │         Server          │
                 │                         │
                 │    📄 📄 📄 Files        │
                 │                         │
                 └────────────┬────────────┘
                              │
                              ▼
                       Server crashes
                              │
                              ▼
                         Files gone

```

3️⃣  ❌  Difficult to handle high traffic - jab traffic increase hoga, server has to handle api requests, file upload, download, processing. IO operation becomes bottleneck, server slow down ho jayega. performance drop (bad user experience)

4️⃣  ❌  No CDN (Slow Delivery) - local files are serves directly from server only becoz koi CDN nhi hai.  suppose server hamara india me hai and usa se koi request server pe aa rha hai toh without cdn automatically response slow ho jayega. cdn ka kaam yahi hai ki nearest location pe caching (files server krna).  downloading bhot slow hoga.

5️⃣  ❌  No backups - backups nhi nhi hoga. disk failure, server crash  restart (the risk of data loss is very high). without backup data loss ho hi jayega. backup ki need hoti h taki tum restore kr sako.

6️⃣  ❌  Hard to manage storage - large files ko locally manage krna difficult hota hai. There are n problems -
	Disk get full, need manual cleanup, need to resize server and other problems

7️⃣  ❌  Security risk - files ko khud se secure krna padega ki koi unauthorised access toh nhi kr rha. encryption decryption krna pdta hai which is not easy.

✅ Solution :- Cloud storage is better solution like managed services (S3, blob storage, Cloudinary, Imagekit). ye service khud cheeze mange krti hai, scale hoti hai , security provide krti hai, CDN provide krti hai jaise s3 + CloudFront. 
server is decoupled form your cloud storage. server en files ko urls ke through consume karge. server crash ho kuch bhi ho humari files safe hogi.
	Highly Scalable, Global Accessible, CDN integration, High Availability, Security by default, No server load, Easy integration.
```
							┌──────────┐
							│   User   │
							└────┬─────┘
							     │
							     ▼
							┌──────────────┐
							│   Backend    │
							└──────┬───────┘
							       │
							       │ Upload
							       ▼
							┌──────────────────┐
							│       S3         │
							│                  │
							│   📄 📄 📄 Files │
							└────────┬─────────┘
							         │
							         │ Serve files
							         ▼
							┌──────────────────┐
							│       CDN        │
							└────────┬─────────┘
							         │
							         │
							         ▼
							      User gets
							       the file
```

MinIO (open source) for backend - S3 compatible storage. MinIO is high performance object storage system designed to store unstructured data like 
images 📸  videos 🎥  pdf 📄  backups 💾

```
folder ------> bucket  (top level container hold objects)
file ------> object  (actual files)
file properties ------> meta data


An object consist of 
1. file data (binary data)
2. meta data
3. object key ---> (path)

👉 Object key is unique identier of file. object keys are used to uniquely identify the objects.

meta data is extra information about file
1. content-type
2. size
3. uploaded-by


👉 Acccess key and Secret key (like username, password)
in order to access file from s3 (authenticate krne ke liye)

Endpoint - url where minio server runs (spring me yahi endpoint use hoga)

minIO server - main engine (handle api server, manger bucket/objects, handle stprage & replication)
```


API Layer in S3, storage backend in S3, data on local disks within distributed nodes s3 (high performance io operation) .... isko bbad me samjhenge.

-------








