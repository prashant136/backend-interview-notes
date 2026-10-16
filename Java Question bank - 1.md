**Question 1:  Can we run java without public static void main**
```java
### before java v.25, give error

### from java 25 (dono valid hai)
class HelloWorld {
	public static void main(String[] args) {
		System.out.println("Hello, world!");
	}	
}

class HelloWorld {
	void main() {  
		System.out.println("Hello, world!");  
	}
	
	// ye bhi valid hai...
	void main(String[] args) {  
		System.out.println("Hello, world!");
	}
}
```

**Question 2:  Can we write java file with no class in it ? can we override the main method**

```
yes, from java 25. we must write our method directyl in file, no wrapper class needed. However, compiler still creates a class under the hood. because JVM strictly requires class to execute code.

void main() {
    System.out.println("Look Ma, no class!");
}


No, we can't, becoz main is static and static methods are never overridden. If we write the same main method in child class, that's not overriding. That is method hiding, which method runs is decided at compile time based on the class name, not at runtime based on object.
```

**Question 3:  Can JVM call our overloaded main**

The answer is No, We can definitely overlaod main as many versions as we want, it compiles file. but the jvm only looks for one exact signature: main with a String[ ] param. It completely ignores others.
If we want our overloaded version to run, we have to call ot ourselves from inside the real main.

```
public class OverloadMain {

    // 1. JVM calls THIS method at launch
    public static void main(String[] args) {
        System.out.println("JVM launched main(String[] args)");

        // Calling overloaded versions manually:
        main(10);
        main("Hello", "World");
    }

    // 2. Overloaded main (JVM ignores this at startup)
    public static void main(int number) {
        System.out.println("Overloaded main with int::);
    }

    // 3. Overloaded main (JVM ignores this at startup)
    public static void main(String str1, String str2) {
        System.out.println("Overloaded main with two Strings::);
    }
}
```


**Question 4: Can we print something without writing a main method**

answer varies. 
Until java 6, yes. we could System.out.println() inside a static block, and it would print before the jvm looked for main. The static block runs at class loading time.
but from java 7 onward, JVM checks for the main method first. JVM --> first confirms `main` exists, then loads the static block and finally invokes main().

**Question 5: Can a class extends itself ? Can a class extends two classes ?**
No. otherwise compiler will give us a cyclic inheritance error. reason - if class A extends A, then to create an object of A, the constructor has to call parent constructor, which is A itself.

No, java doesn't allow multiple inheritance of classes. (reason is diamond problem)

If both parents have the same method, the child has no way to decide which one to
inherit. java simply avoid this confusion.

------

**Question 6: Can an interface extends multiple interfaces**

Yes, An interface can extends as many interfaces as it wants. why is it allowed ?
becoz interfaces traditionally had only method declarations, no body and no state. So there was nothing to conflict. No fields, no constructors, no implementation to choose between.

```
interface A { 
    void m1(int a);
}

interface B { 
    void m1();
}

// Ye COMPILE HO JAYEGA (Overloading rule)
interface C extends A, B { 
    // C inherits both m1(int) and m1() seamlessly as overloaded methods
}


######## ------------------------------
interface A { 
    int m1(); // Returns int
}

interface B { 
    void m1(); // Returns void
}

// COMPILE ERROR: m1() in B clashes with m1() in A; incompatible return type
interface C extends A, B { 

}

######### --------------------------

interface A {  
    void m1();  
}  
  
interface B {  
    void m1(); 
}  
  
// m1() from interface C override m1 from A and m1() from B
interface C extends A, B {  
    void m1();  
}
```

**Question 7: Can two interfaces have the same default method**

Yes, they can.
form java 8 we got default methods, which means interfaces now have implementations. so diamond problem came back.
Java handles it by forcing us to decide. If we implement both interfaces, we must override that method ourselves. So Java didn't solve the diamond prob here, it must make us solve it.

```java
interface InterfaceA {
    default void display() { .... }
}

interface InterfaceB {
    default void display() { .... }
}

// COMPILE ERROR: diamond problem
class ClassC implements InterfaceA, InterfaceB {
    // Fails to compile because of ambiguity!
}

// solution -
class ClassC implements InterfaceA, InterfaceB {
    @Override
    public void display() { .... }
}
```

----------

**Question 8: Can an interface have private methods**

Yes, from java 9 onwards. the reason is very simple. once we had multiple default methods, these default methods often needed the same helper code (to share common logic). Earlier we had no way to share that without making it public.

