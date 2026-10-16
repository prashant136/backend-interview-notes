
##### [ kuch most important yaad rakho ] 
1. Agar object locked nhi h to kabhi bhi thread nhi chalega. means object locked/unlocked ke condition me thread ka concept aata h. jab hum object/class level syncronized keyword use krte h. syncronized method, syncronized block dono.
2. syncronized keyword use krte h phir jab koi thread t1 execute hota hai. sabse phle wo thread t1 object ko lock aqauire krta h phir execute hota h. object lock hoga to koi dusra thread t2 usee same time me use nhi kar payega. esme bhi do condition h. ya toh thread t1 pura execute hoga toh lock release karega. nhi to phir thread t1 waiting state me chala jayega lock realease krne ke baad. agar t1 waiting state me chala gya lock release krne ke baad phir koi dursa thread t2 us ko execute karega. (lekin thread t2 waiting state nhi nhi hona chahiye. uske liye hum notify() use krte h).
3. wait(), notify(), notifyAll() ye tino object class me already present hota h. thread class ya runnable interface me nhi. In method ko hum syncronized method ya block me use krte h, wo bhi exception handling ke sath, nhi to error dega. (InterruptedException)



**Question 1: Thread vs Runnable? Creating threads in diff ways? Runnable as lambda ?**

```java
// implement Runnable - prefered
class World implements Runnable {
    public void run() {
	    for(;;) {
	        System.out.println(Thread.currentThread().getName());
        }
    }
}

public class Main {
	public static void main() {
		Thread t1 = new Thread(new World());
		t1.start();   // creates a new thread and then executes run()
	}
}
```

```java
class World extends Thread {
    public void run() {
	    for(;;) {
	        System.out.println(Thread.currentThread().getName());
        }
    }
}

public class Main {
	public static void main() {
		Thread t1 = new World();
		t1.start();
	}
}
```

- if class is extends some other classes, then use Runnable interface. (prefered)
```java
class A {}
class B extends A {}

class B extends A implements Runnable {

}
```

Runnable + Lambda 

```java
Thread t = new Thread(() -> {
    System.out.println("Task running");
});

t.start();
```

----

**Question 2: Different ways of synchronization of threads in Java ?**


```java
// Synchronized method - Entire method is locked.
public synchronized void increment() {
    count++;
}

// Synchronized block - Only critical section is locked. More fine-grained.
public void increment() {
    synchronized (this) {
        count++;
    }
}

// Static synchronized - Lock is on the Class object, not individual object.
public static synchronized void increment() {
    count++;
}
```

example -

```java

class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }
    
    public void decrement() {
        synchronized (this) {
            count--;
        }
    }
    
    public int getCount() {
        return count;
    }
    
    public static synchronized void printMessage() {
        System.out.println(
            Thread.currentThread().getName() + " -> Static method"
        );
    }
}

public class Main {
	// InterruptedException jaruri h for exception handling
    public static void main(String[] args) throws InterruptedException {

        Counter counter = new Counter();

        // Synchronized method
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        // Synchronized block
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.decrement();
            }
        });

        t1.start();
        t2.start();

        t1.join();  
        t2.join();

        System.out.println("Final count: " + counter.getCount());

        // Static synchronized method
        Thread t3 = new Thread(Counter::printMessage);
        Thread t4 = new Thread(Counter::printMessage);

        t3.start();
        t4.start();

        t3.join();
        t4.join();
    }
}
```


⭐️ `thread.join()` ka simple meaning:

> Main thread, jis thread par `join()` laga hai uske complete hone tak wait karega.


------------

**Question 3: What is the difference between synchronizing on a method vs synchronizing ?**

synchronizing method -
- **Poora method lock** ho jata hai.
- Ek time par **sirf ek thread** is object ke is synchronized method ko execute karega.
- Simple hai, but kabhi-kabhi unnecessary code bhi lock ho jata hai.

synchronizing block -
- **Sirf block ke andar ka code lock** hota hai.
- Baaki method ko multiple threads simultaneously execute kar sakte hain.
- Isliye **more fine-grained control** milta hai.

