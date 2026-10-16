##### [ kuch most important yaad rakho ] 
1. Comparator ko natural sorting jo class ke andar krte h. Isme compareTo() method ko override krte h aur Comparable<??> implement.
2. Comparator ko custom sorting ke liye use krte h, jo class ke bahar hoti h.

**Question 1: Comparator vs Comparable? Which one provides natural sorting?**
**Implement same for employee (id, name. salary,age) to sort in descending order of salary?**

```java
class Student implements Comparable<Student> {
    int age;

    public int compareTo(Student s) {
        return this.age - s.age;
    }
} // Student → age ke basis par sort
```


> **"Comparable defines the natural/default ordering inside the class, while Comparator allows us to define custom sorting logic externally."**

```java
import java.util.*;

class Employee {
    int id;
    String name;
    int salary;
    int age;

    Employee(int id, String name, int salary, int age) {
        this.id = id;
        this.name = name;
        this.salary = salary;
        this.age = age;
    }

    @Override
    public String toString() {
        return id + " " + name + " " + salary + " " + age;
    }
}

public class Main {
    public static void main(String[] args) {

        List<Employee> employees = new ArrayList<>();

        employees.add(new Employee(1, "Rahul", 50000, 25));
        employees.add(new Employee(2, "Amit", 80000, 30));
        employees.add(new Employee(3, "Neha", 60000, 27));

        // Salary descending
        employees.sort((e1, e2) -> e2.salary - e1.salary);

        for (Employee e : employees) {
            System.out.println(e);
        }
    }
}
```


--------

**Question 2: Let say there is a Customer class. We add objects of Customer class to an ArrayList. How can we sort the Customer objects in ArrayList by using customer firstName attribute of Customer class?**

```java
import java.util.*;

class Customer {
    int id;
    String firstName;

    Customer(int id, String firstName) {
        this.id = id;
        this.firstName = firstName;
    }

    @Override
    public String toString() {
        return id + " " + firstName;
    }
}

public class Main {
    public static void main(String[] args) {

        List<Customer> customers = new ArrayList<>();

        customers.add(new Customer(1, "Rahul"));
        customers.add(new Customer(2, "Amit"));
        customers.add(new Customer(3, "Neha"));

        // Sort by firstName
        customers.sort((c1, c2) ->
                c1.firstName.compareTo(c2.firstName)
        );

        for (Customer c : customers) {
            System.out.println(c);
        }
    }
}
```


**Only remember 4 methods -**

`compareTo()` for String
	example - c1.name.compareTo(c2.name)
 
`Integer.compare()` for  int
	Integer.compare(e1.salary, e2.salary)
	similarly for double - Double.compare()

`Comparator.comparing()` 



A list of students objects { id, name, marks }.  sort karo marks ascending aur agar marks same hain → Name ascending.

```java
import java.util.*;

class Student {
    int id;
    String name;
    int marks;

    Student(int id, String name, int marks) {
        this.id = id;
        this.name = name;
        this.marks = marks;
    }

    public String toString() {
        return id + " " + name + " " + marks;
    }
}

public class Main {
    public static void main(String[] args) {

        List<Student> list = new ArrayList<>();

        list.add(new Student(1, "Rahul", 90));
        list.add(new Student(2, "Amit", 80));
        list.add(new Student(3, "Neha", 90));
        list.add(new Student(4, "Ankit", 80));
		
		// use this -
        list.sort(
            Comparator.comparing((Student s) -> s.marks)
                      .thenComparing(s -> s.name)
        );
        
        // or this - (without Comparator.comparing, thenComparing)
		list.sort((s1, s2) -> {
		    if (s1.marks != s2.marks) {
		        return Integer.compare(s1.marks, s2.marks);
		    }
		    return s1.name.compareTo(s2.name);
		});

        System.out.println(list);
    }
}
```


------

**Question 4: Collections class? When to use synchronized collections? Arrays class ?**

```
Collections.sort(list);
Collections.reverse(list);
Collections.max(list);
Collections.min(list);
```

**Collections class**

> **"Collections is a utility class that provides helper methods to operate on collections, like sorting, reversing, searching, and creating synchronized wrappers."**


**Synchronized collections kab use karein?**

