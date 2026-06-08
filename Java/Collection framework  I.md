


### Collections Class

Utility class used to perform operations on Collection objects such as sorting, reversing, shuffling, finding min/max, and creating synchronized collections. (`java.util.Collections`)
`Collections` is a **utility class** that provides methods to work with Collection objects like List, Set, Queue, etc.

### Arrays Class

Utility class used to perform operations on arrays such as sorting, searching, comparing, filling, and converting arrays to strings or lists.

### Synchronized Collections

Thread-safe wrappers around collections created using `Collections.synchronizedXXX()`. Useful for multi-threaded access, but in modern applications `ConcurrentHashMap` and other concurrent collections are usually preferred


# Synchronized Collections in Java

A **Synchronized Collection** is a collection where all methods are wrapped with a lock so that only one thread can access it at a time.

Java provides synchronized versions of collections through the `Collections` utility class.

```
List<String> list = Collections.synchronizedList(new ArrayList<>());

Set<String> set = Collections.synchronizedSet(new HashSet<>());

Map<Integer, String> map = Collections.synchronizedMap(new HashMap<>());
```


# Question: Why Do We Need Synchronized Collections?

Normal collections are **not thread-safe**.

```
List<String> list = new ArrayList<>();

// Thread-1
list.add("A");

// Thread-2
list.add("B");
```

If multiple threads modify the collection simultaneously:

- Data corruption can happen
- Elements may be lost
- Unexpected exceptions may occur

To prevent this, Java introduced synchronized collections.

---

# Internal Working

When you create:

```
List<String> list = Collections.synchronizedList(new ArrayList<>());
```

Java internally wraps every method with a lock.

Simplified:

```
public synchronized boolean add(E e) {
    return list.add(e);
}

public synchronized E get(int index) {
    return list.get(index);
}
```

Only one thread can execute a method at a time.

---

# Example

## Without Synchronization

```
List<Integer> list = new ArrayList<>();

Runnable task = () -> {
    for(int i=0; i<1000; i++) {
        list.add(i);
    }
};

new Thread(task).start();
new Thread(task).start();
```

Expected:

```
2000 elements
```

Actual:

```
Maybe 1950
Maybe 1987
Maybe exception
```

because multiple threads modify the same list.

## With Synchronization

```
List<Integer> list =
    Collections.synchronizedList(new ArrayList<>());

Runnable task = () -> {
    for(int i=0; i<1000; i++) {
        list.add(i);
    }
};

new Thread(task).start();
new Thread(task).start();
```

Result:

```
Always 2000 elements
```

because every operation acquires the same lock.

---

# Important Interview Question

## Is synchronizedList completely thread-safe?

Many developers say:

```
Yes
```

Interview answer:

```
No
```

Individual operations are thread-safe.  But compound operations are not.

---

# Iterating Over Synchronized Collection

Another famous interview question.

Wrong:

```
List<String> list =
    Collections.synchronizedList(new ArrayList<>());

for(String s : list) {
    System.out.println(s);
}
```

Another thread may modify the collection during iteration.

Result:

```
ConcurrentModificationException
```

---

## Correct

```
synchronized(list) {
    for(String s : list) {
        System.out.println(s);
    }
}
```

Always lock the collection while iterating.

---

# Drawback of Synchronized Collections

Only one thread can access the collection at a time.

```
Thread-1 -> get()
Thread-2 -> wait

Thread-3 -> wait
```

Even read operations are blocked.

Performance becomes poor under high concurrency.

---

# Better Alternative (Modern Java)

Use collections from package:

```
java.util.concurrent
```

Examples:

|Old Collection|Better Concurrent Collection|
|---|---|
|synchronizedList|CopyOnWriteArrayList|
|synchronizedMap|ConcurrentHashMap|
|synchronizedSet|ConcurrentSkipListSet|

---

# When to Use Synchronized Collections?

### Use when:

✅ Small application
✅ Few threads

```
Collections.synchronizedList(...)
```




`CopyOnWriteArrayList` is a thread-safe variant of `ArrayList` in Java. It achieves thread safety by creating a fresh copy of the underlying array for every mutation operation (e.g., add, set, remove). It is highly optimized for read-heavy, write-rarely scenarios but is very costly to update.

```
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

list.add("Java");
list.add("Spring");
list.add("Kafka");
```

# Why was it introduced?

Normal `ArrayList` is **not thread-safe**.

Suppose:

```
ArrayList<String> list = new ArrayList<>();
```

Thread-1:

```
for(String s : list){
    System.out.println(s);
}
```

Thread-2:

```
list.add("New Item");
```

You may get:

```
ConcurrentModificationException
```

because one thread is iterating while another thread is modifying.

To solve this problem efficiently for **read-heavy applications**, Java provides `CopyOnWriteArrayList`.

