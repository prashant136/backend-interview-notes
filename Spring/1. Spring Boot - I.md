
#  Question: IOC? Purpose and benefits of IOC container in Spring ? DI ? types? explain to the interviewer ?

## IoC in Spring

**IoC means Inversion of Control.**

Normally, our class creates its own dependencies using `new`.

```
class OrderService {
    private PaymentService paymentService = new PaymentService();
}
```

But in Spring, **the control of object creation is given to Spring Container**.

```
@Service
class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Here, `OrderService` is not creating `PaymentService`.  
**Spring creates it and gives it to OrderService.**

---

## IoC Container

**IoC Container is the core part of Spring that creates, manages, wires, and destroys objects.**

These Spring-managed objects are called **Beans**.

### Purpose of IoC Container

Interview answer:

> The purpose of IoC Container is to manage the lifecycle of application objects. It creates beans, injects their dependencies, manages configuration, and provides fully ready objects to the application.

---

## Benefits of IoC Container

1. **Loose coupling**  
    Classes do not create dependencies directly.
2. **Easy testing**  
    We can inject mock objects easily.
3. **Centralized object management**  
    Spring manages bean creation and lifecycle.
4. **Better maintainability**  
    Changing implementation is easier.
5. **Supports dependency injection**  
    Dependencies are automatically provided by Spring.

---

## DI in Spring

**DI means Dependency Injection.**

It is the practical way to achieve IoC.

> DI means Spring provides the required dependency object to a class instead of the class creating it itself.

Example:

```
@Service
class PaymentService {
    public void pay() {
        System.out.println("Payment done");
    }
}

@Service
class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

`OrderService` depends on `PaymentService`, and Spring injects it.

---

## Types of Dependency Injection

### 1. Constructor Injection

```
@Service
class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**Best and recommended approach.**

Use when dependency is mandatory.

Interview line:

> Constructor injection is preferred because it makes dependencies mandatory, supports immutability using final fields, and makes unit testing easier.

---

### 2. Setter Injection
In Setter Injection, Spring injects the dependency using a setter method after the object is created.

```
@Service
class OrderService {
    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Here Spring calls:

```
orderService.setPaymentService(paymentServiceBean);
```

automatically. Use when dependency is optional.

Interview line:

> Setter injection is useful when dependency is optional or can be changed after object creation.

---

### 3. Field Injection
Field Injection means Spring injects the dependency **directly into the class field** using reflection.

```
@Service
class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Here Spring automatically creates `PaymentService` bean and puts it into `paymentService` field.
Not recommended for production.

Interview line:

> Field injection is simple, but not recommended because it makes testing harder and hides dependencies.

---

## Why Is Field Injection Not Recommended in Spring ?

**Field Injection is not recommended because it makes the class harder to test, hides dependencies, and can lead to NullPointerExceptions. Constructor Injection is preferred because dependencies are explicit and can be made immutable using `final`.**

---

## Example of Field Injection (Not Recommended)

```
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;

    public void placeOrder() {
        paymentService.pay();
    }
}
```

### Problems

### 1. Hidden Dependencies

Looking at the constructor, you can't tell what dependencies the class needs.

```
OrderService service = new OrderService();
```

Nothing indicates that `PaymentService` is required.

---

### 2. Difficult Unit Testing

Suppose you want to test `OrderService`.

With field injection:

```
OrderService service = new OrderService();
```

`paymentService` is null.

You must use Reflection or Spring Test Context to inject it.

```
service.placeOrder(); // NullPointerException
```

---

### 3. Can Create NullPointerException

If Spring does not inject the dependency for some reason:

```
paymentService.pay();
```

You get:

```
NullPointerException
```

at runtime.

---

### 4. ==Cannot Make Dependency Final==

This is invalid:

```
@Autowired
private final PaymentService paymentService;
```

Because Spring injects after object creation.

So field injection prevents immutability.
---

## Final Interview Answer

> IoC means Inversion of Control. In Spring, instead of a class creating its own dependencies using `new`, the Spring IoC Container creates and manages those objects. These objects are called beans.
> 
> The IoC Container is responsible for creating beans, injecting dependencies, managing bean lifecycle, and providing fully configured objects to the application.
> 
> Dependency Injection is the way Spring implements IoC. It means dependencies are injected into a class by Spring instead of the class creating them manually.
> 
> There are mainly three types of DI: constructor injection, setter injection, and field injection. Constructor injection is preferred because it makes dependencies mandatory, supports immutability, and improves testability. Setter injection is used for optional dependencies, and field injection is generally avoided in production.

----


# Question: Singleton in Spring vs Singleton implementation in java ? Is it safe to assume that a Singleton bean is thread safe in Spring Framework? Singleton type ?

##### Java Singleton: 

> Only **one object** of a class can exist in the entire JVM/application.

Example:

```
public class DatabaseConnection {