Agar **multiple threads same collection ko access/modify** kar rahe hain aur collection itself thread-safe nahi hai:

```
ArrayList
HashMap
HashSet

// ye thread safe nhi hoti. isilye hum ye use krte h.

List<Integer> list =
    Collections.synchronizedList(new ArrayList<>());

Map<Integer, String> map =
    Collections.synchronizedMap(new HashMap<>());
```


> **"We use synchronized collections when multiple threads access a non-thread-safe collection concurrently and we need basic thread safety."**

⚠️ Modern applications me high-concurrency cases ke liye `ConcurrentHashMap`, `CopyOnWriteArrayList` jaise concurrent collections bhi consider karte hain.

**Arrays class**


> **"Arrays is a utility class that provides methods for common operations on arrays, such as sorting, searching, comparing and copying."**


----

**Question 5: What is CopyOnWriteArrayList? How it is different from ArrayList and vector in Java ?**

```
CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>();
list.add(10);
list.add(20);
```

Agar koi thread read kar raha hai aur doosra thread write karta hai, reader ko usually **old snapshot** milta hai. Isliye read ke time locking nahi hoti.

```
ArrayList
→ read and write dono fast hai but NOT thread-safe (single threaded)

Vector
→ Thread-safe because methods synchronized hain (Locking ki wajah se slower)
→ Old/legacy approach

CopyOnWriteArrayList
→ Thread-safe
→ Read ke time lock nahi
→ Write par copy create - Expensive (copy banti hai)
→ Read-heavy applications ke liye - Very good for concurrent reads
```

Interview me exactly bolo:

> **"CopyOnWriteArrayList is a thread-safe list designed for read-heavy scenarios. On every write, it creates a new copy of the underlying array, so reads can happen without locking. Compared to ArrayList, it is thread-safe; compared to Vector, it provides better concurrent read performance but writes are more expensive."**


-----------

**Working of CopyOnWriteArrayList - no need for interview**

**CopyOnWriteArrayList ka naam hi uska concept batata hai**:

Read → same array use karo
Write → array ki new copy banao


```
1. [READ]
ek list → [10, 20, 30] hai. threat t1 usko read krta hai.

list.get(1) - Ye directly current array se 20 read karega.

list → [10, 20, 30]
          ↑
        READ <- thread t1

array ki koi copy nahi banti. Locking bhi nahi hoti.

2. [WRITE]
ab usi list ko thread t2 kuch write karega.

list.add(40) - Purane array ko modify nahi karega instead nayi copy banega.

Old array:  [10, 20, 30]

             ↓ copy

New array:  [10, 20, 30, 40]

Phir list internally naye array ko point karegi.

list → [10, 20, 30, 40] -> final list


3. [READ + WRITE ~ Agar same time read + write ho]
assume karo list [10, 20, 30] hai. 
thread t1 → READ
thread t2 → WRITE

Thread-1 purane array ko read karegi aur Thread-2 meanwhile new copy bana raha hoga. Read disturb nahi hoga.

isiliye :-
READ  → fast, no locking
WRITE → copy create → expensive
```


> CopyOnWriteArrayList maintains a snapshot-like array for reads. When a write occurs, it creates a new copy with the modification and then replaces the reference. Therefore reads don't require locking, but writes are expensive.

--------

**Question 5: What is the scenario to use ConcurrentHashMap in Java ? internal working ?**

Agar **multiple threads same Map ko simultaneously read/write** kar rahe hain, to `ConcurrentHashMap` use kar sakte ho.

ConcurrentHashMap<Integer, String> map = new ConcurrentHashMap<>();
map.put(1, "Alice");
map.put(2, "Bob");


> ConcurrentHashMap is a thread-safe implementation of Map designed for concurrent access. We use it when multiple threads need to read and modify a map simultaneously, especially in high-concurrency applications.








------

**Question 6: How will you create an empty Map in Java ?**

Mutable empty Map -

Map<Integer, String> map = new HashMap<>();  // baad me data add kar sakte ho.

Immutable empty Map -

Map<Integer, String> map = Collections.emptyMap();
Map<Integer, String> map = Map.of();

Isme data add nhi kr sakte, immutable h.