----------

**Question 4: What is the difference between synchronized instance methods and synchronized static methods?**


```java
public synchronized void increment() {
    count++;
}
```

🔒 Lock **object (`this`)** par lagta hai.
Counter c1 = new Counter();
Counter c2 = new Counter();
c1, c2 dono objects ke lock alag hai.  Isliye `c1` aur `c2` ke methods **simultaneously run** kar sakte hain.




```java
public static synchronized void print() {
    System.out.println("Hello");
}
```

🔒 Lock **class (`Counter.class`)** par lagta hai. Isliye `Counter` ke **saare objects ke liye same lock** hai.
Counter c1 = new Counter();
Counter c2 = new Counter();
Agar `c1` aur `c2` dono static synchronized method access kar rahe hain → **ek time par sirf ek thread**.


```
instance synchronized → object lock
static synchronized   → class lock
```


----------------

**Question 5: What object is used as the lock when you declare a method as synchronized? ( Jab kisi method ko `synchronized` declare karte hain, toh lock kis object par lagta hai ? )**

For a synchronized instance method, the lock is acquired on the current object (`this`). For a static synchronized method, the lock is acquired on the class object.

-------------

**Question 6: Can constructors be synchronized? Why or why not?**

No bro, Java me constructors ko `synchronized` nahi kar sakte.

```java
class MyClass {
	public synchronized MyClass() { }  // ❌ Compile error
	
	//...
}
```

why: 
Constructor ka kaam **new object create/initialize karna** hai. Lekin constructor execute hote time object **abhi fully create/initialize nahi hua hota**, so constructor ko synchronize karne ka concept nahi hai.

> **"No, constructors cannot be synchronized in Java because synchronization requires a lock on an object, while a constructor is used to create and initialize the object itself."**

----------

**Question 7: What happens if two threads call two different synchronized methods on the same object ?**

Ha, **dono methods synchronized hain aur same object par call ho rahe hain**, toh **ek time par sirf ek thread execute karega**.

```
class Counter { 
	public synchronized void method1() { } 
	public synchronized void method2() { }
}

Counter obj = new Counter();
Thread-1 → obj.method1();
Thread-2 → obj.method2();
```


Dono methods ka lock **same `obj` (`this`)** hai.

So:

```
Thread-1 → method1() → 🔒 obj lock
Thread-2 → method2() → ⏳ waits
```

Thread-1 jab lock release karega, tab Thread-2 `method2()` execute karega.


> **"If two threads call different synchronized instance methods on the same object, they cannot execute simultaneously because both methods acquire the same object's lock."**

**Important:** Agar objects different hain:

```
obj1.method1();
obj2.method2();
```

toh **dono simultaneously execute ho sakte hain**, kyunki locks different objects ke hain.

------

**Question 8: What is the impact of synchronized on performance compared to ReentrantLock ?**

Dono ka basic kaam **thread ko lock karna** hai. Normal use case mein performance difference usually **bahut bada nahi hota**.

```
synchronized
     ↓
JVM khud lock/unlock manage karti hai

ReentrantLock
     ↓
Developer lock/unlock explicitly manage karta hai
```

**`synchronized`**

- Simple locking ke liye good.
- Modern JVM isko optimize karti hai.
- Lock/unlock automatically hota hai.

**`ReentrantLock`**

- `tryLock()` → lock available hai ya nahi check kar sakte ho.
- Fairness option → waiting threads ko fair chance.
- Interruptible locking → waiting thread ko interrupt kar sakte ho.
- In advanced scenarios, ye flexibility performance/concurrency tuning me help kar sakti hai.

Important interview point

**Ye mat bolna:**

> "`ReentrantLock` is always faster than synchronized." ❌

Correct:

> **"Modern JVMs optimize synchronized very well, so for basic locking their performance is generally comparable. ReentrantLock is useful when we need advanced features like tryLock, fairness, or interruptible locking."**

Shortcut

```
Basic locking      → synchronized
Advanced control   → ReentrantLock
Performance        → generally comparable
```

--------

**Question 9:  Race Condition vs Visibility problem? How volatile solves visibility ?**