# ArrayList vs CopyOnWriteArrayList

| Feature                 | ArrayList                                | CopyOnWriteArrayList           |
| ----------------------- | ---------------------------------------- | ------------------------------ |
| Thread Safe             | ❌ No                                     | ✅ Yes                          |
| Concurrent Modification | ❌ Throws ConcurrentModificationException | ✅ No Exception                 |
| Read Performance        | ✅ Fast                                   | ✅ Very Fast                    |
| Write Performance       | ✅ Fast                                   | ❌ Expensive                    |
| Iterator                | Fail-Fast                                | Fail-Safe (Snapshot)           |
| Memory Usage            | Low                                      | High                           |
| Suitable For            | Single Thread                            | Multi-threaded Read Heavy Apps |
# Interview Answer (2-Minute Version)

> `CopyOnWriteArrayList` is a thread-safe implementation of List from the `java.util.concurrent` package. On every modification operation such as add, remove, or update, it creates a new copy of the underlying array and replaces the old one. Because iterators work on a snapshot of the array, it does not throw `ConcurrentModificationException` during concurrent reads and writes. It is best suited for read-heavy and write-light scenarios such as listener lists, subscribers, and configuration caches. Compared to `ArrayList`, it provides thread safety and fail-safe iteration at the cost of higher memory usage and slower write operations.


# Fail-Fast vs Fail-Safe Iterator

This is one of the most common Java interview questions.

---

# Simple Definition

## Fail-Fast Iterator

If a collection is modified while an iterator is traversing it, the iterator **immediately throws `ConcurrentModificationException`**.

Examples:

```
ArrayListHashMapHashSet
```

---

## Fail-Safe Iterator

If a collection is modified while an iterator is traversing it, the iterator **does not throw exception**.

Instead, it works on a **separate copy (snapshot)** of the collection.

Examples:

```
CopyOnWriteArrayListConcurrentHashMap
```

# Interview One-Liner

> A Fail-Fast iterator (like `ArrayList`) throws `ConcurrentModificationException` when the collection is modified during iteration because it detects changes using `modCount`. A Fail-Safe iterator (like `CopyOnWriteArrayList`) works on a snapshot copy of the collection, so modifications do not affect the iterator and no exception is thrown.



### Question: Explain **why snapshot is needed**, **how it works**, and **what happens during concurrent modification**.

---

# Interview Answer

> In `CopyOnWriteArrayList`, when an iterator is created, it gets a reference to the current array. This array acts as a snapshot of the list at that moment. If another thread modifies the list later, `CopyOnWriteArrayList` creates a completely new array instead of modifying the existing one. Therefore, the iterator continues reading from the old snapshot and is unaffected by concurrent modifications. That's why it never throws `ConcurrentModificationException`.


---

# Question: What is ConcurrentHashMap ? What is the scenario to use ConcurrentHashMap in Java ?

## First Build Intuition

Imagine you have a normal `HashMap`.

```
Map<String, Integer> map = new HashMap<>();
```

Now suppose:
- Thread-1 is updating the map.
- Thread-2 is reading the map.
- Thread-3 is deleting from the map.

All at the same time.

### Problem

`HashMap` is **not thread-safe**.

---

## Solution #1: Collections.synchronizedMap()

```
Map<String, Integer> map =  Collections.synchronizedMap(new HashMap<>());
```

This makes the map thread-safe.

### But New Problem

Only **one thread** can access the map at a time.

```
Thread-1 reading
Thread-2 waiting
Thread-3 waiting
Thread-4 waiting
```

Even readers wait.

This becomes slow in high-concurrency applications.

---

# Why ConcurrentHashMap Was Introduced

Idea:

> Instead of locking the entire map, lock only the part that is being modified.

So multiple threads can work simultaneously.

```
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
```

---

# Definition (Interview)

> ConcurrentHashMap is a thread-safe implementation of Map that allows multiple threads to read and update data concurrently with better performance than synchronizedMap.

---

# Real-Life Analogy

Imagine a library.
### synchronizedMap

One lock on the main gate.

```
Reader enters
Everyone waits
Writer enters
Everyone waits
```

---

### ConcurrentHashMap

Different sections.

```
Science Section
History Section
Math Section
```

Person reading Science does not block someone reading History.

Much faster.

---
# Production Scenario #1 - User Session Cache

Suppose:

```
10,000 users online
```

Store sessions:

```
ConcurrentHashMap<String, UserSession>

sessionMap.put(sessionId, session);
```

Multiple requests:

```
Thread-1 -> User A
Thread-2 -> User B
Thread-3 -> User C
```

All access simultaneously.

Perfect use case.

---

# Production Scenario #2 - API Rate Limiter

Store request count.

```
ConcurrentHashMap<String, Integer>

IP -> Request Count
```

Example:

```
192.168.1.1 -> 50
192.168.1.2 -> 25
```

Many requests update counts concurrently.

---

# Production Scenario #3 - In-Memory Cache

```
ConcurrentHashMap<Long, Product>

productCache.put(id, product);
```

Thousands of threads:
- reading products
- updating products

simultaneously.

---

# Production Scenario #4 - Microservices

Store temporary data:

```
ConcurrentHashMap<String, CompletableFuture<?>>
```

Used in:

- Kafka consumers
- RabbitMQ consumers
- Request tracking systems

---

# Important Interview Question

## Why not HashMap?

```
HashMap
```

Not thread-safe.

Concurrent updates may corrupt data.

---

## Why not synchronizedMap?

```
Collections.synchronizedMap()
```

Thread-safe but locks entire map.

Poor performance under heavy traffic.

---

## Why ConcurrentHashMap?

```
ConcurrentHashMap
```

- Thread-safe
- Better concurrency
- Better throughput
- Locking only where required

---

# Null Question (Very Common)

### HashMap

Allows:

```
map.put(null, "A");
map.put("A", null);
```

---

### ConcurrentHashMap

Not allowed.

```
map.put(null, "A");
map.put("B", null);
```

Throws:

```
NullPointerException
```

---
# Atomic Operations (Important)

Suppose:

```
map.get(key);
map.put(key, value);
```

Not atomic. Two threads may interfere.

Use:

```
map.putIfAbsent(key, value);
```

Only inserts if absent.

---

Another important method:

```
map.computeIfAbsent(key, k -> new ArrayList<>());
```

Very common in production.

Example:

```
userOrders.computeIfAbsent(
    userId,
    k -> new ArrayList<>()
);
```

Creates list only once safely.

# Interview Answer (2-Minute Version)

> ConcurrentHashMap is a thread-safe implementation of the Map interface designed for high-concurrency environments. Unlike HashMap, it supports safe concurrent access by multiple threads. Unlike Collections.synchronizedMap(), it does not lock the entire map for every operation. In Java 8+, it uses CAS and bucket-level locking, allowing multiple threads to read and update different parts of the map simultaneously. It is commonly used in caches, session stores, rate limiters, and shared in-memory data structures in multithreaded applications.

>Use `ConcurrentHashMap` whenever multiple threads need to read and update a shared Map concurrently with good performance. It is the standard choice for thread-safe in-memory data structures in Java applications.


## Question: How will you create an empty Map in Java?

### Short Interview Answer

```
Map<String, Integer> map = new HashMap<>();
```

This creates an empty `HashMap`.

If I need an immutable empty map:

```
Map<String, Integer> map = Collections.emptyMap();
```

Or in Java 9+:

```
Map<String, Integer> map = Map.of();
```

---

### Follow-up Interview Explanation

**Mutable Empty Map**

```
Map<String, Integer> map = new HashMap<>();
```

We can add/update entries later.

```
map.put("A", 10);
```

---

**Immutable Empty Map**

```
Map<String, Integer> map = Map.of();
```

or

```
Map<String, Integer> map = Collections.emptyMap();
```

Trying to add data:

```
map.put("A", 10);
```

Throws:

```
UnsupportedOperationException
```

---

### Production Usage

- `HashMap` → Most commonly used.
- `Map.of()` → Configuration values, constants.
- `Collections.emptyMap()` → Return empty result instead of `null`.

Example:

```
public Map<String, User> getUsers() {    return Collections.emptyMap();}
```

Avoids:

```
NullPointerException
```

---

## Question: What is the difference between remove() method of Collection and remove() method of Iterator?


> `Collection.remove()` removes an element directly from the collection. If used while iterating, it may throw `ConcurrentModificationException`. `Iterator.remove()` removes the current element safely during iteration and is the recommended approach when traversing a collection.
---

## Question: Between an Array and ArrayList, which one is the preferred collection for storing objects?

> Generally, `ArrayList` is preferred because it provides dynamic resizing, generics support, and useful APIs. Arrays are preferred only when the size is fixed and maximum performance or memory efficiency is required. In most enterprise applications, we use `ArrayList`.

---

## Question: Is it possible to replace Hashtable with ConcurrentHashMap in Java?

Yes, in most modern Java applications, we can replace `Hashtable` with `ConcurrentHashMap`.

`Hashtable` is old and makes the **entire map locked** for every operation. So only one thread can read/write at a time.

`ConcurrentHashMap` is better because it allows **multiple threads to read and write safely** with better performance.

> Yes, `ConcurrentHashMap` is usually preferred over `Hashtable` because it is thread-safe like Hashtable but gives much better performance in concurrent environments.

---

## Question:  How CopyOnWriteArrayList class is different from ArrayList and Vector classes?

### `ArrayList`

Not thread-safe.