> For a mutable empty Map, I use `new HashMap<>()`. For an immutable empty Map, I can use `Collections.emptyMap()` or `Map.of()`.

---------

**Question 7: How will you convert a Map to a List in Java?**

```
// Keys ki List:
List<Integer> keys = new ArrayList<>(map.keySet());


// Values ki List:
List<String> values = new ArrayList<>(map.values());


// Entries ki List:
List<Map.Entry<Integer, String>> entries = new ArrayList<>(map.entrySet());


// Yaad rakho:
keySet()   → keys
values()   → values
entrySet() → key + value
```

-----

**Question 8: Shallow copy deep copy kya hota hai? Map, set, arraylist me shallow aur deep copy kaise bnate h ?**

Maan lo:

```
Person p1 = new Person();
Person p2 = p1;
```

Yahan copy nahi bani. Dono **same object** ko point kar rahe hain.

Shallow Copy -

> **Outer/container ki copy banti hai, but andar ke objects same reference hote hain.**


```
Original List       Copy List
    ↓                   ↓
 [A, B, C]          [A, B, C]
    ↑                   ↑
    └──── same objects ─┘
```

Agar `A` ek mutable object hai aur uske andar change karo, dono lists me change dikhega.


Deep Copy -

> **Outer/container bhi naya aur andar ke objects bhi naye.**

```
Original List       Copy List
    ↓                   ↓
 [A1, B1, C1]       [A2, B2, C2]
```

`A1` aur `A2` alag objects hain.
A1 change karoge → A2 unaffected.

> In a shallow copy, the outer collection is copied but the objects referenced inside it are shared. In a deep copy, both the collection and the objects inside it are independently copied.

**Hashmap**

```java
// shallow copy
Map<Integer, Employee> map1 = new HashMap<>();
Map<Integer, Employee> map2 = new HashMap<>(map1);

// map2 me chage krne se map1 me bhi reflect hoga.

//deep copy
Map<Integer, Employee> map2 = new HashMap<>();

for (Map.Entry<Integer, Employee> entry : map1.entrySet()) {
    map2.put(
        entry.getKey(),
        new Employee(entry.getValue())
    );
}
// dono alag hai.
```

**ArrayList**

```java
// shallow copy
List<Employee> list1 = new ArrayList<>();
List<Employee> list2 = new ArrayList<>(list1);

// deep copy
List<Employee> list2 = new ArrayList<>();

for (Employee e : list1) {
    list2.add(new Employee(e));
}
```

**HashSet**

```java
// shallow copy
Set<Employee> set1 = new HashSet<>();
Set<Employee> set2 = new HashSet<>(set1);

// deep copy
Set<Employee> set2 = new HashSet<>();

for (Employee e : set1) {
    set2.add(new Employee(e));
}
```


-------

**Question 8: Why we cannot create a generic array in Java ?**

```
T[] arr = new T[10];  ❌ 

List<T> list = new ArrayList<>();  ✅ use this
```


Interview me exactly bolo:

> We cannot directly create a generic array because Java generics use type erasure, so the actual type of T is not available at runtime, while arrays require their component type at runtime for type checking.

----

**Question 9: Diff between an Array and ArrayList, which one is the preferred collection for storing objects?**

```java
Student[] students = new Student[10];   // arrays

List<Student> students = new ArrayList<>();  // arraylist
```


> I would generally prefer ArrayList for storing objects because it is dynamically resizable and provides useful collection APIs. I would use an array when I know the fixed size or specifically need array semantics.

---------

**Question 10: Internal working of Arraylist{should know when n how new array is created and data copied} ?**

Ye answer yaad kar lo:

> ArrayList internally uses a dynamically resizable array, typically an Object array. When we add an element and there is available capacity, it is inserted directly into the next position. When the internal array becomes full, ArrayList creates a larger array, copies the existing elements into the new array, and then adds the new element. Because resizing doesn't happen on every insertion, add() has amortized O(1) time complexity, although a resize operation itself takes O(n).