So now an interface can have `private` and `private static` methods, purely for internal reuse. they're not visible to implementing classes.

```java
public interface PaymentProcessor {

    // Default methods introduced in Java 8
    default void processCreditCard(double amount) {
        logTransaction("Credit Card", amount); // Shared private helper
        // Card processing logic...
    }

    default void processUPI(double amount) {
        logTransaction("UPI", amount); // Shared private helper
        // UPI processing logic...
    }

    // Static method introduced in Java 8
    static void printAuditHeader() {
        formatAuditLog("SYSTEM AUDIT"); // Shared private static helper
    }

    // 1. Private instance method (Java 9+) - visible only inside this interface
    private void logTransaction(String type, double amount) {
        System.out.println("Processing " + type + " payment of $" + amount);
    }

    // 2. Private static method (Java 9+) - can be called by default or static methods
    private static void formatAuditLog(String message) {
        System.out.println("=== " + message + " ===");
    }
}
```
-------

**Question 9: Can an interface have constructor**

No, A constructor exists to initailize the state of an object. An interface has no state to initialize. Whatever varibles we declared in interface are automtically `public static final`, which means they are constants, initialized once at the interface.

---------

**Question 10: Can an abstract class have constructor**

Yes, people think "we can't create an object of an abstract class", so why would it have a constructor ?" but remember, when we create a child class object, the parent constructor always runs first. so the abstract class constructor is needed to initailized the fields that the child inherits.
If we don't wrote it, java add a default constructor anyway.


-------
Question 10: Can we create an object of an abstract class

No, the object belongs to only subclass inherit form abstract class. So directly, never but through subclass, we can always.

----
Question :: Can we override static method
No, overriding resolved at runtime based on actual object.

-----------

----------

--------


Question :: What happens internally when you create an object using new ?

When we create an object using new, Java does several steps -
1.) memory is allocated -> java first creates space in memory (heap) for new object.
2.) default values are set -> all variables of the object are given default values, like 0 for numbers, false for boolean and null for objects.
3.) constructor is called -> set the values to object
4.) reference is return -> finally the address (ref.) of the object is returned and stored in our variable. 

------

Question :: Can we create an object without new keyword ? If yes, how ?
there are several methods like -
- using Clone()
- using Reflection
- using deserialization
- using factory method

-------
Question :: Can a class be declared without any variable or methods ?

Yes, class can be declared without any variables or methods. java still creates a .class file. it still has a default constructor and we can create its objects.

---

Question :: marker class ? When we need a class just to represent its type so we create empty class like this and this is also called as marker class.

--------

Question :: What is the difference between object and object reference ?
First, Object and object ref. aren't the same. Object is the real data, stored in the heap and created using 'new' keyword whereas object ref. is a variable that stores the address of the object, stored in the stack memory and used to access the object.
Student s = new Object();  // 's' is the ref. and its point to the actual object in heap.

--------

Question :: Does assigning null to reference delete the object ?
No, suppose in example -
Student s = new Student(); when we do s = null, the reference is removed, the object is still these but unreachable. Object is not immediately destroyed, Garbage collector will remove it later.

-----------
Question :: Can a class have only static members ? If yes then is it still be object oriented ?
Answer is yes.

-------

Question :: Is encapsulation only about making variables private ?
Answer is No. encapsulation means, keeping data safe by making variables private and controlling how it is accessed by getters and setters.

-----
Question :: If a class has private fields but public setters, is it truly encapsulated ?
Yes, private field  + public setter (partial encapsulated).  
Encapsulation = If there is field age and set age method, so we cannot set negative number in the age attribute. validation inside setter.

Question :: Why do we use getters setter when we can make fields public and setting getting directly?
encapsulated hiding data by private variable. using getter and setter (private field doesn't expose outside).

Question :: What are anonymous classes?
An anonymous class is a class without a name. an inner class that does not have a name and is declared and instantiated at the same time in a single expression.

Question :: Why do we need abstract classes when we already have interfaces?


Question :: Can abstract class exits without any abstract method ?
Yes.

Question :: Why can interface not have instance variable ?
interface is contract, not an object blueprint. instance variable belong to objects. interface doesn't create objects.

Question :: 

Question :: Can an interface exist without any methods ?
Yes, it is called marker interface. Serializable and Cloneable.

Question :: What exactly makes an interface a functional interface ?
An interface 

Question :: 

Question :: 



polymorphism
abstraction
marker interface