    private static DatabaseConnection instance;

    private DatabaseConnection() {
    }

    public static DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
}
```

---
### Spring Singleton Bean

Spring Singleton means:

> Spring creates **one bean object per Spring IoC container**.

Example:

```
@Component
public class UserService {
}
```

By default, Spring beans are singleton scoped.

So this:

```
@Autowired
private UserService userService;
```

will inject the **same UserService object** wherever needed.

Important difference:

|Java Singleton|Spring Singleton|
|---|---|
|One object per JVM|One object per Spring container|
|Class controls creation|Spring container controls creation|
|Uses private constructor/static method|Uses IoC container|
|Harder to test|Easier to test/mock|
|Manual lifecycle|Spring manages lifecycle|

Interview line:

> Java Singleton is a design pattern implemented inside the class, while Spring Singleton is a bean scope managed by the Spring IoC container.

## Types of Singleton Pattern -

### 1. Lazy Singleton

```
class LazySingleton {

    private static LazySingleton instance;

    private LazySingleton() {
        System.out.println("LazySingleton object created");
    }

    public static LazySingleton getInstance() {
        if (instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }

    public void showMessage() {
        System.out.println("Hello from LazySingleton");
    }
}

public class Main {
    public static void main(String[] args) {

        LazySingleton obj1 = LazySingleton.getInstance();
        LazySingleton obj2 = LazySingleton.getInstance();

        obj1.showMessage();

        System.out.println(obj1 == obj2);
    }
}

// Output - true
```

```
Lazy Singleton is NOT thread-safe.
```

Why?

Imagine two threads:

```
Thread-1 checks instance == null → true
Thread-2 checks instance == null → true

Thread-1 creates object
Thread-2 also creates object
```

So two objects can be created.

---
### 2. Eager Singleton

```
class EagerSingleton {

    private static final EagerSingleton INSTANCE = new EagerSingleton();

    private EagerSingleton() {
        System.out.println("EagerSingleton object created");
    }

    public static EagerSingleton getInstance() {
        return INSTANCE;
    }

    public void showMessage() {
        System.out.println("Hello from EagerSingleton");
    }
}

public class Main {
    public static void main(String[] args) {

        EagerSingleton obj1 = EagerSingleton.getInstance();
        EagerSingleton obj2 = EagerSingleton.getInstance();

        obj1.showMessage();

        System.out.println(obj1 == obj2);
    }
}

// Output - true
```

This is **thread-safe**.

Why?

Because object is created when class loads:

```
private static final EagerSingleton INSTANCE = new EagerSingleton();
```

JVM class loading is thread-safe.

So multiple threads cannot create multiple objects.

---

### 3. Thread-Safe Singleton using synchronized -

```
class SynchronizedSingleton {

    private static SynchronizedSingleton instance;

    private SynchronizedSingleton() {
        System.out.println("SynchronizedSingleton object created");
    }

    public static synchronized SynchronizedSingleton getInstance() {
        if (instance == null) {
            instance = new SynchronizedSingleton();
        }
        return instance;
    }