```
// internal working -

1. Internally arraylist Object[] elementData; 
   
2. agar Space available hai, new element directly next position par add ho jata hai - O(1)

3. agar Array full ho gya hai toh ArrayList new larger array create karegi.
jiska size = capacity by about 1.5X

	Old array
	[10][20][30][40][50][60][70][80][90][100]
	                    ↓ copy
	New array
	[10][20][30][40][50][60][70][80][90][100][ ][ ][ ][ ][ ]

Then old elements new array me copy hogi. Old array eventually garbage collection ke liye eligible ho jata hai.

4. why arralist fast ? 
   mostly add O(1) time leti hai. 
   BUT jab resize hota hai to add O(n) time leti hai. lekin resize har baar nahi hota, isliye average complexity O(1) hai.

```


----


**Question 11: HashMap { should know hashing and then index generation, data stored in buckets in array and hash Collisions lead to linked list creation, red black tree when? need of Hashcode, equals, Immutable classes } ? What happens when you put a key object in a HashMap that is already present ?**


```
Internal working of hashmap -

1️⃣ hashmap internally Bucket array use krta hai. har bucket me { key + value } hota hai.
	Bucket array
	   ↓
	[0] → Node
	[1] → null
	[2] → Node → Node
	[3] → null

2️⃣ hashmap.put("alice", 30) tab kya hota hai ?
   key.hashCode() - key se hash value generate hogi. HashMap hash value ko use karke bucket array ka index calculate karta hai.

	suppose index = 3 mila toh data bucket me store ho jayegi.
	flow => hashCode → hash → bucket index

3️⃣ Collision ka case -
   map.put(key1, "A");
   map.put(key2, "B");
   
   Agar dono same bucket me aa jaye toh collision kahenge. ab isme do scenario h jo eqauls() se decide hogi -
   a.) key1.equals(key2) = false (keys diffrent hai) ~ same bucket me linked list form hogi.
   b.) key1.equals(key2) = true (keys same hai) ~ old value replace ho jayegi.
   
4️⃣ Already present key ko put karenge toh ?
   map.put("Alice", 100); 
   map.put("Alice", 200);
   
   If the key already exists, HashMap replaces the old value with the new value.

5️⃣ Red-Black Tree kab banta hai ?
   Agar ek bucket me bahut saare collisions ho jaate hain, linked list long ho sakti hai. Java 8+ me bucket ko RB Tree me convert kiya ja sakta hai.
   
bucket nodes >= 8 + table capacity >= 64 = Treeification
why tree ? list lambi hogi toh search O(n) time legi, aur tree me O(log n).

6️⃣ hashCode() aur equals() ki need kyun?
HashMap ko do cheezein identify karni hoti hain:

hashCode() - bucket locate karne me help.
equals() - Bucket ke andar actual key same hai ya different.
   
```

**Custom object ko HashMap key banana** -
```java
class Employee {
    int id;
    String name;

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }

    @Override
    public boolean equals(Object obj) {
        // compare id and name
    }
}

// Agar tum custom object ko key bana rahe ho, toh equals() aur hashCode() ka contract correctly maintain karna important hai.
```

Important Rule -
```java
Agar two objects equals() ke according equal hain, toh unka hashCode() bhi same hona chahiye.
```


Ab interview me EXACTLY kaise explain karna hai?

Step 1 - HashMap internally uses an array of buckets.

Step 2 - When we put a key-value pair, HashMap calculates the key's hashCode and uses the hash to determine the bucket index.

Step 3 - The entry is stored in that bucket.

Step 4 - If multiple keys map to the same bucket, a hash collision occurs. HashMap handles collisions using a linked list, and in Java 8+ a heavily-collided bucket can be converted into a Red-Black Tree.

Step 5 - When checking whether a key already exists, hash is used first and then equals() is used to identify the actual key.

Step 6 - If the key already exists, put() replaces its old value; it doesn't create another entry for the same key.

------

**Question 12: What are the differences between a HashMap and a Hashtable in Java?**

HashMap -

```java
Map<Integer, String> cache = new HashMap<>();

cache.put(101, "Prashant");
cache.put(102, "Rahul");

String user = cache.get(101);
```

Hashtable -

```java
Map<Integer, String> cache = new Hashtable<>();

cache.put(101, "Prashant");
cache.put(102, "Rahul");
```

Dono Map hain, but **thread-safety aur null handling** mein major difference hai.

**Differences -**