Race Condition - Jab **multiple threads same data ko same time modify** karte hain aur result execution order par depend karta hai.
**Solution:** `synchronized`, `Lock`, `AtomicInteger` etc.

Visibility Problem - Ek thread ne variable change kiya, but **dusre thread ko updated value immediately dikhe hi nahi**.

> Race condition occurs when multiple threads access and modify shared data concurrently, causing incorrect results. Visibility problem occurs when one thread's update is not visible to another thread. Volatile guarantees visibility of the latest value across threads, but it does not provide atomicity.

**Atomicity** ka simple matlab:

> **Koi operation ya toh poora complete hoga, ya bilkul nahi hoga — beech mein doosra thread interfere nahi kar sakta.**

Atomicity means an operation happens as one indivisible unit. No other thread can observe or interfere with the operation in the middle.

```
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();


Atomicity  → operation tootega nahi
Visibility → latest value dikhegi
```

-------

**Question 10: What is a daemon thread in Java? How will you make a user thread into daemon thread if it has already started?**

Daemon thread ek **background thread** hota hai jo application ke main/user threads ke liye support kaam karta hai.


```java
Thread t = new Thread(() -> {
    while (true) {
        System.out.println("Background work...");
    }
});

t.setDaemon(true);
t.start();
```

Jab **saare user threads finish ho jaate hain**, JVM daemon threads ko automatically terminate kar deti hai.


```
User Thread   → JVM iske finish hone ka wait karti hai
Daemon Thread → JVM iske liye wait nahi karti
```

Common examples: background monitoring, cleanup/support tasks.



> **Agar user thread already start ho chuka hai, kya usko daemon bana sakte hain?**

**NO.**

Thread t = new Thread(...);
t.start();
t.setDaemon(true);   // ❌ IllegalThreadStateException

`setDaemon(true)` **thread start hone se pehle** call karna zaroori hai.


Correct:

```
Thread t = new Thread(...);

t.setDaemon(true);   // ✅
t.start();            // ✅
```

### Interview me exactly bolo:

> **"A daemon thread is a background thread that doesn't prevent the JVM from exiting. We must call `setDaemon(true)` before `start()`. Once a thread has started, its daemon status cannot be changed."**

------

A. agar sare thread execute ho gye h. aur demon thread me koi eisa task h jisse thread block ho gya h. to kya main thread use terminate kr dega ?

B. agar koi user thread aisa h jo thread block kr dia h.  Main thread us user thread ka wait karega (thread block) ya terminate karega ?  demon thread ka kya hoga ?

A. Daemon thread blocked ho gaya toh?

```
Main thread      → DONE
User threads     → DONE
Daemon thread    → BLOCKED / WAITING
```

Phir bhi **JVM terminate ho jayegi**.

Daemon thread ke blocked hone se JVM uska wait **nahi karti**.

⚠️ Main thread daemon ko explicitly terminate nahi karta. **JVM shutdown hoti hai**, aur daemon thread bhi uske saath khatam ho jata hai.



B. User thread blocked ho gaya toh?

```
Main thread      → DONE
User thread      → BLOCKED ⏳
Daemon thread    → RUNNING
```

JVM **terminate nahi hogi**, kyunki ek user thread abhi alive hai.

JVM wait karegi jab tak **saare non-daemon/user threads finish nahi ho jaate**.

```
User thread blocked
       ↓
JVM alive rahegi
       ↓
User thread unblock/finish
       ↓
All user threads finished
       ↓
JVM exits
       ↓
Daemon threads terminate
```

Ek line me yaad rakho:

> **JVM daemon thread ka wait nahi karti, but user thread ka wait karti hai.**

Aur ek correction: **main thread bhi ek user (non-daemon) thread hota hai**, lekin woh kisi blocked user thread ko khud "wait" nahi karta unless tum explicitly `join()` etc. use karo. **JVM** alive rehti hai because user thread alive hai.

---------

**Question 11: Runnable vs Callable ?**

