

### How can I use user defined class as a key in Hashmap ? What method do we need to override ? Do we need to make the class immutable ?

Yes, you can absolutely use a user-defined class as a key in a `HashMap`. In fact, this is a very common Java interview question.

#### What do you need to override?

You should override **both**:

1. `hashCode()`
2. `equals()`

These two methods work together.

##### Why?

A `HashMap` stores entries using this process:

```
put(key, value)
      |
      v
key.hashCode()
      |
      v
Find Bucket
      |
      v
Compare using equals() if multiple keys exist in same bucket
```

Similarly, when retrieving:

```
get(key)
      |
      v
key.hashCode()
      |
      v
Find Bucket
      |
      v
equals() with existing keys
      |
      v
Return Value
```

---

#### Example

```
class Employee {

    private int id;
    private String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;

        if (!(obj instanceof Employee))
            return false;

        Employee other = (Employee) obj;

        return id == other.id &&
               Objects.equals(name, other.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}

public class Main {
    public static void main(String[] args) {
      Map<Employee, String> map = new HashMap<>();

      Employee e1 = new Employee(1, "John");
      Employee e2 = new Employee(1, "John");

      map.put(e1, "Developer");

      System.out.println(map.get(e2)); 
    }
}
```

Output

```
Developer
```

Because

- `e1.hashCode() == e2.hashCode()`
- `e1.equals(e2) == true`

---

#### What if we don't override hashCode()?

Suppose you only override `equals()`.

```
Employee e1 = new Employee(1, "John");
Employee e2 = new Employee(1, "John");

map.put(e1, "Developer");

System.out.println(map.get(e2));
```

Output

```
null
```

Why?

Because the default `hashCode()` from `Object` is based on the object's identity (memory address). So:

```
e1.hashCode() -> 12345

e2.hashCode() -> 98765
```

Different buckets are used.

`equals()` is never even called because the objects are not found in the same bucket.

---

#### What if we override hashCode() but not equals()?

Then both objects may go into the same bucket.

But Java compares them using `equals()`.

Default `equals()` from `Object` compares references.

```
e1 == e2
```

which is

```
false
```

So

```
map.get(e2)
```

returns

```
null
```

---

#### Do we need to make the class immutable?

**It is not mandatory, but it is strongly recommended.**

This is one of the most important interview points.

Consider:

```
class Employee {

    int id;

    Employee(int id) {
        this.id = id;
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }

    @Override
    public boolean equals(Object obj) {
        Employee e = (Employee) obj;
        return id == e.id;
    }
}
```

Now,

```
Employee emp = new Employee(1);

map.put(emp, "Developer");
```

The bucket is calculated using

```
hashCode() = hash(1)
```

Now change the key:

```
emp.id = 2;
```

Now

```
hashCode() = hash(2)
```

But the object is **still stored in the bucket for `hash(1)`**.

Now:

```
System.out.println(map.get(emp));
```

Output

```
null
```

The object became "lost" inside the `HashMap`.

---

#### Immutable version

```
class Employee {

    private final int id;
    private final String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // getters only
}
```

No setters.

Now the hash code never changes.

This is why immutable classes make excellent `HashMap` keys.

Examples from the JDK include:

- String
- Integer
- Long

All are immutable.

-----------------
---

## What are Functional interfaces ? Why they were added in Java8 ? Is Functional interfaces annotation mandatory?

A **Functional Interface (FI)** is an interface that contains **exactly one abstract method**.

It can have:

- ✅ One abstract method (**mandatory**)
- ✅ Any number of `default` methods
- ✅ Any number of `static` methods
- ✅ Methods inherited from `Object` (`toString()`, `equals()`, `hashCode()`)

### Example

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}
```

Here `add()` is the only abstract method, so it is a Functional Interface.

---

### Why were Functional Interfaces added in Java 8?

Before Java 8, whenever we wanted to pass behavior (a function) to another method, we had to create:

- A separate class
- An anonymous inner class

This resulted in lots of boilerplate code.

### Before Java 8

```java
interface Greeting {
    void sayHello();
}

public class Demo {
    public static void main(String[] args) {

        Greeting greeting = new Greeting() {
            @Override
            public void sayHello() {
                System.out.println("Hello");
            }
        };

        greeting.sayHello();
    }
}
```

Notice how much code is required just to implement one method.

---

### Java 8 introduced Lambda Expressions

Since a Functional Interface has only one method, Java knows exactly which method the lambda is implementing.

```
Greeting greeting = () -> System.out.println("Hello");