1. `HashMap` **thread-safe nahi hai**. Hashtable thread safe hota h. `Hashtable` ke methods traditionally **synchronized** hote hain. Isliye basic operations thread-safe hain.


**Lekin:** iska matlab ye nahi ki Hashtable modern concurrent programming ke liye preferred hai. `ConcurrentHashMap`usually better choice hai.

2. HashMap → allows null

```
HashMap<Integer, String> map = new HashMap<>();

map.put(null, "ABC");
map.put(1, null);
```

Hashtable → null allowed nahi

```
Hashtable<Integer, String> table = new Hashtable<>();

table.put(null, "ABC"); // ❌
table.put(1, null);     // ❌

// Exception aayegi.
```


3. Performance

Because `Hashtable` methods are synchronized, it generally has more synchronization overhead.

But interview mein ye **mat bolna ki Hashtable always slow hai**. Better bolo:

> “HashMap generally has less synchronization overhead, while Hashtable synchronizes its operations.”

4. Legacy

`Hashtable` **legacy class** hai. `HashMap` Java Collections Framework ka modern Map implementation hai.
Modern concurrent requirement ke liye: ConcurrentHashMap


🎯 Interview mein exactly bolo

> HashMap and Hashtable both implement Map, but HashMap is not synchronized whereas Hashtable is synchronized. HashMap allows one null key and multiple null values, while Hashtable does not allow null keys or values. HashMap generally has less synchronization overhead. Hashtable is a legacy class, and for modern concurrent applications I would generally prefer ConcurrentHashMap.


-------------------------

**Question 13: ConcurrentHashMap ka internal working ?**

