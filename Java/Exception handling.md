
**Question 1: Throwable class and Object class** ?

In Java, `Object` and `Throwable` are two fundamental classes in the `java.lang` package.

*The Object class is the root of the entire Java class hierarchy, while the Throwable class is the root of Java's exception handling hierarchy and directly inherits from Object.*

```
		       java.lang.Object
		              │
		      java.lang.Throwable
		        ┌─────┴─────┐
		  java.lang.Error   java.lang.Exception
		                          │
		                     java.lang.RuntimeException
		                     		                     

Error Class: Represents catastrophic situations such as hardware failure, memory exhaustion (OutOfMemoryError), or stack overflow (StackOverflowError). Your application should not try to catch these errors because recovery is generally impossible.

RuntimeException Class: A special subclass of `Exception` that represents unchecked exceptions.
```

-----

**Question 2: What is the base class for Error and Exception classes in Java ?**

Answer: Throwable Class

Interview me exactly bolo:

> Throwable is the superclass of both Error and Exception in Java. Error represents serious problems generally not expected to be handled by the application, while Exception represents conditions that an application can handle.

-----

**Question 3: What is a finally block ? Output of following**

 Interview mein exactly bolo:

>  The finally block is used for cleanup code and normally executes whether an exception occurs or not.

```
public class FinallyTest {
	public static int testMethod() {
		try {
			System.out.println("In try block");
			return 10;
		} catch (Exception e) {
			System.out.println("In catch block");
			return 20;
		} finally {
			System.out.println("In finally block");
			return 30;
		}
	}
	
	public static void main(String[] args) {
		System.out.println("Result: " + testMethod());
	}
}

// output - 30
If both try and finally contain return statements, the return from finally overrides the return from try or catch.
```


```java
public class Main {
	public static int testMethod() {
		try {
			System.out.println("In try block");
		    throw new Exception("Some Exception Occured..");
		} catch (Exception e) {
			System.out.println("In catch block: " + e.getMessage());
		    e.printStackTrace(); // It outputs the exact line numbers, class names, and execution path that led to an exception.
			return 20;
		} finally {
			System.out.println("In finally block");
			return 30;
		}
	}
	
	public static void main(String[] args) {
		System.out.println("Result: " + testMethod());
	}
}

// e.printStackTrace();  -> // Highly useful for local debugging. generally discouraged in production-level code.
// use logger -> Production systems (Log4j, SLF4j)


/*
 * ouptut -
	In try block
	In catch block: Some Exception Occured..
	java.lang.Exception: Some Exception Occured..
		at Main.testMethod(Main.java:19)
		at Main.main(Main.java:31)
	In finally block
	Result: 30
*/
```

-----

**Question 4: In Java, what are the differences between a Checked and Unchecked? Examples? How would you handle these?**

```
Checked Exception - Compiler compile-time par check karta hai. 
Agar handle nahi kiya, code compile nahi hoga.
Ex - IOException, SQLException, FileNotFoundException

try {
    FileReader file = new FileReader("data.txt");
} catch (FileNotFoundException e) {
    // handle
}

// OR throws kar sakte ho:

void readFile() throws IOException {
    // code
}

```


```
Unchecked Exception - Compiler force nahi karta handle karne ke liye.
Ye generally programming/runtime problems hoti hain.

Examples: NullPointerException, ArithmeticException, ArrayIndexOutOfBoundsException

String s = null;
s.length();   // NullPointerException

But blindly `catch (Exception)` ya `catch (NullPointerException)` karna usually solution nahi hai; root cause fix karna better hai.
```


 🎯 Interview mein exactly bolo:

> Checked exceptions are checked by the compiler, so they must either be handled using try-catch or declared using throws. Examples are IOException and SQLException. Unchecked exceptions are subclasses of RuntimeException and are not checked by the compiler. Examples are NullPointerException and IllegalArgumentException. For unchecked exceptions, we generally fix the root cause, and use try-catch when we can meaningfully recover from the error.

Checked → 
	Compiler says: handle me or declare me.  
	Handle -> try-catch / throws

Unchecked → 
	Compiler doesn't force you. 
	Handle -> Usually fix root cause;  try-catch when recovery is meaningful.

-----

**Question 5: Can we create a finally block without creating a catch block? In what scenarios, a finally block will not be executed?**

Interview mein bolo:

> Yes, a finally block can be used without a catch block. catch isn't mandatory. A try block can be followed by either catch, finally, or both. Finally is commonly used for cleanup code.


**Kab finally execute nhi hoga -**

System.exit() call ho gaya - 

```
try {
    System.exit(0);   
} finally {
    System.out.println("Finally");
}
```

In Java, System.exit() terminates the currently running Java Virtual Machine (JVM). It is used to instantly stop the execution of a program.

Infinite loop / stuck execution

```
try {
    while (true) {
    }
} finally {
    System.out.println("Finally");
}
```

Yaha program `try` se bahar hi nahi aa raha, so `finally` tak execution nahi pahunchta.

Interview ke liye bas ye yaad rakho

> **“Finally normally executes regardless of whether an exception occurs or a return statement is present. It may not execute if the JVM terminates abruptly, such as through System.exit(), JVM crash, or process termination.”**

finally tab nhi chalega - jab JVM terminate ho jaye.

-------------

**Question 6: What is the concept of Exception Propagation? Example by code ?**

Exception Propagation kya hai -

Jab ek method mein exception aata hai aur us method mein **handle nahi hota**, to exception **caller method ke paas chala jata hai**.
Agar caller bhi handle nahi karta, to uske caller ke paas chala jata hai.