```
List<String> list = new ArrayList<>();
```

Good for normal single-threaded use.

---

### `Vector`

Thread-safe, but old.

Every method is synchronized, so performance is poor.

```
List<String> list = new Vector<>();
```

---

### `CopyOnWriteArrayList`

Thread-safe, but works differently.

Whenever you modify it, it creates a **new copy of the internal array**.

```
List<String> list = new CopyOnWriteArrayList<>();
list.add("A"); // creates new copied array internally
```

Best when:

```
Reads are very high
Writes are very low
```

Example: list of event listeners, config list, observers.


**One-line interview answer:**

> `CopyOnWriteArrayList` is thread-safe like Vector, but instead of locking every read/write, it creates a fresh copy on modification, so it is best for read-heavy and write-rare scenarios.

---

## Question: Why ListIterator has add() method but Iterator does not have ?

`Iterator` works for all collections:

```
Set
Queue
List
```

But not every collection has a fixed position/index.

So `Iterator` only supports basic operations:

```
hasNext()
next()
remove()
```

### ListIterator in Java - 

ListIterator is an advanced cursor in Java used to traverse elements of List implementations. It extends the Iterator interface.
- Works only with List implementations
- Supports bidirectional traversal (forward & backward)
- Allows modification of elements using add(), set(), and remove()
## Interesting Fact about ListIterator

There is no current element in ListIterator. Its cursor always lies between the previous and next elements. The previous() will return to the previous elements and the next() will return to the next element. Therefore, for a list of length n, there are n+1 possible cursors.
### Forward and Backward Traversal

#### 1. Forward direction iteration

- **hasNext():** This method returns true when the list has more elements to traverse while traversing in the forward direction
- **next():** This method returns the next element of the list and advances the position of the cursor.
- **nextIndex():** This method returns the index of the element that would be returned on calling the __next()__ method.

#### 2. Backward direction iteration

- **hasPrevious():** This method returns true when the list has more elements to traverse while traversing in the reverse direction
- **previous():** This method returns the previous element of the list and shifts the cursor one position backward.
- **previousIndex():** This method returns the index of the element that would be returned on calling the __previous()__ method.

A `List` has order and position/index, so Java knows exactly **where to add** the new element.

```
import java.util.*;

public class GFG {

    public static void main(String[] args) {
        // list of names
        List<String> names = new LinkedList<>();
        names.add("learn");
        names.add("from");
        names.add("Geeksforgeeks");

        // Getting ListIterator
        ListIterator<String> listIterator = names.listIterator();

        // Traversing elements
        System.out.println("Forward Direction Iteration:");
        while (listIterator.hasNext()) {
            System.out.println(listIterator.next());
        }

        // Traversing elements, the iterator is at the end
        // at this point
        System.out.println("Backward Direction Iteration:");
        while (listIterator.hasPrevious()) {
            System.out.println(listIterator.previous());
        }
    }
}

// Output -
Forward Direction Iteration:
learn
from
Geeksforgeeks
Backward Direction Iteration:
Geeksforgeeks
from
learn
```

## Methods of ListIterator

|Method|Description|
|---|---|
|[add(E e)](https://www.geeksforgeeks.org/java/list-adde-ele-method-in-java-with-examples/)|This method inserts the specified element into the list.|
|[hasNext()](https://www.geeksforgeeks.org/java/hasnext-method-in-java-collections/)|This returns true if the list has more elements to traverse.|
|[hasPrevious()](https://www.geeksforgeeks.org/java/hasprevious-method-in-java-collections/)|This returns true if the list iterator has more elements while traversing the list in the backward direction.|
|[next()](https://www.geeksforgeeks.org/java/next-method-in-java-collections/)|This method returns the next element and increases the cursor by one position.|
|[nextIndex()](https://www.geeksforgeeks.org/java/get-previous-and-next-index-using-java-listiterator/)|This method returns the index of the element which would be returned on calling the next() method.|
|[previous()](https://www.geeksforgeeks.org/java/previous-method-in-java-collections/)|This method returns the previous element of the list and shifts the cursor one position backward|
|[previousIndex()](https://www.geeksforgeeks.org/java/get-previous-and-next-index-using-java-listiterator/)|This method returns the index of the element which would be returned on calling the previous() method.|
|[remove()](https://www.geeksforgeeks.org/java/remove-element-arraylist-java/)|This method removes the last element from the list that was returned on calling next() or previous() method element from.|
|[set(E e)](https://www.geeksforgeeks.org/java/arraylist-set-method-in-java-with-examples/)|This method replaces the last element that was returned on calling next() or previous() method with the specified element.|
**One-line interview answer:**

> `Iterator` is generic for all collections, so it cannot safely define where to add an element. `ListIterator` is specific to List, which has ordering and index positions, so it provides `add()` method.
---