    public void showMessage() {
        System.out.println("Hello from SynchronizedSingleton");
    }
}

public class Main {
    public static void main(String[] args) {

        SynchronizedSingleton obj1 = SynchronizedSingleton.getInstance();
        SynchronizedSingleton obj2 = SynchronizedSingleton.getInstance();

        obj1.showMessage();

        System.out.println(obj1 == obj2);
    }
}

// true
```

Important:

```
Thread-safe, but slower because every call to getInstance() is synchronized.
```

---

### 4. Double-Checked Locking Singleton

```
class DoubleCheckedLockingSingleton {

    private static volatile DoubleCheckedLockingSingleton instance;

    private DoubleCheckedLockingSingleton() {
        System.out.println("DoubleCheckedLockingSingleton object created");
    }

    public static DoubleCheckedLockingSingleton getInstance() {

        if (instance == null) {
            synchronized (DoubleCheckedLockingSingleton.class) {
                if (instance == null) {
                    instance = new DoubleCheckedLockingSingleton();
                }
            }
        }

        return instance;
    }

    public void showMessage() {
        System.out.println("Hello from DoubleCheckedLockingSingleton");
    }
}

public class Main {
    public static void main(String[] args) {

        DoubleCheckedLockingSingleton obj1 =
                DoubleCheckedLockingSingleton.getInstance();

        DoubleCheckedLockingSingleton obj2 =
                DoubleCheckedLockingSingleton.getInstance();

        obj1.showMessage();

        System.out.println(obj1 == obj2);
    }
}

// true
```

Important:

```
volatile is required to avoid visibility and object creation ordering issues.
```


### 5. Bill Pugh Singleton

```
public class DatabaseConnection {

    private DatabaseConnection() {
        System.out.println("DatabaseConnection object created");
    }

    private static class SingletonHelper {
        private static final DatabaseConnection INSTANCE = new DatabaseConnection();
    }

    public static DatabaseConnection getInstance() {
        return SingletonHelper.INSTANCE;
    }

    public void connect() {
        System.out.println("Connected to database");
    }
}

public class Main {
    public static void main(String[] args) {

        DatabaseConnection db1 = DatabaseConnection.getInstance();
        DatabaseConnection db2 = DatabaseConnection.getInstance();

        db1.connect();

        System.out.println(db1 == db2);
    }
}
```

Output:

```
DatabaseConnection object created
Connected to database
true
```

---

### How Bill Pugh Works

```
private static class SingletonHelper {    
    private static final DatabaseConnection INSTANCE = new DatabaseConnection();
}
```

This inner class is **not loaded immediately**.

It loads only when we call:

```
DatabaseConnection.getInstance()
```

So object creation is lazy.

Also, Java class loading is thread-safe by default.

So this is:

```
Lazy loading + thread-safe + no synchronized overhead
```

Interview line:

> Bill Pugh Singleton uses a static inner helper class. The instance is created only when getInstance() is called, and JVM class loading guarantees thread safety.

---

### 6. Enum Singleton

```
public enum DatabaseConnectionEnum {

    INSTANCE;

    DatabaseConnectionEnum() {
        System.out.println("DatabaseConnectionEnum object created");
    }