```
1️⃣ ConcurrentHashMap bhi hashmap ki trah bucket me cheeze store krta hai.

ConcurrentHashMap
        ↓
   Bucket Array
        ↓
 ┌────┬────┬────┬────┐
 │ B0 │ B1 │ B2 │ B3 │
 └────┴────┴────┴────┘
       ↓
      Node
       ↓
   key → value
   

ConcurrentHashMap ka flow hashmap jaisa hi hoga. kuch differnces hai jo locking se releated hai. jo hum aage dekhenge.


------------------------------------------------------------------------------ 


2️⃣ main diffrence locking ka hota hai.

👉 Hashtable ka conceptual idea - Jab Ek thread t1 synchronization hold karega, tab durse thread t2 ke operations ko block kar dega. 
Poore map par ek single global lock laga dega.

             🔐 MAP LOCK
                 |
        ┌────────┼────────┐
        ↓        ↓        ↓
       B0       B1       B2

👉 ConcurrentHashMap - Poore map par ek single global lock nahi lgata.
Different buckets par operations concurrently ho sakte hain. (Every bucket has differnt lock)

       B0       B1       B2       B3
       🔐       🔐        🔐       🔐


------------------------------------------------------------------------------


3️⃣ get() kaise karega -

ConcurrentHashMap<Integer, String> cache = new ConcurrentHashMap<>();
cache.put(101, "Prashant");
cache.get(102, "Dasi");
cache.get(103, "Rita");

102
 ↓
key.hashCode()
 ↓
hash value
 ↓
bucket index
 ↓
bucket
 ↓
Node find
 ↓
key compare
 ↓
"Dasi"

get() generally locking nahi karta. Isliye multiple threads simultaneously read kar sakte hain:

Thread 1 → get(101) ──┐
Thread 2 → get(102) ──┼──> ConcurrentHashMap
Thread 3 → get(103) ──┘

🚩 Ye cache ke liye bahut useful hai because cache mein reads usually bahut zyada hote hain.


------------------------------------------------------------------------------


4️⃣ Collision -

Exactly HashMap ki tarah collision handle hota hai. 
pahle bucket linkedlist me convert hoga.
Heavy collision hone par Red-Black Tree mein treeification hogi. (Java 8+)


------------------------------------------------------------------------------

5️⃣ put (Most important) - Isme do condition hai.

✅ Case 1: Bucket empty hai - CAS (comapre and Swap Algo)

Single thread (HashMap) ki condition me CAS ki need nhi hogi. but muiltiple thread (ConcurrentHashMap) ki condition me jaroorat padegi.

Suppose cache mein:
	Bucket 5 → EMPTY

Do threads same time value daalne aaye:
	Thread 1 → put(101, "A")
	Thread 2 → put(201, "B")

Agar simple insertion hota:
	Thread 1 → check empty
	Thread 2 → check empty

	Thread 1 → A daal diya
	Thread 2 → B daal diya

Problem: dono ne empty dekha tha.
Ab kaun jeeta? Race condition ho sakti hai. Isliye CAS use hota hai.

### CAS ka simple meaning:
Check karo bucket abhi bhi empty hai. Agar empty hai toh mera data daal do.
	


					 ┌────┬────┬────┬
	Thread 1 ----->	 │   bucket B5  │   <----- Thread 2
					 └────┴────┴────┴					       
						
	
	Do thread ek time pr bucket 5 (jo emprty hai) inset karneka try kr rhe hai. Thread t1 pehle successfully insert kr di, then Thread t2 ko pata chal gaya:
"Oh, kisi aur thread ne already insert kar diya."

So sirf ek thread successfully insert karega. Isi wajah se concurrent environment mein empty bucket par bhi CAS useful hai.


✅ Case 2: Bucket mein Node present ho - Collision (Fine-grained lock)

Fine-grained lock means = Poore map par ek single lock nahi lagta.

Already Node me value present hai. ab tum inset krne ka try kr rhe ho.
ConcurrentHashMap 'bucket ke head node par synchronized lock' laga dega. Puri ConcurrentHashMap lock nahi hogi. Sirf relevant bin/bucket ke modification path par synchronization hoti hai.

			Bucket 2
			   ↓
			🔐 Node
			   ↓
			Node
			   ↓
			Node
meanwhile jo other buckets hai unhe concurrently proceed kar sakte hain.
Bucket 5 → Thread 2 ka operation
Bucket 8 → Thread 3 ka operation

Ek cache ka example lelo -

Imagine 1000 users ka cache:

        ConcurrentHashMap
              |
     ┌────────┼────────┐
     ↓        ↓        ↓
   B1 🔐     B2 🔐     B3 🔐
     ↑        ↑        ↑
   T1         T2       T3
   
Agar T1 B1 modify kar raha hai:  
T1 → B1 🔐

iska matlab ye nahi: 
T2 → WAIT ❌
T3 → WAIT ❌

T2/T3 different buckets par kaam kar sakte hain.

If the target bucket is empty, ConcurrentHashMap can use CAS to insert without locking. If the bucket already contains nodes and we need to modify it, it uses fine-grained synchronization on that bin instead of locking the entire map.



#### COMPLETE FLOW OF PUT -

		put(key, value)
		      ↓
			hash
		      ↓
		bucket index
		      ↓
		Is bucket empty?
		    /       \
		  YES        NO
		   ↓          ↓
		 CAS       existing node
		   ↓          ↓
		insert     compare key
		              ↓
		       ┌──────┴──────┐
		       ↓             ↓
		   same key      different key
		       ↓             ↓
		  replace value   collision
		                     ↓
		              synchronized bin
		                     ↓
		                add/update


------------------------------------------------------------------------------


6️⃣ Ek important interview point

ConcurrentHashMap null allow nahi karta.

ConcurrentHashMap<Integer, String> map = new ConcurrentHashMap<>();
map.put(null, "ABC"); // ❌
map.put(1, null);     // ❌


	HashMap            → null allowed
	ConcurrentHashMap  → null not allowed
	Hashtable          → null not allowed
	
```

🎯 Interview mein exactly bolo

> ConcurrentHashMap internally uses a bucket array similar to HashMap. In Java 8+, reads generally don't require locking. For updates, if the bucket is empty, CAS can be used to insert the node. For existing bins, fine-grained synchronization is used on the relevant bin instead of locking the entire map. Collisions are handled using linked nodes, and heavily-collided bins can be converted to Red-Black Trees. Older Java 7 implementations used segments, but Java 8+ uses bucket-level concurrency rather than segments.


 **🧠 5 words mein:**
**Read → no lock generally**  
**Empty write → CAS**  
**Collision write → fine-grained lock**  
**Heavy collision → Tree**  
**Java 7 → Segments, Java 8+ → Buckets**