greeting.sayHello();
```

Much cleaner and easier.

---

### Why only one abstract method?

Because a lambda expression doesn't specify the method name. 
Because lambda represents ONE behavior.
lambda can map to only one method.

Example:

```
() -> System.out.println("Hello");
```

Java has to know **which method** this lambda represents.

If there were two abstract methods:

```
interface Test {
    void method1();
    void method2();
}
```

Then

```
Test t = () -> System.out.println("Hello");
```

would be ambiguous.

Java wouldn't know whether the lambda implements `method1()` or `method2()`.

Therefore a Functional Interface must have **exactly one abstract method**.

---

#### Why are Functional Interfaces important?

They are the foundation of many Java 8 features:

- Lambda Expressions
- Method References
- Stream API
- Optional API
- CompletableFuture
- Collections (`forEach()`, `removeIf()`, `sort()`, etc.)

Example:

```
List<String> names = List.of("A", "B", "C");

names.forEach(name -> System.out.println(name));
```

Here `forEach()` expects a `Consumer<T>`, which is a Functional Interface.

---

#### Is `@FunctionalInterface` annotation mandatory?

**No.** It is **optional**, but strongly recommended.

Without the annotation:

```
interface Greeting {
    void hello();
}
```

This is still a valid Functional Interface.

---

#### With the annotation

```
@FunctionalInterface
interface Greeting {
    void hello();
}
```

It behaves the same at runtime, but the compiler now verifies that the interface remains functional.

---

#### Why use `@FunctionalInterface`?

Suppose later someone accidentally adds another abstract method:

```
@FunctionalInterface
interface Greeting {

    void hello();

    void bye();
}
```

Compiler error:

```
Unexpected @FunctionalInterface annotation
Greeting is not a functional interface
Multiple non-overriding abstract methods found
```

Without the annotation:

```
interface Greeting {

    void hello();

    void bye();
}
```

The code compiles, but it is no longer usable with lambda expressions.

The annotation helps catch this mistake early.

---

#### Can a Functional Interface have default and static methods?

Yes.

```java
@FunctionalInterface
interface Vehicle {

    void start();

    default void stop() {
        System.out.println("Stopped");
    }

    static void fuelType() {
        System.out.println("Petrol");
    }
}
```

This is still a Functional Interface because there is only **one abstract method** (`start()`).

---

#### Interview Answer (2-minute version)

> A Functional Interface is an interface that contains exactly one abstract method. It was introduced in Java 8 to support lambda expressions and method references, allowing developers to pass behavior as data with much less boilerplate code than anonymous inner classes. A Functional Interface can also have any number of `default`, `static`, and inherited `Object` methods because only abstract methods count toward the "single abstract method" rule. The `@FunctionalInterface` annotation is optional, but it is recommended because it enables compile-time validation, ensuring that no additional abstract methods are accidentally added and the interface remains compatible with lambda expressions.


### What is a Lambda Expression?

A **Lambda Expression** is a **short way of implementing the single abstract method of a Functional Interface**.

#### Relationship Between Functional Interface and Lambda

A **lambda expression cannot exist by itself**.
It **must always implement a Functional Interface**.

```
Functional Interface
        │
        │ provides one abstract method
        ▼
Lambda Expression
        │
        │ implements that method
        ▼
Object of Functional Interface
```

multiple parameter FI -
```java
@FunctionalInterface
interface Functional {
    int operation(int a, int b);
}

public class Test {
    public static void main(String[] args) {
        // Using lambda expressions to define the operations
        Functional add = (a, b) -> a + b;
        Functional multiply = (a, b) -> a * b;

        // Using the operations
        System.out.println(add.operation(6, 3));  
        System.out.println(multiply.operation(4, 5));  
    }
}

// Output -
9
20
```


## Various types or categories of FI ? Any predefined Fis(ex. Runnable)? What is the difference between Predicate, Function, and Supplier interfaces ?

### Categories of Functional Interfaces

Functional Interfaces can be divided into **two categories**.

#### 1. Custom Functional Interfaces

These are created by developers.

Example:

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}
```

Usage:

```
Calculator c = (a, b) -> a + b;
```

---

#### 2. Predefined Functional Interfaces

Java 8 introduced many ready-made Functional Interfaces in the package:

```
java.util.function
```

These eliminate the need to create common interfaces yourself.

---

#### Other Predefined Functional Interfaces (Before Java 8)

Even before Java 8, Java already had several interfaces that qualified as Functional Interfaces.

|Interface|Abstract Method|Purpose|
|---|---|---|
|`Runnable`|`run()`|Execute a task|
|`Callable<V>`|`call()`|Execute a task and return a value|
|`Comparator<T>`|`compare()`|Compare two objects|
|`Comparable<T>`|`compareTo()`|Natural ordering|
|`ActionListener`|`actionPerformed()`|Event handling|

---

#### Java 8 Predefined Functional Interfaces

The four most important ones are:

|Interface|Takes Input?|Returns Value?|Main Method|
|---|---|---|---|
|`Predicate<T>`|✅ Yes|✅ boolean|`test()`|
|`Function<T,R>`|✅ Yes|✅ Yes|`apply()`|
|`Consumer<T>`|✅ Yes|❌ No|`accept()`|
|`Supplier<T>`|❌ No|✅ Yes|`get()`|

These four cover almost every use case.

---

### 1. Predicate

**Predicate** means **a condition** or **a test**.

It answers:

> Is this true or false?

#### Method

```
boolean test(T t);
```

#### Example

```
Predicate<Integer> isEven = n -> n % 2 == 0;

System.out.println(isEven.test(10));
System.out.println(isEven.test(9));
```

Output

```
true
false
```

#### Use Cases

- Filtering data
- Validation
- Conditions
- Stream API `filter()`

Example:

```
List<Integer> nums = List.of(1,2,3,4,5);

nums.stream()
    .filter(n -> n % 2 == 0)
    .forEach(System.out::println);
```

---

### 2. Function

A **Function** converts one value into another.

Input → Output

#### Method

```
R apply(T t);
```

#### Example

```
Function<String, Integer> length = str -> str.length();

System.out.println(length.apply("Java"));
```

Output

```
4
```

Another example:

```
Function<Integer, Integer> square = n -> n * n;
```

Input:

```
5
```

Output:

```
25
```

#### Use Cases

- Data transformation
- DTO conversion
- Entity → DTO
- Stream `map()`

Example:

```
names.stream()
     .map(String::toUpperCase)
     .forEach(System.out::println);
```

---

### 3. Consumer

A **Consumer** accepts data and performs an action.

It **doesn't return anything**.

#### Method

```
void accept(T t);
```

#### Example

```
Consumer<String> print = s -> System.out.println(s);

print.accept("Hello");
```

Output

```
Hello
```

### Use Cases

- Printing
- Logging
- Saving
- Sending emails
- Updating database

Example:

```
list.forEach(System.out::println);
```

`forEach()` internally uses a `Consumer`.

---

### 4. Supplier

#### Meaning

A **Supplier** supplies or generates data.

It takes **no input** but returns a value.

#### Method

```
T get();
```

#### Example

```
Supplier<Double> random = () -> Math.random();

System.out.println(random.get());
```

Every call returns a new random value.

Another example:

```
Supplier<String> greeting = () -> "Welcome";
```

### Use Cases

- Generate IDs
- Generate timestamps
- Lazy loading
- Object creation

---

### Visual Representation

```
Predicate
Input -------> true / false

Example:
18 -------> true
17 -------> false
```

```
Function
Input -------> Output

"Java" -------> 4
5 -------> 25
```

```
Consumer
Input -------> Action

"Hello" -------> print
100 -------> save
```

```
Supplier
No Input -------> Output

-------> Random Number
-------> UUID
-------> Current Time
```

---

### Real-World Example (E-commerce)

Imagine an online shopping application:

```
Customer Order
      │
      ▼
Predicate
"Is payment successful?"
Returns: true/false

      │
      ▼
Function
Convert OrderEntity → OrderDTO

      │
      ▼
Consumer
Send confirmation email

      │
      ▼
Supplier
Generate Order ID
```

------
------

## What is the default method, and why is it required ?

A **default method** is a method inside an interface that has a **body (implementation)**.

Before Java 8:

- Interfaces could only declare methods.
- No implementation was allowed.

After Java 8:

Interfaces can contain methods with implementation by using the `default` keyword.

Example:

```
interface Vehicle {

    void start();   // Abstract method

    default void stop() {
        System.out.println("Vehicle Stopped");
    }
}
```