| Runnable                                        | Callable                              |
| ----------------------------------------------- | ------------------------------------- |
| `run()`                                         | `call()`                              |
| return type - void                              | **value return** kar sakta hai        |
| Checked exception directly throw nahi kar sakta | Checked exception throw kar sakta hai |

```java
// Runnable example -
Runnable task = () -> {
    System.out.println("Task running");
};

Thread t = new Thread(task);
t.start();
```


Maan lo hume calculation karni hai:

```java
// Callable example
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) throws Exception {

        ExecutorService executor = Executors.newSingleThreadExecutor();

        Callable<Integer> task = () -> {
            return 10 + 20;
        };

        Future<Integer> future = executor.submit(task);

        Integer result = future.get();

        System.out.println(result); // 30

        executor.shutdown();
    }
}
```


> **Runnable = kaam karo**  
> **Callable = kaam karo + result do**

Interview me:

> **"Runnable is used when we don't need a result, while Callable is used when we need a result from the task. Callable also supports checked exceptions, and its result is obtained through Future."**

-----


**Question 11: Wait(), notify(), notifyAll()? Present in which class and why ?**

Without proper communication mechanisms, threads might end up in inefficient busy-waiting states, leading to wastage of CPU resources and potential deadlocks.

**Object Class in Java**

Object class (in Java.lang) is the root of the Java class hierarchy. Every class in Java either directly or indirectly extends Object. It provides essential methods like toString(), equals(), hashCode(), clone() and several others that support object comparison, hashing, debugging, cloning and synchronization.

- Acts as the root of all Java classes
- Defines essential methods shared by all objects
- Supports thread communication (wait(), notify(), notifyAll())


Read this article below  (for understanding) :-

**Synchronization**

Synchronization in Java is the process of controlling access to shared resources by multiple threads. It makes sure that only one thread can access a resource at a time, preventing inconsistencies and ensuring data integrity. Synchronization is typically achieved using the `synchronized` keyword, which can be applied to methods or blocks of code.

When a thread enters a synchronized method or block, it acquires the lock (or monitor) for the object or class. While the thread holds the lock, no other thread can enter any synchronized method or block associated with that object or class. When the thread exits the synchronized method or block, it releases the lock, allowing other threads to acquire it.

**Inter-Thread Communication**

Java provides the `wait()`, `notify()`, and `notifyAll()` methods for inter-thread communication. 


The Role of `wait()`, `notify()`, and `notifyAll()` -

The `wait()`, `notify()`, and `notifyAll()` methods are used to manage the execution of threads that need to wait for specific conditions or events. These methods must be called from synchronized context, meaning they should be used within synchronized blocks or synchronized methods.

- `wait()`: This method causes the current thread to wait until another thread invokes the `notify()` or `notifyAll()` method on the same object. It releases the lock on the object and allows other threads to acquire it. The waiting thread remains in the waiting state until it is notified.
- `notify()`: This method wakes up a single thread that is waiting on the object's monitor. If multiple threads are waiting, one of them is chosen to be awakened. The exact thread chosen depends on the thread scheduler.
- `notifyAll()`: This method wakes up all threads that are waiting on the object's monitor. These threads compete for the lock, and one of them will proceed once the lock is released.


The `wait()` Method -

The `wait()` method causes the current thread to wait until another thread invokes the `notify()` or `notifyAll()` method on the same object. When a thread calls `wait()`, it releases the lock on the object and enters the waiting state. This allows other threads to acquire the lock and proceed with their execution.


How `wait()` Works -

When a thread calls the `wait()` method, the following sequence of events occurs:

1. **Release Lock**: The thread releases the lock it holds on the object.
2. **Enter Waiting State**: The thread enters the waiting state and is added to the object’s wait set. The wait set is a collection of threads that are waiting for the object’s monitor.
3. **Wait for Notification**: The thread remains in the waiting state until another thread calls `notify()` or `notifyAll()` on the same object.
4. **Reacquire Lock**: Once notified, the thread attempts to reacquire the lock on the object. If successful, it exits the waiting state and resumes execution from the point where it called `wait()`.