    public void connect() {
        System.out.println("Connected to database using Enum Singleton");
    }
}

public class Main {
    public static void main(String[] args) {

        DatabaseConnectionEnum db1 = DatabaseConnectionEnum.INSTANCE;
        DatabaseConnectionEnum db2 = DatabaseConnectionEnum.INSTANCE;

        db1.connect();

        System.out.println(db1 == db2);
    }
}
```

Output:

```
DatabaseConnectionEnum object created
Connected to database using Enum Singleton
true
```

---

### Why Enum Singleton is Best - 

Enum Singleton is considered the safest Singleton implementation in Java.

Enum Singleton protects from:

```
Reflection
Serialization
Cloning
Thread issues
```

Example:

```
DatabaseConnectionEnum.INSTANCE.connect();
```

Interview line:

> Enum Singleton is the safest Singleton implementation because Java internally guarantees only one enum instance, even during serialization and reflection attacks.

## Final Interview Summary

Say this:

> Basic lazy singleton is not thread-safe because multiple threads can enter the null check at the same time and create multiple objects. Eager singleton is thread-safe because the instance is created during class loading, and JVM class loading is thread-safe. Bill Pugh Singleton is better because it gives lazy loading and thread safety without synchronization. Enum Singleton is the safest implementation because it also protects from serialization and reflection issues.

> Basic Lazy Singleton is not thread-safe. Eager Singleton is thread-safe but creates the object early. Synchronized Singleton is thread-safe but slower. Double-Checked Locking gives lazy loading with better performance, but it must use volatile.

---

### Question:  Stateless Bean vs Stateful Bean ?

#### What is State?

> Data stored inside instance variables (class fields).

Example:

```
public class UserService {
    private String currentUser; // state
}
```

Here `currentUser` is the object's state.

---

### Stateless Bean -

A Stateless bean does **not store request/user-specific data in instance variables**.

All data is passed through method parameters and local variables.


```
@Service
public class UserService {

    public String getUserName(Long userId) {
        return "Prashant";
    }
}
```

Here:

```
public String getUserName(Long userId)
```

- userId comes as parameter
- no instance variable used
- no shared data

So this bean is Stateless.

---
### Real Production Example

```
@Service
public class ProductService {

    @Autowired
    private ProductRepository repository;

    public Product getProduct(Long id) {
        return repository.findById(id);
    }
}
```

Most Spring:

```
@Service
@Repository
@RestController
```

classes are Stateless.

---

### Stateful Bean

A Stateful bean stores request-specific or user-specific data in instance variables.

Example:

```
@Service
public class ShoppingCartService {

    private Long userId;

    public void setUser(Long userId) {
        this.userId = userId;
    }

    public Long getUser() {
        return userId;
    }
}
```

Here:

```
private Long userId;
```

is shared by all threads.

## Interview Answer

> A Stateless bean does not store request-specific data in instance variables. It only uses method parameters and local variables, making it naturally thread-safe and suitable for Spring Singleton scope.
> 
> A Stateful bean stores mutable data in instance variables. Since Spring Singleton beans are shared across multiple requests, Stateful beans can lead to race conditions and thread-safety issues. Therefore, Spring services are generally designed as Stateless beans.

### Quick Rule

```
Uses only method parameters → Stateless → Safe
Stores mutable instance variables → Stateful → Unsafe
```

This is why almost every `@Service`, `@Repository`, and `@RestController` you see in production Spring applications is designed to be **stateless**.

## Is Spring Singleton Bean Thread-Safe?

> No, it is not safe to assume that a Spring Singleton bean is thread-safe. Spring guarantees only one instance per container, but thread safety depends on how we write the bean. If the bean is stateless, it is usually safe. If it has shared mutable state, then it is not thread-safe.

Best rule:

```
Spring Singleton Bean + Stateless = Safe
Spring Singleton Bean + Mutable shared state = Not Safe
```

----

# Question: Lifecycle of a bean ?

---

# Question: Autowiring? Different methods of autowiring ? 

Autowiring is Spring’s way of automatically injecting dependencies into beans. It can be done using constructor injection, setter injection, or field injection. Constructor injection is preferred because it makes dependencies mandatory, supports immutability, and is easier to test.

- Constructor Injection ✅
- Setter Injection ✅
- Field Injection ❌ (not recommended)


# What if we implement same interface in two classes and we have to inject one of the class using autowiring? @Primary, @Qualifier, Convention over configuration in qualifying by object name?


---

# Question: @Configuration @Bean and @Component ? does @Configuration @Bean is use togather or saperate?

# 1. @Component

Use when **you own the source code** of the class.

Example:

```
@Component
public class EmailService {