Here:

- `start()` → Abstract method
- `stop()` → Default method

---

### Why was the Default Method introduced?

#### The Problem Before Java 8

Imagine Java has an interface:

```
interface Animal {

    void eat();
}
```

Thousands of classes implement it.

```
class Dog implements Animal {

    @Override
    public void eat() {
        System.out.println("Dog is eating");
    }
}
```

```
class Cat implements Animal {

    @Override
    public void eat() {
        System.out.println("Cat is eating");
    }
}
```

Now suppose Java wants to add a new feature:

```
interface Animal {

    void eat();

    void sleep();
}
```

Now every implementing class breaks.

Imagine this happening to millions of Java projects after a JDK upgrade.

That would completely break backward compatibility.

---

### [ Solution: Default Method ]

Instead of making every class implement the new method, Java provides a default implementation.

```
interface Animal {

    void eat();

    default void sleep() {
        System.out.println("Sleeping...");
    }
}
```

Now:

```
class Dog implements Animal {

    @Override
    public void eat() {
        System.out.println("Eating");
    }
}
```

No compile error.

Dog automatically gets `sleep()`.

```
Dog d = new Dog();

d.sleep();
```

Output

```
Sleeping...
```

---

### Real Reason Java 8 Needed Default Methods

Java 8 introduced the **Stream API**.

Java wanted to add methods like:

```
Collection.stream()

List.sort()

Iterable.forEach()

Collection.removeIf()
```

Suppose `List` originally looked like:

```
interface List<E> {

    int size();

    boolean add(E e);
}
```

Java 8 wanted to add

```
stream()
sort()
replaceAll()
```

Without default methods:

Every implementation would have to change.

Examples:

- `ArrayList`
- `LinkedList`
- `Vector`
- `Stack`
- Third-party list implementations

Thousands of libraries would stop compiling.

Instead, Java added them as default methods.

That's why your old Java code continued to work after upgrading to Java 8.

---

#### Can We Override a Default Method?

Yes.

```
interface Vehicle {

    default void horn() {
        System.out.println("Normal Horn");
    }
}
```

```
class BMW implements Vehicle {

    @Override
    public void horn() {
        System.out.println("BMW Horn");
    }
}
```

Output

```
BMW Horn
```

The class's implementation takes precedence over the default one.

---

### Multiple Default Methods (Diamond Problem)

Suppose:

```
interface A {

    default void display() {
        System.out.println("A");
    }
}
```

```
interface B {

    default void display() {
        System.out.println("B");
    }
}
```

Now:

```
class Test implements A, B {

}
```

Compiler error:

```
Duplicate default methods
```

Java doesn't know which implementation to use.

You must override it.

```
class Test implements A, B {

    @Override
    public void display() {
        A.super.display();
    }
}
```

Or

```
B.super.display();
```

You explicitly choose the implementation.



### Question: Can Functional Interface Extend Another Interface?

YES — But with Rules
```
A functional interface can extend another interface ONLY IF:

Total abstract methods count remains ONE
```

```
interface Parent{
  default void print(){
    System.out.println("Hello");
  }
}

@FunctionalInterface
interface Child extends Parent{
  void show();
}
```

Valid because: Abstract methods = 1 only - show() fn.
print() is default method, not abstract.

Invalid Example

```
interface A {
  void test();
}

interface B {
  void demo();
}

@FunctionalInterface
interface C extends A, B {
  
}
```

Compile Error.

Because now:
```
Abstract methods:
1. test()
2. demo()
   
Total = 2 abstract methods.
```

Not functional anymore.

------------
------

## difference between lambdas and anonymous inner classes in terms of scope and performance ?

### Short Interview Answer (1 minute)

> Lambda expressions and Anonymous Inner Classes both provide implementations of functional interfaces, but they differ in scope, syntax, and runtime behavior.
> 
> Lambdas do not create a new scope or a new `this` reference. The `this` keyword inside a lambda refers to the enclosing class.
> 
> Anonymous Inner Classes create a separate class object, so `this` refers to the anonymous class itself.
> 
> In terms of performance, lambdas are generally more efficient because Java uses the `invokedynamic` instruction to create them lazily, whereas anonymous classes generate separate `.class` files and require object creation.
> 
> Therefore, lambdas are preferred for implementing Functional Interfaces, while anonymous classes are useful when multiple methods or state are required.