It’s important to note that the `wait()` method must be called from within a synchronized block or method. If it is called outside of a synchronized context, the program will throw an `IllegalMonitorStateException`.

Overloaded `wait()` Methods -

The `wait()` method is overloaded to provide additional functionality. The three versions are:

- `wait()`: Waits indefinitely until notified.
- `wait(long timeout)`: Waits for the specified amount of time (in milliseconds) or until notified, whichever comes first.
- `wait(long timeout, int nanos)`: Waits for the specified amount of time (in milliseconds and nanoseconds) or until notified, whichever comes first.

```java
package ThreadCommunication;  
  
class SharedResource {  
    private int data;  
    private boolean available = false;  
  
    public synchronized void produce(int value) throws InterruptedException {  
        while (available) {  
            wait();  
        }  
        data = value;  
        available = true;  
        notify();  
    }  
  
    public synchronized int consume() throws InterruptedException {  
        while (!available) {  
            wait();  
        }  
        available = false;  
        notify();  
        return data;  
    }  
}  
  
public class WaitNotifyExample {  
    public static void main(String[] args) {  
        SharedResource resource = new SharedResource();  
  
        Thread producer = new Thread(() -> {  
            try {  
                for (int i = 0; i <= 5; i++) {  
                    resource.produce(i);  
                    System.out.println("Produced: " + i);  
                }  
            } catch (InterruptedException e) {  
                Thread.currentThread().interrupt();  
            }  
        });  
  
        Thread consumer = new Thread(() -> {  
            try {  
                for (int i = 0; i <= 5; i++) {  
                    int value = resource.consume();  
                    System.out.println("Consumed: " + value);  
                }  
            } catch (InterruptedException e) {  
                Thread.currentThread().interrupt();  
            }  
        });  
  
        producer.start();  
        consumer.start();  
    }  
}
```



https://medium.com/@AlexanderObregon/javas-wait-notify-and-notifyall-explained-87ec05b6ad65

---------

**Question 12: ExecutorService? Type of thread pool? What considerations while creating thread pool?**

`ExecutorService` Java ka framework hai jo **threads ko manage karta hai** aur tasks ko thread pool me execute karta hai.

```
ExecutorService executor = Executors.newFixedThreadPool(3);

executor.submit(() -> {
    System.out.println("Task running");
});

executor.shutdown();
```

**Interview:**

> "`ExecutorService` is used to manage a pool of threads and execute submitted tasks without manually creating and managing threads."

---

**Types of Thread Pool -** 

**Fixed Thread Pool**

```
Executors.newFixedThreadPool(5);
```

➡️ Fixed number of threads.

**Cached Thread Pool**

```
Executors.newCachedThreadPool();
```

➡️ Threads dynamically create/reuse hote hain.

**Single Thread Pool**

```
Executors.newSingleThreadExecutor();
```

➡️ Sirf **1 thread**, tasks sequentially execute hote hain.

**Scheduled Thread Pool**

```
Executors.newScheduledThreadPool(2);
```

➡️ Task ko **delay ya periodically** execute karne ke liye.

Shortcut -

```
Fixed     → fixed number
Cached    → create/reuse dynamically
Single    → one thread
Scheduled → delayed/periodic tasks
```

---

**Thread pool create karte time kya consider karna hai?**

Main 4 cheezein yaad rakho:

① Task type

- **CPU-intensive** → threads usually CPU cores ke around
- **I/O-intensive** → more threads useful ho sakte hain because threads I/O wait karte hain

 ② Number of threads

Bahut kam → CPU/resources underutilized  
Bahut zyada → context switching + memory overhead

 ③ Queue size

Agar tasks threads se fast aa rahe hain, queue grow karegi.

➡️ **Unbounded queue** → memory problem ho sakti hai.

 ④ Shutdown

Application/service finish hone par:
	executor.shutdown();


 Interview me complete answer:

> **"ExecutorService manages a pool of threads and executes tasks. Common thread pools are fixed, cached, single, and scheduled. While creating a pool, I consider whether tasks are CPU or I/O intensive, pool size, queue capacity, task rejection behavior, and proper shutdown."**

--------