    public void sendEmail() {
        System.out.println("Sending email");
    }
}
```

Spring scans the package:

```
@ComponentScan
```

Finds:

```
@Component
public class EmailService
```

Creates bean automatically.

Equivalent to:

```
EmailService emailService = new EmailService();
```

and stores it inside Spring Container.

---

## Real Life Example

Your project classes:

```
UserServiceOrderServicePaymentServiceEmailService
```

You wrote these classes.

So use:

```
@Service
public class UserService {}
```

```
@Repository
public class UserRepository {}
```

```
@Controller
public class UserController {}
```

These are specialized versions of:

```
@Component
```

Internally:

```
@Service
@Component
```

```
@Repository
@Component
```

```
@Controller
@Component
```

---

# 2. @Bean

Suppose class belongs to a third-party library.

```
ObjectMapper
RestTemplate
AmazonSQS
KafkaProducer
```

You cannot modify these classes.

You cannot write:

```
@Component
public class ObjectMapper
```

because source code isn't yours.

So Spring gives:

```
@Bean    // @Bean means third party
```

---

Example

```
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {

        return new ObjectMapper();
    }
}
```

Spring executes:

```
new ObjectMapper();
```

and stores it as a bean.

Now:

```
@Autowired
private ObjectMapper objectMapper;
```

works.

---

# 3. @Configuration

Now comes the confusing part.

Many people think:

```
@Configuration
```

creates bean.

Wrong.

Its job is:

```
Marks a class that contains bean definitions.
```

Example:

```
@Configuration
public class AppConfig {

}
```

Nothing happens.

No bean created.

---

Bean gets created only when:

```
@Bean
```

is present.

```
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

Here:

```
@Configuration -> configuration class
@Bean -> bean definition
```

---

# Do @Configuration and @Bean Work Together?

### Yes, most commonly.

```
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

This is the standard usage.

---

# Can @Bean Exist Without @Configuration?

Yes.

```
@Component
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

Still works.

But there is a subtle difference.

---

# Important Interview Point

Consider:

```
@Configuration
public class AppConfig {

    @Bean
    public Engine engine() {
        return new Engine();
    }

    @Bean
    public Car car() {
        return new Car(engine());
    }
}
```

### With @Configuration

Spring ensures:

```
Only one Engine bean exists.
```

Even though:

```
engine()
```

is called inside:

```
car()
```

 Spring intercepts the call and returns the already managed singleton bean. Only ONE Engine object exists.

Result:

```
Singleton Engine
Singleton Car
```

---

### Without @Configuration

```
@Component
public class AppConfig {

    @Bean
    public Engine engine() {
        return new Engine();
    }

    @Bean
    public Car car() {
        return new Car(engine());
    }
}
```

Now:

```
engine()
```

creates a fresh object.

Result:

```
Multiple Engine objects may be created.
```

This is why Spring recommends:

```
@Configuration + @Bean
```

for configuration classes.

---


# @Primary and @Qualifier

Suppose you have an interface:

```
public interface PaymentService {
    void pay();
}
```

And two implementations:

```
@Component
public class CreditCardPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("Credit Card Payment");
    }
}
```

```
@Component
public class UpiPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("UPI Payment");
    }
}
```

Now Spring container has **2 beans** of type `PaymentService`.

When you write:

```
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring gets confused:

> "Which PaymentService should I inject?"

Result:

```
NoUniqueBeanDefinitionException
expected single matching bean but found 2
```

---

## Solution 1: @Primary

Use when one implementation should be the **default choice**.

```
@Component
@Primary
public class UpiPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("UPI Payment");
    }
}
```

Spring automatically injects:

```
UpiPaymentService
```

because it is marked as primary.

---

## Real Production Example

Suppose:

```
NotificationService
```

has 3 implementations:

```
EmailNotificationService
SmsNotificationService
PushNotificationService
```

90% of your application sends Email.

So mark Email as primary.

```
@Component
@Primary
public class EmailNotificationService
```

Now all injections automatically get Email unless explicitly overridden.

---

## Solution 2: @Qualifier

Use when you want a **specific bean**.

```
@Component
public class CreditCardPaymentService implements PaymentService {}
```

```
@Component
public class UpiPaymentService implements PaymentService {}
```

Inject specific bean:

```
@Service
public class OrderService {