-------
-----

## Asynchronous Programming? CompletableFuture?

#### Interview Definition

> Asynchronous programming is a programming model where a task starts executing without blocking the current thread. Instead of waiting for the task to finish, the thread continues executing other work, and the result is processed later when it becomes available.

### What is CompletableFuture?

#### Interview Definition

> CompletableFuture is a Java class introduced in Java 8 that represents the result of an asynchronous computation. It allows us to execute tasks in the background, combine multiple asynchronous operations, and process the results once they complete.

### Common Methods

#### supplyAsync()

Runs a task that returns a value.

```
CompletableFuture<String> future =
    CompletableFuture.supplyAsync(() -> {

        return "Hello";
    });
```

---

#### runAsync()

Runs a task that doesn't return anything.

```
CompletableFuture.runAsync(() -> {
    System.out.println("Email Sent");
});
```

---

Difference

```
runAsync()
```

returns

```
CompletableFuture<Void>
```

while

```
supplyAsync()
```

returns

```
CompletableFuture<T>
```

---

#### thenApply()

Transforms result.

```
CompletableFuture<String> future =
CompletableFuture.supplyAsync(() -> "John")
                 .thenApply(name -> name.toUpperCase());
```

Output

```
JOHN
```

---

#### thenAccept()

Consumes result.

```
future.thenAccept(System.out::println);
```

---

#### thenRun()

Runs after completion.

No input.

No output.

```
future.thenRun(() -> {

    System.out.println("Completed");

});
```

---

#### thenCompose()

Used when the second async task depends on the first.

Example

```
Get User
      ↓
Get Orders(userId)
```

```
getUser()
.thenCompose(user -> getOrders(user.getId()));
```

---

# 8. thenCombine()

Used when tasks are independent.

```
Get User
        \
         \
          Combine

         /
Get Orders
```

```
future1.thenCombine(future2, (user, orders) -> {
  return new Dashboard(user, orders);
});
```

---

#### allOf()

Wait for multiple tasks.

```
CompletableFuture.allOf(
	future1,
	future2,
	future3
).join();
```

Useful for

```
Dashboard APIs
```

---

#### anyOf()

Returns whichever completes first.

Useful

```
Multiple cache servers

CDNs

Mirror APIs
```

---

#### Exception Handling

```
future.exceptionally(ex -> {
	return "Default";
});
```

or

```
future.handle((result, ex) -> {

});
```

---

#### join() vs get()

#### get()

```
future.get();
```

Throws checked exceptions

```
InterruptedException

ExecutionException
```

---

#### join()

```
future.join();
```

Throws unchecked exception

```
CompletionException
```

Cleaner.

Most Spring Boot developers prefer

```
join()
```

---

# 13. Real Spring Boot Example

Dashboard API

Without async

```
User user = userService.getUser();

Orders orders = orderService.getOrders();

Wishlist wishlist = wishlistService.getWishlist();
```

Total

```
6 seconds
```

---

Using CompletableFuture

```
CompletableFuture<User> user =
    CompletableFuture.supplyAsync(() - >
        userService.getUser());

CompletableFuture<Orders> orders =
    CompletableFuture.supplyAsync(() - >
        orderService.getOrders());

CompletableFuture<Wishlist> wishlist =
    CompletableFuture.supplyAsync(() - >
        wishlistService.getWishlist());

CompletableFuture.allOf(
    user,
    orders,
    wishlist
).join();
```

Total

```
2 seconds
```

---

### Spring Boot Async

Enable async

```
@EnableAsync
```

Method

```
@Async
public CompletableFuture<User> getUser() {

    return CompletableFuture.completedFuture(user);

}
```

Called by

```
CompletableFuture<User> future = userService.getUser();
```

---

### When should we use CompletableFuture?

Use it when tasks are **independent** and can run in parallel, such as:

- Calling multiple microservices.
- Reading from different databases or caches.
- Fetching dashboard data from several sources.
- Sending notifications (email, SMS, push) without blocking the main request.
- Performing background processing that doesn't need to delay the user response.

Avoid it when:

- Tasks must execute strictly one after another.
- The work is CPU-intensive and parallelism would overwhelm the server.
- The asynchronous overhead outweighs the performance benefit.

---

### Common Interview Questions

#### Q3. What is the difference between `runAsync()` and `supplyAsync()`?