**Question 13: What is the fundamental difference between wait() and sleep() methods?**

Bro, **fundamental difference = lock**. Bas ye yaad rakho:

|`wait()`|`sleep()`|
|---|---|
|`Object` class ka method|`Thread` class ka method|
|**Lock release karta hai** 🔓|**Lock release nahi karta** 🔒|
|Communication ke liye (`notify`)|Thread ko temporarily pause karne ke liye|
|`synchronized` context me call karna hota hai|`synchronized` ki requirement nahi|

```
wait():
lock 🔒 → wait() → lock release 🔓 → waiting
                     ↓
                  notify()
                     ↓
                lock acquire → continue
```

```
sleep():
lock 🔒 → sleep() → lock still held 🔒 → wake up → continue
```

Interview me exactly bolo:

> **"`wait()` is used for thread communication and releases the monitor lock, whereas `sleep()` is used for pausing a thread and does not release any lock."**

**Shortcut:**  
👉 `wait()` = **lock chhod do + wait karo**  
👉 `sleep()` = **so jao, lock mat chhodo**

-----------

**Question 14: What is the use of ThreadLocal class ?**


`ThreadLocal` ka use **har thread ke liye alag variable/value maintain karne** ke liye hota hai.

Matlab multiple threads same `ThreadLocal` use karein, phir bhi **har thread ki value independent hoti hai**.

```java
ThreadLocal<Integer> count = ThreadLocal.withInitial(() -> 0);

Thread-1 → count = 10
Thread-2 → count = 20
```

Thread-1 ko `10` hi milega, Thread-2 ko `20`.

example

```java
ThreadLocal<String> user = new ThreadLocal<>();

Thread t1 = new Thread(() -> {
    user.set("Alice");
    System.out.println(user.get());  // Alice
});

Thread t2 = new Thread(() -> {
    user.set("Bob");
    System.out.println(user.get());  // Bob
});

t1.start();
t2.start();
```

Yahan `user` shared object hai, **but value thread-specific hai**.

Use case

- User/request context
- Transaction context
- Per-thread database connection/context

Interview me bolo:

> **"ThreadLocal provides thread-local storage, meaning each thread gets its own independent copy of a variable, so threads don't need to synchronize access to that variable."**

**Shortcut:**  
`ThreadLocal = same variable, but each thread ki apni value.`


--------

**Question 15:  There are two threads T1 and T2? How will you ensure that these threads run
in sequence T1, T2 in Java?**

Bro, agar **T1 ka poora kaam pehle aur uske baad T2** chalana hai, simplest way hai **`join()`**.

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {

        Thread T1 = new Thread(() -> {
            System.out.println("T1 running");
        });

        Thread T2 = new Thread(() -> {
            System.out.println("T2 running");
        });

        T1.start();

        T1.join();   // Main waits until T1 finishes

        T2.start();  // Then T2 starts
    }
}
```


Question 16:  predict the behaviour ?

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {

        Thread T1 = new Thread(() -> {
            System.out.println("T1 running");
        });

        Thread T2 = new Thread(() -> {
            System.out.println("T2 running");
        });

        T1.start();
        T2.start();

        T1.join();
    }
}
```

Execution

```
Main
 ↓
T1.start()  → T1 running
 ↓
T2.start()  → T2 running
 ↓
T1.join()
 ↓
Main T1 ka wait karega
```

Ab maan lo:

```
T1 → running ⏳
T2 → running ⏳
Main → T1 ke finish hone ka wait
```

**T2 independently execute karta rahega.**

Agar T1 pehle finish ho gaya:

```
T1 → finished ✅
T2 → finished ✅
Main → T1.join() se return
Main → end
JVM → terminate
```

Agar T2 slow hai:

```
T1 → finished ✅
Main → T1.join() complete
Main → end
T2 → still running ⏳
```

**JVM immediately terminate nahi hogi**, kyunki T2 ek **user/non-daemon thread** hai aur abhi running hai.

```
Main finished
     ↓
T2 still running
     ↓
JVM waits for T2
     ↓
T2 finishes
     ↓
JVM terminates
```