```java
public class Test {

    static void method3() {
        int x = 10 / 0;   // Exception
    }

    static void method2() {
        method3();
    }

    static void method1() {
        method2();
    }

    public static void main(String[] args) {

        try {
            method1();
        } catch (ArithmeticException e) {
            System.out.println("Exception handled");
        }
    }
}

## flow of error -
ArithmeticException
	↓
method3
   ↓
method2
   ↓
method1
   ↓
main
```

Interview ye bolo -
> Exception propagation is the process by which an unhandled exception moves up the call stack from the current method to its caller, and then to higher-level callers until it is handled. If nobody handles it, it becomes an uncaught exception and the thread terminates.

---------

**Question 7: When we override a method in a Child class, can we throw an additional Exception that is not thrown by the Parent class method ?**

means, Parent class ke method mein agar koi checked exception declared nahi hai, to kya Child class us overridden method mein naya checked exception throw/declare kar sakti hai?

Answer: No — checked exception ke case mein.

```
class Parent {
    void show() {
    }
}

class Child extends Parent {
    @Override
    void show() throws IOException {   // ❌ Compile error
    }
}
```

Parent method:

```
void show()
```

koi checked exception declare nahi karta.

Child:

```
void show() throws IOException
```

IOException add kar raha hai. Allowed nahi hai.


**2. Lekin unchecked exception allowed hai**

```
class Parent {
    void show() {
    }
}

class Child extends Parent {
    @Override
    void show() throws RuntimeException {   // ✅ allowed
    }
}
```

Because `RuntimeException` unchecked exception hai.

Parent mein exception tha toh?

```
class Parent {
    void show() throws IOException {
    }
}

class Child extends Parent {
    @Override
    void show() throws FileNotFoundException {
    }
}
```

✅ Allowed.

Because `FileNotFoundException` is a subclass of `IOException`.


#### 🧠 Main rule

Overriding method cannot throw a broader/new checked exception than the parent method.

Allowed:

```
Parent: IOException
Child: FileNotFoundException ✅
```

Not allowed:

```
Parent: IOException
Child: SQLException ❌
```

Also:

```
Parent: no checked exception
Child: IOException ❌
```

🎯 Interview mein exactly bolo:

> An overriding method cannot throw a new or broader checked exception than the overridden parent method. It can throw the same checked exception, a subclass of it, or no exception. Unchecked exceptions are not restricted by this rule.

-------

**Question 8: When we override a method in a Child class, can we throw an additional Exception that is not thrown by the Parent class method ?**







------------




















-------

**Question 15: try-with-resources**

Managing resources like file handles, database connections, and network sockets in Java can be a challenging task. Traditionally, developers relied on the `try-finally` blocks to make sure that resources are properly closed after their operations are complete. However, this approach is error-prone and can lead to resource leaks if not implemented correctly. Java's `try-with-resources` statement, introduced in Java 7, provides a more efficient and less error-prone way to manage resources.

**Resource - **  A resource is any object that implements the `AutoCloseable` interface, which includes the `Closeable` interface as well. Resources often represent external or system-level entities that need to be explicitly released when no longer in use. Common examples include file streams, database connections, network sockets, and other I/O-related objects.

The `AutoCloseable` interface defines a single method:

```java
public interface AutoCloseable {  
	void close() throws Exception;  
}
```

Any class that implements this interface can be used within a `try-with-resources` block. Additionally, classes that implement the older `Closeable` interface can also be used in `try-with-resources`, making it easy for developers familiar with `Closeable` to adopt this newer resource management feature. The `Closeable` interface is a subinterface of `AutoCloseable` and defines the `close()` method with a more specific `IOException` exception:

```java
public interface Closeable extends AutoCloseable {
    void close() throws IOException;
}
```


```
// syntax -

try (ResourceType resource = new ResourceType()) {  
    // Code that uses the resource  
} catch (ExceptionType e) {  
    // Exception handling code  
}

// where `ResourceType` is the class type, and `resource` is the variable name.
```

**Handling Multiple Resources -** 

One of the powerful features of `try-with-resources` is its ability to manage multiple resources in a single statement. When you need to work with several resources simultaneously, you can declare them all within the same `try` block, separated by semicolons:

```java
try (BufferedReader br = new BufferedReader(new FileReader("file1.txt"));  
     BufferedReader br2 = new BufferedReader(new FileReader("file2.txt"))) {  
  
    String line1, line2;  
    while ((line1 = br.readLine()) != null && (line2 = br2.readLine()) != null) {  
        System.out.println("File1: " + line1);  
        System.out.println("File2: " + line2);  
    }  
} catch (IOException e) {  
    e.printStackTrace();  
}
```

**Exception Handling and Suppressed Exceptions -**

An essential aspect of the `try-with-resources` statement is its handling of exceptions. If an exception is thrown within the `try` block and another occurs while closing the resource, Java does not discard the second exception. Instead, the second exception is added to the first as a "suppressed exception." This makes sure that you don’t lose critical information about what went wrong during resource management.

Consider the following scenario:

```java

class CustomResource implements AutoCloseable {
    @Override
    public void close() {
        throw new RuntimeException("some Exception..."); // Suppressed exception
    }
}


try (CustomResource resource = new CustomResource()) {  
    throw new Exception("Exception during processing");  // Priamry exception
} catch (Exception e) {  
    System.err.println("Caught: " + e.getMessage());  
    for (Throwable suppressed : e.getSuppressed()) {  
        System.err.println("Suppressed: " + suppressed.getMessage());  
    }  
}

```

If the `close()` method of `CustomResource` throws an exception, that exception will be added to the list of suppressed exceptions of the primary exception thrown inside the `try` block. The output would include details about both the original exception and the suppressed exception, providing complete information about what went wrong.