|`runAsync()`|`supplyAsync()`|
|---|---|
|No return value|Returns a value|
|`CompletableFuture<Void>`|`CompletableFuture<T>`|

---

#### Q4. What is the difference between `thenCompose()` and `thenCombine()`?

- **`thenCompose()`**: Use when the second async operation depends on the result of the first.
- **`thenCombine()`**: Use when two async operations are independent and their results need to be merged.

---

#### Q5. What is the difference between `get()` and `join()`?

|`get()`|`join()`|
|---|---|
|Throws checked exceptions|Throws unchecked `CompletionException`|
|Requires `try/catch`|Cleaner to use in many applications|

### [ real world use-case catalog ingestion example async programing ]

--------
---------

## Diff between terminal and intermediate operations in Java 8? Test ability to identify in real programs ?


## Parallel streams? When to use? What are the pitfalls of using parallelStream() in  web application?



---------
-------







# Strings 

## What is the meaning of Immutable in the context of String class in Java ?

In Java, **`String` is immutable**, which means once a String object is created, its content cannot be modified.

#### Interview Answer (30 seconds)

> "In Java, String is immutable, which means after a String object is created, its value cannot be changed. If we perform any operation like `concat()`, `replace()`, or `toUpperCase()`, Java creates a new String object instead of modifying the existing one."

Example:

```
String s = "Hello";
s.concat(" World");

System.out.println(s);
```

**Output:**

```
Hello
```

Why?

Because `concat()` creates a new String, but we didn't store it.

Correct way:

```
String s = "Hello";
s = s.concat(" World");

System.out.println(s);
```

Output:

```
Hello World
```

---

#### Another Example

```
String s1 = "Java";
String s2 = s1;

s1 = "Spring";

System.out.println(s1); // Spring
System.out.println(s2); // Java
```

Explanation:

- `"Java"` object never changed.
- `s1` now points to a new object `"Spring"`.
- `s2` still points to `"Java"`.

---

### Why did Java make String immutable?

Interviewers love this question.

Answer:

1. **Security**
    - Strings are used for passwords, database URLs, file paths, network connections.
    - If Strings were mutable, someone could change these values unexpectedly.
2. **String Pool**
    - Multiple variables can share the same String object.
    - Immutability makes this sharing safe and memory efficient.
3. **Thread Safety**
    - Multiple threads can use the same String object without synchronization because it never changes.
4. **HashMap Performance**
    - `String` caches its `hashCode()`.
    - Since the value never changes, the hash code also never changes, making lookups faster.

---

```
String s = "Hello";

        +---------+
s ----> | "Hello" |
        +---------+

s = s.concat(" World");

        +---------+
        | "Hello" |
        +---------+

        +---------------+
s ----> | "Hello World" |
        +---------------+
```

The original `"Hello"` object is **not modified**.

---

#### One-Line Interview Definition

> "Immutable means the object's state cannot be changed after creation. In Java, String is immutable, so any modification operation creates a new String object instead of changing the existing one."


-------
------
-----

## How many ways are there in Java to create a String object ? What is String interning ?

There are **2 main ways**.

#### Method 1: Using String Literal (Recommended)

```
String s1 = "Java";
```

#### What happens?

- JVM first checks the **String Constant Pool (SCP)**.
- If `"Java"` already exists, it returns the existing object.
- Otherwise, it creates a new object in the String Pool.

Example:

```
String s1 = "Java";
String s2 = "Java";
```

Memory:

```
String Pool

+---------+
| "Java"  |
+---------+
   ↑   ↑
  s1  s2
```

Only **one object** is created.

---

#### Method 2: Using new Keyword

```
String s1 = new String("Java");
```

#### What happens?

JVM performs two steps:

1. Checks the String Pool.
2. Creates a **new object in Heap memory**.

Memory:

```
String Pool

+---------+
| "Java"  |
+---------+

Heap

+---------+
| "Java"  |   <-- New Object
+---------+
      ↑
      s1
```

If the pool doesn't already contain `"Java"`, Java first creates it there and then creates the separate heap object.

---

### Interview Answer

> There are mainly two ways to create a String:
> 
> 1. Using a String literal (`String s = "Java";`) which uses the String Constant Pool.
> 2. Using the `new` keyword (`String s = new String("Java");`) which always creates a new object in the heap.


	The String Constant Pool (SCP), also known as the String Intern Pool, is a specialized memory region located within the **Java Heap Memory** that caches and stores unique string literals.