    @Autowired
    @Qualifier("creditCardPaymentService")
    private PaymentService paymentService;
}
```

Spring injects:

```
CreditCardPaymentService
```

---

## How does Spring know this name?

By default bean name = class name with lowercase first letter.

```
CreditCardPaymentService
```

becomes

```
creditCardPaymentService
```

Similarly:

```
UpiPaymentService
```

becomes

```
upiPaymentService
```

---

# Custom Bean Name

You can define your own name.

```
@Component("credit")
public class CreditCardPaymentService implements PaymentService {}
```

```
@Component("upi")
public class UpiPaymentService implements PaymentService {}
```

Injection:

```
@Autowired
@Qualifier("upi")
private PaymentService paymentService;
```

---

# Constructor Injection (Recommended)

Instead of field injection:

```
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
        @Qualifier("upi") PaymentService paymentService) {

        this.paymentService = paymentService;
    }
}
```

Production code usually uses constructor injection.

# What if both are used?

Example:

```
@Component
@Primary
public class UpiPaymentService
        implements PaymentService {
}
```

```
@Component
public class CreditCardPaymentService
        implements PaymentService {
}
```

Injection:

```
@Autowired
@Qualifier("creditCardPaymentService")
private PaymentService paymentService;
```

Who wins?

✅ `@Qualifier`

Spring injects:

```
CreditCardPaymentService
```

because qualifier is more specific.

---

# Interview Answer (2 Minute Version)

> When multiple beans of the same type exist in Spring, autowiring becomes ambiguous and Spring throws `NoUniqueBeanDefinitionException`.
> 
> `@Primary` is used to mark one bean as the default bean that Spring should inject when no specific bean is requested.
> 
> `@Qualifier` is used to explicitly specify which bean should be injected.
> 
> If both `@Primary` and `@Qualifier` are present, `@Qualifier` takes precedence because it provides an exact bean selection.
> 
> In production, `@Primary` is useful for default implementations, while `@Qualifier` is used when different implementations are required in different parts of the application.

### Easy Memory Trick

```
@Primary = Default Choice
@Qualifier = Exact Choice
```


# # What is @Inject ?

`@Inject` is a **Dependency Injection annotation** defined by the Java standard.

Package:

```
import jakarta.inject.Inject;
```

or older:

```
import javax.inject.Inject;
```

It is part of:

- JSR-330 (Java Dependency Injection Specification)
- Standard Java DI annotation
- Not Spring-specific

Think:

```
@Inject  -> Java Standard
@Autowired -> Spring Framework
```

---

### Why was @Inject created?

Suppose today your application uses Spring.

Tomorrow company wants:

- Spring
- Google Guice
- CDI (Jakarta EE)

If you use:

```
@Autowired
```

your code becomes Spring dependent.

If you use:

```
@Inject
```

your code follows Java standard and can work with multiple DI frameworks.

---
### Service

```
@Service
public class EmailService {

    public void sendEmail() {
        System.out.println("Sending Email");
    }
}
```

### Using @Inject

```
@Component
public class NotificationService {

    @Inject
    private EmailService emailService;

    public void notifyUser() {
        emailService.sendEmail();
    }
}
```

Spring sees:

```
@Inject
```

and injects dependency exactly like:

```
@Autowired
```

# Interview Answer (2-Minute Version)

**What is @Inject?**

`@Inject` is a standard dependency injection annotation defined by JSR-330 (Jakarta Inject). It is framework-independent and can be used in Spring, CDI, and Guice. Spring fully supports `@Inject`, so it works similarly to `@Autowired`.

**Difference from @Autowired?**

- `@Autowired` is Spring-specific.
- `@Inject` is Java standard.
- `@Autowired` supports `required=false`.
- `@Inject` does not.
- For multiple beans, `@Autowired` uses `@Qualifier`, while `@Inject` uses `@Named`.

**Production Preference?**

Use constructor injection. In modern Spring Boot, a single constructor doesn't need `@Autowired` or `@Inject` at all. Spring automatically injects the dependency.