---------------------------

**Question 14: Why Concurrenthashmap when already HashTable present?

interview me ye bolo -
> Hashtable is thread-safe, but its methods are synchronized, which limits concurrency because multiple threads may contend on the same synchronization mechanism. ConcurrentHashMap is designed for high-concurrency access. In Java 8+, reads generally don't require locking, and updates use CAS and fine-grained synchronization at the bucket/bin level instead of locking the entire map.


**Segmentation ? Locking ?

> In Java 7 it used segmentation, where different segments had independent locks.

**Segmentation Java 7 ke ConcurrentHashMap mein tha. Java 8+ mein segment-based design nahi hai.** Interview mein ye distinction bolna important hai.

Segmentation kya tha ? Purane **Java 7 ConcurrentHashMap** ko imagine karo:

```
ConcurrentHashMap
       |
  -----------------
  |       |       |
Segment Segment Segment
  1        2       3
  
Har segment ka apna lock tha.
	Segment 1 → keys A, B, C
	Segment 2 → keys D, E, F
	Segment 3 → keys G, H, I

Agar:
	Thread 1 → Segment 1 mein write
	Thread 2 → Segment 2 mein write
	
to dono simultaneously kaam kar sakte the.

Lekin agar dono:
	Thread 1 → Segment 1
	Thread 2 → Segment 1

to same segment ka lock chahiye → ek ko wait karna padega.

Java 7 CHM = multiple segments + each segment has its own lock.

Java 8+ ConcurrentHashMap mein `Segment[]` architecture nahi hai. 
Writing ke time entire Map ko lock nahi karta.
Instead fine-grained locking use hoti hai.

```


---

**Question 15: Concurrent modification ?**

It occurs when a collection is structurally modified while an iterator is iterating over it, in a way that the iterator does not expect. For example, removing an element directly from an ArrayList inside a for-each loop can cause ConcurrentModificationException. ArrayList iterators are fail-fast and detect this unexpected modification.


Agar pooche **"Does concurrent mean multiple threads?"**

> **“Not necessarily. ConcurrentModificationException can occur even in a single thread when we modify a collection directly while iterating over it.”**


**Iterator bolta hai:**

> Main collection traverse kar raha hoon, tum beech mein structure mat badlo. 😄

-----

**Question 16: Iterator  aur splitIterator ?**

Iterator - Iterator ek object hai jo Collection ke elements ko one-by-one traverse/read karne ke liye use hota hai.

I**terator ke 3 important methods -**

Interview ke liye mainly ye 3 yaad rakho:

1.`hasNext()`  check karta hai - Kya next element available hai?  return true/false

2.`next()` - Next element deta hai.

3.`remove()` - Iterator ke through current element remove kar sakte ho.
Ye important hai because iteration ke time safe removal ke liye use kiya ja sakta hai.

**Real example**

```java
List<Integer> list = new ArrayList<>();

list.add(10);
list.add(20);
list.add(30);

Iterator<Integer> it = list.iterator();

while (it.hasNext()) {
    int x = it.next();

    if (x == 20) {
        it.remove();
    }
}

Before:
10 20 30

After:
10 30
```

**Iterator kyun chahiye?**
	Tum bol sakte ho: "Bhai `for` loop already hai, Iterator kyun?"

**Iterator common traversal mechanism provide karta hai. (Different collections mein)**
ArrayList, LinkedList, HashSet, TreeSet etc.
sabki internal implementation different hai.

Lekin: collection.iterator()
se tum unko common way mein traverse kar sakte ho.


> **Iterator = collection ke elements ko one-by-one traverse karne ka standard way.**


----------

**Question 17: In which scenario, LinkedList is better than ArrayList in Java ?**

🎯 Interview mein exactly bolo

> **“LinkedList can be better than ArrayList when we frequently insert or remove elements from the beginning or from a position where we already have an iterator/reference. LinkedList can perform that insertion or deletion in O(1), whereas ArrayList may need to shift elements, which takes O(n). However, if we need to first traverse to find that position, LinkedList still takes O(n), and ArrayList generally has better cache locality and random access.**

**ArrayList = Read/Access fast**  
**LinkedList = Modification at known node fast**