-----
----

## Why Java Uses String Literals?

Because they save **memory** and improve **performance**.

Without the String Pool:

```
String s1 = "Java";
String s2 = "Java";
String s3 = "Java";
```

Three separate objects would be created.

```
Heap

+--------+
| Java   |
+--------+

+--------+
| Java   |
+--------+

+--------+
| Java   |
+--------+
```

This wastes memory.

---

With the String Pool:

```
String Pool

+--------+
| Java   |
+--------+
 ↑  ↑  ↑
s1 s2 s3
```

Only **one object** is created.

#### Advantages of String Literals

#### 1. Memory Optimization

Duplicate Strings share the same object.

#### 2. Better Performance

No need to create duplicate objects.

#### 3. Faster Comparisons

References can be shared, making some operations more efficient.

#### 4. Reduced Garbage Collection

Fewer duplicate objects mean less work for the GC.

----
-----

## Question -

```
String a = "Java";
a = "Python";
a = "CPP";
a = "Rust";
a = "Redis";
a = "Last String";

what happend to prevoius all strings. please explain it in the point of garbage collection.
```

Let's see what happens step by step.

---

#### Step 1

```
String a = "Java";
```

JVM creates `"Java"` in the **String Constant Pool**.

```
String Pool

+--------+
| Java   | <--- a
+--------+
```

---

#### Step 2

```
a = "Python";
```

Since String is immutable:

- `"Java"` is **NOT modified**.
- A new String `"Python"` is created in the String Pool.
- `a` now points to `"Python"`.

```
String Pool

+--------+
| Java   |
+--------+

+--------+
| Python | <--- a
+--------+
```

Notice:

`"Java"` still exists.

---

#### Step 3

```
a = "CPP";
```

```
String Pool

Java
Python

CPP  <--- a
```

Again,

- `"Python"` is not changed.
- `a` simply starts referring to `"CPP"`.

---

Continue...

```
a = "Rust";
a = "Redis";
a = "Last String";
```

Now the pool contains

```
Java
Python
CPP
Rust
Redis
Last String  <--- a
```

---

### Are all previous Strings Garbage?

**Not immediately.**

This is the important interview point.

The String Constant Pool is a special area that stores interned strings. A string in the pool becomes **eligible for garbage collection only when there are no live references to it anywhere in the application**, and the JVM determines it is no longer needed.

In your example:

```
String a = "Java";
a = "Python";
```

After the reassignment:

- No variable refers to `"Java"` anymore.
- Therefore, `"Java"` becomes **eligible for garbage collection** (assuming no other reference to it exists).

The same happens for `"Python"`, `"CPP"`, `"Rust"`, etc., as `a` keeps changing.

---

### Does Garbage Collector remove them immediately?

**No.**

This is a common misconception.

Garbage Collection works like this:

```
Eligible for GC  ≠  Immediately deleted
```

The JVM decides **when** to run the Garbage Collector based on memory needs and its own algorithms.

So after

```
a = "Python";
```

`"Java"` is simply waiting until the JVM runs GC.

---

### What if another variable points to it?

```
String a = "Java";
String b = "Java";

a = "Python";
```

Memory:

```
Java  <--- b

Python <--- a
```

Now `"Java"` **cannot** be garbage collected because `b` still references it.

---

#### What if we assign the same literal again?

```
String a = "Java";
a = "Python";

String c = "Java";
```

Does Java create another `"Java"`?

**No.**

It reuses the existing `"Java"` from the String Pool if it hasn't been garbage collected.

```
Java <--- c

Python <--- a
```

No duplicate object is created.

---

### Interview Answer (Best)

> When we keep assigning new String literals to the same variable, Java does not modify the existing String because Strings are immutable. Instead, it creates (or reuses) the new literal in the String Constant Pool and changes the reference of the variable. The previous String object becomes **eligible for garbage collection** only if no other references point to it. It is not removed immediately—the JVM's Garbage Collector decides when to reclaim that memory.

---

#### One important correction to a common misconception

Many people say:

> "Strings in the String Pool are never garbage collected."

This was true in very old JVM implementations (before Java 7, where the pool lived in PermGen). **In modern Java (Java 7+), interned strings in the String Pool are ordinary heap objects and can be garbage collected when they are no longer reachable.**


--------
------

## String, StringBilder and StringBuffer object ?

### 1. String

`String` objects are **immutable**, meaning once created, their value **cannot be changed**.

```
String s = "Hello";
s = s + " World";
```

What actually happens:

```
String s = "Hello";          // Object 1
s = "Hello World";           // Object 2 created
```

The original `"Hello"` object is not modified.


---

### 2. StringBuilder

`StringBuilder` is **mutable**, meaning you can modify the same object without creating new ones.

```
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");

System.out.println(sb);
```

Output:

```
Hello World
```

No new object is created for every append.

### When to use?

- Many string modifications
- Single-threaded applications
- Best performance

---

### 3. StringBuffer

`StringBuffer` is also **mutable**, but it is **thread-safe** because its methods are synchronized.

```
StringBuffer sb = new StringBuffer("Hello");
sb.append(" World");
```

Output:

```
Hello World
```

### When to use?

- Multiple threads access the same object
- Thread safety is required

---

# Difference

|Feature|String|StringBuilder|StringBuffer|
|---|---|---|---|
|Mutable?|❌ No|✅ Yes|✅ Yes|
|Thread Safe?|N/A|❌ No|✅ Yes|
|Performance|Slow for modifications|Fastest|Slower than StringBuilder|
|Object creation|New object every modification|Same object|Same object|
|Introduced|Java 1.0|Java 5|Java 1.0|

---

### Why is StringBuilder faster?

Suppose:

```
String s = "";

for(int i = 0; i < 1000; i++) {
    s += i;
}
```

This creates **1000+ new String objects**, making it slow.

Using `StringBuilder`:

```
StringBuilder sb = new StringBuilder();

for(int i = 0; i < 1000; i++) {
    sb.append(i);
}
```

The same object is reused, so it's much faster.

---

### Interview Answer (30 seconds)

> **String** is immutable, so every modification creates a new object.
> 
> **StringBuilder** is mutable and not thread-safe, making it the fastest option for string manipulation in single-threaded applications.
> 
> **StringBuffer** is mutable and thread-safe because its methods are synchronized. It's used when multiple threads modify the same string, though it's slower than `StringBuilder` due to synchronization.

#### Easy way to remember

- **String** → **Cannot change** (Immutable)
- **StringBuilder** → **Can change + Fast + Single Thread**
- **StringBuffer** → **Can change + Thread Safe + Multi Thread**


## Arrange the three classes String, StringBuffer and StringBuilder in the order of efficiency for String processing operations?

For **string processing operations** (like concatenation, `append()`, `insert()`, `delete()`), the order of efficiency is:

#### Most Efficient → Least Efficient

```
StringBuilder > StringBuffer > String
```

#### Why?

#### 1. StringBuilder ⭐ (Most Efficient)

- Mutable
- No synchronization
- Fastest performance
- Best for single-threaded applications

```
StringBuilder sb = new StringBuilder();
sb.append("Java");
sb.append(" Spring");
```

---

#### 2. StringBuffer

- Mutable
- Thread-safe (methods are synchronized)
- Slightly slower than `StringBuilder` because synchronization adds overhead

```
StringBuffer sb = new StringBuffer();
sb.append("Java");
```

---

#### 3. String ⭐ (Least Efficient for modifications)

- Immutable
- Every modification creates a new object
- Slowest for repeated string operations

```
String s = "Java";
s = s + " Spring"; // New String object created
```

---

#### Interview Answer (20 seconds)

> **For string processing operations, the efficiency order is:**
> 
> **StringBuilder > StringBuffer > String**
> 
> - **StringBuilder** is the fastest because it is mutable and not synchronized.
> - **StringBuffer** is slightly slower because it is synchronized for thread safety.
> - **String** is the slowest for repeated modifications because it is immutable and creates a new object for every change.


-----
-----
------


### Diff between or how many objects are created when

```java
String s=new String(“Shiv”);
String s1=”Shiv”;
String s2=”Shiv”;

// output - 2
```


### Concept of interning? Output of below program

```java
String s1 = "Java";
String s2 = "Java";
String s3 = new String("Java");
String s4 = s3.intern();

System.out.println(s1 == s2);System.out.println(s1 == s3);
System.out.println(s1 == s4);
```

```
Pool
-------
"Java"
 ↑   ↑   ↑
s1  s2  s4

Heap
-------
new String("Java")
 ↑
s3
```

```
true
false
true
```

